---
title: "Terraform patterns: loops" # Title of the blog post.
date: 2021-12-21T11:24:21+01:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/terraform.png" # Sets featured image on blog post.
#thumbnail: "/images/path/terraform.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: true # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - kubernetes
  - azure
  - terraform
showShare: false
---

# Create multiple resources with a loop

If you want to create multiple instances of, say, an Azure resource group, you can add a for_each argument.
The for_each argument accepts a map or a set, and creates an instance for each item in that map or set.

So you can create a map of key value pairs (aka a dictionary) and use it to define multiple resource groups:

```
resource "azurerm_resource_group" "rg" {
    for_each = {
        projectx-dev-we = "westeurope"
        projectx-dev-us = "eastus"
    }
    
    name     = "${each.key}-rg"
    location = each.value
    tags     = var.tags
}
```

Sure enough you can also refactor the map as a variable

```
variable "groups" {
    default = {
        projectx-prod-we = "westeurope"
        projectx-prod-us = "eastus"
    }
}

resource "azurerm_resource_group" "otherrg" {
    for_each = var.groups

    name     = "${each.key}-rg"
    location = each.value
    tags     = var.tags
}
```

## Reference the resource group

What if you want to create a storage account in each of the created resource groups?
You could reference the resource group that has been created in the previous step as follows: 

```
resource "azurerm_storage_account" "storage" {
    name = "projectxprodwestorage"
    account_replication_type = "LRS"
    account_tier = "Standard"
    location = azurerm_resource_group.otherrg["projectx-prod-we"].location
    resource_group_name = azurerm_resource_group.otherrg["projectx-prod-we"].name
}
```

## A more complex example

A map has just some keys and values. They can contain many things of just one type.
But what if I want to use strings, lists and so on to create my new resource, in a for-each loop?
For  example, I have a vnet and it contains multiple subnets. So there is a one-to-many relationship. 
[Here](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/subnet) is its definition.

The subnet has a name (of type string) and address_prefixes (of type list). 
Objects to the rescue! Objects contain a specific set of things of many types, and they have name whih we can refer to.

First, let's create the vnet:

```
resource "azurerm_virtual_network" "vnet" {
    address_space       = ["172.16.0.0/16"]
    location            = azurerm_resource_group.otherrg["projectx-prod-we"].location
    name                = "${var.prefix}-vnet"
    resource_group_name = azurerm_resource_group.otherrg["projectx-prod-we"].name
}
```

Let's now define the subnets.


### Use a map of objects

As the variable type we could use a map of objects. The key is the name of the subnet instance, and the value is a complex object with the subnet properties.
The map has the following definition:

```
variable "subnets" {
  type = map(object({
    address_prefixes  = list(string)
    service_endpoints = list(string)
  }))
}
```

We can then define the default value of the variable: 

```
variable "subnets" {
    type    = map(object({
    address_prefixes     = list(string)
    service_endpoints    = list(string)
    }))
    default = {
        "db-subnet" = {
            address_prefixes     = ["172.16.1.0/24"]
            service_endpoints    = ["Microsoft.AzureCosmosDB","Microsoft.Sql"]
        },
        "generic-subnet" = {
            address_prefixes     = ["172.16.2.0/24"]
            service_endpoints    = ["Microsoft.Storage","Microsoft.KeyVault"]
        }
    }
}
```

Finally, we can create the subnets as follows:

```
resource "azurerm_subnet" "subnet" {
    for_each = var.subnets
    name                 = "${var.prefix}-${each.key}"
    resource_group_name  = azurerm_resource_group.otherrg["projectx-prod-we"].name
    virtual_network_name = azurerm_virtual_network.vnet.name
    address_prefixes     = each.value.address_prefixes
    service_endpoints    = each.value.service_endpoints
}
```

### Use a list of objects

We can also use a list of objects, but then we can only use name and one extra property.

```
variable "subnets_list" {
  description = "Required. A map of string, object with the subnet definition"
  type    = list(object({
    name = string
    address_prefixes     = list(string)
    service_endpoints    = list(string)
  }))
  default = [
    {
      name : "firewall_subnet"
      address_prefixes : ["10.12.0.0/24"]
      service_endpoints : ["Microsoft.Sql"]
    },
    {
      name : "jumpbox_subnet"
      address_prefixes : ["10.12.1.0/24"]
      service_endpoints : ["Microsoft.Sql"]
    }
  ]
}
```

