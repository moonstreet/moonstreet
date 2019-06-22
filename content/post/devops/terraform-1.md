---
title: "Azure Kubernetes Services, Terraform and Jenkins (part 1)"
date: 2019-06-21T11:29:13+01:00
draft: false
image: "uploads/monitoring.jpeg"
tags: ["devops"]
topic: ["devops"]
author: "Jacqueline"
---

# Terraform

This blogseries is about my adventures in AKS, Terraform and Jenkins. This first article is about Terraform.

This assumes you have an Azure account.

## Prepare

Create a service principal with contributor permissions on Azure Subscription level. We can fine tune the permissions later.

```sh
AZ_SUB="your subscription id"
AZ_SECRET="S3cr3tPasw0rd"
NAME="terraform-partner-sub"

az login
az account set --subscription $AZ_SUB
az ad sp create-for-rbac --name $NAME-installation-account --password $AZ_SECRET --years 2
```

Now this account has contributor permissions on the subscription level.
The account is valid for 2 years.  
Save the login info this command returns somewhere really safe.

## Set up Terraform

I already know what I want with Terraform. Apart form an AKS cluster I would also like to build my Jenkins environment with Terraform (in the future, I already have one now built manually). I also already know that I want to reuse code as much as possible. This looks like a nice folderstructure for now:

```
└── infrastructure-as-code
    ├── aks-cluster
    │   ├── dev
    │   └── sandbox
    ├── jenkins
    └── modules

```

Let's install Terraform. Should work on any Linux distro with sudo enabled:

```sh
wget https://releases.hashicorp.com/terraform/0.12.2/terraform_0.12.2_linux_amd64.zip
unzip terraform_0.12.2_linux_amd64.zip
sudo mv terraform /usr/local/bin
```

Let's start with our sandbox environment.

```sh
cd aks-cluster/sandbox
```

The first thing we need to do is create a Terraform file and confugure the Azure provider.

```sh
cat > infrastructure.tf <<'EOF'
provider "azurerm" {
  subscription_id = "${var.az_subscription_id}"
  client_id       = "${var.az_client_id}"
  client_secret   = "${var.az_secret}"
  tenant_id       = "${var.az_tenant_id}"
  version         = "=1.28.0"
}
EOF
```

I need to declare these variable somewhere. Let's create a vars file for that.

```sh
cat > vars.tf <<'EOF'
variable "az_subscription_id" {}
variable "az_client_id" {}
variable "az_secret" {}
variable "az_tenant_id" {}
}
EOF
```

And finally, initialize the variables. This is done in a `tfvars` document. Since version 0.12 it supports the json format as well.

```sh
cat > sandbox.tfvars.json <<'EOF'
{
"az_subscription_id":"your subscription id",
"az_client_id":"your app id",
"az_secret": "****"
"az_tenant_id":"your tenant id"
}
EOF
```

Yes, I am happy. Let's now initialize Terraform to see if everything is OK:

```
terraform init
terraform plan --var-file sandbox.tfvars.json -out terraform_plan -input=false
```

I am adding `-out terraform_plan -input=false` because I don't want to manually input anything. This comes in handy later when you run this in a pipeline.
This should download the Terraform azurerm plugin and tell you that everything is OK.

## Save Terraform state elsewhere (in Azure storage)

We don't want to keep our state in source control or on our local computer. It might get lost and it contains some sensitive information about the environment.  
Quickly create a storage account, add blob storage and then add a container named terraform.  
Add the following on top of terraform.tf:

```
terraform {
  backend "azurerm" {
    storage_account_name = "microbotstorage"
    container_name       = "terraform"
    key                  = "sandbox_tfstate_file##"
    access_key           = "***"
  }
}
```

And we are save. Also look here: https://docs.microsoft.com/en-us/azure/terraform/terraform-backend

## Build the Network

Add the following to the infrastructure.tf file:

```
resource "azurerm_resource_group" "network" {
  name     = "${var.prefix}-${var.environment}-${var.location_abbreviation}-network-rg"
  location = "${var.az_region}"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "${var.prefix}-${var.environment}-${var.location_abbreviation}-vnet"
  address_space       = ["${var.az_vnet_address_prefix}"]
  location            = "${var.az_region}"
  resource_group_name = "${azurerm_resource_group.network.name}"
}

resource "azurerm_subnet" "k8s-subnet" {
  name                 = "${var.prefix}-${var.environment}-${var.location_abbreviation}-kubernetes-subnet"
  resource_group_name  = "${azurerm_resource_group.network.name}"
  virtual_network_name = "${azurerm_virtual_network.vnet.name}"
  address_prefix       = "${var.az_vnet_kubernetes_subnet}"
  service_endpoints    = ["Microsoft.Sql", "Microsoft.Storage"]
}
```

