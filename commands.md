# docker swarm mode commands

## 1. creating docker-machine alias as dm for windows

```
rem dm.bat
@echo off
echo.
docker-machine.exe %*
```

## 2. creating and joining swarms

### *change DockerNAT to external using wifi switch*

```
dm ls
dm rm manager1

dm -debug create manager1

dm create -d hyperv --hyperv-virtual-switch DockerNAT manager1
dm create -d hyperv --hyperv-virtual-switch DockerNAT worker1
dm create -d hyperv --hyperv-virtual-switch DockerNAT worker2

docker swarm leave --force
docker swarm init
docker swarm join --token SWMTKN-1-2f9qbahafdkk89t1phfygmpari5s14wv1s981nqompfi5nhzkg-ckhcu9wh8as3nb4eh1jfed4wr 192.168.1.228:2377
```

## 3. docker node commands

```
docker node ls

docker node rm mqbiya141rckk1n5nhy5jhiti

docker node promote worker1
docker node demote worker1

docker node update --availability drain manager1
docker node update --availability active manager1

docker node update --label-add zone=1 manager1
docker node update --label-add zone=2 worker1
docker node update --label-add zone=3 worker2

docker node inspect manager1
docker node inspect -f '{{.Spec.Labels}}' manager1
```

## 4. docker service commands

* setting up visualizer

```
docker service create \
--constraint=node.role==manager \
--mode=global \
--publish mode=host,target=8080,published=8080 \
--mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
--name=viz \
dockersamples/visualizer
```

* various

```
docker service ls
docker service logs viz
docker service inspect viz
docker service ps viz
```

* 1-web-app

```
docker service create \
--constraint node.labels.zone!=1 \
--replicas 2 \
--placement-pref 'spread=node.labels.zone' \
-e NODE_NAME='{{.Node.Hostname}}' \
-p 80:80 \
--name nodenamer \
lrakai/nodenamer:1.0.0

docker service update --image lrakai/nodenamer:1.0.1 nodenamer
docker service scale nodenamer=6
docker service update --rollback-parallelism 2 nodenamer
docker service update --image lrakai/nodenamer:1.0.0 nodenamer
docker service rollback nodenamer
```

## 5. docker stack commands

* various

```
docker stack ls
```

* 2-stack

```
docker stack deploy -c docker-stack.yml demo
docker stack ps demo
docker stack services demo
```