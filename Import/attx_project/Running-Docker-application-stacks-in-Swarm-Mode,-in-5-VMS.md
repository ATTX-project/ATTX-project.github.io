# Building Swarm Mode on a PC or Mac

This guide exemplifies 5 VMs created with Vagrant and configured with Ansible, however one can use Docker Swarm Mode with Docker-Machine (https://docs.docker.com/machine/) as well.

<!-- TOC START min:1 max:3 link:true update:true -->
  - [1. Prerequisites](#1-prerequisites)
    - [Virtualbox, Vagrant and Ansible](#virtualbox-vagrant-and-ansible)
    - [Preparing the environment](#preparing-the-environment)
  - [2. Create the Swarm](#2-create-the-swarm)
    - [Create the cluster from node1:](#create-the-cluster-from-node1)
    - [Adding nodes to the Swarm](#adding-nodes-to-the-swarm)
  - [3. Building and pushing your own stack services with Compose file](#3-building-and-pushing-your-own-stack-services-with-compose-file)
    - [Inspecting stacks](#inspecting-stacks)
    - [Scaling up services (e.g. ElasticSearch)](#scaling-up-services-eg-elasticsearch)

<!-- TOC END -->

## 1. Prerequisites

### Virtualbox, Vagrant and Ansible

- Virtualbox: https://www.virtualbox.org/wiki/Downloads

- Vagrant: https://www.vagrantup.com/downloads.html
  - install vagrant-vbguest plugin (https://github.com/dotless-de/vagrant-vbguest)

- Ansible:
  - install PIP and Ansible's prerequisites:
    ```          
      $ sudo easy_install pip
      $ sudo pip install paramiko PyYAML Jinja2 httplib2 six pycrypto
    ```

  - Install Ansible:
    ```         
    $ sudo pip install ansible
    ```

  - source the setup script to make Ansible available on this terminal session:
    ```
    $ source path/to/your-ansible-clone/hacking/env-setup
    ```

  - the last step needs to be repeated every time new terminal session is open
    and want to use any Ansible command (but you'll probably only need to run
    it once).


### Preparing the environment

Download the Vagrant, Ansible, and SSH config files from
https://github.com/ATTX-project/platform-deployment/tree/feature-swarm-mode/swarm-mode.

Run the following commands:
```
$ vagrant up
$ chmod 600 private-key
$ ansible-playbook provisioning.yml
```
Now you should be able to ssh on `node1` using:
```
$ ssh vagrant@10.10.10.10 -i private-key
```
These are the default IP addresses for the nodes:
```
    10.10.10.10 node1
    10.10.10.20 node2
    10.10.10.30 node3
    10.10.10.40 node4
    10.10.10.50 node5
```

## 2. Create the Swarm
Reference: https://docs.docker.com/engine/swarm/swarm-tutorial/

The cluster is initialised with docker `swarm init`,  executed on a first, seed node.

DO NOT execute docker swarm init on multiple nodes (possibility of multiple disjoint clusters).

### Create the cluster from node1:

`$ docker swarm init`

Checking that Swarm mode is enabled

Run the docker info command:
`docker info`

The output should include:
```
Swarm: active
NodeID: 8jud7o8dax3zxbags3f8yox4b
Is Manager: true
ClusterID: 2vcw2oa9rjps3a24m91xhvv0c
...
```
When running in Swarm mode, each node advertises its address to the others (i.e. it tells them "you can contact me on 10.1.2.3:2377"). If the node has only one IP address (other than 127.0.0.1) it is used automatically, otherwise, you must specify which one to use
(Docker refuses to pick one randomly)

- You can also specify an IP address or an interface name, and also a port number
(otherwise, the default port 2377 will be used)

Example:
`docker swarm init --advertise-addr eth1`

### Adding nodes to the Swarm

Show the worker token again:

`docker swarm join-token worker`

Switch to node2 and copy-paste the `docker swarm join ...` command with the token output that was displayed just before.

Switch back to node1 and display the cluster information (node1 is a manager):

`docker node ls`

The output should be similar to the following:

```
ID             HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
8jud...ox4b *  node1     Ready   Active        Leader
ehb0...4fvx    node2     Ready   Active
```

To add more managers, display the manager token with `docker swarm join-token manager` from any manager node (e.g. node1), and copy-paste the output command to your desired node.

To add more workers, display the manager token with `docker swarm join-token worker` from any manager node (e.g. node1), and copy-paste the output command to your desired node


## 3. Building and pushing your own stack services with Compose file

Docker Compose can be directly used by a Swarm cluster through `docker stack ...` commands (New in Docker Engine 1.13), but it requires docker-compose YML version 3.

For example:

`docker stack deploy my_stack --compose-file my_stack_file.yml`

### Inspecting stacks

`docker stack ps` shows the detailed state of all services of a stack, for example:

`docker stack ps my_stack`

Confirm that we get the same output with the following command:

`docker service ps my_stack_mystack`

### Scaling up services (e.g. ElasticSearch)

Create an ElasticSearch service and scale it up with the `replicas`

```
docker service create --name ES --publish 9200:9200 --replicas 7 \
     elasticsearch:2
```

Monitor your new ElasticSearch service:

```
watch docker service ps search
```
