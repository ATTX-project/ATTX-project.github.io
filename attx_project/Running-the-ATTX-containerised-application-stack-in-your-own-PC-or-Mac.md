# Running the ATTX Stack on a PC or Mac

We hereby exemplify how to run the ATTX stack in your PC or Mac, so that you can try out our Linked Open Data solution. The applications and services in the ATTX stack are fully containerised with Docker (cf. https://www.docker.com/what-container) and thus require that you install Docker CE (https://www.docker.com/community-edition) and Docker Compose (https://docs.docker.com/compose/) in your PC or Mac beforehand.

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Pre-requisite: Install Docker and Docker Compose on your PC or Mac](#pre-requisite-install-docker-and-docker-compose-on-your-pc-or-mac)
  - [Running the ATTX Application Stack with Docker](#running-the-attx-application-stack-with-docker)
  - [Test the Availability of the ATTX Containerised Stack](#test-the-availability-of-the-attx-containerised-stack)

<!-- TOC END -->


## Pre-requisite: Install Docker and Docker Compose on your PC or Mac
The installation instructions for Docker are quite clear and straightforward. For your convenience, please find below the links to the most common Operating Systems in use on Academic environments PCs.

Installing Docker Community Edition on PC (Windows):
Please note that Docker for Windows comes either with Docker CE (windows 10) or with Docker Toolbox (older Windows versions).
* [On PC (Windows 10)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description)
* [On PC (older Windows versions)](https://www.docker.com/products/docker-toolbox)

Installing Docker Community Edition on PC (GNU/Linux):
* [On PC (Ubuntu)](https://store.docker.com/editions/community/docker-ce-server-ubuntu?tab=description)
* [On PC (Fedora)](https://store.docker.com/editions/community/docker-ce-server-fedora?tab=description)

Installing Docker Community Edition on Mac
* [On Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac?tab=description)


**Installing Docker Compose**
GNU/Linux users need to install Docker Compose, Windows and Mac users have Docker Compose already bundled in their Docker installation (but please check your Docker environment as per the instructions below).
* [On all Operating Systems](https://docs.docker.com/compose/install/)

## Running the ATTX Application Stack with Docker
With Docker CE and Docker Compose successfully installed, you can now download the `docker-compose.yml` file that defines the ATTX containerised applications and service to your PC or Mac.

1. Download the ATTX `docker-compose.yml` file from https://github.com/ATTX-project/platform-deployment/blob/dev/dcompose/docker-compose.yml to a directory in your host machine.
2. Open a command line terminal in your host machine (or even the Docker Quickstart Terminal if you're using Windows or Mac) and change your working directory to the one where you downloaded the ATTX `docker-compose.yml` file.
3. In the directory where you downloaded the ATTX `docker-compose.yml` run the command `docker-compose up`. This will start up the containerised ATTX applications and services.
5. You can follow the startup messages of containerised ATTX stack on the terminal. Once the message `_attx_uv_dpus_1 exited with status 0` is displayed, you can assume that the stack is up and running (meaning that all containers have been started and that the ATTX DPUs have been deployed to UnifiedViews).


## Test the Availability of the ATTX Containerised Stack
1. Verify that the ATTX Docker containers are running. Open a new terminal and enter the `docker ps` command. The output should be similar to the following:
```shell
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
6faf8fe639b1        attx-dev:5000/gm-api:latest                           "/entrypoint.sh"         3 minutes ago       Up 3 minutes        0.0.0.0:4302->4302/tcp                           build_gmapi_1
f9f2a83adf38        tenforce/unified-views-frontend:latest                "catalina.sh run"        3 minutes ago       Up 3 minutes        0.0.0.0:8080->8080/tcp                           build_frontend_1
b88cb73c9710        tenforce/unified-views-backend:latest                 "/bin/bash /bin/st..."   3 minutes ago       Up 3 minutes        5001/tcp                                         build_backend_1
7daf4b1bcd17        attx-dev:5000/wf-api:latest                           "/entrypoint.sh"         3 minutes ago       Up 3 minutes        0.0.0.0:4301->4301/tcp                           build_wfapi_1
a3fd681bad00        attx-dev:5000/attx-fuseki:latest                      "/docker-entrypoin..."   3 minutes ago       Up 3 minutes        0.0.0.0:3030->3030/tcp                           build_fuseki_1
8f909dae60ea        attx-dev:5000/attx-es5:latest                         "/docker-entrypoin..."   3 minutes ago       Up 3 minutes        0.0.0.0:9210->9210/tcp, 0.0.0.0:9310->9310/tcp   build_elasticsearch5_1
547f6c10aea6        tenforce/unified-views-mariadb:feat-compact-modular   "docker-entrypoint..."   3 minutes ago       Up 3 minutes        3306/tcp                                         build_mysql_1
3a5c2751a642        attx-dev:5000/essiren:latest                          "/docker-entrypoin..."   3 minutes ago       Up 3 minutes        0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   build_essiren_1
```

2. Please keep in mind that if you launched the ATTX Stack with a command line terminal, the ATTX services will be using the `localhost` IP address (`127.0.0.1`). In case you used the Docker QuickStart Terminal, the IP address in use will the one of the Docker Machine (typically `192.168.99.100`, but please check in the startup messages of Docker QuickStart Terminal).
3. Open the UnifiedViews login page at `http://127.0.0.1:8080/unifiedviews/` (or `http://<docker_machine_ip>/unifiedviews/`)
4. Check the availability of Apache Jena/Fuseki at `http://127.0.0.1:3030` (or `http://<docker_machine_ip>:3030`)
5. Verify ElasticSearch 1.3.4 (with SIREN plugin) at `http://127.0.0.1:9200/_cat` (or `http://<docker_machine_ip>:9200/_cat`).
6. Check that ElasticSearch 5.x is available at `http://127.0.0.1:9210/_cat` (or `http://<docker_machine_ip>:9210/_cat`).
