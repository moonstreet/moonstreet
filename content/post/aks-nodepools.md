---
title: "Azure Kubernetes node pools with Terraform" # Title of the blog post.
date: 2020-12-29T13:30:21+01:00 # Date of post creation.
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


**In Azure Kubernetes Service (AKS), nodes of the same configuration are grouped together into node pools.** These node pools contain the underlying VMs that run your applications.
The initial number of nodes and their size (SKU) is defined when you create an AKS cluster, which creates a system node pool. To support applications that have different compute or storage demands, you can create additional user node pools.

I just copied the above text from [here](https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools) because it is just right. To have a full understanding of node pools I encourage you to read the whole article. Also, if you plan to run Azure Kubernetes in production, I can recommend [this article](https://docs.microsoft.com/en-us/azure/aks/concepts-clusters-workloads) as well. It's all about the sizing baby!

This post is about deploying node pools with Terraform. I assume a bit of prior knowledge about Azure and Terraform modules.

Because we run multiple instances of AKS I thought to make the number of node pools and their properties variable. [This](https://www.danielstechblog.io/terraform-working-with-aks-multiple-node-pools-in-tf-azure-provider-version-1-37/) article directed me in that direction.

At work, we have a git repo with multiple cluster definitions (I treat them like cattle). The clusters are deployed in a Jenkins pipeline.

## Terraform config

My goal is to create 3 node pools:

-   a system node pool for system pods
-   an infra node pool for infra pods (Vault, Elasticsearch and Prometheus to be precise)
-   an app node pool for our LOB apps

If you define an AKS cluster, following the Terraform [documentation](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/kubernetes_cluster) you will note that there is a default node pool block, but there isn't a definition of for extra node pools. In fact, there is a separate resource, namely the [azurerm_kubernetes_cluster_node_pool](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/kubernetes_cluster_node_pool) resource.

My definition of the azurerm_kubernetes_cluster_node_pool is like this:

```
resource "azurerm_kubernetes_cluster_node_pool" "pools" {
  lifecycle {
    ignore_changes = [
      node_count
    ]
  }

  for_each = var.az_aks_additional_node_pools
  kubernetes_cluster_id = azurerm_kubernetes_cluster.kube.id
  name                  = each.value.name
  mode                  = each.value.mode
  node_count            = each.value.node_count
  vm_size               = each.value.vm_size
  availability_zones    = ["1", "2"]
  max_pods              = 250
  os_disk_size_gb       = 128
  node_taints           = each.value.taints
  node_labels           = each.value.labels
  enable_auto_scaling   = each.value.cluster_auto_scaling
  min_count             = each.value.cluster_auto_scaling_min_count
  max_count             = each.value.cluster_auto_scaling_max_count
}
```
We will be using the for_each expression to be able to define and deploy multiple nodepools later.
The variable is defined as follows:

```

variable "az_aks_additional_node_pools" {
  type = map(object({
    node_count                     = number
    name                           = string
    mode                           = string
    vm_size                        = string
    taints                         = list(string)
    cluster_auto_scaling           = bool
    cluster_auto_scaling_min_count = number
    cluster_auto_scaling_max_count = number
    labels                         = map(string)
  }))
}
```
Look at 'taints' and 'labels': taint is a list of strings whereas labels are a map of strings. It took me an hour or so to figure this out, but I was also watching television at the same time. You need the labels, and the taints to configure your workloads (deployments and statefulsets) to direct the pods to the correct node pool.

Finally, this is how I define 3 node pools for a cluster. This will result in:


```
  az_aks_additional_node_pools = {
    systempool = {
      node_count = 1
      mode = "System"
      name = "system"
      vm_size    = "Standard_B2ms"
      zones      = ["1", "2"]
      taints = [
        "CriticalAddonsOnly=true:NoSchedule"
      ]
      labels = null
      cluster_auto_scaling           = false
      cluster_auto_scaling_min_count = null
      cluster_auto_scaling_max_count = null
    }
    infrapool = {
      node_count = 1
      name = "infra"
      mode = "User"
      vm_size    = "Standard_B2ms"
      zones      = ["1", "2"]
      taints = [
        "InfraAddonsOnly=true:NoSchedule"
      ]
      labels = {
        nodepool: "infra"
      }
      cluster_auto_scaling           = false
      cluster_auto_scaling_min_count = null
      cluster_auto_scaling_max_count = null
    }
    apppool = {
      node_count                     = 2
      name                           = "app16"
      mode                           = "User"
      vm_size                        = "Standard_A2m_v2"
      zones                          = ["1", "2"]
      taints                         = null
      labels = {
        nodepool: "app16"
      }
      cluster_auto_scaling           = true
      cluster_auto_scaling_min_count = 2
      cluster_auto_scaling_max_count = 4
    }
  }
```

## Cleaning up the default node pool

When the cluster and its node pools are deployed, I let Jenkins clean up de the default node pool because it's no longer needed.

```shell
az aks nodepool delete --resource-group $CLUSTER_FULL_NAME-rg --cluster-name $CLUSTER_FULL_NAME --name default
```

## Resources and inspiration
- https://www.danielstechblog.io/
- https://docs.microsoft.com/en-us/azure/aks/concepts-clusters-workloads
- https://docs.microsoft.com/en-us/azure/aks/use-multiple-node-pools



