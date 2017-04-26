# Using Consul for Service Discovery in Docker Swarm

Hereby you can find a report of a test implementation of [Consul](https://www.consul.io/) as a [Service Discovery](Service-Discovery.md) solution in a Docker Swarm cluster.

<!-- TOC START min:1 max:3 link:true update:false -->
1. Deployment on Docker Swarm
2. Possibilities
3. Limitations and solutions
4. Conclusions
<!-- TOC END -->


## 1. Deployment on Docker Swarm

For this exercise, we deployed a 3-node Docker Swarm with one Master and two workers (identical to the one running in cPouta).

Such 3-node swarm was created with docker-machine (requires not only [Docker Machine](https://docs.docker.com/machine/install-machine/), but also  [Virtualbox](https://www.virtualbox.org/wiki/VirtualBox)), and can be automated by a [BASH script](https://github.com/ATTX-project/platform-deployment/blob/feature-docker-machine/swarm-mode-docker-machine/createSwarm.sh).


Once the Swarm is created, we can deploy Consul in our Docker Machine Swarm hosts. Given that Consul doesn't yet support deployment in native Docker Swarm mode, it will have to be deployed in host mode, as per the following `docker-compose.yml` example file:

```
version: '2'

services:
  consul:
    container_name: consul
    image: progrium/consul
    ports:
      - 8500:8500
      - 8301:8301
      - 8300:8300
    command: -server -bootstrap

  consul-server:
    container_name: consul
    image: consul
    network_mode: host
    environment:
      - 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}'
    command: agent -server -bind=$DOCKER_IP -bootstrap-expect=1 -client=$DOCKER_IP

  consul-agent:
    container_name: consul
    image: consul
    network_mode: host
    environment:
      - 'CONSUL_LOCAL_CONFIG={"leave_on_terminate": true}'
    command: agent -bind=$DOCKER_IP -retry-join=$CONSUL_SERVER_IP -client=$DOCKER_IP

```

This example `docker-compose.file` can be used to deploy the Consul Server in our running Swarm Master with the following commands:
1. `$ export DOCKER_IP=$(docker-machine ip swarm-1)`
2. `$ docker-compose -f docker-compose-proxy.yml up -d consul-server`

With the Consul Server running, we can test Consul's Key-Value Store HTTP API:
1. Check the status of Consul's Key-Value Store HTTP API (should be empty at this point): ` curl "http://$(docker-machine ip swarm-1):8500/v1/catalog/service/web"`
2. Create a test entry in Consul's Key-Value Store: `$ curl -X PUT -d 'this is a test' "http://$(docker-machine ip swarm-1):8500/v1/kv/msg1"`
3. Retrieve your key value entry from Consul's HTTP API: `curl "http://$(docker-machine ip swarm-1):8500/v1/kv/msg1?raw"`


We can now deploy the Consul agents to the other two nodes in the cluster:

```
$ export CONSUL_SERVER_IP=$(docker-machine ip swarm-1)
$ for i in 2 3; do eval $(docker-machine env swarm-$i); export DOCKER_IP=$(docker-machine ip swarm-$i); docker-compose -f docker-compose-proxy.yml up -d consul-agent; done
```

And test the availability of the Consul agents in the worker nodes and that replication between the different instances is working:
1. Create a Key-Value entry in the Consul Agent running in the second Swarm node: `$ curl -X PUT -d 'this is another test' "http://$(docker-machine ip swarm-2):8500/v1/kv/msg2"`
2. Verify that our new Key-Value has been replicated to the Consul Server: `$ curl "http://$(docker-machine ip swarm-1):8500/v1/kv/msg2?raw"`
3. Check that it has been replicated to the the Consul Agent running in the third Swarm Node: `$ curl "http://$(docker-machine ip swarm-3):8500/v1/kv/msg2?raw"`

{TO BE CONTINUED}

## 2. Possibilities

## 3. Limitations and solutions

## 4. Conclusions