We need to add some extra variables to our vars file. The vars file looks like this. I used the extended notation for some of the variables for good measure.

```
variable "environment" {
  type        = "string"
  description = "Name of the environment to deploy to"
  default     = "dev"
}
variable "prefix" {
  type        = "string"
  description = "Prefix for the deployment name"
  default     = "app"
}

variable "az_subscription_id" {}
variable "az_client_id" {}
variable "az_secret" {}
variable "az_tenant_id" {}
variable "location_abbreviation" {}
variable "az_region" {}
variable "az_vnet_kubernetes_subnet" {}
variable "az_vnet_address" {}
```

Let's install this:

```
terraform plan --var-file sandbox.tfvars.json -out terraform_plan -input=false
terraform apply --var-file sandbox.tfvars.json -out terraform_plan -input=false
```

Now have a quick look and you should see these resources are installed in Azure.

## Modularize

Let's make a module of the network creation.
In our module folder, we'll create a dir named vnet and we put two files.

```
└── infrastructure-as-code
    ├── aks-cluster
    │   ├── dev
    │   └── sandbox
    │       ├── infrastructure.tf
    │       ├── sandbox.tfvars.json
    │       ├── terraform_plan
    │       └── vars.tf
    ├── jenkins-agent
    └── modules
        └── vnet
            ├── vnet.tf
            └── vnetvars.tf
```

We cut and paste this portion from sandbox/infrastructure.tf to modules/vnet/vnet.tf:

```sh
resource "azurerm_resource_group" "network" {
  name     = "${var.prefix}-${var.environment}-${var.location_abbreviation}-network-rg"
  location = "${var.az_region}"
}

# Create virtual network and subnets
resource "azurerm_virtual_network" "vnet" {
  name                = "${var.prefix}-${var.environment}-${var.location_abbreviation}-vnet"
  address_space       = ["${var.az_vnet_address}"]
  location            = "${var.az_region}"
  resource_group_name = "${azurerm_resource_group.network.name}"
}

resource "azurerm_subnet" "k8s-subnet" {
  name                 = "${var.prefix}-${var.environment}-${var.location_abbreviation}-kubernetes-subnet"
  resource_group_name  = "${azurerm_resource_group.network.name}"
  virtual_network_name = "${azurerm_virtual_network.vnet.name}"
  address_prefix       = "${var.az_vnet_kubernetes_subnet}"
  service_endpoints    = ["Microsoft.Sql", "Microsoft.Storage"]
}

```

And in modules/vnet/vnetvars.tf we copy and paste:

```sh
variable "environment" {}
variable "prefix" {}
variable "location_abbreviation" {}
variable "az_region" {}
variable "az_vnet_kubernetes_subnet" {}
variable "az_vnet_address" {}
```

And now sandbox/infrastructure looks like this:

```sh
terraform {
  backend "azurerm" {
    storage_account_name = "microbotstorage"
    container_name       = "terraform"
    key                  = "sandbox_tfstate"
    access_key           = "LXzVqR8apD0wkCFFE6OcBed+CvgeDZklAL5YvHg79EPSS45UCsGp/XKCS974pgMghgDo3kdFwqfoHhoQ3fBvXw=="
  }
}

provider "azurerm" {
  subscription_id = "${var.az_subscription_id}"
  client_id       = "${var.az_client_id}"
  client_secret   = "${var.az_secret}"
  tenant_id       = "${var.az_tenant_id}"
  version         = "1.28.0"
}

module "vnet" {
  source                    = "../../modules/vnet"
  prefix                    = "${var.prefix}"
  environment               = "${var.environment}"
  location_abbreviation     = "${var.location_abbreviation}"
  az_vnet_address           = "${var.az_vnet_address}"
  az_region                 = "${var.az_region}"
  az_vnet_kubernetes_subnet = "${var.az_vnet_kubernetes_subnet}"
}

```

Jeez. It looks like duplication but it isn't really. The modules need variables declared, but so does our sandbox configuration that consumes the module. We can now add more environments and supply different values for the modules.  
Let's carry on and create the cluster, as a module.

## Add the cluster

