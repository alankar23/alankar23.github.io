---
layout: page
title: 'Kubernetes Disaster Recovery: A Guide to etcd Backup'
author: <author_id>
tags: [linux, docker, kubernetes]
---

# Kubernetes Etcd Backup

## Outline

- Introduction
  k8s and etcd intro

- Prerequisites
  kubernetes, and etcd version 
  docker on backup system
- Understanding etcd
  working in short
  certs needed for authentication
- How to Back Up etcd
  
- Restoring etcd


# Backing Up Kubernetes: A Deep Dive into etcd

Ever wondered what happens when your Kubernetes cluster has a bad day? Let's talk about backing up one of its most critical components - etcd, your cluster's "source of truth."

## Introduction

Kubernetes (K8s) is like a conductor orchestrating a complex symphony of containers. Behind this orchestration lies etcd, a distributed key-value store that maintains the entire state of your cluster. Think of etcd as Kubernetes' memory bank - it stores everything from your pod configurations to your secrets. Lose etcd, and you're essentially losing your cluster's brain!

## Prerequisites

Before we dive into the backup process, make sure you have:

- Kubernetes cluster (well duh)
- Docker installed on your backup system
- Access to the Kubernetes control plane (master node)
- ~~A cup~~ Multiple cups of chai (optional but recommended ☕)

Pro tip: While these are the minimum requirements, it's always good practice to test your backup strategy on a development cluster first.

## Understanding etcd

### Authentication: The Certificates You'll Need

Security first! To interact with etcd, you'll need these certificates:

- `/etc/kubernetes/pki/etcd/ca.crt`: The etcd CA certificate
- `/etc/kubernetes/pki/etcd/server.crt`: Server certificate
- `/etc/kubernetes/pki/etcd/server.key`: Server key

These certs are your pass to etcd. Without them, you're not getting in!

💡 **Extra Tips Worth Knowing:**
- Store your backup certificates separately from your cluster
- Regular backup testing is as important as the backup itself
- Consider using automated backup solutions for production environments
- Keep track of your etcd version - it matters for restore operations

Remember: A backup is only as good as its latest test restore!

## How to Back Up etcd

### Cluster etcd Information

To back up etcd, you’ll need the etcd version and the necessary certificates. Start by retrieving the details of your etcd pod:
```
$ kubectl describe pods -n kube-system etcd-k8s-master

Name:                 etcd-k8s-master                                                                                                                                                                               
Namespace:            kube-system  
Priority:             2000001000   
Priority Class Name:  system-node-critical
Node:                 k8s-master/192.168.100.10
Start Time:           Fri, 01 Nov 2024 01:38:00 +0530     
Labels:               component=etcd                                                                      
                      tier=control-plane                                                                  
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.100.10:2379
                      kubernetes.io/config.hash: 0a0430dc440a1ab0ac89aac4cefec68c
                      kubernetes.io/config.mirror: 0a0430dc440a1ab0ac89aac4cefec68c
                      kubernetes.io/config.seen: 2024-09-07T10:46:31.843036711+05:30
                      kubernetes.io/config.source: file         
Status:               Running                       
IP:                   192.168.100.10                                                                     
IPs:                                             
  IP:           192.168.100.10                                                                           
Controlled By:  Node/k8s-master
Containers:                                                                                               
  etcd:                                
    Container ID:  containerd://2e02f2cbadcd4084acbd34374d397a94df6bfe0af9e4c8532e741320880e0b6d          
    Image:         registry.k8s.io/etcd:3.5.12-0                                                          
    Image ID:      registry.k8s.io/etcd@sha256:44a8e24dcbba3470ee1fee21d5e88d128c936e9b55d4bc51fbef8086f8ed123b                                                                                                      
    Port:          <none>                                                                                 
    Host Port:     <none>          
    Command:                                       
      etcd                         
      --advertise-client-urls=https://192.168.100.10:2379                                                
      --cert-file=/etc/kubernetes/pki/etcd/server.crt                                                     
      --client-cert-auth=true                                                                             
      --data-dir=/var/lib/etcd                     
      --experimental-initial-corrupt-check=true
      --experimental-watch-progress-notify-interval=5s                                                    
      --initial-advertise-peer-urls=https://192.168.100.10:2380
      --initial-cluster=k8s-master=https://192.168.100.10:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.10:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.100.10:2380 
      --name=k8s-master
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
```
### etcd Version:

