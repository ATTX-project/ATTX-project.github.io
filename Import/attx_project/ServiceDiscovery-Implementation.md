# Service Discovery Implementation

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Consul in Practice](#consul-in-practice)
    - [Registering a Service](#registering-a-service)
    - [(De)Registering a Service](#deregistering-a-service)
    - [Making Use of Registered Services](#making-use-of-registered-services)
    - [Storing Key/Value](#storing-keyvalue)
  - [References](#references)

<!-- TOC END -->

## Consul in Practice

Service Discovery became an important component to most environments who can’t be satisfied with static and manual configuration of components.

### Registering a Service

### (De)Registering a Service

### Making Use of Registered Services

### Storing Key/Value

What would be the best way to store this?

* Using one key per root and storing all the root tree as JSON object
* Using key per 1-level children - and storing all grandchildren as JSON
* or storing each grand children per key

In other words:

* /foo {json}
* /foo/bar {json}
* /foo/bar/zar {json}

## References

* http://www.giantflyingsaucer.com/blog/?p=5579
* http://proxy.dockerflow.com/
* https://labs.magnet.me/nerds/2015/10/26/consultant-configuration-management-with-consul.html - for using the K/V Store
* https://www.spirulasystems.com/blog/2015/06/25/building-an-automatic-environment-using-consul-and-docker-part-1/
* https://www.spirulasystems.com/blog/2015/07/02/automatic-environment-using-consul-and-docker-swarm-part-2/
* https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/
* http://www.dblooman.com/consul/2016/09/11/Consul-and-Service-discovery,-how-it-can-help/
