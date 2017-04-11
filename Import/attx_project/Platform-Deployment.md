This ATTX Platform Deployment describes the necessary steps for setting up the whole ATTX Project or working with individual components.

**Github Repository: [platform-deployment](https://github.com/ATTX-project/platform-deployment)**

## Prerequisites
Preconditions for running the ATTX stack:
* 8 GB RAM (minimum)
* Docker Engine (https://docs.docker.com/engine/installation/)
* Docker Compose (https://docs.docker.com/compose/)

## Install steps

_TO DO: ATTX Install steps and Requirements to get started short description here._

Assuming that those preconditions are satisfied, the following commands can be used to deploy and start-up the ATTX Stack:
* `curl https://raw.githubusercontent.com/ATTX-project/platform-deployment/dev/docker-compose.yml > <your_attx_directory>/docker-compose.yml`
* `sudo docker-compose -f <your_attx_directory>/docker-compose.yml up -d` - for running in detached mode or `sudo docker-compose up -f <your_attx_directory>/docker-compose.yml` for running with console output
* Check the containers are running `docker ps`
* Shut down the containers `sudo docker-compose down`

### How to deploy and start the ATTX platform automatically in RHEL7 and Centos7:
1. Pre-requisite: sharing your SSH public keys with the target host
2. Download the "single_host_deployment.yml" Ansible playbook
3. Edit the "hosts" and "remote_user" (must have sudo rights) as appropriate
4. Run the ansible-playbook (e.g. `ansible-playbook -i hosts --ask-become-pass single_host_deployment.yml`)

## Technical Documentation

_TO DO: ATTX Technical Documentation short description here such as provisioning resources and necessary requirements._

## Gradle tasks

Build artifacts should be created/downloaded under build/{image} directory.

### build-image-attx-dpus



### build-all-images