The etcd version can be found in the image tag of the pod:

    Image:registry.k8s.io/etcd:3.5.12-0
### Certificates:
Ensure you have the correct certificate paths to access etcd securely:

	--cert-file=/etc/kubernetes/pki/etcd/server.crt
	--key-file=/etc/kubernetes/pki/etcd/server.key                                           
	--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
### Certificate Host Location:

The certificate files are mounted from the host to the pod, as seen in the `Volumes` section

  Volumes:
    etcd-certs:
      Type:          HostPath (bare host directory volume)
      Path:          /etc/kubernetes/pki/etcd
      HostPathType:  DirectoryOrCreate

### **Prep**

To start, we need to copy the certificates from the master node’s directory `/etc/kubernetes/pki/etcd/` to your local machine’s `etcd/certs` directory for authentication with etcd.  

Example of the file structure in the `etcd/certs` directory:  
```bash
$ ls -l etcd/certs
-rwxrwxrwx 1 user user 1 KiB Sun Nov 3 14:49:02 2024 ca.crt
-rwxrwxrwx 1 user user 1 KiB Sun Nov 3 14:49:02 2024 server.crt
-rwxrwxrwx 1 user user 1 KiB Sun Nov 3 14:49:02 2024 server.key
```

Next, we’ll use Docker to run the etcd container, and set up a cron job to regularly execute the container and capture a snapshot of the etcd data.

### **Dockerfile**

#### **Why Run etcd in a Docker Container?**  
Running the etcd backup container in Docker offers flexibility for future upgrades. If we decide to upgrade our cluster (including etcd) to a newer version, we’ll also need to upgrade the etcd backup system. Instead of manually managing version updates, we can simply update the image tag in our Docker Compose file, making the process much easier.

#### **Our Dockerfile**  

```
services:
  etcd:
    image: bitnami/etcd:3.5.12
    environment:
      ETCDCTL_API: 3
    entrypoint: ["etcdctl","--endpoints", "https://192.168.100.10:2379",   "--cacert=/certs/ca.crt", "--cert=/certs/server.crt", "--key=/certs/server.key", "snapshot", "save", "/backup/snapshot.db"]
    volumes:
      - ./etcd/certs:/certs
      - ./backup:/backup
```

In this setup, we've mounted the necessary certificates and the backup directory into the container. We use the `etcdctl snapshot save` command to create the snapshot backup.

#### **Running the Container**  
When we run the container using Docker Compose, we should see the following output:  

```bash
$ docker compose up  
[+] Running 2/0
 ⠿ Network etcd-backup_default   Created                                                                                                      0.0s  
 ⠿ Container etcd-backup-etcd-1  Created                                                                                                      0.0s
Attaching to etcd-backup-etcd-1
etcd-backup-etcd-1  | {"level":"info","ts":"2024-11-15T08:19:48.879484Z","caller":"snapshot/v3_snapshot.go:65","msg":"created temporary db file","path":"/backup/snapshot.db.part"}
etcd-backup-etcd-1  | {"level":"info","ts":"2024-11-15T08:19:49.029314Z","logger":"client","caller":"v3@v3.5.12/maintenance.go:212","msg":"opened snapshot stream; downloading"}
etcd-backup-etcd-1  | {"level":"info","ts":"2024-11-15T08:19:49.029366Z","caller":"snapshot/v3_snapshot.go:73","msg":"fetching snapshot","endpoint":"https://192.168.100.10:2379"}
etcd-backup-etcd-1  | {"level":"info","ts":"2024-11-15T08:20:13.297681Z","logger":"client","caller":"v3@v3.5.12/maintenance.go:220","msg":"completed snapshot read; closing"}
etcd-backup-etcd-1  | {"level":"info","ts":"2024-11-15T08:20:13.355176Z","caller":"snapshot/v3_snapshot.go:88","msg":"fetched snapshot","endpoint":"https://192.168.100.10:2379","size":"49 MB","took":"24 seconds ago"}
etcd-backup-etcd-1  | {"level":"info","ts":"2024-11-15T08:20:13.355275Z","caller":"snapshot/v3_snapshot.go:97","msg":"saved","path":"/backup/snapshot.db"}
etcd-backup-etcd-1  | Snapshot saved at /backup/snapshot.db
```

