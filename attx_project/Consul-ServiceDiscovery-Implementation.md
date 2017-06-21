# Consul Service Discovery Implementation

<!-- TOC START min:1 max:4 link:true update:false -->
  - [Consul in Practice](#consul-in-practice)
    - [Registering a Service with Consul](#registering-a-service-with-consul)
    - [Making Use of Registered Services](#making-use-of-registered-services)
      - [Using Consul Tags](#using-consul-tags)
    - [Storing Key/Value pairs in Consul](#storing-keyvalue-pairs-in-consul)
    - [Consul and Docker SWARM](#consul-and-docker-swarm)
      - [Using Consul Catalog for Service Discovery](#using-consul-catalog-for-service-discovery)
      - [Using Consul K/V for Service Discovery](#using-consul-kv-for-service-discovery)
  - [References](#references)

<!-- TOC END -->


## Consul in Practice

In order to understand what components, when they communicate and what/how they communicate to each other using Service Discover component, we envision the following:
* (micro)services offered by the Graph, Workflow and Distribution components require to register services such as Graph-Manager-API, UVProvovenance-API, ElasticSearch or any other of processing components - see [Administrator's User Guide](User-Guide-Administrator.md);
* configurations specific to the Graph Store (e.g. Fuseki endpoint(s)) or the MariaDB configuration for UVProvovenance-API or several (micro)services parameters will be stored in the Consul KV store;
* Accessing the registered (micro)services can be achieved either directly from the Consul DNS/HTTP API or via the proxy/router (for additional load balancing), meanwhile accessing the KV store should be done directly from Consul HTTP API - these solutions fit into the [Service Discovery patterns](http://microservices.io/patterns/index.html#service-discovery).

### Registering a Service with Consul

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

### Consul and Docker SWARM

As illustrated in [Consul for Service Discovery on Docker Swarm](Consul-for-Service-Discovery-on-Docker-Swarm.md) we are making use of [Docker Flow Proxy](http://proxy.dockerflow.com) and [Docker Swarm Listener](http://swarmlistener.dockerflow.com/) in order to implement a Server-Side Service Discovery pattern. However there are a few naming conventions that should be taken into consideration.

#### Using Consul Catalog for Service Discovery

In this solution there are two main components the Service Discovery (where service are registered in the Consul catalogue of service for being monitored, and possible for healthcheck) and the Service Registry (where the services are registered using the Docker Swarm listener and used by Docker Flow proxy in the Consul K/V store).

* Docker SWARM
    - serviceName = name of the docker container in the network
* Docker Flow Proxy
    - serviceName = name of the container in the network
    - servicePath = /[service name from consul]/endpoint
* Consul
    - catalog
      - serviceName = application type of service
      - serviceID = service name from SWARM
      - tag = v1
    - kv
      - serviceName/description = "Description of the the service"
      - serviceName/dataset = '/ds'

----

* Docker SWARM
    - serviceName = ES1
    - serviceName = ES5
* Docker Flow Proxy
    - elasticsearch 1.3
        - serviceName = ES1
        - servicePath = /ES1/jsonRestAPI/
        - reqPathSearch = /ES1/jsonRestAPI/
        - reqPathReplace = /
    - elasticsearch 5.x
        - serviceName = ES5
        - servicePath = /ES5/jsonRestAPI/
        - reqPathSearch = /ES5/jsonRestAPI/
        - reqPathReplace = /
* Consul
    - catalog
    ```json
    [
        {
            "serviceName": "jsonRestAPI",
            "serviceID": "ES1",
            "tags": ["v1"]
        },
        {
            "serviceName": "jsonRestAPI",
            "serviceID": "ES5",
            "tags": ["v1"]
        }
    ]
    ```
    - kv
      /ES1/description = "REST API that uses old version of the ES"
      /ES5/description = "REST API implemented with the current version of the ES"
* Querying for services of type "jsonRestAPI" and version "v1":
`http://proxy/consul/catalog/service/jsonRestAPI?tag=v1`
    - Returns: serviceIDs
* Access the service (e.g. indexing): `proxy/[serviceID]/[serviceName]/[endpoint]`
    ```
    PUT http://proxy/ES1/jsonRestAPI/[indexName]/[document type]
    ```

#### Using Consul K/V for Service Discovery

In this solution we would be using the services are registered using the Docker Swarm listener and used by Docker Flow proxy in the Consul K/V store in order to discover new services, and client side processing.

In order to add new values to the Consul K/V store, we will have to pass some environment variables to the container and at the same time follow the structure of the docker flow proxy for registering service parameters.

* Docker SWARM
    - serviceName = name of the docker container in the network
* Docker Flow Proxy
    - serviceName = name of the container in the network
    - servicePath = /[service name from consul]/endpoint
* Consul
    - kv
      - /docker-flow/serviceName/description = "Description of the the service"
      - /docker-flow/serviceName/dataset = '/ds'

----

* Docker SWARM
    - serviceName = ES1
    - serviceName = ES5
* Docker Flow Proxy
    - elasticsearch 1.3
        - serviceName = ES1
        - servicePath = /ES1/jsonRestAPI/
        - reqPathSearch = /ES1/jsonRestAPI/
        - reqPathReplace = /
    - elasticsearch 5.x
        - serviceName = ES5
        - servicePath = /ES5/jsonRestAPI/
        - reqPathSearch = /ES5/jsonRestAPI/
        - reqPathReplace = /
* Consul
    - kv
      /docker-flow/jsonRestAPI_ES1_v1/description = "REST API that uses old version of the ES"
      /docker-flow/jsonRestAPI_ES5_v1/description = "REST API implemented with the current version of the ES"
* Querying for services of type "jsonRestAPI" and version "v1":
    - `http://proxy/consul/v1/kv/docker-flow/service/jsonRestAPI_ES1_v1?keys`
    -  `http://proxy/consul/v1/kv/docker-flow/service/jsonRestAPI_ES1_v1?recurse`
* Querying for services of type "jsonRestAPI":
    - `http://proxy/consul/v1/kv/docker-flow/service/jsonRestAPI_?keys`
    -  `http://proxy/consul/v1/kv/docker-flow/service/jsonRestAPI_?recurse`
* Querying for services of type "jsonRestAPI" ES1 versions:
    - `http://proxy/consul/v1/kv/docker-flow/service/jsonRestAPI_ES1?keys`
* Querying for parameter keys of type "jsonRestAPI" ES1 version "v1":
    - `http://proxy/consul/v1/kv/docker-flow/service/jsonRestAPI_ES1_v1?keys`
* Access the service (e.g. indexing): `proxy/[serviceID]/[serviceName]/[endpoint]`
    ```
    PUT http://proxy/ES1/jsonRestAPI/[indexName]/[document type]
    ```

## References

* https://stackoverflow.com/questions/42054051/service-discovery-in-docker-without-using-consul
* https://stackoverflow.com/questions/39132292/how-to-deploy-consul-using-docker-1-12-swarm-mode
* Installing Hashicorp’s Consul and using Python 3 to communicate with it http://www.giantflyingsaucer.com/blog/?p=5579
* http://proxy.dockerflow.com/
* https://labs.magnet.me/nerds/2015/10/26/consultant-configuration-management-with-consul.html - for using the K/V Store
* https://www.spirulasystems.com/blog/2015/06/25/building-an-automatic-environment-using-consul-and-docker-part-1/
* https://www.spirulasystems.com/blog/2015/07/02/automatic-environment-using-consul-and-docker-swarm-part-2/
* https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/
* http://www.dblooman.com/consul/2016/09/11/Consul-and-Service-discovery,-how-it-can-help/
* https://technologyconversations.com/2016/03/21/docker-flow-proxy-on-demand-haproxy-service-discovery-and-reconfiguration/
