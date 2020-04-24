# Kubeflow with juju
## Overview
This procedure describe procedures of deploying belwo k8s cluster;
- 1 master, 1 worker and 1 worker with GPU
- Kubeflow on top of the k8s cluster

This tutorial k8s bundle include loadbalancer, Certificate Authority(EasyRSA), calico CNI and K8s v1.15.


## Prerequisite

- DNS
  - Name resolution entry
```
  hostname                   description
  -------------------------- ----------------
  vcenter.vsphere.local      vCenter server
  pesxiknl01.vsphere.local   ESXi \#1
  pesxiknl02.vsphere.local   ESXi \#2
  URL on Internet            
```

  - System-resolv
     --statusでDNSが正しくセットされていること、上記の名前が解決できることを確認

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

**※no\_proxy
(Ubuntu側のプロキシ除外変数)にCIDR（/24等）を設定した場合、curl/wgetは効くがno-proxy（jujuの変数）には効かない**

**no-proxyには以下のようにすべてのアドレスを指定する\
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

`juju add-cloud \--local  
`juju add-credential vsphere  

1. Create bootstrap.conf
```
primary-network: InternalNetworkVM  # Network for 
datastore: datastore2               # 
```
1. Create Controller VM by bootstrap command

```
juju bootstrap vsphere vsphere-00 --config bootstrap.conf --show-log
```
1. Create bundle.yaml
```
```
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
## Add Persistent Storage for Kubeflow statefull workload

1. Add StrageClass
1. Deploy Kubeflow
  Follow this instruction
  https://www.kubeflow.org/docs/started/k8s/kfctl-k8s-istio/


