# Deploying ATTX Components on Docker Swarm

This guide exemplifies how to deploy the ATTX components on Docker Swarm, by setting up OpenStack volumes for  data persistence and using a v3 Docker Composer YML definition file.

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Setting up OpenStack Volumes for Data Persistence](#setting-up-openstack-volumes-for-data-persistence)
  - [Deploying the ATTX Components](#deploying-the-attx-components)
<!-- TOC END -->

Given that the ATTX components are containerised, we need to ensure data persistence between container restarts. In practice, ATTX components will need access to file systems in data volumes in order to write and read data.

The ATTX Docker Compose v3 YML file defines the mappings between the internal containers and file systems, and the following list states the containers that need data persistence, as well as the mappings between internal and file system data storage:
* UnifiedViews DB (MySQL): `/uv-data/mysql:/var/lib/mysql`;
* ElasticSearch 2.2 (with Siren plugin): `/attx-data/essiren:/usr/share/elasticsearch/data`;
* Elasticsearch 5.2: `/attx-data/elasticsearch5:/usr/share/elasticsearch/data`;
* Fuseki: `/attx-data/fuseki:/data/fuseki`;
* GM-API: `/attx-data/gm-api:/app/data`.

In short, the procedure will consist in first setting up OpenStack volumes for data persistency, and secondly in deploying the ATTX components defined in v3 Docker Composer YML with Docker Swarm commands.

**Pre-requisite:** Docker Swarm formed on Cpouta and essential components provisioned (cf. [Provisioning ATTX Components on CSC Open Stack \(cPouta\)](Provisioning-ATTX-Components-on-CSC-Open-Stack-cPouta.md)).

## Setting up OpenStack Volumes for Data Persistence

**1.** Create a persistent storage volume for UnifiedViews (e.g. 'uv-data') and attach it to the Swarm Master, as per instructions in https://research.csc.fi/pouta-persistent-volumes.


**2.** Create a persistent storage volume for ATTX components (e.g. 'attx-data') and attach it to the first Swarm worker, as per instructions in https://research.csc.fi/pouta-persistent-volumes.


**3.** Next, you should partition the new volumes, label them, create new file systems on them, and mount them. If you wish, you can follow the above-mentioned CSC instructions as they are, but for consistency with  existing ATTX practices we do recommend you to tag the partitions as `uv-data` and `attx-data` and mount them in `/uv-data` and `/attx-data` respectively:

**3.1**
Create a label ("uv-data") for the partition that will be use to store UnifiedViews data:
```shell
sudo e2label /dev/vdc1 uv-data
```

Create a `/uv-data` mount point and mount the volume:
```shell
sudo mkdir /uv-data
sudo mount /dev/vdc1 /uv-data
```

And taking the "data" label and "/data" change into account in /etc/fstab:
```shell
sudo echo "LABEL=uv-data    /uv-data    ext4    defaults    0    0" >> /etc/fstab
```

Now you can create the directories that will be used by the UnifiedViews containers who need data persistence (unified-views-mariadb)
```shell
sudo mkdir /uv-data/mariadb
```

**3.2.**
Create a label (`attx-data`) for the partition that will be use to store ATTX data:
```shell
sudo e2label /dev/vdc1 attx-data
```

Create a ``/attx-data` mount point and mount the volume:
```shell
sudo mkdir /attx-data
sudo mount /dev/vdc1 /attx-data
```

And taking the `attx-data` label and `/attx-data` change into account in ``/etc/fstab`:
```shell
sudo echo "LABEL=attx-data    /attx-data    ext4    defaults    0    0" >> /etc/fstab
```

Now you can create the directories that will be used by the ATTX containers that need data persistence (UnifiedViews DB, Fuseki, ElasticSearch 1.3.4, ElasticSearch 5.2)
```shell
sudo mkdir /attx-data/essiren
sudo mkdir /attx-data/elasticsearch5
sudo mkdir /attx-data/fuseki
sudo mkdir /attx-data/gmapi
```

## Deploying the ATTX Components
**1.**
With the data volumes in place (as well as the Provisioning, cf. [Provisioning ATTX Components on CSC Open Stack cPouta](Provisioning-ATTX-Components-on-CSC-Open-Stack-cPouta.md)), it's now time deploy the ATTX application stack in Docker Swarm.

Download the latest ATTX Docker Swarm YML stack (e.g. https://github.com/ATTX-project/platform-deployment/blob/dev/swarm-mode-cpouta/attx-swarm.yml) to your Swarm Master node (e.g. attx-swarm-1), to `/home/cloud-user/swarm-mode-cpouta`, for example.

To deploy the ATTX application stack in Docker Swarm from `/home/cloud-user/swarm-mode-cpouta`, run:
```shell
docker stack deploy --compose-file attx-swarm.yml attx
```

Verify that all services are starting up:
```shell
docker service ls
```

The output should be similar as follows (mind that the `uv-dpus` and `attx-shared` should run once and then exit):
```shell
cloud-user@attx-swarm-1:~/swarm-mode-cpouta$ docker service ls
ID            NAME                 MODE        REPLICAS  IMAGE
2ghu5r2ig9ii  attx_essiren         replicated  1/1       attxtest/essiren:latest
57b62rzt0dlw  attx_backend         replicated  1/1       tenforce/unified-views-backend:latest
j8bbprqi695s  attx_wfapi           replicated  1/1       attxtest/wf-api:latest
lttan3dficf0  attx_mysql           replicated  1/1       tenforce/unified-views-mariadb:feat-compact-modular
n37grvnuclbt  attx_uv-dpus         replicated  0/1       tenforce/unified-views-add-dpus:latest
nc4x8h88h84y  attx_attx-dpus       replicated  1/1       attxtest/uv-attx-dpus:latest
ozwzjngvavwn  attx_frontend        replicated  1/1       tenforce/unified-views-frontend:latest
te3o51dqwtmg  attx_shared          replicated  0/1       attxtest/uv-attx-shared:latest
v2e06blmfzan  attx_elasticsearch5  replicated  1/1       attxtest/attx-es5:latest
vmvjvz2swvp1  attx_fuseki          replicated  1/1       attxtest/attx-fuseki:latest
wyi8p9zl770c  attx_gmapi           replicated  1/1       attxtest/gm-api:latest
```

**2.**
Once the ATTX services are running, you can also check whether UnifiedViews is operational. The UnifiedViews login page should be available at `http://<Swarm_public_IP_address>:8080/unifiedviews` and you should able to login.
