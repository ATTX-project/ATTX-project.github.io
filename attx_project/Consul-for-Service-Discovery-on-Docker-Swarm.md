# Consul for Service Discovery in Docker Swarm

Hereby you can find a report of a test implementation of [Consul](https://www.consul.io/) as a [Service Discovery](Service-Discovery-Solutions.md) solution in a [Docker Swarm](https://docs.docker.com/engine/swarm/) cluster.

<!-- TOC START min:1 max:3 link:true update:false -->
  - [1. Deployment on Docker Swarm](#1-deployment-on-docker-swarm)
  - [2. Possibilities](#2-possibilities)
  - [3. Limitations and a possible solution](#3-limitations-and-a-possible-solution)
  - [4. Automatic service registration](#4-automatic-service-registration)
  - [5. Conclusions](#5-conclusions)
  - [6. References](#6-references)

<!-- TOC END -->

## 1. Deployment on Docker Swarm

For this exercise, we deployed a 3-node Docker Swarm with one Master and two workers (identical to the one running in cPouta).

Such 3-node swarm was created with docker-machine (requires not only [Docker Machine](https://docs.docker.com/machine/install-machine/), but also  [Virtualbox](https://www.virtualbox.org/wiki/VirtualBox)), and can be automated by a [BASH script](https://github.com/ATTX-project/platform-deployment/blob/feature-docker-machine/swarm-mode-docker-machine/createSwarm.sh).


Once the Swarm is created, we can deploy Consul in our Docker Machine Swarm hosts. Given that Consul doesn't yet support deployment in native Docker Swarm mode, it will have to be deployed in [host mode](https://docs.docker.com/compose/compose-file/#networkmode), as per the following `docker-compose.yml` example file:

```yml
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

This example `docker-compose.yml` can be used to deploy the Consul Server in our running Swarm Master with the following commands:
1. `eval $(docker-machine env swarm-1)`
2. `export DOCKER_IP=$(docker-machine ip swarm-1)`
3. `docker-compose -f docker-compose-proxy.yml up -d consul-server`

With the Consul Server running, we can test Consul's Key-Value Store HTTP API:
1. Check the status of Consul's Key-Value Store HTTP API (should be empty at this point):
`curl "http://$(docker-machine ip swarm-1):8500/v1/catalog/service/web"`
2. Create a test entry in Consul's Key-Value Store: `curl -X PUT -d 'this is a test' "http://$(docker-machine ip swarm-1):8500/v1/kv/msg1"`
3. Retrieve your key value entry from Consul's HTTP API: `curl "http://$(docker-machine ip swarm-1):8500/v1/kv/msg1?raw"`


We can now deploy the Consul agents to the other two nodes in the cluster:

```bash
export CONSUL_SERVER_IP=$(docker-machine ip swarm-1)
for i in 2 3; do eval $(docker-machine env swarm-$i); \
export DOCKER_IP=$(docker-machine ip swarm-$i); \
docker-compose -f docker-compose-proxy.yml up -d consul-agent; \
done
```

And test the availability of the Consul agents in the worker nodes and that replication between the different instances is working:
1. Create a Key-Value entry in the Consul Agent running in the second Swarm node: `curl -X PUT -d 'this is another test' "http://$(docker-machine ip swarm-2):8500/v1/kv/msg2"`
2. Verify that our new Key-Value has been replicated to the Consul Server: `curl "http://$(docker-machine ip swarm-1):8500/v1/kv/msg2?raw"`
3. Check that it has been replicated to the the Consul Agent running in the third Swarm Node: `curl "http://$(docker-machine ip swarm-3):8500/v1/kv/msg2?raw"`

At the end of this initial step of our exercise, our system will look like this:
```
+---------------------------------------------------------------------------------+
|                                    DOCKER SWARM                                 |
|                                                                                 |
+---------------------------------------------------------------------------------+

+------------------+             +------------------+          +------------------+
|   +----------+   |             |   +----------+   |          |   +----------+   |
|   |  Consul  |   |             |   |  Consul  |   |          |   |  Consul  |   |
|   |  server  <--------------------->  agent   <------------------>  agent   |   |
|   |          |   |             |   |          |   |          |   |          |   |
|   +^---------+   |             |   +----------+   |          |   +----------+   |
|    |             |             |                  |          |                  |
|    |swarm-1      |             |     swarm-2      |          |      swarm-3     |
+------------------+             +------------------+          +------------------+
     |
     |
     |
+----+---+
|  User  |
|requests|
+--------+

```

## 2. Possibilities

With Consul is thus possible to run a Service Discovery service in Docker Swarm (albeit in "host" mode), that enables us to register the ATTX services via a HTTP API, and query the registered information as well.

Consul also replicates the registered information across its instances across the Swarm, thus enabling true elasticity in case new containers need to be replicated to new nodes

By registering and replicating service information, a Service Discovery solution such as Consul also brings in the possibility of scaling up stateful services that don't scale up so well with Docker Swarm (e.g. stateful services without a persistent storage volume, such as Consul).


## 3. Limitations and a possible solution

Given that Consul has to be run in host mode (it doesn't support running natively in Swarm mode yet), scaling up or adding more Consul Server instances in new nodes in our Swarm will pose challenges in terms of fault tolerance and elasticity.

If the Consul Server node goes down, then so go the Consul cluster and hence we lose Service Discovery.

Furthermore, if we need to scale up the Docker Swarm and deploy a new Consul Server instance in a new node, what will be the status of the Key-Value Store of that new Consul instance?

One possible solution is to use Consul in conjunction with a distributed reverse proxy and load balancer. [Docker Flow Proxy](https://github.com/vfarcic/docker-flow-proxy), is one such solution. It is a Swarm-enabled implementation of [HA Proxy](http://www.haproxy.org/) that integrates well with Consul that would bring true fault tolerance and elasticity to our ATTX Docker Swarm, and could be re-used as a reverse proxy as well.

We can thus start by creating a new Swarm network and attach to it all services that should be accessible through a reverse proxy (all other services would use a "attx-backend" network):

`docker network create --driver overlay proxy`

Hence, an example service (e.g. "Nginx") would use the "proxy" and "attx-backend" networks:
```shell
docker service create --name my_web \
--publish 49001:80 \
--network attx-backend \
--network proxy nginx
```

With the reverse proxy network in place, we can now deploy the Docker Flow Proxy across our Swarm, and configure it to communicate with the Consul services:

```shell
docker service create --name proxy \
-p 80:80 -p 443:443 -p 8080:8080 \
--network proxy -e MODE=swarm --replicas 3 \
-e CONSUL_ADDRESS="$(docker-machine ip swarm-1):8500,(docker-machine ip swarm-2):8500,$(docker-machine ip swarm-3):8500" \
vfarcic/docker-flow-proxy
```

Once the Proxy is up and running (check with `docker service ps proxy`), our system's status can be represented by the following diagram:

```
+--------+
|  User  |
|requests|
+-------^+
        |
        |
        |
+---------------------------------------------------------------------------------+
|       |                                                                         |
|       |                             DOCKER SWARM                                |
|       |                                                                         |
|       |                                                                         |
|       |                                                                         |
|      +v-----+                        +------+                       +------+    |
|      |Docker|                        |Docker|                       |Docker|    |
|      |Flow  <------------------------>Flow  <----------------------->Flow  |    |
|      |Proxy |                        |Proxy |                       |Proxy |    |
|      +--+---+                        +--+---+                       +--+---+    |
|         |                               |                              |        |
+---------------------------------------------------------------------------------+
          |                               |                              |
+------------------+             +------------------+            +----------------+
|   +-----v----+   |             |   +----v-----+   |            |   +---v----+   |
|   |  Consul  |   |             |   |  Consul  |   |            |   | Consul |   |
|   |  server  <--------------------->  agent   <--------------------> agent  |   |
|   |          |   |             |   |          |   |            |   |        |   |
|   +----------+   |             |   +----------+   |            |   +--------+   |
|                  |             |                  |            |                |
|     swarm-1      |             |     swarm-2      |            |    swarm-3     |
+------------------+             +------------------+            +----------------+

```

We can test Docker Flow Proxy by issuing it to distribute a Service Discovery registration/reconfiguration request across the Swarm's Consul instances for our Nginx test service:

```json
curl "$(docker-machine ip swarm-1):8080/v1/docker-flow-proxy/reconfigure?serviceName=my_web&servicePath=/var/www&port=49001&distribute=true"
```

And we can test whether the service registration is stored in Consul:

`curl "http://$(docker-machine ip swarm-1):8500/v1/kv/docker-flow/my_web?recurse"`

The output should include Service Discovery data in key value format, and be available in all Consul instances (replace `docker-machine ip swarm-1` with `docker-machine ip swarm-2` and `docker-machine ip swarm-3`):

```javascript
{
LockIndex: 0,
Key: "docker-flow/my_web/path",
Flags: 0,
Value: "L3Zhci93d3c=",
CreateIndex: 14224,
ModifyIndex: 14384
},
{
LockIndex: 0,
Key: "docker-flow/my_web/pathtype",
Flags: 0,
Value: "cGF0aF9iZWc=",
CreateIndex: 14221,
ModifyIndex: 14381
},
{
LockIndex: 0,
Key: "docker-flow/my_web/port",
Flags: 0,
Value: "NDkwMDE=",
CreateIndex: 14218,
ModifyIndex: 14377
},
```

Let's scale up the instances of Docker Flow Proxy and check whether the status was replicated.

Increase the number of instances from three to six:
`docker service scale proxy=6`

Find in which node is running instance number 6 of the Docker Flow Proxy ("proxy.6") and the respective ID:
```shell
NODE=$(docker service ps proxy | grep "proxy.6" | awk '{print $4}')
eval $(docker-machine env $NODE)
ID=$(docker ps | grep "proxy.6" | awk '{print $1}')
```

Query proxy.6 to check whether the service registration for the "my_web" service has been replicated.
`docker exec -it $ID cat /cfg/haproxy.cfg`.

The service registration information should be as follows (like originally registered right after service creation):

```config
frontend services
    bind *:80
    bind *:443
    mode http

    acl url_my_web49001 path_beg /var/www
    use_backend my_web-be49001 if url_my_web49001

backend my_web-be49001
    mode http
    server my_web my_web:49001
```

## 4. Automatic service registration
We used used manual service registration in the examples above. But what if we would need that our services deployed in Docker Swarm would register themselves and their own exposed application endpoints every time they would be started up in Swarm?

One possible solution is to deploy a Listener that can understand Service Registration parameters as part of the Swarm service (which we could specify as service startup options in our `docker-compose.yml` stack) and would pass them on to the distributed Flow Proxy, which would then distribute them to Consul's KV Store.

An interesting implementation of such a solution is [Docker Flow Swarm Listener](http://swarmlistener.dockerflow.com/), which integrates well with Docker Flow Proxy, being able to forward it the same [HTTP configuration parameters](http://proxy.dockerflow.com/usage/).

We can deploy Docker Flow Proxy as a service in our running swarm (where we already have deployed Consul and Docker) trough the following command:

```bash
docker service create --name swarm-listener \
    --network proxy \
    --mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
    -e DF_NOTIFY_CREATE_SERVICE_URL=http://proxy:8080/v1/docker-flow-proxy/reconfigure \
    -e DF_NOTIFY_REMOVE_SERVICE_URL=http://proxy:8080/v1/docker-flow-proxy/remove \
    --constraint 'node.role==manager' \
    vfarcic/docker-flow-swarm-listener
```

This will configure the `swarm-listener` service to forward service registration requests to Flow Proxy's own service registration interface ("http://proxy:8080/v1/docker-flow-proxy/reconfigure", the same that we have used above for manual service registration), which will then will be distributed to Consul's KV Store.

With our new `swarm-listener` up and running (check with `docker service ps swarm-listener`), we can then check the service registration for a test image ("pygradle"), that exposes two applications ("/p/0.1/index" and "/health"):

```bash
docker service create --name pygradle \
-p 4300:4300 --network proxy \
--label com.df.port=4300 \
--label com.df.notify=true \
--label com.df.distribute=true \
--label com.df.servicePath=/p/health,/p/0.1/index \
--label com.df.reqPathSearch='/p/' \
--label com.df.reqPathReplace='/' \
blankdots/test_restapi:plain
```

Once the test "pygradle" service is up, (check with `docker service ps pygradle`), we can query Consul's Key-Value Registry for the registered service parameters:

`curl "http://$(docker-machine ip swarm-1):8500/v1/kv/docker-flow/pygradle?recurse"`


```json
[{"LockIndex":0,"Key":"docker-flow/pygradle/color","Flags":0,"Value":null,"CreateIndex":1826,"ModifyIndex":1852},{"LockIndex":0,"Key":"docker-flow/pygradle/consultemplatebepath","Flags":0,"Value":null,"CreateIndex":1821,"ModifyIndex":1846},{"LockIndex":0,"Key":"docker-flow/pygradle/consultemplatefepath","Flags":0,"Value":null,"CreateIndex":1820,"ModifyIndex":1850},{"LockIndex":0,"Key":"docker-flow/pygradle/domain","Flags":0,"Value":null,"CreateIndex":1823,"ModifyIndex":1847},{"LockIndex":0,"Key":"docker-flow/pygradle/hostname","Flags":0,"Value":null,"CreateIndex":1824,"ModifyIndex":1848},{"LockIndex":0,"Key":"docker-flow/pygradle/path","Flags":0,"Value":"L3AvaGVhbHRoLC9wLzAuMS9pbmRleA==","CreateIndex":1822,"ModifyIndex":1853},{"LockIndex":0,"Key":"docker-flow/pygradle/pathtype","Flags":0,"Value":"cGF0aF9iZWc=","CreateIndex":1825,"ModifyIndex":1849},{"LockIndex":0,"Key":"docker-flow/pygradle/port","Flags":0,"Value":"NDMwMA==","CreateIndex":1819,"ModifyIndex":1854}]
```

And we check the proxy functionality for our test container ("pygradle") published service paths ("/p/health" and "/p/0.1/index", simple HTTP GET and HTTP POST endpoints, respectively):

```bash
curl -i $(docker-machine ip swarm-1)/p/health
HTTP/1.1 200 OK
Server: gunicorn/19.6.0
Date: Mon, 22 May 2017 14:19:53 GMT
content-length: 0
content-type: application/json; charset=UTF-8


curl -i $(docker-machine ip swarm-1)/p/0.1/index
HTTP/1.1 405 Method Not Allowed
Server: gunicorn/19.6.0
Date: Mon, 22 May 2017 14:20:06 GMT
content-length: 0
content-type: application/json; charset=UTF-8
allow: POST, OPTIONS
```

At this point, we have reached the following architectural schema when setting up Consul as a Service Registration and Discovery in Docker Swarm, while trying to:
1. Ensure consistency and distributability of Consul's stateful Key-Value Store, and fault tolerance plus elasticity of Consul instances.
2. Automatic Service Registration / De-Registration of Docker Swarm services at startup/shutdown.


```
+--------+
|  USER  |
|REQUESTS|
+-------++
        |
        |
+---------------------------------------------------------------------------------+
|       |                                                                         |
|     +-+------+                    DOCKER SWARM                                  |
|     | Swarm  |                                                                  |
|     |service |                                                                  |
|     +-+------+                                                                  |
|       |        +--->  SD registration/de-registration                           |
|     +-+------+                                                                  |
|     | Swarm  |                                                                  |
|     |Listener|                                                                  |
|     +-+------+                                                                  |
|       |        +--->  TCP/HTTP requests                                         |
|      +v-----+                        +------+                       +------+    |
|      |Docker|                        |Docker|                       |Docker|    |
|      |Flow  <------------------------>Flow  <----------------------->Flow  |    |
|      |Proxy |                        |Proxy |                       |Proxy |    |
|      +--+---+                        +--+---+                       +--+---+    |
|         |                               |                              |        |
+---------------------------------------------------------------------------------+
|                               |                              |
+------------------+             +------------------+            +----------------+
|   +-----v----+   |             |   +----v-----+   |            |   +---v----+   |
|   |  Consul  |   |             |   |  Consul  |   |            |   | Consul |   |
|   |  server  <--------------------->  agent   <--------------------> agent  |   |
|   |          |   |             |   |          |   |            |   |        |   |
|   +----------+   |             |   +----------+   |            |   +--------+   |
|                  |             |                  |            |                |
|     swarm-1      |             |     swarm-2      |            |    swarm-3     |
+------------------+             +------------------+            +----------------+

```

## 5. Conclusions

1. Consul is a distributed Service Discovery solution with a well documented API (https://www.consul.io/api/index.html) and [Key-Value Store](https://www.consul.io/api/kv.html), whose features compare well to other solutions (cf. https://attx-project.github.io/Service-Discovery-Solutions.html).
2. Consul doesn't support running in "swarm" mode yet, but it can be run in "host" mode across a [Docker Swarm](https://docs.docker.com/engine/swarm/). The downside is that this creates a non-fault-tolerant Consul cluster, should the Consul Server instance go down. It also makes uncertain the Key-Value Store status of a new Consul Server instance in a scaled-up Docker Swarm.
3. Consul's Docker Swarm limitations can be addressed by using [Docker Flow Proxy](http://proxy.dockerflow.com/) capabilities of distributing Service Discovery registration/reconfiguration and query requests between Consul servers and agents. Docker Flow Proxy can also be used as a distributed reverse proxy for the ATTX Project. Nevertheless, service registration calls to Flow Proxy must be triggered either manually by the user or by programmatically by the container (at startup as part of its entrypoint script).
4. Automatic service is registration/de-registration can be implement with [Docker Swarm Listener](http://swarmlistener.dockerflow.com/), which can be run as a Docker Swarm service. This evidently requires that the service registration KV parameters are known, in order to be parametrised at startup time, whether by command line or in a `docker-compose.yml` file.
5. It's possible to automate the provisioning of Consul and Docker Flow Proxy in ATTX Docker Swarm through [Docker Compose files](https://docs.docker.com/compose/compose-file/), and its deployment via BASH scripts in [Ansible](https://www.ansible.com/) playbook.

## 6. References
* [Docker Swarm](https://docs.docker.com/engine/swarm/)
* [Consul](https://www.consul.io/)
* [Docker Proxy Flow](http://proxy.dockerflow.com/)
* [Docker Swarm Listener ](http://swarmlistener.dockerflow.com/)