And voilà! Our etcd backup container is now up and running, successfully saving the snapshot.

### **Automating the Backup**

#### **Shell Script**

Create the backup shell script (`etcd.sh`), which will run the Docker Compose command and compress each snapshot file with a date and time stamp to reduce its size:

```
$ cat etcd.sh

#!/bin/bash

# Change this to your project directory
cd /home/alankar/etcd-backup

docker compose up 

# Define source and destination directories
SOURCE_DIR="backup/snapshot.db"
DEST_alankar="backup" 

# Get the current time for the zip file name and today's date for the directory name
CURRENT_TIME=$(date +"%H-%M")
TODAY_DATE=$(date +"%d-%m")


# Define the zip file name and the destination directory path
ZIP_NAME="etcd-$CURRENT_TIME.zip"
DEST_DIR="$DEST_alankar/$TODAY_DATE"

# Create the destination directory if it doesn't exist
mkdir -p "$DEST_DIR"

# Create the zip file
zip -r "$DEST_DIR/$ZIP_NAME" "$SOURCE_DIR"

# Output the result
echo "Created zip file: $DEST_DIR/$ZIP_NAME"
```

Basically this script runs the Docker Compose backup and then compresses the resulting snapshot file, saving it with a timestamp to help organize backups.

#### **Crontab**

We’ll now schedule this script to run every 12 hours using **crontab**. This way, backups will be taken automatically without manual intervention.

```
$ crontab -l
0 */12 * * * /home/alankar/etcd-backup/etcd.sh >/home/alankar/etcd-backup/etcd.log 
```
We are running our script at every 12 th our of the day and spitting the output in `etcd.log`

#### **Checking the Backup Output**

You can check the log file (`etcd.log`) to see the backup process in action:
```
$ cat etcd.log 
Run Time is 12-00
Attaching to etcd-etcd-1
etcd-etcd-1  | Snapshot saved at /backup/snapshot.db
etcd-etcd-1 exited with code 0
  adding: backup/snapshot.db (deflated 69%)
Created zip file: backup/15-11/etcd-12-00.zip
```
You can also verify the backups in the `backup` directory:

```
$ ls -l  backup/
drwxr-xr-x 2 alankar alankar     4096 Nov 15 12:00 15-11
-rw------- 1 alankar alankar 48611360 Nov 15 12:00 snapshot.db
$  ls -l  backup/15-11/
total 29024
-rw-r--r-- 1 alankar alankar 14872336 Nov 15 00:00 etcd-00-00.zip
-rw-r--r-- 1 alankar alankar 14847133 Nov 15 12:00 etcd-12-00.zip
```


### **Restoring etcd**

To restore an etcd snapshot:

1. **Extract the Snapshot**:  
   Decompress the zip file to retrieve the `snapshot.db`.

2. **Use the Compose File**:  
   In the restore folder, you'll find a `member` directory, which you’ll need to copy to your master node.

```
services:
  etcd:
    image: bitnami/etcd:3.5.12
    environment:
      ETCDCTL_API: 3
    entrypoint: ["etcdctl","--data-dir", "/restore/", "snapshot","restore", "/backup/snapshot.db"]
    volumes:
      - ./backup:/backup
      - ./restore:/restore
```
```
$ ls restore/
 member
```
Copy the member directory to your master nod.

3. **Update the `etcd.yaml` Static Pod**:  
Copy the member directory to your master node and edit the `etcd.yaml` static pod at `/etc/kubernetes/manifests`


Change this path to
```
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
```

To this:
```
  - hostPath:
      path: /var/newData/etcd
      type: DirectoryOrCreate
    name: etcd-data
```

4. **Wait for the Changes to Reflect**: 

After editing the static pod file, wait a few minutes for the changes to take effect. Your etcd will now be restored from the snapshot.