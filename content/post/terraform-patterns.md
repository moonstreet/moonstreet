---
title: "Terraform patterns: conditionals" # Title of the blog post.
date: 2021-12-21T08:24:21+01:00 # Date of post creation.
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


This is a back to basics post about a Terraform pattern: conditionals.

## Conditionals: if then else

Consider a resource group name. Imagine we want the resource group to follow the rules of naming convention but in other cases we don't want to.
So if there is a naming convention, implement that, if not than do not.

Imagine a naming convention `<projectname>-<environment>-<resource>`. In this case every resource has a certain prefix. Now we can say: if the prefix is set, please follow that naming, else just take the variable of the full name.

This is where the ternary operator comes in.

If var.prefix is an empty string then the result is "my-prefix-rg", but otherwise it is the actual value of var.rg_name:


| What  | Means  |
|---|---|
|  if var.prefix | condition  |
|  ?  | then  |
|   | do stuff   |
|  : | else   |
|   | do other stuff  |


```
# condition ? true_val : false_val
name  = var.rg_name == null ? "${var.prefix}-rg" : var.rg_name
```
A full example


main.tf
```
resource "azurerm_resource_group" "rg" {
  name     = var.rg_name == null ? "${var.prefix}-rg" : var.rg_name
  location = var.location
  tags     = var.tags
}
```

variables.tf
```
variable prefix {
  default = "projextx-dev"
}

variable rg_name {
  default = null
}

# let's just tags for fun
variable "tags" {
  default = {
    owner = "jacqueline",
    department = "research"
  }
}
```

## What about the null value?

Terraform v0.12 allows assigning the special value null to an argument to mark it as "unset". So a module can allow its caller to conditionally override a value while retaining the default behavior if the value is not defined.

In other words:
* null can be used with conditionals
* in other cases: null does not mean a resource does not get created. It just means its default behaviour will be applied.

https://www.hashicorp.com/blog/terraform-0-12-conditional-operator-improvements

The next pattern will be loops.


