---
layout: post
title: Push a multi architecture image to Docker Hub
show-img: true
tags: [docker, howto]
image: /img/docker.png
---
To create Docker images with multi architecture support, create your Docker image as usual with a tag 
for the architecture and push them to the Docker Hub. 
After all images has been pushed, create a manifest file and push them as :latest image to the Docker Hub.

**Create and push manifest file**
```
$ docker manifest create --amend \
    jackgruber/manifest:latest \
    jackgruber/manifest:armhf \
    jackgruber/manifest:amd64

$ docker manifest push jackgruber/manifest:latest
```

You can inspect a manifest through follow command:
```
docker manifest inspect jackgruber/manifest
```
<img src="/img/posts/2018-05-13/manifest_inspect.jpg">

Now the matching version for the architecture is loaded with the ```docker run jackgruber/manifest``` command.


## Enable experimental mode
At the moment the manifest command is experimental and must be enabled.
For enabling this, edit the ```~/.docker/config.json``` and add following option ```"experimental": "enabled"```.
![Enable experimental mode #1](/img/posts/2018-05-13/config.json.png)

## manifest blob unknown
![manifest blob unknown #2](/img/posts/2018-05-13/bloberror.jpg)  
If you get follow error, delete the corresponding folder for the manifest in ```~/.docker/manifests/``` an create the manifest again.
