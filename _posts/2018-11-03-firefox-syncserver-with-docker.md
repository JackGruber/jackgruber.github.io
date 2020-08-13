---
layout: post
title: Setup your own Firefox Sync Server with Docker
show-img: true
tags: [docker, howto, network, cloud]
thumbnail-img: /assets/img/cloud.png
cover-img: "/assets/img/head/cloud.jpg"
---
If you don't want your bookmarks, tabs, cronics, ... to be stored by Firefox in the Firefox cloud, you can run your self-hosted Firefox Sync server.
After that, only the Mozilla-hosted accounts server ([https://accounts.firefox.com](https://accounts.firefox.com)) is used, but the data is stored locally.

{: .box-note}
You can safely use the Mozilla-hosted Firefox Accounts server in combination with a self-hosted sync storage server. The authentication and encryption protocols are designed so that the account server does not know the user’s plaintext password, and therefore cannot access their stored sync data.

{: .box-warning}
Two docker containers are created, a Firefox Sync Server and an nginx proxy, which acts as a reverse SSL proxy. 
Docker-compose is used to easily create and configure the containers. 

## Build the Syncserver
First you have to create a docker syncserver image. 
```
git clone https://github.com/mozilla-services/syncserver.git
cd syncserver
docker build -t syncserver .
```

## Create Configuration
```
mkdir /opt/ff-syncserver
mkdir /opt/ff-syncserver/db
chown 1000 /opt/ff-syncserver/db
chmod 700 /opt/ff-syncserver/db
```

Create the `docker-compose.yml` and `proxy.conf` files in the folder `/opt/ff-syncserver` with contents below. 
Replace `<URL>` in the `docker-composer.yml` with your URL (For which the SSL certificate was also issued) and `<KEY>` with the output of `head -c 20 /dev/urandom | sha1sum | awk '{print $1}'`.

##### docker-compose.yml
```
version: '3.7'
services:
  syncserver:
     # https://github.com/mozilla-services/syncserver.git
     thumbnail-img: syncserver
     volumes:
       - /opt/ff-syncserver/db/:/tmp
     environment:
       - SYNCSERVER_PUBLIC_URL=https://<URL>:5001
       - SYNCSERVER_SECRET=<KEY>
       - SYNCSERVER_SQLURI=sqlite:////tmp/syncserver.db
       - SYNCSERVER_BATCH_UPLOAD_ENABLED=true
       - SYNCSERVER_FORCE_WSGI_ENVIRON=true
       - SYNCSERVER_ALLOW_NEW_USER=true
       - PORT=5000   
     restart: always
     networks:
       front:
         aliases:
           - ffsync
  nginx:
    # https://github.com/nginxinc/docker-nginx
    thumbnail-img: nginx:alpine
    ports:
      - "5001:443/tcp"
    volumes:
      - /opt/ff-syncserver/nginx_proxy.conf:/etc/nginx/conf.d/proxy.conf:ro
      - /opt/ff-syncserver/cert.cer:/data/cert.cer:ro
      - /opt/ff-syncserver/cert.key:/data/cert.key:ro
    depends_on:
      - "syncserver"
    networks:
      - front
    restart: always


networks:
  front:
    driver: bridge
```
  
  
##### proxy.conf
```
server {
    listen 443 ssl;
    server_name _;
    ssl_certificate       /data/cert.cer;
    ssl_certificate_key   /data/cert.key;
    ssl_protocols TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_redirect off;
        proxy_read_timeout 120;
        proxy_connect_timeout 10;
        proxy_pass http://ffsync:5000/;
    }
}
```

## Certificate
Create an SSL certificate and copy the certificate (`cert.crt`) and the private key (`cert.key`) in PEM format to `/opt/ff-syncserver`.
Change the permission for both files:
```
chown 1000 cert.crt
chown 1000 cert.key
chmod 400 cert.crt
chmod 400 cert.key
```

## Start the docker containers
Start the containers with `docker-compose up -d`.

## Configure your Firefox
You need to change the sync server address on each Firefox installation.

* Open in Firefox `about:config` 
* Search for `identity.sync.tokenserver.uri` 
* Set the value to `https://<YOURURL>:5001/token/1.0/sync/1.5`

Now you can login to your Firefox account and the data is syncronized to your own syncserver.  


### Links:
* [Mozilla Syncserver Git repository](https://github.com/mozilla-services/syncserver)
* [Official howto from Mozilla](https://docs.services.mozilla.com/howtos/run-sync-1.5.html)
* [Nginx Docker Git repository](https://github.com/nginxinc/docker-nginx)


{: .box-error}
Firefox for Android Version 62.0.3 ignore the Port in `identity.sync.tokenserver.uri`  
[Bugzilla 1482462](https://bugzilla.mozilla.org/show_bug.cgi?id=1482462)

 