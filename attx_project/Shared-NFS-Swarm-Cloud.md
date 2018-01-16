# Shared NFS Volume in Docker Swarm (on cPouta)

Please find hereby the procedure to set-up a shared NFS file system in Docker Swarm with the [Netshare Docker volume plugin](http://netshare.containx.io/)

1. Create a "/d/attx-shared" directory in the ATTX Master (attx-swarm-1): `sudo mkdir -p /d/attx-shared`

2. Create a persistent volume and mount it to /d/attx-shared (instructions here: https://research.csc.fi/pouta-persistent-volumes)

3. Install NFS tools to setup Master as NFS server:
```
$ sudo apt-get update
$ sudo apt-get install nfs-kernel-server nfs-common
```

4. Configure your the new NFS server (attx-swarm-1) to share the "/d/attx-shared" directory:

```
$ sudo vi /etc/exports
```
 Add the "/d/attx-shared" directory plus the IP address of the hosts that should read and write form it:
```
/d/attx/-shared <attx-swarm-2_IP_ADDRESS>(rw,fsid=0,insecure,no_subtree_check,sync)

/d/attx/-shared <attx-swarm-3_IP_ADDRESS>(rw,fsid=0,insecure,no_subtree_check,sync)
```

Restart the NFS server to take changes into effect:
`# sudo systemctl restart nfs-kernel-server`

5. On the NFS clients (attx-swarm-2 and attx-swarm-3), create a "/d/attx-shared" directory as mount point for the NFS export from the server (attx-swarm-1):

`$ sudo mkdir -p /d/attx-shared`

Install the NFS client tools and test mounting the NFS export into the local "/d/attx-shared":

```
# sudo apt-get install nfs-common
# mount -t nfs -o proto=tcp,port=2049 <nfs-server-IP>:/ /d/attx-shared
```

Remember to un-mount it afterwards (`unmount /d/attx-shared`).

5. Install and configure the NetShare Docker volume plugin in all hosts as per http://netshare.containx.io/docs/getting-started

6. Check your attx-swarm.yml file:

With an NFS server and shared volume in place, and the other swarm node as NFS clients, and Docker NFS Volume plugin installed in all nodes, the changes to attx-shared.yml are as such:
Volumes:
```
  attx-shared:
    driver_opts:
      type:  nfs
      o: addr=192.168.1.31,rw
      device: ":/d/attx-shared"
```
Services:
```
volumes:
  - attx_shared:/attx-shared
```
