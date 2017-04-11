# Deploying-ATTX-Components-on-Docker-Swarm

**Pre-requisite:** Docker Swarm formed on Cpouta and essential components provisioned \(cf. Provisioning-ATTX-Components-on-CSC-Open-Stack-cPouta.md\).

**1.** Create a persistent storage volume and attach it to the Swarm Master, as per instructions in [https://research.csc.fi/pouta-persistent-volumes](https://research.csc.fi/pouta-persistent-volumes).

**2.** Next, you should partition the new volume, label the new partition, create a new filesystem on it, and mount the volume. If you wish, you can follow the above-mentioned CSC instructions as they are, but for consistency with ATTX existent practices it's perhaps more clear to to tag the partition as "data" and mount it in "/data" as well:

Create a label \("data"\) for the partition:

```
sudo e2label /dev/vdc1 data
```

Create a "/data" mount point and mount the volume:

```
sudo mkdir /data
sudo mount /dev/vdc1 /data
```

And taking the "data" label and "/data" change into account in /etc/fstab:

```
sudo echo "LABEL=data    /data    ext4    defaults    0    0" >> /etc/fstab
```

Now you can create the directories that will be used by the ATTX containers who need data persistency \(UnifiedViews DB, Fuseki, ElasticSearch Siren, ElasticSearch 5\)

```
sudo mkdir /data/unifiedviews
sudo mkdir /data/essiren
sudo mkdir /data/elasticsearch5
```

**3.**

