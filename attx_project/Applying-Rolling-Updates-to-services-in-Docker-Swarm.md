# Applying Rolling Updates to services in Docker Swarm

Hereby you can find instructions on how to applying rolling updates to services running in Docker Swarm.

The service being used used as an example is [Redis](https://hub.docker.com/_/redis/), but in practice it can be a [ATTX component](https://hub.docker.com/u/attxproject/dashboard/).

Pre-requisite: available Docker Swarm up and running.

1. Deploy service in Docker Swarm:
`docker service create --replicas 3 --name redis --update-delay 10s elasticsearch:5.2.0`

2. Check that the new service has been replicated:
`docker service inspect --pretty redis`

The "Service Mode" should be "replicated", and the number of "Replicas" should be "3":
```shell
ID:		ttjg5xkhg2fxtf1bmtdsp8q01
Name:		redis
Service Mode:	Replicated
 Replicas:	3
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
 Max failure ratio: 0
ContainerSpec:
 Image:		redis:3.0.6@sha256:6a692a76c2081888b589e26e6ec835743119fe453d67ecf03df7de5b73d69842
Resources:
Endpoint Mode:	vip
```
`docker service ls`
```
ttjg5xkhg2fx  redis       replicated  3/3       redis:3.0.6
```

You can also use the `docker service ps redis` command to verify that the status of the new service:
```shell
ID            NAME     IMAGE        NODE     DESIRED STATE  CURRENT STATE          ERROR  PORTS
1t638vum32s6  redis.1  redis:3.0.6  swarm-3  Running        Running 8 minutes ago         
qgtvi12qczt3  redis.2  redis:3.0.6  swarm-2  Running        Running 8 minutes ago         
q2sgd0ua12q2  redis.3  redis:3.0.6  swarm-1  Running        Running 8 minutes ago         

```

3. Issue the rolling update to a new version of the service (3.0.7 in this example):
`docker service update --image redis:3.0.7 redis`

Check the "UpdateStatus" of the "redis" service update (should be "completed"):
```shell
ID:		ttjg5xkhg2fxtf1bmtdsp8q01
Name:		redis
Service Mode:	Replicated
 Replicas:	3
UpdateStatus:
 State:		completed
 Started:	41 seconds
 Completed:	1 second
 Message:	update completed
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
 Max failure ratio: 0
ContainerSpec:
 Image:		redis:3.0.7@sha256:b562e3febd22fe3b0db9df909c251910ae4f1a962339e52b3d69e72f3121bd64
Resources:
Endpoint Mode:	vip
```

Finally, check that all instances of the 3.0.6 version of the "redis" service have been shutdown, and that all instances of new 3.0.7 version are running:
```shell
docker service ps redis
ID            NAME         IMAGE        NODE     DESIRED STATE  CURRENT STATE           ERROR  PORTS
iy10bjo2udtp  redis.1      redis:3.0.7  swarm-3  Running        Running 3 minutes ago          
1t638vum32s6   \_ redis.1  redis:3.0.6  swarm-3  Shutdown       Shutdown 3 minutes ago         
koo6pupp8pg8  redis.2      redis:3.0.7  swarm-2  Running        Running 3 minutes ago          
qgtvi12qczt3   \_ redis.2  redis:3.0.6  swarm-2  Shutdown       Shutdown 3 minutes ago         
0kg6bgr4zr85  redis.3      redis:3.0.7  swarm-1  Running        Running 3 minutes ago          
q2sgd0ua12q2   \_ redis.3  redis:3.0.6  swarm-1  Shutdown       Shutdown 3 minutes ago         
```
