---
processors : ["Neoverse-N1"]
software : ["linux"]
title: "Deploy WordPress"
type: docs
weight: 3
hide_summary: true
description: >
    Learn how to deploy WordPress.
---

## Pre-requisites

* Physical machines or cloud nodes with Ubuntu installed.

## Deploy the AKS cluster

* We use three yaml files to deploy WordPress: kustomization.yaml, mysql-deployment.yaml, and wordpress-deployment.yaml.

Add the folowing code in **kustomization.yaml** file

```console
secretGenerator:
  literals:
  - password=YourPasswordHere
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
```

Add the folowing code in **mysql-deployment.yaml** file

```console
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress                                                                                                                                                                                                                                                                        
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: managed-csi
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:                                                                                                                                                                                                                                                                             
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:8.0.30
        name: mysql
        env:
        - name: MYSQL_DATABASE
          value: wordpressdb
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: MYSQL_USER
          value: mysqluser
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
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

* Initializes a working directory containing Terraform configuration files.
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
az aks get-credentials --resource-group arm-aks-demo-rg-calm-rabbit --name arm-aks-cluster-demo
```
* Run the following command to see the status of the nodes. They should be in the ready state.

```console
kubectl get nodes
```

* Run the following command to see the current pods running on the cluster.

```console
kubectl get pods -A
```

[<-- Return to Learning Path](/content/en/cloud/clair/#sections)
