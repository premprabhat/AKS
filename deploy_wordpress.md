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

Add the folowing code in **wordpress-deployment.yaml** file

```console
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
  loadBalancerSourceRanges: ["0.0.0.0/0"]
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
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
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:6.0.2-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: WORDPRESS_DB_NAME
          value: wordpressdb
        - name: WORDPRESS_DB_USER
          value: mysqluser
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

* Now apply the configuration.

```console
kubectl apply -k ./
```

* Check on the volume claims by running the following command:
```console
kubectl get pvc
```

* Check on the WordPress and MySQL pods by running the following command:
```console
kubectl get pods
```

* Now that the pods are running, we can verify that the persistent volumes have been attached by running the following command:

```console
kubectl get volumeattachments
```
* To get the external IP address of the WordPress deployment run the following command:

```console
kubectl get svc
```

* Now use the external IP and open it in the browser. WordPress welcome screen will be displayed in the browser.



[<-- Return to Learning Path](/content/en/cloud/clair/#sections)
