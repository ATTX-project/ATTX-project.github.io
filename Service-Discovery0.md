## Service Discovery

Overview based on:
* **Ecosystem** - Documentation, Active Development, Open License, Ease of Use
* **Features** - Service Discovery, Health Checking etc.

| Service Tool                                                                     | Additional Tools                                                                                           | Ok | Ecosystem | Features                                                |
|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|----|-----------|---------------------------------------------------------|
| [Apache Zookeeper](http://zookeeper.apache.org/)                                 | [Apache Curator](http://curator.apache.org/)                                                               |    | ✔️️         | ~ (requires a lot of plugins to have complete features) |
| [etcd](https://github.com/coreos/etcd)                                           | [Registrator](https://github.com/gliderlabs/registrator) [confd](https://github.com/kelseyhightower/confd) |    | ✔️️         | ~ with some effort                                      |
| [Consul](https://www.consul.io)                                                  | [Consul Template](https://github.com/hashicorp/consul-template)                                            | ✔️️  | ✔️️         | ✔️️                                                       |
| [Airbnb SmartStack](http://nerds.airbnb.com/smartstack-service-discovery-cloud/) | [nerve](https://github.com/airbnb/nerve) and [synapse](https://github.com/airbnb/synapse)                  |    | ~         | ✔️️                                                       |

## References

* Comparison of Zookeeper, etcd and Consul - https://technologyconversations.com/2015/09/08/service-discovery-zookeeper-vs-etcd-vs-consul/
* Curator Service Discovery based on Zookeeper and introduction to Service Discovery - https://github.com/Netflix/curator/wiki/Service-Discovery
* Service Discovery and Distributed Configuration Stores -  https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-service-discovery-and-distributed-configuration-stores
* Service Discovery in a Microservices Architecture https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/
* Service Discovery with Docker and Consul
  * Part 1 - http://www.smartjava.org/content/service-discovery-docker-and-consul-part-1
  * Part 2 - http://www.smartjava.org/content/service-discovery-docker-and-consul-part-2
