# Deploying the ATTX Semantic Broker on Kontena

<!-- TOC START min:1 max:4 link:true update:false -->
  - [What is Kontena](#what-is-kontena)
  - [Installing Kontena CLI](#installing-kontena-cli)
  - [Creating a Kontena Stack](#creating-a-kontena-stack)
  - [Registering Kontena CLI to Kontena Master](#registering-kontena-cli-to-kontena-master)
  - [Creating a Kontena grid](#creating-a-kontena-grid)
  - [Creating nodes on the grid](#creating-nodes-on-the-grid)
  - [Creating Kontena volumes](#creating-kontena-volumes)
  - [WIP](#wip)
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
- A Kontena Master: Kontena's control and monitoring node.

Kontena's services are configured trough an YML file, which describes:
- The container image for the service;
- Networking definitions;
- Scaling properties;
- Stateful/stateless attributes.

With such YML file, it's also possible to link together the defined services, so as to create an application stack.

Hence, it could be described more as a "container as a service platform", instead of a "Docker container management system".

Given the growing complexity of the ATTX Semantic Broker application stack, we decided to try out Kontena by testing the deployment of our stack with it. Hereby you can find our report.
