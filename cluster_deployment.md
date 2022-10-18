---
processors : ["Neoverse-N1"]
software : ["linux"]
title: "AKS Cluster Deployment Configuration"
type: docs
weight: 3
hide_summary: true
description: >
    Learn how to AKS Cluster Deployment Configuration.
---

## Pre-requisites

* Physical machines or cloud nodes with Ubuntu installed.
* Install 

## Build Nginx from source

Then follow [this documentation](https://armkeil.blob.core.windows.net/developer/Files/pdf/white-paper/guidelines-for-deploying-nginx-plus-on-aws.pdf) to setup Basic static file server.

### Steps in brief

NOTE: The below mentioned steps are used to setup Basic static file server(upstreams).

*  Switch to root user:

```console
sudo su -
```

*  Set the below parameters:

```console
sysctl net.ipv4.ip_local_port_range=”32768 60999”
sysctl net.ipv4.tcp_max_syn_backlog=60999
sysctl net.core.somaxconn=60999
sysctl net.ipv4.tcp_autocorking=0
```

* Add the following code in /etc/nginx/nginx.conf:

```console
user www-data;
worker_processes auto;
worker_rlimit_nofile 1000000;
pid /run/nginx.pid;
events {
 worker_connections 1024;
 accept_mutex off;
 multi_accept off;
}
http {
 ##
 # Basic Settings
 ##
 sendfile on;
 tcp_nopush on;
 tcp_nodelay on;
 keepalive_timeout 75;
 keepalive_requests 1000000000;
 types_hash_max_size 2048;
 include /etc/nginx/mime.types;
 default_type application/octet-stream;

 ##
 # Logging Settings
 ##

 access_log off;
 # error_log off will not turn off error logs. Error logs will redirect to /usr/share/nginx/off
 # The below comes the closest to actually turning off error logs.
 error_log /dev/null crit;
 ##
 # Virtual Host Configs
 ##
 include /etc/nginx/conf.d/*.conf;
 include /etc/nginx/sites-enabled/*;
}
```

* Add the following code into /etc/nginx/conf.d/default.conf:

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
