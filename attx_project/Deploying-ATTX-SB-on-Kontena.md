# Deploying the ATTX Semantic Broker on Kontena

### Table of Contents
<!-- TOC START min:1 max:4 link:true update:false -->
  - [What is Kontena and why do we test it](#what-is-kontena-and-why-do-we-test-it)
  - [Installing Kontena CLI](#installing-kontena-cli)
    - [Registering Kontena CLI to Kontena Master](#registering-kontena-cli-to-kontena-master)
  - [Creating a Kontena grid](#creating-a-kontena-grid)
  - [Provisioning nodes on the grid](#provisioning-nodes-on-the-grid)
  - [Creating Kontena volumes](#creating-kontena-volumes)
  - [Creating and deploying a Kontena Stack](#creating-and-deploying-a-kontena-stack)
    - [Using Digital Ocean Block Storage](#using-digital-ocean-block-storage)
    - [Deploying Kontena on CSC's Open Stack (cPouta)](#deploying-kontena-on-cscs-open-stack-cpouta)
  - [Pre-Requisites](#pre-requisites)
      - [Kontena Master](#kontena-master)
      - [Kontena grid and nodes](#kontena-grid-and-nodes)
  - [Conclusions](#conclusions)
  - [References](#references)

<!-- TOC END -->

## What is Kontena and why do we test it

[Kontena](https://www.kontena.io/) is a microservices orchestration and management platform. As such, it treats containers as computational instances of managed services.

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

Kontena's CLI (Command Line Interface) [requires Ruby 2.1 or latest](https://www.ruby-lang.org/en/documentation/installation/). With Ruby in place, installing Kontena in Ubuntu 16.04 LTS is quite straightforward:

```
$ gem install --user-install kontena-cli
$ export PATH=$PATH:/home/<username>/.gem/ruby/2.3.0/bin
```

Alternatively if running MacOS, [there is an installer available](https://github.com/kontena/kontena/releases/latest).

Using kontena-cli is also intuitive, given its tree-like command structure and help function (e.g. `kontena master --help`). Auto-completing of Kontena's commands can be enabled by adding `which kontena > /dev/null && . "$( kontena whoami --bash-completion-path )"` in the users "~/.bashrc" file.

### Registering Kontena CLI to Kontena Master

All Kontena service management operations go trough a master node. Now that we have kontena-cli, we can register our client to the Kontena Master. In the case of our trial, we used a previously provisioned master node on Kontena Cloud and the respective token ("`<token_id>`") (https://www.kontena.io/cloud):

`$ kontena master join https://<kontena_master>.kontena.cloud <token_id>`

This command will open a default browser window -in our case Firefox- and ask for your Kontena Cloud account and password. Upon successful registration, the browser will exit and the following command will be displayed on the CLI terminal:

`Authenticated to Kontena Master <kontena_master> at https://<kontena_master>.kontena.cloud as <kontena_cloud_username>`

N.B.: Kontena Cloud is Kontena's hosted master cloud. It make it easier to manage, audit, and follow up the performance of your Kontena infrastructure it will host your Kontena masters, but not the nodes or storage: it's up to the user to provide the computational resources (e.g. own hardware, cloud providers, etc.). Nevertheless, it must be also noted that Kontena makes it quite easy to deploy and manage nodes and storage almost anywhere.

## Creating a Kontena grid

In Kontena, a grid is a cluster-type logical unit that groups a set of nodes plus connectivity, DNS-based service discovery, storage settings, provision for VPN access and Load Balancing configuration.

Now that we are logged in to a Kontena Master, we can provision such services by creating a grid:

`$ kontena grid create attx`


Something interesting in Kontena's proposition is that its nodes can be located either in a data centre, or in your laptop, or even in different cloud providers. Furthermore, once a Kontena grid is created, an overlay network (based on [Weave](https://www.weave.works/)) for that is automatically created as well, enabling communication for all containerised services no matter where the host nodes are located.

A new grid will also bring an [etcd](https://coreos.com/etcd) DNS-based service which will automatically register newly deployed services, and whose key-value store can be accessed in case our applications need key-value Service Discovery.

## Provisioning nodes on the grid

Now that we have our Kontena grid, we can create three nodes to provide us with computing infrastructure. In our case, we'll provision those nodes in our development environment with [Vagrant](https://www.vagrantup.com/docs/installation/index.html) (though we could have used AWS or Digital Ocean, for example).

`$ kontena vagrant node create`

We'll be asked for the number of nodes and RAM:

```shell
> How many nodes?:  3
> Choose a size  4096MB
```

We can list the created nodes and ssh into them:

```shell
$ kontena node ls
NAME               VERSION   STATUS   INITIAL   LABELS
⊛ frosty-hill-15   1.2.2     online   1 / 1     provider=vagrant
⊛ young-river-48   1.2.2     online   -         provider=vagrant
⊛ late-meadow-36   1.2.2     online   -         provider=vagrant
```
```
$ kontena node ssh frosty-hill-15
Last login: Fri Jun 30 09:56:32 UTC 2017 from 10.0.2.2 on pts/0
Container Linux by CoreOS stable (1409.5.0)
```
Thus it's confirmed that Kontena plugins use [CoreOS](https://coreos.com/) as operating system for its nodes.


## Creating Kontena volumes

At this point, we have a grid (TCP/IP connectivity, DNS service discovery, etc.), and nodes (computing resources), but not persistent storage yet.

Given that the Docker version of [UnifiedViews](https://unifiedviews.eu/) requires a [shared volume container](https://hub.docker.com/r/tenforce/unified-views-shared/), let's proceed by creating an "attx-uv-shared" shared named volume in Kontena:

`$ kontena volume create --driver local --scope stack attx-uv-shared`

And since we also need to share files between our Semantic Broker components, let's create a "attx-db-shared" volume as well:

`$ kontena volume create --driver local --scope stack attx-sb-shared`

In this trial, we have created two Kontena volumes in our attx grid, that will be using the nodes' file systems (`--driver local `), and that can be shared between  services in our Kontena stack (`--scope stack`).


## Creating and deploying a Kontena Stack

With a Kontena infrastructure platform in place, it's time to deploy our ATTX Semenatic Broker service stack. In Kontena, these are configured in a [YAML file](https://www.kontena.io/docs/references/kontena-yml.html), which [supports most of the Docker-Compose v2 variables](https://www.kontena.io/docs/references/docker-compose-support.html) but with a structure similar (e.g. in volume declaration) to [Docker Compose v3 YML](https://docs.docker.com/compose/compose-file/).

In practice (after some trial and error, but also with friendly help from Kontena staff), starting with [the Docker Compose v3 YAML file that we use with Docker Swarm](https://github.com/ATTX-project/platform-deployment/blob/dev/attx-kontena/attx-kontena.yml), we were able to create a Kontena stack file ([attx-kontena.yml](https://github.com/ATTX-project/platform-deployment/blob/dev/attx-kontena/attx-kontena.yml)), which we could deploy as following:

`$ kontena stack install --name attx attx-kontena.yml`

And verify its basic properties, as configured in [attx-kontena.yml](https://github.com/ATTX-project/platform-deployment/blob/dev/attx-kontena/attx-kontena.yml):

```shell
$ kontena stack ls

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

```shell
$ kontena service ls

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


`$ kontena service show  attx/frontend`
```yml
attx-demo/attx/frontend:
  created: 2017-06-30T09:50:35.909Z
  updated: 2017-06-30T09:50:35.909Z
  stack: attx-demo/attx
  desired_state: running
  image: attxproject/unified-views-frontend:stable-1.2
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


### Using Digital Ocean Block Storage

Let's start by creating a Kontena grid computing environment for this exercise:
```shell
$ kontena grid create do-test
[warn] Option --initial-size=1 is only recommended for test/dev usage
[done] Creating do-test grid      
[done] Switching scope to do-test grid     
```

And some Kontena Vagrant nodes in our local environment too:

```shell
kontena vagrant node create
> How many nodes?:  2
> Choose a size  4096MB
[done] Generating Vagrant config     
[done] Triggering CoreOS Container Linux box update
```

Since Kontena's Digital Ocean plug-in makes it quite easy to manage droplets and block storage volumes, let's start by installing it on our kontena-cli environment:

```shell
$ kontena plugin install digitalocean
 [done] Installing plugin digitalocean     
Installed plugin digitalocean version 0.3.0

```

And follow on by creating a Kontena node in Digital Ocean:
```shell
$ kontena digitalocean node create
> DigitalOcean API token:  
> Choose a datacenter region:  Frankfurt 1
> Select CoreOS channel Stable
> Choose droplet size:  4GB/2CPU/60GB ($40/mo)
> How many droplets?:  1
 [done] Creating DigitalOcean droplet long-morning-64      
 [done] Waiting for node long-morning-64 join to grid attx-do      



```

At this point, we have then 2 nodes running in Vagrant in our local environment, and one in Digital Ocean:

```shell
$ kontena node ls
NAME                  VERSION   STATUS   INITIAL   LABELS
⊛ long-morning-64     1.2.2     online   1 / 1     region=fra1,az=fra1,provider=digitalocean,master
⊛ hidden-field-17     1.2.2     online   -         provider=vagrant
⊛ falling-silence-7   1.2.2     online   -         provider=vagrant
```

Now we should login to [Digital Ocean](https://www.digitalocean.com/) to create a block storage volume and attach it to our newly created kontena node (long-morning-64). Once that is done, we can configure the attached volume as follows:

```shell
$ kontena node ssh long-morning-64
$ sudo mkfs.ext4 -F /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-01
$ sudo mkdir -p /mnt/volume-fra1-01; sudo mount -o discard,defaults /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-01 /mnt/volume-fra1-01; echo /dev/disk/by-id/scsi-0DO_Volume_volume-fra1-01 /mnt/volume-fra1-01 ext4 defaults,nofail,discard 0 0 | sudo tee -a /etc/fstab
```

So now that we have a "/mnt/volume-fra1-01" volume that we can use for UnifiedViews MySQL database, we can edit the mysql service defintion  in the attx stack yml file to use the block storage device mount point:
```yml
mysql:
  image: attxproject/unified-views-mariadb:stable-1.2
  ports:
    - "3306:3306"
  stateful: true
  environment:
    MYSQL_ROOT_PASSWORD: "UV"
    MYSQL_PASSWORD: "UV"
  volumes:
    - /mnt/volume-fra1-01:/var/lib/mysql
  affinity:
    - label==master
```

And then we can deploy the service (but have to create shared volumes first with `kontena volume create --driver local --scope stack attx-uv-shared`, and `kontena volume create --driver local --scope stack attx-sb-shared`):

```shell
$ kontena stack install --name attx attx-do-kontena.yml
```

When the stack finishes deploying, we can check that indeed the "mysql" service is running in the scheduled Digital Ocean node and using the defined block storage device with the `kontena service show attx/mysql` command:

```yml
attx-do/attx/mysql:
  created: 2017-08-03T13:52:48.276Z
  updated: 2017-08-03T13:52:48.276Z
  stack: attx-do/attx
  desired_state: running
  image: attxproject/unified-views-mariadb:stable-1.2
  revision: 1
  stack_revision: 1
  stateful: yes
  scaling: 1
  strategy: ha
  deploy_opts:
  dns: mysql.attx.attx-do.kontena.local
  affinity:
    - label==master
  env:
    - MYSQL_ROOT_PASSWORD=UV
    - MYSQL_PASSWORD=UV
  ports:
    - 3306:3306/tcp
  volumes:
    - /mnt/volume-fra1-01:/var/lib/mysql
  instances:
    attx/mysql/1:
      scheduled_to: long-morning-64
      deploy_rev: 2017-08-03 13:52:49 UTC
      rev: 2017-08-03 13:52:49 UTC
      state: running
      containers:
        attx.mysql-1 (on long-morning-64):
          dns: mysql-1.attx.attx-do.kontena.local
          ip: 10.81.128.73
          public ip: 207.154.202.178
          status: running
```

And what if we check the "/mnt/volume-fra1-01" on the long-morning-64 host?
```shell
long-morning-64 volume-fra1-01 # ls -l
total 176200
-rw-rw----. 1 999 rkt-admin    16384 Aug  3 13:54 aria_log.00000001
-rw-rw----. 1 999 rkt-admin       52 Aug  3 13:54 aria_log_control
-rw-rw----. 1 999 rkt-admin 50331648 Aug  3 14:21 ib_logfile0
-rw-rw----. 1 999 rkt-admin 50331648 Aug  3 13:53 ib_logfile1
-rw-rw----. 1 999 rkt-admin 79691776 Aug  3 14:21 ibdata1
drwx------. 2 999 rkt-admin    16384 Aug  3 12:05 lost+found
-rw-rw----. 1 999 rkt-admin        0 Aug  3 13:53 multi-master.info
drwx------. 2 999 rkt-admin     4096 Aug  3 13:53 mysql
drwx------. 2 999 rkt-admin     4096 Aug  3 13:53 performance_schema
-rw-rw----. 1 999 rkt-admin    24576 Aug  3 13:54 tc.log
drwx------. 2 999 rkt-admin     4096 Aug  3 13:54 unified_views
```

Success :-) Though this wasn't a complicated exercise, using Kontena to deploy the ATTX SB stack in a mixed Digital Ocean and Vagrant environment is quite easy (at least compared to setting up a similar environment using Docker Swarm).


### Deploying Kontena on CSC's Open Stack (cPouta)


Hereby, we exemplify how to deploy a Kontena master, grid, and 3 nodes to CSC's Open Stack (cPouta), using [ready-made Openstack HEAT templates](https://github.com/ATTX-project/platform-deployment/tree/dev/attx-kontena/cpouta). For this exercise you will need:

1. [A CSC user account](https://research.csc.fi/csc-guide-getting-access-to-csc-services)
2. [A cPouta project](https://research.csc.fi/pouta-application)
3. [The SSH certificate used by your cPouta project](https://research.csc.fi/pouta-connecting-a-virtual-machine)
4. OpenStack CLI tools
5. Sourcing cPouta's environment variables
6. Importing CoreOS image

We assume that 1., 2., and 3., have been satisfied, i.e. that we already have a CSC user account, a cPouta project, and have generated and downloaded the project's SSH certificate.

## Pre-Requisites

Once we have your CSC user account and cPouta project up and and running, we'll need to install the OpenStack CLI. In Ubuntu, this can be done with the "apt" tool:

```
$ sudo apt-get install python-openstackclient python-novaclient python-neutronclient python-glanceclient python-cinderclient
```

Otherwise, if you prefer you can [install the OpenStack CLI tools using PIP](https://research.csc.fi/pouta-install-client).

With the OpenStack CLI tools in place, it's now time to configure the terminal environment (e.g. BASH) and authenticate to cPouta, so that we can interact with CSC's OpenStack via command line. All cPouta projects have their authentication script available for download at their "[Access and Security](https://pouta.csc.fi/dashboard/project/access_and_security/api_access/openrc/)" dashboard panel.

After downloading the authentication script, we can source the environment variables and login to cPouta:

```shell
$ source <your_cpouta_project_name>-openrc.sh
```
The script will ask you for a password, which will be the password of your CSC account.

At this point, the only pre-requisite left is to import the CoreOS image to cPouta. Since Kontena is based on [CoreOS](https://coreos.com/) and cPouta image catalogue doesn’t include a CoreOS image, you'll have to download it and push it to cPouta. You can find the [instructions at CoreOS website](https://coreos.com/os/docs/latest/booting-on-openstack.html), just be sure to note down the name you chose for the image.

```shell
$ glance image-create --name CoreOS \
>   --container-format bare \
>   --disk-format qcow2 \
>   --file coreos_production_openstack_image.img
```

#### Kontena Master

Now we are almost ready to install the Kontena master with a [ready-made HEAT template](https://github.com/ATTX-project/platform-deployment/blob/dev/attx-kontena/cpouta/heat-kontena-master.yml). We just need to generate its SSL certificate:

```shell
$ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
$ cat certificate.crt privateKey.key > kontena_master.pem
```

Recommendation: generate Kontena master SSL certificate in the same folder where you downloaded the [HEAT templates](https://github.com/ATTX-project/platform-deployment/tree/dev/attx-kontena/cpouta).

Before installing Kontena master with the HEAT template, we should edit its parameters header in order to fit our cPouta project environment (the image name, for example):

```yml
parameters:
  key_name:
    type: string
    default: Cpouta Test
    # replace above with your key pair name as per: https://pouta.csc.fi/dashboard/project/access_and_security/
  az:
    type: string
    default: nova
    # above should be nova
  public_network:
    type: string
    default: project_2000561
    # replace the above with your project_name / tenant name e.g project_2000561
  public_ip_pool:
    type: string
    default: public
    # above should be public
  name:
    type: string
    default: kontena-master
    # this can stay as kontena-master
  volume_size:
    type: number
    default: 20
  image:
    type: string
    default: CoreOS
    # the above should be the name you gave to the CoreOS image in cPouta
```

So now that we have Kontena's master SSL certificate and parameters in place, we can proceed with the installation:

```shell
$ openstack stack create --parameter image=CoreOS --template heat-kontena-master.yml Kontena-Master
```
And then check the installation status:

```shell
$ openstack stack list
+--------------------------------------+----------------+-----------------+----------------------+--------------+
| ID                                   | Stack Name     | Stack Status    | Creation Time        | Updated Time |
+--------------------------------------+----------------+-----------------+----------------------+--------------+
| 0e74c0f1-cd21-4903-8ce8-2c66649ec05d | Kontena-Master | CREATE_COMPLETE | 2017-08-02T15:10:39Z | None         |
+--------------------------------------+----------------+-----------------+----------------------+--------------+
```

We can also test the SSH conection to the Kontena Master (be sure to [double check Kontena Master public floating IP](https://pouta.csc.fi/dashboard/project/instances/):

```shell
$ export SSL_IGNORE_ERRORS=true
$ ssh -i ~/.ssh/<cpouta_project_key>.pem core@<kontena_master_floating_public_ip>
```

It's also a good idea to retrieve and write down the initial admin code of our new Kontena master, so that we can register our local Kontena CLI to it:
```shell
$ openstack stack show Kontena-Master
| outputs               | - description: Initial admin code                                                                                                |
|                       |   output_key: initial_admin_code                                                                                                 |
|                       |   output_value: <INITIAL_ADMIN_CODE>


$ kontena master login --code <INITIAL_ADMIN_CODE> https://<kontena_master_floating_public_ip>:8443  
```




#### Kontena grid and nodes

With our local Kontena CLI registered to the Kontena master, we can create a grid and deploy nodes:

```shell
$ kontena grid create --initial-size=3 cpouta-grid
 [done] Creating cpouta-grid grid      
 [done] Switching scope to cpouta-grid grid    
```

Each Kontena grid has its own token, so be sure to retrieve it as well (the grid token will be needed to deploy the nodes into the grid with the HEAT template)

```shell
$ kontena grid show --token cpouta-grid
<GRID_TOKEN>
```
At this point, we should edit the [HEAT template for creating the grid services and nodes](https://github.com/ATTX-project/platform-deployment/blob/dev/attx-kontena/cpouta/heat-grid.yml), and make sure that the values of the parameters matches the ones that were used used for creating the Master.

Once that's in place, we can then install the Kontena grid services and nodes:

```shell
$ openstack stack create --parameter grid_name=<grid_name> --parameter kontena_master_uri=https://<master_public_ip>:8443 --parameter grid_token='<grid_token>' --parameter image=CoreOS -t heat-grid.yml <grid_name>

```

And after a few minutes, confirm that the installation was successful:

```shell
$ openstack stack show cpouta-grid
+-----------------------+-------------------------------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                                         |
+-----------------------+-------------------------------------------------------------------------------------------------------------------------------+
| id                    | 847453da-ebcf-493d-8770-fa8b2d575b0e                                                                                          |
| stack_name            | cpouta-grid                                                                                                                   |
| description           | Heat template to setup multi AZ grid                                                                                          |
|                       |                                                                                                                               |
| creation_time         | 2017-08-18T10:21:08Z                                                                                                          |
| updated_time          | None                                                                                                                          |
| stack_status          | CREATE_COMPLETE                                                                                                               |
| stack_status_reason   | Stack CREATE completed successfully                                                                                           |
```

And also, we can check whether we have new Kontena nodes and whether we can create a test Nginx service:
```shell
$ kontena node ls
NAME                             VERSION   STATUS   INITIAL   LABELS
⊛ cpouta-grid-node-2.novalocal   1.3.4     online   1 / 3     -
⊛ cpouta-grid-node-1.novalocal   1.3.4     online   2 / 3     -
⊛ cpouta-grid-node-3.novalocal   1.3.4     online   3 / 3     -


$ kontena service deploy nginx
 [done] Deploying service nginx      
⊛ Deployed instance cpouta-grid/null/nginx-1 to node cpouta-grid-node-3.novalocal
madeira@lx6-hulib-2:~/workspace/ATTX/cPouta$ kontena node ls
NAME                             VERSION   STATUS   INITIAL   LABELS
⊛ cpouta-grid-node-2.novalocal   1.3.4     online   1 / 3     -
⊛ cpouta-grid-node-1.novalocal   1.3.4     online   2 / 3     -
⊛ cpouta-grid-node-3.novalocal   1.3.4     online   3 / 3     -

$ kontena service ls
NAME      INSTANCES   STATEFUL   STATE     EXPOSED PORTS
⊝ nginx   1 / 1       no         running   0.0.0.0:80->80/tcp

$ kontena service show nginx
instances:
    nginx/1:
      scheduled_to: cpouta-grid-node-3.novalocal
      deploy_rev: 2017-08-18 10:29:34 UTC
      rev: 2017-08-18 10:29:34 UTC
      state: running
      containers:
        nginx-1 (on cpouta-grid-node-3.novalocal):
          dns: nginx-1.cpouta-grid.kontena.local
          ip: 10.81.128.13
          public ip: <node_public_ip>
          status: running

$ curl -k "<node_public_ip>"
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Conclusion: we have successfully deployed Kontena on cPouta :-)


## Conclusions

Based in our trial, we can say that:
- As a microservices orchestration and management platform, Kontena is quite user-friendly, especially given its focus on service management instead of container management;
- [Kontena's grid](https://www.kontena.io/docs/using-kontena/grids.html) infrastructure concept is quite convenient for the deployment, management, and development of a microservices application stack, given not only it  includes Service Discovery, load balancing, overlay and VPN networking functionalities, but also that it enables the provisioning of nodes in different cloud environments;
- [Kontena Cloud](https://www.kontena.io/cloud) makes it easy to deploy, manage, audit, and follow the performance of the infrastructure and services;
- [Volume management in Kontena](https://www.kontena.io/docs/using-kontena/volumes.html) is straightforward and it can use use all Docker Volume plugins. Kontena can use volumes from different cloud providers (e.g. Digital Ocean block storage, AWS S3, etc.), and it can use data storage engines (though it's up to the user to deploy and configure them - It would be nice to be able to use something like ```kontena digitalocean volume create```);
- Kontena's stacks aren't exactly the same as Docker Compose stacks, and there's no conversion tool available. Reference to [Kontena's stack documentation](https://www.kontena.io/docs/references/kontena-yml.html) is advised. Expect some trial and error when editing complex Docker Compose YML files into Kontena YML.
- Technical support from Kontena is to-the-point and quite friendly, and since it's provided by Kontena's development team, it gives it the feeling of being "by developers for developers".
- Kontena's Vagrant and Digital Ocean plugins are quite easy to use, and make it possible to store important data (e.g. UnifiedViews MySQL database) in cloud-based block storage volumes.

## References
* [Kontena](https://www.kontena.io/)
* [Kontena Cloud](https://www.kontena.io/cloud)
* [Kontena Documentation](https://www.kontena.io/docs/)
* [Kontena Github](https://github.com/kontena/kontena)
