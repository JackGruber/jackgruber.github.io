---
layout: post
title: "gnupg cannot connect to keyserver"
subtitle: "keyserver receive failed: Connection refused"
tags: [docker, error, troubleshooting]
category: troubleshooting
bigimg: "/img/head/console.jpg"
---
On a docker project where i had to add gnupg keys i always got an error `keyserver receive failed: Connection refused`.
After a while of troubleshooting, i realized it's not a connection or DNS problem. 
But that the Dirmngr did not use the configured DNS servers of the system.
```
gpg --keyserver ipv4.pool.sks-keyservers.net --recv-keys "3D06A59ECE730EB71B511C17CE752F178259BD92"
gpg: keyserver receive failed: Connection refused
```

This option forces the use of the system’s standard DNS resolver code.
```console
echo standard-resolver >> $HOME/.gnupg/dirmngr.conf;
```
If you have already try to recive keys, you need to kill the dirmngr with `pkill dirmngr`.

We can now retrieve keys again:
```shell
gpg --keyserver ipv4.pool.sks-keyservers.net --recv-keys "3D06A59ECE730EB71B511C17CE752F178259BD92"
gpg: key CE752F178259BD92: 51 signatures not checked due to missing keys
gpg: key CE752F178259BD92: public key "Isaac Bennetch <bennetch@gmail.com>" imported
gpg: no ultimately trusted keys found
gpg: Total number processed: 1
gpg:               imported: 1
```


## Dockerfile example
```dockerfile
FROM alpine
RUN apk add --no-cache gnupg

RUN mkdir $HOME/.gnupg; \
    chmod 700 $HOME/.gnupg; \
    echo standard-resolver >> $HOME/.gnupg/dirmngr.conf;

RUN gpg --keyserver ipv4.pool.sks-keyservers.net --recv-keys "3D06A59ECE730EB71B511C17CE752F178259BD92"
```
