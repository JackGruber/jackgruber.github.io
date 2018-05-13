---
layout: post
title: Push a multi architecture image to Docker Hub
show-img: true
image: /img/hello_world.jpeg
---
To create docker images with multi architecture support, create your docker image as usual with a tag 
for the architecture and push them to the Docker Hub. 
After all images has been pushed, create a manifest file and push them as :latest image to the Docker Hub.

**Create and push manifest file**
```
docker manifest create --amend \
    jackgruber/manifest:latest \
    jackgruber/manifest:armhf \
    jackgruber/manifest:amd64

docker manifest push jackgruber/manifest:latest
```

You can inspect a manifest through follow command:
```
docker manifest inspect jackgruber/manifest
```

At the moment the manifest command is experimental and must be enabled.
For enabling this, edit the ```~/.docker/config.json``` and add following option ```"experimental": "enabled"```.
<img src="/img/posts/drafts/config.json.png">


