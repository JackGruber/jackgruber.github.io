---
layout: post
title: Overlapping subnetworks or how to change Docker default subnet size?
show-img: true
tags: [docker, howto, error, troubleshooting, network]
image: /img/docker.png
bigimg: "/img/head/container.jpg"
---

Several services already running were no longer available from subnets after a new container was created using `docker-compose`.   
The control of the IP addresses showed that a new network 192.168.X.0/20 was created, which overlapped the existing networks and therefore the communication did not work anymore. 

By default, Docker uses the following subnet areas for creating new networks:
```
172.X.0.0/16
192.168.X.0/20
```
Also `docker-compose` uses these network areas when creating new networks, this can lead to problems in larger networks. 

To prevent this, you can specify in the `/etc/docker/daemon.json` the network areas to be used, so that there are no overlaps with your own networks. 
```
"default-address-pools":
[
	{"base":"172.17.0.0/16","size":24},
	{"base":"172.90.0.0/16","size":24}
]
```

`base` = Network area from which a subnet of `size` is selected for the new network.


To test the configuration, 50 networks were created and their subnet displayed and deleted.
{% raw %}
```
for i in {1..50}; do docker network create net-$i; done
for i in $(docker network ls -q); do docker network inspect --format='{{.Name}} {{.IPAM.Config}}' $i; done
for i in {1..50}; do docker network rm net-$i; done
```
{% endraw %}

{: .box-warning}
This option is only available since version 18.06!