This is how I defined the AKS cluster, with monitoring support.

```sh
resource "azurerm_resource_group" "k8s" {
  name     = "${var.prefix}-${var.environment}-${var.location_abbreviation}-cluster-rg"
  location = "${var.az_region}"
}
resource "azurerm_resource_group" "logging" {
  name     = "${var.prefix}-${var.environment}-${var.location_abbreviation}-logging-rg"
  location = "${var.az_region}"
}

resource "azurerm_log_analytics_workspace" "space" {
  name                = "${var.prefix}-${var.environment}-${var.location_abbreviation}-loganalytics-workspace"
  location            = "${var.az_region}"
  resource_group_name = "${azurerm_resource_group.logging.name}"
  sku                 = "PerGB2018"
}

resource "azurerm_log_analytics_solution" "oms" {
  solution_name         = "ContainerInsights"
  location              = "${azurerm_log_analytics_workspace.space.location}"
  resource_group_name   = "${azurerm_log_analytics_workspace.space.resource_group_name}"
  workspace_resource_id = "${azurerm_log_analytics_workspace.space.id}"
  workspace_name        = "${azurerm_log_analytics_workspace.space.name}"

  plan {
    publisher = "Microsoft"
    product   = "OMSGallery/ContainerInsights"
  }
}

resource "azurerm_kubernetes_cluster" "k8s" {
  name                = "${var.prefix}-${var.environment}-${var.location_abbreviation}-cluster"
  location            = "${azurerm_resource_group.k8s.location}"
  resource_group_name = "${azurerm_resource_group.k8s.name}"
  dns_prefix          = "${var.prefix}${var.environment}${var.location_abbreviation}"
  kubernetes_version  = "${var.az_aks_version}"

  service_principal {
    client_id     = "${var.az_client_id}"
    client_secret = "${var.az_secret}"
  }

  addon_profile {
    oms_agent {
      enabled                    = true
      log_analytics_workspace_id = "${azurerm_log_analytics_workspace.space.id}"
    }
  }

  agent_pool_profile {
    name            = "agentpool"
    count           = "2"
    vm_size         = "${var.az_aks_vm_size}"
    os_type         = "Linux"
    os_disk_size_gb = 60
    vnet_subnet_id  = "${var.az_vnet_kubernetes_subnet_id}"
    max_pods        = 110
  }

  network_profile {
    docker_bridge_cidr = "172.17.0.1/16"
    dns_service_ip     = "10.2.0.10"
    network_plugin     = "azure"
    service_cidr       = "10.2.0.0/16"
  }

  role_based_access_control {
    enabled = true
  }
}

```

The variables:

```sh
variable "prefix" {}

variable "environment" {}

variable "location_abbreviation" {
  type        = "string"
  description = "Abbreviation of the location"
  default     = "we"
}

variable "az_region" {
  type        = "string"
  description = "Azure region for deployment"
  default     = "West Europe"
}

variable "az_aks_version" {
  type        = "string"
  description = "Kubernetes Version - az aks get-versions --location westeurope --output table"
  default     = "1.12.7"
}

variable "az_aks_vm_size" {
  type        = "string"
  description = "Azure VM node size e.g. Standard_D2s_v3 or Standard_B4ms - az vm list-sizes --location westeurope"
  default     = "Standard_D2s_v3"
}

variable "az_vnet_kubernetes_subnet_id" {}

variable "az_client_id" {}

variable "az_secret" {}

```

So we already built the vnet, and now we want to put this Kubernetes cluster in it.
_We need a reference to the subnet created by the vnet module!_ So how does that work?

We add a file called modules/vnet/outputs.tf and add this piece of code to it:

```
output "k8s_subnet_id" {
  value = "${azurerm_subnet.k8s-subnet.id}"
}

```

We can reference that value with `"${module.vnet.k8s_subnet_id}"`, like this:

```sh

module "aks" {
  source                = "../../modules/aks"
  prefix                = "${var.prefix}"
  environment           = "${var.environment}"
  location_abbreviation = "${var.location_abbreviation}"

  az_region                    = "${var.az_region}"
  az_vnet_kubernetes_subnet_id = "${module.vnet.k8s_subnet_id}"
  az_aks_version               = "${var.az_aks_version}"
  az_client_id                 = "${var.az_client_id}"
  az_secret                    = "${var.az_secret}"
}
```

https://stackoverflow.com/questions/51213871/terraform-provider-variable-sharing-in-modules
