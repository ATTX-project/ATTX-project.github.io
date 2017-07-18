# Deploying the ATTX Semantic Broker on Kontena

<!-- TOC START min:1 max:4 link:true update:false -->
  - [What is Kontena](#what-is-kontena)
  - [Installing Kontena CLI](#installing-kontena-cli)
    - [Registering Kontena CLI to Kontena Master](#registering-kontena-cli-to-kontena-master)
  - [Creating a Kontena grid](#creating-a-kontena-grid)
  - [Provisioning nodes on the grid](#creating-nodes-on-the-grid)
  - [Creating Kontena volumes](#creating-kontena-volumes)
  - [Creating and deploying a Kontena Stack](#creating-a-kontena-stack)
  - [Work in progress](#wip)
    - [Using Digital Ocean Block Storage]
    - [Deploying Kontena on CSC Open Stack (cPouta)]
      - [Kontena Master]
      - [Kontena grid and nodes]
      - [Kontena volumes]
  - [Conclusions](#conclusions)
  - [References](#references)

<!-- TOC END -->


## What is Kontena and why do we test it

Kontena is a microservices orchestration and management platform. As such, it treats containers as computational instances of managed services.

As a platform, Kontena consists of:
- Several Kontena Nodes: the virtual machines that run the containerised Kontena services;
- A Kontena Master: Kontena's control and monitoring node;

Kontena management is done via Kontena CLI, a Ruby-based Command Line Interface.

Kontena's services are configured trough an YML file, which describes:
- The container image for the service;
- Networking definitions;
- Scaling properties;
- Stateful/stateless attributes.

With such YML file, it's also possible to link together the defined services, so as to create an application stack.

Hence, given our experience it could be described more as a "container as a service platform", instead of a "Docker container management system".

Given the growing complexity of the ATTX Semantic Broker application stack, we decided to try out Kontena by testing the deployment of our stack with it. Hereby you can find our report.

## Installing Kontena CLI

Kontena's CLI (Command Line Interface) [requires Ruby 2.1 or latest](https://www.ruby-lang.org/en/documentation/installation/). With Ruby in place installing Kontena is straightforward:

`$ sudo gem install kontena-cli`

Using kontena-cli is also intuitive, given its tree-like command structure and help function (e.g. `sudo kontena master --help`).

### Registering Kontena CLI to Kontena Master

All Kontena service management operations go trough a master node. Now that we have kontena-cli, we can register our client to the Kontena Master. In the case of our trial, we used a previously provisioned master node on Kontena Cloud and the respective token ("`<token_id>`") (https://www.kontena.io/cloud):

`$ sudo kontena master join https://<kontena_master>.kontena.cloud <token_id>`

This command will open a Firefox browser window and ask for your Kontena Cloud account and password. Upon successful registration, Firefox will exit and the following command will be displayed on the CLI terminal:

`Authenticated to Kontena Master <kontena_master> at https://<kontena_master>.kontena.cloud as <kontena_cloud_username>`

N.B.: Kontena Cloud is Kontena's hosted master cloud. It make it easier to manage, audit, and follow up the performance of your Kontena infrastructure it will host your Kontena masters, but not the nodes or storage: it's up to the user to provide the computational resources (e.g. own hardware, cloud providers, etc.). Nevertheless, it must be also noted that Kontena makes it quite easy to deploy and manage nodes and storage almost anywhere.

## Creating a Kontena grid

In Kontena, a grid is a cluster-type logical unit that groups a set of nodes plus connectivity, DNS-based service discovery, storage settings, provision for VPN access and Load Balancing configuration.

Now that we are logged in to a Kontena Master, we can create a provision such services by creating a grid:

`$ sudo kontena grid create attx`


Something interesting in Kontena's proposition is that its nodes can be located either in a data centre, or in your laptop, or even in different cloud providers. Furthermore, once a Kontena grid is created, an overlay network (based on [Weave](https://www.weave.works/)) for that is automatically created as well, enabling communication for all containerised services no matter where the host nodes are located.

A new grid will also bring an [etcd](https://coreos.com/etcd) DNS-based service which will automatically register newly deployed services, and whose key-value store can be accessed in case our applications need key-value Service Discovery.

## Provisioning nodes on the grid

Now that we have our Kontena grid, we can create three nodes to provide us with computing infrastructure. In our case, we'll provision those nodes in our development environment with [Vagrant](https://www.vagrantup.com/docs/installation/index.html) (though we could have used AWS or Digital Ocean, for example):

`sudo kontena vagrant node create`

We'll be asked for the number of nodes and RAM:

```
> How many nodes?:  3
> Choose a size  4096MB
```

We can list the created nodes and ssh into them:

```
$ sudo kontena node ls
NAME               VERSION   STATUS   INITIAL   LABELS
⊛ frosty-hill-15   1.2.2     online   1 / 1     provider=vagrant
⊛ young-river-48   1.2.2     online   -         provider=vagrant
⊛ late-meadow-36   1.2.2     online   -         provider=vagrant
```
```
$ sudo kontena node ssh frosty-hill-15
Last login: Fri Jun 30 09:56:32 UTC 2017 from 10.0.2.2 on pts/0
Container Linux by CoreOS stable (1409.5.0)
```
Thus it's confirmed that Kontena uses [CoreOS](https://coreos.com/) as operating system for its nodes.


## Creating Kontena volumes

At this point, we have a grid (TCP/IP connectivity, DNS service discovery, etc.), and nodes (computing resources), but not persistent storage yet.

Given that the Docker version of [UnifiedViews](https://unifiedviews.eu/) requires a [shared volume container](https://hub.docker.com/r/tenforce/unified-views-shared/), let's proceed by creating an "attx-uv-shared" shared named volume in Kontena:

`$ sudo kontena volume create --driver local --scope stack attx-uv-shared`

And since we also need to share files between our Semantic Broker components, let's create a "attx-db-shared" volume as well:

`$ sudo kontena volume create --driver local --scope stack attx-sb-shared`

In this trial, we have created two Kontena volumes in our attx grid, that will be using the nodes' file systems (`--driver local `), and that can be shared between  services in our Kontena stack (`--scope stack`).


## Creating and deploying a Kontena Stack

With a Kontena infrastructure platform in place, it's time to deploy our ATTX Semenatic Broker service stack. In Kontena, these are configured in a [YAML file](https://www.kontena.io/docs/references/kontena-yml.html), which [supports most of of the Docker-Compose v2 variables](https://www.kontena.io/docs/references/docker-compose-support.html) but with a structure similar (e.g. in volume declaration) to [Docker Compose v3 YML](https://docs.docker.com/compose/compose-file/).

In practice (after some trial and error, but also with friendly help from Kontena staff), starting with [the Docker Compose v3 YAML file that we use with Docker Swarm](https://github.com/ATTX-project/platform-deployment/blob/dev/swarm-mode-cpouta/attx-swarm.yml), we were able to create a Kontena stack file ([attx-kontena.yml](https://github.com/ATTX-project/platform-deployment/blob/feature-kontena/attx-kontena/attx-kontena.yml)), which we could deploy as following:

`$ sudo kontena stack create -name attx attx-kontena.yml`

And verify its basic properties, as configured in [attx-kontena.yml](https://github.com/ATTX-project/platform-deployment/blob/feature-kontena/attx-kontena/attx-kontena.yml):

```
$ sudo kontena stack ls

NAME            ⊝ attx
STACK           attxproject/attx:0.1.11
SERVICES        9
STATE           running
EXPOSED PORTS   *:3306->3306/tcp,*:3030->3030/tcp,*:9210->9210/tcp,*:9310->9310/tcp,*:8066->8066/tcp,*:8080->8080/tcp,*:4302->4302/tcp
NAME            ⊝ vpn
STACK           kontena/vpn:1.3.1
SERVICES        1
STATE           running
EXPOSED PORTS   *:1194->1194/udp
```

And it was also straightforward to get more information about the newly-deployed stack services and their properties:

```
$ sudo kontena service ls

NAME              INSTANCES   STATEFUL   STATE     EXPOSED PORTS
⊝ attx/gmapi      1 / 1       yes        running   0.0.0.0:4302->4302/tcp
⊝ attx/attxdpus   1 / 1       no         running
⊝ attx/frontend   1 / 1       no         running   0.0.0.0:8080->8080/tcp
⊝ attx/backend    1 / 1       no         running   0.0.0.0:8066->8066/tcp
⊝ attx/shared     1 / 1       no         running
⊝ attx/wfapi      1 / 1       no         running
⊝ attx/es5        1 / 1       yes        running   0.0.0.0:9210->9210/tcp,0.0.0.0:9310->9310/tcp
⊝ attx/fuseki     1 / 1       yes        running   0.0.0.0:3030->3030/tcp
⊝ attx/mysql      1 / 1       yes        running   0.0.0.0:3306->3306/tcp
⊝ vpn/server      1 / 1       yes        running   0.0.0.0:1194->1194/udp

```

```
$ sudo kontena service show  attx/frontend

attx-demo/attx/frontend:
  created: 2017-06-30T09:50:35.909Z
  updated: 2017-06-30T09:50:35.909Z
  stack: attx-demo/attx
  desired_state: running
  image: attxtest/unified-views-frontend:stable-1.2
  revision: 1
  stack_revision: 1
  stateful: no
  scaling: 1
  strategy: ha
  deploy_opts:
  dns: frontend.attx.attx-demo.kontena.local
  affinity:
    - label==master
  ports:
    - 8080:8080/tcp
  volumes:
    - attx-uv-shared:/unified-views
    - attx-sb-shared:/attx-shared
  links:
    - shared
    - mysql
    - backend
  instances:
    attx/frontend/1:
      scheduled_to: blue-sky-55
      deploy_rev: 2017-06-30 09:52:28 UTC
      rev: 2017-06-30 09:52:28 UTC
      state: running
      containers:
        attx.frontend-1 (on blue-sky-55):
          dns: frontend-1.attx.attx-demo.kontena.local
          ip: 10.81.128.111
          public ip: 128.214.91.110
          status: running
```
Practical result: successful deployment of our Semantic Broker stack, all applications and services operational as expected.

## Work in progress

### Using Digital Ocean Block Storage

### Deploying Kontena on CSC Open Stack (cPouta)
#### Kontena Master
#### Kontena grid and nodes
#### Kontena volumes

## Conclusions

Based in our trial, we can say that:
- As a microservices orchestration and management platform, Kontena is quite user-friendly, especially given its focus on service management instead of container management;
- [Kontena's grid](https://www.kontena.io/docs/using-kontena/grids.html) infrastructure concept is quite convenient for the deployment,  management, and development of a microservices application stack, given not only it  includes Service Discovery, load balancing, overlay and VPN networking functionalities, but also that it enables the provisioning of nodes in different cloud environments;
- [Kontena Cloud](https://www.kontena.io/cloud) makes it easy to deploy, manage, audit, and follow the performance of the infrastructure and services;
- [Volume management in Kontena](https://www.kontena.io/docs/using-kontena/volumes.html) is straightforward and it can use use all Docker Volume plugins. Kontena can use volumes from different cloud providers (e.g. Digital Ocean block storage, AWS S3, etc.), and it can use data storage engines (though it's up to the user to deploy and configure them);
- Kontena's stacks aren't exactly the same as Docker Compose stacks, and there's no conversion tool available. Reference to [Kontena's stack documentation](https://www.kontena.io/docs/references/kontena-yml.html) is advised. Expect some trial and error when editing complex Docker Compose YML files into Kontena YML.
- Technical support from Kontena is to-the-point and quite friendly, and provided by Kontena's development team, giving it the feel of being "by developers for developers".

## References
[Kontena](https://www.kontena.io/)
[Kontena Cloud](https://www.kontena.io/cloud)
[Kontena Documentation](https://www.kontena.io/docs/)
[Kontena Github](https://github.com/kontena/kontena)
