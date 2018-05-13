---
layout: post
title: Docker multi arch images
show-img: true
image: /img/hello_world.jpeg
---
to create docker images with multi architecture support, create your docker image as usual for every platform. 
Then tag the images with a tag for the architecture and push them to docker. 
After all images has been pushed, create a manifest file an push them as :latest image to the hub.

** Create and push manifest **
```
docker manifest create --amend \
    jackgruber/manifest:latest \
    jackgruber/manifest:armhf \
    jackgruber/manifest:amd64

docker manifest push jackgruber/manifest:latest
```

You can inspect a manifest throu follow command:
```
docker manifest inspect jackgruber/manifest
```
