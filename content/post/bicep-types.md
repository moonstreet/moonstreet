---
title: "Azure Bicep cheatsheet"
date: 2024-05-04T09:13:46+01:00
draft: true
featured: false
toc: false 
menu: main
#featureImage: "/images/terraform.png"
#thumbnail: "/images/path/terraform.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 50 
codeLineNumbers: true 
figurePositionShow: true
categories:
  - Technology 
tags:
  - azure
showShare: false
---    

# Azure Bicep cheatsheet
For my work I need to use Azure Bicep more and more. And slowly I come to appreciate it.

## Useful links
These are 2 links I can't do without:

* [Azure Verified Modules](https://github.com/Azure/bicep-registry-modules) 
* [Bicep Reference](https://learn.microsoft.com/en-us/azure/templates/)

## Bicep in 5 short sentences
Azure resources can be deployed via a REST endpoint: the Azure Resource Manager REST API. That API accepts a JSON payload that represents the desired state of the infrastructure you want to deploy. Bicep is a DSL that provides a more readable syntax. Bicep files are compiled (transpiled) into ARM Template JSON files.

## Cheatsheet
From 1 to 100 in 1 second. The use of modules is assumed.

### Optional parameters
You have an optional parameter. When a value is assigned, use it. When not, omit the parameter.

*Method 1*

Give the parameter a default empty value. Then use the `empty()` function to evaluate.

Declaration:

```
@description('Optional.Specifies the dnsPrefix. If empty it will default to aks-<suffix>.') 
param dnsPrefix string = ''
```
Implementation:

```
dnsPrefix: !empty(dnsPrefix) ? dnsPrefix : 'aks-${suffix}'
```
Explanation:

' ' is considered an empty string in Bicep. The !empty() function returns true if the string is not empty (i.e., has a length greater than 0) and false if the string is empty. If no value is provided for the dnsPrefix parameter, it will default to an empty string (''). When the !empty(dnsPrefix) condition is evaluated, it will be false because an empty string is considered empty.


*Method 2*

Use an object. When the object has the property, use it, else set it to null.

Declaration:

```
@description('Optional.Specifies the dnsPrefix. If empty it will default to aks-<suffix>.') 
param dnsPrefix string = ''
```
Implementation:

```
dnsPrefix: !empty(dnsPrefix) ? dnsPrefix : 'aks-${suffix}'
```
Explanation:

' ' is considered an empty string in Bicep. The !empty() function returns true if the string is not empty (i.e., has a length greater than 0) and false if the string is empty. If no value is provided for the dnsPrefix parameter, it will default to an empty string (''). When the !empty(dnsPrefix) condition is evaluated, it will be false because an empty string is considered empty.



