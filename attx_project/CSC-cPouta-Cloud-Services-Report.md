# CSC Cloud Services (cPouta) Report

Hereby you can find a report about our experience in the usage of CSC IaaS Cloud Services (cPouta) for deploying the ATTX stack on 3-node and 5-node Docker Swarm configurations, formed by three Ubuntu 14.04 VMs in the cPouta environment (https://research.csc.fi/cpouta).

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Obtaining a CSC Account](#obtaining-a-csc-account)
  - [Applying for a cPouta project](#applying-for-a-cpouta-project)
  - [Pricing and IaaS Resource Packages](#pricing-and-iaas-resource-packages)
  - [Deployment Test Scenarios](#deployment-test-scenarios)
    - [5-node Configuration](#5-node-configuration)
    - [3-node Configuration](#3-node-configuration)
  - [Customer Services](#customer-services)
  - [Documentation](#documentation)
  - [API](#api)
  - [Tools/services](#toolsservices)
  - [Conclusions](#conclusions)
  - [ATTX links](#attx-links)

<!-- TOC END -->

## Obtaining a CSC Account

Obtaining a CSC account is a pre-requisite for accessing the cPouta environment's IaaS resources. Requesting a CSC account for Academic use has found to be straightforward, requiring only filling up the respective form @https://research.csc.fi/accounts-and-projects.

Once the form has been processed (the applicant being informed by email), it's possible to login to CSC's Scientist's User Interface @https://sui.csc.fi/.

## Applying for a cPouta project
A CSC account doesn't automatically provide cPouta resources to the new user. The user must apply for cPouta resources to be assigned to his/hers relevant research project (ATTX-2016 in our case). In practice, this means applying for an _Academic CSC Project_ @https://sui.csc.fi/group/sui/resources-and-applications.

Here, the webui has been found to be not entirely intuitive (one must click on "Academic CSC Project" before the application form is displayed at the need of the page) but the instructions have been helpful (cf. https://research.csc.fi/accounts-and-projects).

Once the Academic CSC Project is created, it's possible to invite one's research team members and consequently grant them access to cPouta.

## Pricing and IaaS Resource Packages
CSC's cPouta usage is free for Finnish academic use, subject to the acceptance of a research project application (as described above), which in our case was straightforward.

The free research project usage includes a IaaS resources basic package that consists of:
* max 8 VM instances:
* max 8 vCPUs
* 33 GB RAM
* 2 public floating IPs
* 20 firewall rule sets ("security groups")
* 10 volumes
* 1000 GB of volume disk space

As elaborated below in the "Deployment Test Scenarios" section, 8 vCPUs is a too small package for forming up a Docker Swarm with High availability (cf. https://docs.docker.com/datacenter/ucp/2.0/guides/high-availability/), but it's possible to apply for additional IaaS resources.

The pricing of additional CSC's computing resources and services can be found at https://research.csc.fi/pricing-of-computing-services, and the allocation of additional vCPUs can be done in the "Resources and Applications" section in SUI (https://sui.csc.fi/group/sui/resources-and-applications).


## Deployment Test Scenarios
We experimented two testing scenarios for the deployment of a Docker Swarm cluster for the ATTX stack were tried: a 5-node configuration (three Swarm managers and two Worker nodes)  and a 3-node configuration (one Swarm manager and two Worker nodes). Given the computational limitations of the Basic project resources, namely the VCPUs related available image flavours, the 3-node scenario became preferred.

For the provisioning procedure that we used, cf. [Provisioning-ATTX-Components-on-CSC-Open-Stack-(cPouta)](Provisioning-ATTX-Components-on-CSC-Open-Stack-cPouta.md).


### 5-node Configuration
Five VMs running Ubuntu-14.04 from cPouta's image catalog (cf. https://research.csc.fi/pouta-flavours), three allocated as Docker Swarm Master (cPouta's "standard.small") and two as Workers ("standard.tiny"). Two vCPUs for the Manager nodes, and one for the workers (only eight vCPUs available in the Basic package). Result: not enough vRAM for the ElasticSearch Docker containers. Scaling up the VM's assigned vRAM proved unfeasible, as that would actually require resizing the VM's image from "standard.tiny" to "standard.small" and we were already at the limit of eight vCPus.


### 3-node Configuration
Three VMs running Ubuntu-14.04 from cPouta's image catalog (cf. https://research.csc.fi/pouta-flavours), one allocated as Docker Swarm Masters ("standard.large", four vCPUs), and two as Workers ("standard.small", two vCPUs each). This combination was proved to deliver setup that allocated sufficient about of vRAM for running the ATTX stack, but no HA as there will be no Swarm Master failover-

The summary of this combo is as follows:
* Master Node:
    * Flavor: standard.large
    * RAM: 7.8GB
    * VCPUs: 4 VCPU
    * Disk: 80GB
    * Attached external volume: 500 GB ("/data")

* Worker Nodes:
    * Flavor: standard.small
    * RAM: 2GB
    * VCPUs: 2 VCPU
    * Disk 80GB

## Customer Services
CSC's customer support services have been found to be highly professional, technically useful, and cordial. Queries to email requests (servicedesk@csc.fi) were typically answered on the same day, addressed queries directly, clarified technical issues, and referred to correct documentation.


## Documentation
We had good experience with the CSC's Cpouta documentation (https://research.csc.fi/cpouta): clear, easy to read, and all configuration commands are correct and in good sequence.

Though CSC' documentation requires at least basic technical understanding of Cloud Computing, GNU/Linux, TCP/IP, SSH, and firewalls, basic information is also provided about those topics @https://research.csc.fi/csc-guide, and training webinars as well @https://www.csc.fi/web/training.


## API
The basic OpenStack API is available for orchestration of resources. It can be used for example for create VMs, allocate storage and administrate networks. ATTX could use it to automate changes configuration of the OpenStack platform so that one doesn't need/have to use the (nice) UI.


## Tools/services
* Scientist User Interface (https://sui.csc.fi): managing projects/resources/users;
* cPouta (https://pouta.csc.fi/): managing cPouta IaaS resources (VMs, storage, firewall rules);
* cPouta's OpenStack APIs (for orchestration of IaaS resources);
* CSC Training (https://www.csc.fi/web/training): training courses and material on cloud computing for science.


## Conclusions
1. Obtaining a CSC account for Academic research usage is easy.
2. Access to cPouta for basic IaaS resources ("Academic CSC Project") is straightforward, but the application form could be more intuitive. The process is well documented, though.
3. The basic resource package for an Academic CSC project is sufficient for deploying a containerised Web Services cluster, but not enough for Docker Swarm High Availability.
4. CSC's Customer Service is highly professional and useful.
5. cPouta's documentation is reliable and of good practical help.
6. CSC's tools and services are quite practical and easy to use, though evidently it requires at least a basic technical understanding of Cloud Computing, GNU/Linux, TCP/IP, SSH, and firewalls.


## ATTX links
* [Provisioning for ATTX](Provisioning-ATTX-Components-on-CSC-Open-Stack-cPouta.md)
* [Deployment of ATTX components](ATTX-Broker-Deployment.md)
