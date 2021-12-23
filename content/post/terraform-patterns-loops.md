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

# Create multiple instances with a loop

If you want to create multiple instances of simular objects, you can add a for_each argument in the resource or module block.
The for_each argument accepts a map or a set of strings, and creates an instance for each item in that map or set. 


You can create a map of key value pairs and use them to define multiple resource groups:

```hcl
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

```hcl
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

When creating a storage account, you can reference the resource group that has been created with the for_each argument as follows: 

```hcl
resource "azurerm_storage_account" "storage" {
    name = "projectxprodwestorage"
    account_replication_type = "LRS"
    account_tier = "Standard"
    location = azurerm_resource_group.otherrg["projectx-prod-we"].location
    resource_group_name = azurerm_resource_group.otherrg["projectx-prod-we"].name
}
```

## A more complex example

What if I have a more complex resource to create. A map has just a key and a value. Or Maps contain many things of one type. Objects contain a specific set of things of many types.
Let's take a vnet as an example. A vnet has some subnets which is a complexer type.

First, let's create the vnet:

```hcl
resource "azurerm_virtual_network" "vnet" {
    address_space       = ["172.16.0.0/16"]
    location            = azurerm_resource_group.otherrg["projectx-prod-we"].location
    name                = "${var.prefix}-vnet"
    resource_group_name = azurerm_resource_group.otherrg["projectx-prod-we"].name
}
```

As the variable type we use a map of objects. The key is the name of the subnet instance, and the value is a complex object with the subnet properties.
The subnet variable object types are incomplete, because Terraform will fail when using nested variables.
And optional properties are not (yet?) supported. 

So the map construct will look as follows:

```hcl
variable "subnets" {
  type = map(object({
    address_prefixes  = list(string)
    service_endpoints = list(string)
    //location = string - cannot use nested variables
    //resource_group_name = string - cannot use nested variables 
  }))
}
```


We can then define the default value of the variable: 

```hcl
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

```hcl
resource "azurerm_subnet" "subnet" {
    for_each = var.subnets
    name                 = "${var.prefix}-${each.key}"
    resource_group_name  = azurerm_resource_group.otherrg["projectx-prod-we"].name
    virtual_network_name = azurerm_virtual_network.vnet.name
    address_prefixes     = each.value.address_prefixes
    service_endpoints    = each.value.service_endpoints
}
```

In the next post we will discuss modules.


https://www.reddit.com/r/Terraform/comments/hyqago/difference_between_maps_and_objects/
https://www.terraform.io/language/meta-arguments/for_each