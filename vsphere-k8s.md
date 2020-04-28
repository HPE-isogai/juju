# Kubeflow with juju x vSphere
## Overview
This procedure describe procedures of deploying belwo k8s cluster;
- 1 master, 1 worker and 1 worker with GPU
- Kubeflow on top of the k8s cluster

This tutorial k8s bundle include loadbalancer, Certificate Authority(EasyRSA), calico CNI and K8s v1.15.


## Prerequisite

- DNS
  - Name resolution entry

| hostname | description |
| --- | --- |
| vcenter host fqdn |  |
| esxi host fqdn | |

  - Confirm DNS settings
    - System-resolv --status
       --Check DNS settings
       --Check name resolution by `dig` with vcenter, esxi, and some URL on internet
              
- DHCP
  - DeployされるマシンはvCenterにルーティング可能なこと
  - dhcp配布されるアドレスがDNSで名前解決できること

- Juju Client Network
  - Connect to internet
  - Connect to vCenter/ESXi
  - Connect to DHCP machines that is provisioned by juju
  - Ensure dns/proxy setting
```
  curl google.com \--head -v -L                         #Internet Connection, make sure 200 OK
  curl https://\<vcenter-hostname\>/sdk --head -v　　   #make sure "Connected"
  curl http://\<some-internal-web-server\> --head --v 　#Internal Connection, make sure 200 OK
```

**no_proxy (Ubutu OS proxy setting) with CIDR address, curl/wget works
**no-proxy (juju proxy env) with CIDR address doesn't work, so specify all address like below
```
  **export no-proxy=\"\`echo 10.1.1.{1..254},\` 10.1.1.255\"**
```

# Procedure
## juju client base os
1. Create virtual machine for juju client  
1. Install Ubuntu  
1. Configure OS settings  
- apt-get update && apt-get upgrade  
- sudoers (NOPASSWD, Optional)  
- Proxy setting (.bashrc)  

## juju k8s deploy 1 master x 1 worker
1. Add Cloud

`juju add-cloud \--local`  
  - add vsphere
`juju add-credential vsphere`  
  - add credential information for your vSphere enviroment
  
1. Create bootstrap.conf
```
primary-network: InternalNetworkVM  # Network for machines that are deployed with juju
datastore: datastore2               # Datastores for machines / ova files that are used by bootstrap / machines
```
1. Create Controller VM by bootstrap command

```
juju bootstrap vsphere vsphere-00 --config bootstrap.conf --show-log
```
1. Create bundle.yaml
<details>
  <summary> bundle.yaml </summary>
    
```
description: For demo purpose, 1 master + GPU workers Kubernetes cluster.
machines:
  "0":
    constraints: cores=2 mem=8G root-disk=32G
  "1":
    constraints: cores=4 mem=16G root-disk=64G
series: bionic
services:
  calico:
    annotations:
      gui-x: '475'
      gui-y: '605'
    charm: cs:~containers/calico
    options:
      cidr: 172.16.0.0/16
  containerd:
    annotations:
      gui-x: '475'
      gui-y: '800'
    charm: cs:~containers/containerd-46
  easyrsa:
    annotations:
      gui-x: '90'
      gui-y: '420'
    charm: cs:~containers/easyrsa-289
    constraints: root-disk=8G
    num_units: 1
    to:
      - '0'
  etcd:
    annotations:
      gui-x: '800'
      gui-y: '420'
    charm: cs:~containers/etcd-478
    constraints: root-disk=8G
    num_units: 1
    options:
      channel: 3.2/stable
    to:
      - '0'
  kubeapi-load-balancer:
    annotations:
      gui-x: '450'
      gui-y: '250'
    charm: cs:~containers/kubeapi-load-balancer-695
    constraints: root-disk=8G
    expose: true
    num_units: 1
    to:
      - '0'
  kubernetes-master:
    annotations:
      gui-x: '800'
      gui-y: '850'
    charm: cs:~containers/kubernetes-master-746
    constraints: cores=2 mem=8G root-disk=32G
    num_units: 1
    to:
      - '0'
  kubernetes-worker:
    annotations:
      gui-x: '90'
      gui-y: '850'
    charm: cs:~containers/kubernetes-worker-588
    constraints: cores=4 mem=16G root-disk=64G
    expose: true
    num_units: 1
    to:
      - '1'
relations:
- - kubernetes-master:kube-api-endpoint
  - kubeapi-load-balancer:apiserver
- - kubernetes-master:loadbalancer
  - kubeapi-load-balancer:loadbalancer
- - kubernetes-master:kube-control
  - kubernetes-worker:kube-control
- - kubernetes-master:certificates
  - easyrsa:client
- - etcd:certificates
  - easyrsa:client
- - kubernetes-master:etcd
  - etcd:db
- - kubernetes-worker:certificates
  - easyrsa:client
- - kubernetes-worker:kube-api-endpoint
  - kubeapi-load-balancer:website
- - kubeapi-load-balancer:certificates
  - easyrsa:client
- - calico:etcd
  - etcd:db
- - calico:cni
  - kubernetes-master:cni
- - calico:cni
  - kubernetes-worker:cni
- - containerd:containerd
  - kubernetes-worker:container-runtime
- - containerd:containerd
  - kubernetes-master:container-runtime
```
</details>

1. Deploy bundle
```
juju deploy ./bundle-k8s-1.15.11-calico.yaml
```
1. Install kubectl with snap on juju client
```
sudo snap install kubectl --channel=1.15/stable --classic
source \<(kubectl completion bash)
juju scp juju scp 0:\~/config \<path-to-kubeconfig-folder\>
KUBECONFIG=\<path-to-kubeconfig-folder\>/config kubectl get node
```

1.  Confirm the progress
 * This takes about an hour
```
watch -c juju status \--color
```
1. Confirm via `kubectl`  
```
KUBECONFIG=\<path-to-kubeconfig-folder\>/config kubectl get pods -n kube-system
```

## Add worker node machine
* Because GPU doesn't attach to machine automatically, we create machine -> add GPU to the machine  

1. Add machine
```
juju add-machine --constraints "cores=4 mem=16G root-disk=64G"
```
1. Shut down the machine
1. Add GPU device for the Node which is added in the previous step via vCenter GUI
1. Boot the machine
1. Confirm GPU
```
lspci |grep NVIDIA
```

## Add GPU worker
1. Add unit
```
juju add-unit kubernetes-worker --to 2
```
1. After deploying completed, Confirm GPU node
```
juju ssh 2 sudo lsmod |grep nvidia  # Confirm roading driver
```
# Deploy Kubeflow
## Add Persistent StorageClass for Kubeflow statefull workload
1. Add StrageClass

## Deploy Kubeflow
1. Deploy Kubeflow
  Follow this instruction
  https://www.kubeflow.org/docs/started/k8s/kfctl-k8s-istio/
  * If you care corporate proxy, follow below instruction
  https://github.com/kubeflow/kfctl/issues/237

## Add PersistentVolumeClaim

| namespace | vpc name | size |
| --- | --- | --- |
| kubeflow | katib-mysql | 10Gi |
| kubeflow | metadata-mysql | 10Gi |
| kubeflow | minio-pv-claim | 20Gi |
| kubeflow | mysql-pv-claim | 20Gi |

## Access Kubeflow Dashboard
TBD

## Deploy Notebook with GPU instance
TBD