The for_each meta-argument accepts a map or a set of strings, so we need to translate the list to a map.
We say: we set the key of the map to the unique subnet.name, and the value is the complete subnet object.

```
resource "azurerm_subnet" "subnet" {
  for_each = { for subnet in var.subnets : subnet.name => subnet }
    name                 = "${var.prefix}-${each.key}"
    resource_group_name  = azurerm_resource_group.otherrg["projectx-prod-we"].name
    virtual_network_name = azurerm_virtual_network.vnet.name
    address_prefixes     = each.value.address_prefixes
    service_endpoints    = each.value.service_endpoints
}
```


## Take aways

* Terraform can give you headaches.
* For_each accepts a map. A map is a dictionary, with a key and a value. The value can be a complex property like an an object or a list.
* When using a list, we need to transform the list to a map by setting a key and a value with the arrow notation. We can set the value to a complex object.

## Complete example

```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=3.2.0"
    }
  }
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
  features {}
}

variable "prefix" {
  default = "headache-dev"
}

variable "network_portion" {
  default = "10.14"
}

//using locals, not variables, because Terraform does not support variables in variables (variable nesting)
locals {
  common_tags = {
    environment   = "sratch"
    creation_date = formatdate("YYYY-MM-01", timestamp())
  }
  subnets-map = {
      "db-subnet" = {
        address_prefixes  = ["${var.network_portion}.1.0/24"]
        service_endpoints = ["Microsoft.AzureCosmosDB", "Microsoft.Sql"]
      }
      "generic-subnet" = {
        address_prefixes  = ["${var.network_portion}.2.0/24"]
        service_endpoints = ["Microsoft.Storage", "Microsoft.KeyVault"]
    }
  }
  subnets-list = [
    {
      name = "fw-subnet"
      address_prefixes  = ["${var.network_portion}.3.0/24"]
      service_endpoints = ["Microsoft.AzureCosmosDB", "Microsoft.Sql"]
    },
    {
      name = "vm-subnet"
      address_prefixes  = ["${var.network_portion}.4.0/24"]
      service_endpoints = ["Microsoft.Storage", "Microsoft.KeyVault"]
    }
  ]
}

resource "azurerm_resource_group" "vnetgroup" {
  location = "westeurope"
  name     = "${var.prefix}-rg"
}

resource "azurerm_virtual_network" "vnet" {
  address_space       = ["${var.network_portion}.0.0/16"]
  location            = "westeurope"
  name                = "${var.prefix}-vnet"
  resource_group_name = azurerm_resource_group.vnetgroup.name
}

resource "azurerm_subnet" "subnet" {
  for_each = { for subnet in local.subnets-list : subnet.name => subnet }
  name                 = "${var.prefix}-${each.key}"
  resource_group_name  = azurerm_resource_group.vnetgroup.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes = each.value.address_prefixes
  service_endpoints = each.value.service_endpoints

}

resource "azurerm_subnet" "subnet2" {
  for_each = local.subnets-map
  name                 = "${var.prefix}-${each.key}"
  resource_group_name  = azurerm_resource_group.vnetgroup.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes = each.value.address_prefixes
  service_endpoints = each.value.service_endpoints
}
```


## Other musings

What I don't like in the above examples, is that my subnet object definition is incomplete. There is no resource_group_name or location, because we use the values of the vnet definition for that. Can't we add those properties to our object definition?  
That would lead to a whole lot of repetition, because we can't use variables in variables. Terraform will fail when using nested variables (can not interpolate variables when using a datastructure as a variable). And optional properties in an object are not (yet?) supported.
What we could do to solve this is to create a local variable. Locals support variable nesting.
Anyway I will stick to using the root value.

In next posts we will discuss modules and maybe also dynamics.

https://stackoverflow.com/questions/58594506/how-to-for-each-through-a-listobjects-in-terraform-0-12
https://www.reddit.com/r/Terraform/comments/hyqago/difference_between_maps_and_objects/
https://www.terraform.io/language/meta-arguments/for_each
