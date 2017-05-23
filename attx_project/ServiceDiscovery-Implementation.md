# Service Discovery Implementation

<!-- TOC START min:1 max:4 link:true update:false -->
  - [Consul in Practice](#consul-in-practice)
    - [Registering a Service](#registering-a-service)
    - [Making Use of Registered Services](#making-use-of-registered-services)
        - [Using Consul Tags](#using-consul-tags)
    - [Storing Key/Value pairs in Consul](#storing-keyvalue-pairs-in-consul)
  - [References](#references)

<!-- TOC END -->


## Consul in Practice

Service Discovery is part of the ATTX Semantic Broker main components as it addresses the need to deal with with static and manual configuration of components and at the same time the need to scale up the (micro)services offered by the broker.

In order to understand what components, when they communicate and what/how they communicate to each other using Service Discover component, we envision the following:
* (micro)services offered by the Graph, Workflow and Distribution components require to register services such as Graph-Manager-API, Workflow-API, ElasticSearch or any other of processing components - see [Administrator's User Guide](User-Guide-Administrator.md);
* configurations specific to the Graph Store (e.g. Fuseki endpoint(s)) or the MariaDB configuration for Workflow-API or several (micro)services parameters will be stored in the Consul KV store;
* Accessing the registered (micro)services can be achieved either directly from the Consul DNS/HTTP API or via the proxy/router (for additional load balancing), meanwhile accessing the KV store should be done directly from Consul HTTP API - these solutions fit into the Service Discovery patterns:
    * Client-Side Service Discovery: http://microservices.io/patterns/client-side-discovery.html
    * Server-Side Service Discovery: http://microservices.io/patterns/server-side-discovery.html

### Registering a Service

Registering a service can be achieved using one of the recommended libraries https://www.consul.io/api/libraries-and-sdks.html e.g. using python-consul

```python
import consul

c = consul.Consul(host='attx-sandbox', port=8500)
# Register Service
c.agent.service.register('restapi',
                         service_id='restapi_1',
                         port=4300,
                         tags=['graph-component'])
```

One of the main questions is when to de-register a service, however a more appropriate issue would be to do a healthcheck (`/health` endpoint already available for GM and WF APIs), also considering some of the services run under supervisor, and will try be restarted, the function of de-registering should be part of shutting down a service.

### Making Use of Registered Services

An overview how we might make use of such services is as described in the diagram below, however two cases can bet identified:
1. accessing a service remote API such as HTTP/REST  at a particular location (host and port) via an client (e.g. browser, DPU);
2. having different services talking to one another.

While in both cases the Server-Side Service Discovery pattern could be used, it seems more feasible, to have the services talking to one another by querying the Service Registry/Discovery service directly or using Docker Flow Proxy (just for registering, TBD if they can talk to one another via Docker Flow Proxy) - see [Consul for Service Discovery on Docker Swarm](Consul-for-Service-Discovery-on-Docker-Swarm.md) for more details.

```
Client          Proxy/Router         Service Registry/Discovery      Service A

                                                 1.register Service A

                                              <--------------------------+
  2. Client request

+-------------------->

                    3. Proxy Asks for Service A

                +-------------------------------->
                ^--------------------------------+

                          4. Proxy Forwards request to Service A

                    +------------------------------------------------------->

             5. Service A responds to the request to Client (via Proxy)

    <----------------------------------------------------------------------+

```


#### Using Consul Tags

Tags could be used to distinguish between either special instances of a service, e.g. "Reasoning with OWL-DL" and "Reasoning with RDFS+" - same service but with different reasoners configure behind.

```json
[{
    "Node": "app-registry-discovery-node1",
    "Address": "x.x.x.x",
    "ServiceID": "api-server-8000",
    "ServiceName": "api-server",
    "ServiceTags": ["dev", "provider-abc"],
    "ServiceAddress": "someserver.domain.com",
    "ServicePort": 8000
}, {
    "Node": "app-registry-discovery-node1",
    "Address": "x.x.x.x",
    "ServiceID": "api-server-8001",
    "ServiceName": "api-server",
    "ServiceTags": ["dev", "provider-xyz"],
    "ServiceAddress": "someserver.domain.com",
    "ServicePort": 8001
}]
```
Accessing services with specific tags would work such as:
`http://consul-server/v1/catalog/service/api-server?tag=dev`

### Storing Key/Value pairs in Consul

Because the Consul provides something that resembles more of a dictionary than a document store, trying to register something like:
```json
{
  "first_level" : {
    "second_level" : {
      "third_level" : {
        "theKey" : "theValue"
      }
    }
  }
}
```

One would have several options (as pointed by: https://groups.google.com/forum/#!topic/consul-tool/DgwWdpntaAY):

* Using one key per root and storing all the root tree as JSON object e.g.`/foo {json}`;
* Using key per 1-level children - and storing all grandchildren as JSON e.g. `/foo/bar {json}`
* or storing each grand children per key e.g `/foo/bar/zar {json}`

the appropriate structure in Consul KV would be:

`/first_level/second_level/third_level/theKey` and the value for that last key would be `theValue` - solution inspired by https://github.com/Cimpress-MCP/git2consul

Another option would be to encode a JSON object to Base64 - see [Base64 encoding and decoding](https://developer.mozilla.org/en/docs/Web/API/WindowBase64/Base64_encoding_and_decoding) or http://stackoverflow.com/questions/34341836/how-to-convert-a-json-object-to-a-base64-string

## References

* Installing Hashicorp’s Consul and using Python 3 to communicate with it http://www.giantflyingsaucer.com/blog/?p=5579
* http://proxy.dockerflow.com/
* https://labs.magnet.me/nerds/2015/10/26/consultant-configuration-management-with-consul.html - for using the K/V Store
* https://www.spirulasystems.com/blog/2015/06/25/building-an-automatic-environment-using-consul-and-docker-part-1/
* https://www.spirulasystems.com/blog/2015/07/02/automatic-environment-using-consul-and-docker-swarm-part-2/
* https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/
* http://www.dblooman.com/consul/2016/09/11/Consul-and-Service-discovery,-how-it-can-help/
* https://technologyconversations.com/2016/03/21/docker-flow-proxy-on-demand-haproxy-service-discovery-and-reconfiguration/
