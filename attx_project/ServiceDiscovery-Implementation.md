# Service Discovery Implementation

Service Discovery is part of the ATTX Semantic Broker main components as it addresses the need to deal with with static and manual configuration of components and at the same time the need to scale up the (micro)services offered by the broker.

Considering the following Service Discovery patterns:
* Client-Side Service Discovery: http://microservices.io/patterns/client-side-discovery.html
* Server-Side Service Discovery: http://microservices.io/patterns/server-side-discovery.html

In the implementation of such a Discovery Service we are exploring two solutions:
* [Kontena.io](https://kontena.io/)
* [Consul.io](https://www.consul.io/) with the following results:
    * [Consul Service Discovery Implementation](Consul-ServiceDiscovery-Implementation.md)
    * [Consul for Service Discovery on Docker Swarm](Consul-for-Service-Discovery-on-Docker-Swarm.md)
