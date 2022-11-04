---
processors : ["Neoverse-N1"]
software : ["linux"]
title: "Deploy and connect to the AKS Cluster"
type: docs
weight: 3
hide_summary: true
description: >
    Learn how to deploy and connect to the AKS Cluster.
---

## Pre-requisites

* Physical machines or cloud nodes with Ubuntu installed.
* Install Terraform, Kubectl and Azure CLI on Arm instance.

## Deploy the AKS cluster

*  AKS deployement configuration:

For AKS deployement the Terraform configuration is broken into four files: providers.tf, variables.tf, main.tf, and outputs.tf

Add the folowing code in **providers.tf** file

```console
terraform {
  required_version = ">=1.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>3.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~>3.0"
    }
  }
}
provider "azurerm" {
  features {}
}
```

Add the folowing code in **variables.tf** file

```console
variable "agent_count" {
  default = 3
}
variable "cluster_name" {
  default = "arm-aks-cluster-demo"
}
variable "dns_prefix" {
  default = "arm-aks"
}
variable "resource_group_location" {
  default     = "eastus"
  description = "Location of the resource group."
}
variable "resource_group_name_prefix" {
  default     = "arm-aks-demo-rg"
  description = "Prefix of the resource group name that's combined with a random ID so name is unique in your Azure subscription."
}                                                                                                                 
variable "ssh_public_key" {
  default = "~/.ssh/id_rsa.pub"
}
```

Add the folowing code in **outputs.tf** file

```console
output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}
```

Add the folowing code in **main.tf** file

```console
# Generate random resource group name
resource "random_pet" "rg_name" {
  prefix = var.resource_group_name_prefix
}
resource "azurerm_resource_group" "rg" {
  location = var.resource_group_location
  name     = random_pet.rg_name.id
}
resource "azurerm_kubernetes_cluster" "k8s" {
  location            = azurerm_resource_group.rg.location
  name                = var.cluster_name
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = var.dns_prefix
  tags                = {
    Environment = "Demo"
  }
  default_node_pool {
    name       = "demopool"
    vm_size    = "Standard_D2ps_v5"
    node_count = var.agent_count
  }                                                                                                 
  linux_profile {
    admin_username = "ubuntu"
    ssh_key {
      key_data = file(var.ssh_public_key)
    }
  }
  identity {
    type = "SystemAssigned"
  }
}
}
```

* Log into Azure using the Azure CLI

```console
az login
```

* Deploy the cluster with Terraform
```console
terraform init
```

* Deploy the cluster with Terraform
```console
terraform apply
```
Once it completes it will output the name of the resource group like this:

* Download the cluster credentials so that we can use the kubectl command.

```console
# HTTPS file server
server {
 listen 443 ssl reuseport backlog=60999;
 root /usr/share/nginx/html;
 index index.html index.htm;
 server_name $hostname;
 ssl on;
 ssl_certificate /etc/nginx/ssl/ecdsa.crt;
 ssl_certificate_key /etc/nginx/ssl/ecdsa.key;
 ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384;
 location / {
     limit_except GET {
         deny all;
     }
     try_files $uri $uri/ =404;
 }
}
```
Here $hostname will be replaced with the DNS of the machine.

Follow the instructions to [create](/key_and_certification.md) ECDSA key and certificate.

NOTE: If you want you can change the content of your HTML file as per your choice.

* Check the configuration for correct syntax run and then start Nginx Server using below commands:

```console
nginx -t -v
systemctl start nginx
```

* To verify the file server is running open the URL in your browser and now the content of your html file will be displayed:

```console
https://<IP>/
```

[<-- Return to Learning Path](/content/en/cloud/clair/#sections)
