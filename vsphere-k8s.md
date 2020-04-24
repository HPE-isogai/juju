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

## Procedure (juju k8s deploy 1 master x 1 worker)

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

## juju k8s deploy 1 master x 1 worker

1. Add machine
```
juju add-machine --constraints "cores=4 mem=16G root-disk=64G"
```
1. Shut down the machine
1. Add GPU device for the Node which is added in the previous step via vCenter GUI
1. Boot the machine

## Add GPU worker

13. Add unit

  -----------------------------------------
  juju add-unit kubernetes-worker \--to 2
  -----------------------------------------

+----------------------------------------------------------------------+
| hitp\@ubu-gw-ext:\~\$ juju status                                    |
|                                                                      |
| Model Controller Cloud/Region Version SLA Timestamp                  |
|                                                                      |
| default vsp-controller-04 vsp-cloud/Hewlett-Packard 2.7.5            |
| unsupported 06:56:27Z                                                |
|                                                                      |
| App Version Status Scale Charm Store Rev OS Notes                    |
|                                                                      |
| calico active 3 calico jujucharms 698 ubuntu                         |
|                                                                      |
| containerd active 3 containerd jujucharms 46 ubuntu                  |
|                                                                      |
| easyrsa 3.0.1 active 1 easyrsa jujucharms 289 ubuntu                 |
|                                                                      |
| etcd 3.2.10 active 1 etcd jujucharms 478 ubuntu                      |
|                                                                      |
| kubeapi-load-balancer 1.14.0 active 1 kubeapi-load-balancer          |
| jujucharms 695 ubuntu exposed                                        |
|                                                                      |
| kubernetes-master 1.15.11 active 1 kubernetes-master jujucharms 746  |
| ubuntu                                                               |
|                                                                      |
| kubernetes-worker 1.15.11 active 2 kubernetes-worker jujucharms 588  |
| ubuntu exposed                                                       |
|                                                                      |
| Unit Workload Agent Machine Public address Ports Message             |
|                                                                      |
| easyrsa/0\* active idle 0 10.0.1.125 Certificate Authority           |
| connected.                                                           |
|                                                                      |
| etcd/0\* active idle 0 10.0.1.125 2379/tcp Healthy with 1 known peer |
|                                                                      |
| kubeapi-load-balancer/0\* active idle 0 10.0.1.125 443/tcp           |
| Loadbalancer ready.                                                  |
|                                                                      |
| kubernetes-master/0\* active idle 0 10.0.1.125 6443/tcp Kubernetes   |
| master running.                                                      |
|                                                                      |
| calico/1\* active idle 10.0.1.125 Calico is active                   |
|                                                                      |
| containerd/1\* active idle 10.0.1.125 Container runtime available    |
|                                                                      |
| kubernetes-worker/0 active idle 1 10.0.1.146 80/tcp,443/tcp          |
| Kubernetes worker running.                                           |
|                                                                      |
| calico/0 active idle 10.0.1.146 Calico is active                     |
|                                                                      |
| containerd/0 active idle 10.0.1.146 Container runtime available      |
|                                                                      |
| kubernetes-worker/1\* active idle 2 10.0.1.157 80/tcp,443/tcp        |
| Kubernetes worker running.                                           |
|                                                                      |
| calico/2 active idle 10.0.1.157 Calico is active                     |
|                                                                      |
| containerd/2 active idle 10.0.1.157 Container runtime available      |
|                                                                      |
| Machine State DNS Inst id Series AZ Message                          |
|                                                                      |
| 0 started 10.0.1.125 juju-b9466a-0 bionic poweredOn                  |
|                                                                      |
| 1 started 10.0.1.146 juju-b9466a-1 bionic poweredOn                  |
|                                                                      |
| 2 started 10.0.1.157 juju-b9466a-2 bionic poweredOn                  |
+----------------------------------------------------------------------+

14. 

+------+
| 15.  |
+------+

16. 
17. 

+------+
| 18.  |
+------+

19. 

20. 

21. Confirm GPU node

+----------------------------------------------------------------------+
| juju ssh 2 sudo lsmod\|grep nvidia                                   |
+======================================================================+
| kubectl describe node \<gpu-node\> \|grep -B 6 gpu                   |
+----------------------------------------------------------------------+
| Outputs similar here:                                                |
|                                                                      |
| \--                                                                  |
|                                                                      |
| > Capacity:                                                          |
| >                                                                    |
| > cpu: 2                                                             |
| >                                                                    |
| > ephemeral-storage: 64860696Ki                                      |
| >                                                                    |
| > hugepages-1Gi: 0                                                   |
| >                                                                    |
| > hugepages-2Mi: 0                                                   |
| >                                                                    |
| > memory: 16426004Ki                                                 |
| >                                                                    |
| > nvidia.com/gpu: 1                                                  |
| >                                                                    |
| > \--                                                                |
| >                                                                    |
| > Allocatable:                                                       |
| >                                                                    |
| > cpu: 2                                                             |
| >                                                                    |
| > ephemeral-storage: 59775617335                                     |
| >                                                                    |
| > hugepages-1Gi: 0                                                   |
| >                                                                    |
| > hugepages-2Mi: 0                                                   |
| >                                                                    |
| > memory: 16323604Ki                                                 |
| >                                                                    |
| > nvidia.com/gpu: 1                                                  |
| >                                                                    |
| > \--                                                                |
| >                                                                    |
| > (Total limits may be over 100 percent, i.e., overcommitted.)       |
| >                                                                    |
| > Resource Requests Limits                                           |
| >                                                                    |
| > \-\-\-\-\-\-\-- \-\-\-\-\-\-\-- \-\-\-\-\--                        |
| >                                                                    |
| > cpu 0 (0%) 0 (0%)                                                  |
| >                                                                    |
| > memory 0 (0%) 0 (0%)                                               |
|                                                                      |
| ephemeral-storage 0 (0%) 0 (0%)                                      |
|                                                                      |
| nvidia.com/gpu 0 0                                                   |
+----------------------------------------------------------------------+
| cat \<\<EOF \| kubectl apply -f --                                   |
|                                                                      |
| kind: Job                                                            |
|                                                                      |
| metadata:                                                            |
|                                                                      |
| name: nvidia-smi                                                     |
|                                                                      |
| labels:                                                              |
|                                                                      |
| name: nvidia-smi                                                     |
|                                                                      |
| spec:                                                                |
|                                                                      |
| template:                                                            |
|                                                                      |
| metadata:                                                            |
|                                                                      |
| labels:                                                              |
|                                                                      |
| name: nvidia-smi                                                     |
|                                                                      |
| spec:                                                                |
|                                                                      |
| containers:                                                          |
|                                                                      |
| \- name: nvidia-smi                                                  |
|                                                                      |
| image: nvidia/cuda                                                   |
|                                                                      |
| command: \[ \"nvidia-smi\" \]                                        |
|                                                                      |
| imagePullPolicy: IfNotPresent                                        |
|                                                                      |
| resources:                                                           |
|                                                                      |
| requests:                                                            |
|                                                                      |
| \# alpha.kubernetes.io/nvidia-gpu: 1                                 |
|                                                                      |
| nvidia.com/gpu: 1                                                    |
|                                                                      |
| limits:                                                              |
|                                                                      |
| \#alpha.kubernetes.io/nvidia-gpu: 1                                  |
|                                                                      |
| nvidia.com/gpu: 1                                                    |
|                                                                      |
| volumeMounts:                                                        |
|                                                                      |
| \- mountPath: /usr/local/nvidia/bin                                  |
|                                                                      |
| name: bin                                                            |
|                                                                      |
| \- mountPath: /usr/lib/nvidia                                        |
|                                                                      |
| name: lib                                                            |
|                                                                      |
| volumes:                                                             |
|                                                                      |
| \- name: bin                                                         |
|                                                                      |
| hostPath:                                                            |
|                                                                      |
| path: /usr/lib/nvidia-375/bin                                        |
|                                                                      |
| \- name: lib                                                         |
|                                                                      |
| hostPath:                                                            |
|                                                                      |
| path: /usr/lib/nvidia-375                                            |
|                                                                      |
| restartPolicy: Never                                                 |
+----------------------------------------------------------------------+
| hitp\@ubu-gw-ext:\~\$ kubectl get pods                               |
|                                                                      |
| NAME READY STATUS RESTARTS AGE                                       |
|                                                                      |
| nvidia-smi-h2wvh 0/1 Completed 0 5s                                  |
|                                                                      |
| hitp\@ubu-gw-ext:\~\$ kubectl logs nvidia-smi-h2wvh                  |
|                                                                      |
| Tue Apr 21 07:17:34 2020                                             |
|                                                                      |
| +\-\-\-\-\-\-\-\-\-                                                  |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\- |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ |
|                                                                      |
| \| NVIDIA-SMI 440.64.00 Driver Version: 440.64.00 CUDA Version: 10.2 |
| \|                                                                   |
|                                                                      |
| \|\-\-\-\-\-\-\-                                                     |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\- |
| \-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ |
|                                                                      |
| \| GPU Name Persistence-M\| Bus-Id Disp.A \| Volatile Uncorr. ECC \| |
|                                                                      |
| \| Fan Temp Perf Pwr:Usage/Cap\| Memory-Usage \| GPU-Util Compute M. |
| \|                                                                   |
|                                                                      |
| \|===========                                                        |
| ====================+======================+======================\| |
|                                                                      |
| \| 0 Tesla P4 On \| 00000000:13:00.0 Off \| 0 \|                     |
|                                                                      |
| \| N/A 29C P8 6W / 75W \| 0MiB / 7611MiB \| 0% Default \|            |
|                                                                      |
| +\-\-\-\-\-\-\-                                                      |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\- |
| \-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ |
|                                                                      |
| +\-\-\-\-\-\-\-\-\-                                                  |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\- |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ |
|                                                                      |
| \| Processes: GPU Memory \|                                          |
|                                                                      |
| \| GPU PID Type Process name Usage \|                                |
|                                                                      |
| \|===========                                                        |
| ==================================================================\| |
|                                                                      |
| \| No running processes found \|                                     |
|                                                                      |
| +\-\-\-\-\-\-\-\-\-                                                  |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\- |
| \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ |
+----------------------------------------------------------------------+

22. Add StrageClass

+-------------------------------------------+
| kubectl create -f - \<\<EOY               |
|                                           |
| apiVersion: storage.k8s.io/v1             |
|                                           |
| kind: StorageClass                        |
|                                           |
| metadata:                                 |
|                                           |
| name: vsphere-storage                     |
|                                           |
| provisioner: kubernetes.io/vsphere-volume |
|                                           |
| parameters:                               |
|                                           |
| diskformat: thin                          |
|                                           |
| EOY                                       |
+-------------------------------------------+

+--------------------------------------------------+
| hitp\@ubu-gw:\~\$ kubectl get sc                 |
|                                                  |
| NAME PROVISIONER AGE                             |
|                                                  |
| vsphere-storage kubernetes.io/vsphere-volume 49s |
+--------------------------------------------------+

  --
  --

23. Deploy Kubeflow

Follow this instruction

https://www.kubeflow.org/docs/started/k8s/kfctl-k8s-istio/

\* Under corporate Proxy configuration

+----------------------------------------------------------------------+
| \#set KUBECONFIG                                                     |
|                                                                      |
| export KUBECONFIG=\<path-to-kubeconfig-file-for-juju-k8s\>           |
|                                                                      |
| mkdir \~/kubeflow-download                                           |
|                                                                      |
| cd \~/kubeflow-download                                              |
|                                                                      |
| tar xvf kfctl\_v1.0.1-0-gf3edb9b\_linux.tar.gz                       |
|                                                                      |
| cd \~                                                                |
|                                                                      |
| export PATH=\$PATH:/home/hitp/kubeflow-download                      |
|                                                                      |
| export KF\_NAME=kubeflow                                             |
|                                                                      |
| export                                                               |
| CONFIG\_URI=\"https://raw.githubusercontent.com/                     |
| kubeflow/manifests/v1.0-branch/kfdef/kfctl\_k8s\_istio.v1.0.2.yaml\" |
|                                                                      |
| \#\#download kfdef(kfctl\_k8s\_istio.v1.0.2.yaml) / manifest         |
|                                                                      |
| cd \~/kubeflow-download                                              |
|                                                                      |
| wget \${CONFIG\_URI}                                                 |
|                                                                      |
| wget                                                                 |
| https://github.com/kubeflow/manifests/archive/v1.0-branch.tar.gz     |
|                                                                      |
| \#\# modify dfdef                                                    |
|                                                                      |
| \`\`\`                                                               |
|                                                                      |
| repos:                                                               |
|                                                                      |
| \- name: manifests                                                   |
|                                                                      |
| uri: file:/home/hitp/kubeflow-download/v1.0-branch.tar.gz            |
|                                                                      |
| version: v1.0.2                                                      |
|                                                                      |
| \`\`\`                                                               |
|                                                                      |
| \#create kustomize files                                             |
|                                                                      |
| kfctl build -V -f kfctl\_k8s\_istio.v1.0.2.yaml                      |
|                                                                      |
| kfctl apply -V -f kfctl\_k8s\_istio.v1.0.2.yaml                      |
+======================================================================+
| \#Create Persitent Volume for these pvc                              |
|                                                                      |
| hitp\@ubu-gw:\~\$ kubectl get pvc -A                                 |
|                                                                      |
| NAMESPACE NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE  |
|                                                                      |
| kubeflow katib-mysql Pending 7m33s(10G                               |
|                                                                      |
| kubeflow metadata-mysql Pending 7m46s(10G                            |
|                                                                      |
| kubeflow minio-pv-claim Pending 7m33s(20G                            |
|                                                                      |
| kubeflow mysql-pv-claim Pending 7m32s(20G                            |
+----------------------------------------------------------------------+

\*If Pod 'Permission Denied' error occurs in the Pod log, set below
setting in the deployment manifest

+------------------+
| securityContext: |
|                  |
| runAsUser: 999   |
|                  |
| fsGroup: 999     |
+------------------+

\#\# Tips

When you recreate user for central dashboard after initial setup, delete
user you create at the welcome procedure.

\#\# Bug fix

\#\# Manual controller case

  VM Name               hostname   IP           Description
  --------------------- ---------- ------------ ---------------------------
  kubeflow-ubu-gw01     ubu-gw01   10.0.1.210   Bastion / Juju Controller
  kubeflow-ubu-k8s-m1   ubu-m1     10.0.1.211   
  kubeflow-ubu-k8s-w1   ubu-w1     10.0.1.212   \+ GPU x 1shu

1.  Deploy Ubuntu machines on vSphere

    a.  Add GPU for

2.  Install juju on ubu-gw01

3.  Create manual controller on ubu-gw01

+----------------------------------------------------------------------+
| juju bootstrap manual/\<ubu-gw01-ip\> juju01                         |
+======================================================================+
| hitp\@ubu-gw01:\~\$ juju models                                      |
|                                                                      |
| Controller: juju01                                                   |
|                                                                      |
| juju                                                                 |
|                                                                      |
| Model Cloud/Region Type Status Machines Cores Access Last connection |
|                                                                      |
| controller\* manual manual available 1 2 admin just now              |
|                                                                      |
| default manual manual available 0 - admin 2 hours ago                |
+----------------------------------------------------------------------+

> ※Controller: juju01 になっておらずdefaultになっている場合は juju
> switch controllerでjuju01に切り替える

4.  Add machines to juju01

+----------------------------------------------------------------------+
| juju add-machine ssh:10.0.1.211                                      |
|                                                                      |
| juju add-machine ssh:10.0.1.212                                      |
+======================================================================+
| hitp\@ubu-gw01:\~\$ juju status                                      |
|                                                                      |
| Model Controller Cloud/Region Version SLA Timestamp                  |
|                                                                      |
| controller juju01 manual 2.7.5 unsupported 12:36:58Z                 |
|                                                                      |
| Machine State DNS Inst id Series AZ Message                          |
|                                                                      |
| 0 started 10.0.1.210 manual: bionic Manually provisioned machine     |
|                                                                      |
| 1 started 10.0.1.211 manual:10.0.1.211 bionic Manually provisioned   |
| machine                                                              |
|                                                                      |
| 2 started 10.0.1.212 manual:10.0.1.212 bionic Manually provisioned   |
| machine                                                              |
+----------------------------------------------------------------------+

5.  Create bundle YAML

6.  Deploy bundle

+----------------------------------------------------------------------+
| ~~juju deploy ./bundle-k8s-calico-jcb.yaml \--map-machines           |
| existing,0=0,1=1~~                                                   |
|                                                                      |
| juju deploy ./bundle-k8s-1.15.11-calico.yaml \--map-machines         |
| existing,0=1,1=2                                                     |
+----------------------------------------------------------------------+

\#\# Remove manual machine

+--------------------------------------------------+
| juju remove-machine \<machine-id\>               |
|                                                  |
| ssh \<machine-ip\>                               |
|                                                  |
| sudo rm -rf /var/lib/juju                        |
|                                                  |
| sudo rm -rf /lib/systemd/system/juju\*           |
|                                                  |
| sudo rm -rf /run/systemd/units/invocation:juju\* |
|                                                  |
| sudo rm -rf /etc/systemd/system/juju\*           |
+--------------------------------------------------+

\#\# vSphere integrator

+----------------------------------------------------------------------+
| hitp\@ubu-gw-ext:\~\$ juju status                                    |
|                                                                      |
| Model Controller Cloud/Region Version SLA Timestamp                  |
|                                                                      |
| default vsp-controller-03 vsp-cloud/Hewlett-Packard 2.7.5            |
| unsupported 03:29:25Z                                                |
|                                                                      |
| App Version Status Scale Charm Store Rev OS Notes                    |
|                                                                      |
| calico active 3 calico jujucharms 698 ubuntu                         |
|                                                                      |
| containerd active 3 containerd jujucharms 46 ubuntu                  |
|                                                                      |
| easyrsa 3.0.1 active 1 easyrsa jujucharms 289 ubuntu                 |
|                                                                      |
| etcd 3.2.10 active 1 etcd jujucharms 478 ubuntu                      |
|                                                                      |
| kubeapi-load-balancer 1.14.0 active 1 kubeapi-load-balancer          |
| jujucharms 695 ubuntu exposed                                        |
|                                                                      |
| kubernetes-master 1.15.11 waiting 1 kubernetes-master jujucharms 746 |
| ubuntu                                                               |
|                                                                      |
| kubernetes-worker 1.15.11 waiting 2 kubernetes-worker jujucharms 588 |
| ubuntu exposed                                                       |
|                                                                      |
| vsphere-integrator blocked 1 vsphere-integrator jujucharms 19 ubuntu |
|                                                                      |
| Unit Workload Agent Machine Public address Ports Message             |
|                                                                      |
| easyrsa/0\* active idle 0 10.0.1.144 Certificate Authority           |
| connected.                                                           |
|                                                                      |
| etcd/0\* active idle 0 10.0.1.144 2379/tcp Healthy with 1 known peer |
|                                                                      |
| kubeapi-load-balancer/0\* active idle 0 10.0.1.144 443/tcp           |
| Loadbalancer ready.                                                  |
|                                                                      |
| kubernetes-master/0\* waiting idle 0 10.0.1.144 6443/tcp Waiting for |
| cloud integration                                                    |
|                                                                      |
| calico/1 active idle 10.0.1.144 Calico is active                     |
|                                                                      |
| containerd/1 active idle 10.0.1.144 Container runtime available      |
|                                                                      |
| kubernetes-worker/0 waiting idle 1 10.0.1.143 80/tcp,443/tcp Waiting |
| for cloud integration                                                |
|                                                                      |
| calico/0 active idle 10.0.1.143 Calico is active                     |
|                                                                      |
| containerd/0 active idle 10.0.1.143 Container runtime available      |
|                                                                      |
| kubernetes-worker/1\* waiting idle 2 10.0.1.129 80/tcp,443/tcp       |
| Waiting for cloud integration                                        |
|                                                                      |
| calico/2\* active idle 10.0.1.129 Calico is active                   |
|                                                                      |
| containerd/2\* active idle 10.0.1.129 Container runtime available    |
|                                                                      |
| vsphere-integrator/0\* blocked idle 3 10.0.1.131 missing credentials |
| access; grant with: juju trust                                       |
|                                                                      |
| Machine State DNS Inst id Series AZ Message                          |
|                                                                      |
| 0 started 10.0.1.144 juju-cfeb1b-0 bionic poweredOn                  |
|                                                                      |
| 1 started 10.0.1.143 juju-cfeb1b-1 bionic poweredOn                  |
|                                                                      |
| 2 started 10.0.1.129 juju-cfeb1b-2 bionic poweredOn                  |
|                                                                      |
| 3 started 10.0.1.131 juju-cfeb1b-3 bionic poweredOn                  |
|                                                                      |
| hitp\@ubu-gw-ext:\~\$                                                |
|                                                                      |
| hitp\@ubu-gw-ext:\~\$ juju trust vsphere-integrator                  |
|                                                                      |
| hitp\@ubu-gw-ext:\~\$ juju status                                    |
|                                                                      |
| Model Controller Cloud/Region Version SLA Timestamp                  |
|                                                                      |
| default vsp-controller-03 vsp-cloud/Hewlett-Packard 2.7.5            |
| unsupported 03:29:53Z                                                |
|                                                                      |
| App Version Status Scale Charm Store Rev OS Notes                    |
|                                                                      |
| calico active 3 calico jujucharms 698 ubuntu                         |
|                                                                      |
| containerd active 3 containerd jujucharms 46 ubuntu                  |
|                                                                      |
| easyrsa 3.0.1 active 1 easyrsa jujucharms 289 ubuntu                 |
|                                                                      |
| etcd 3.2.10 active 1 etcd jujucharms 478 ubuntu                      |
|                                                                      |
| kubeapi-load-balancer 1.14.0 active 1 kubeapi-load-balancer          |
| jujucharms 695 ubuntu exposed                                        |
|                                                                      |
| kubernetes-master 1.15.11 waiting 1 kubernetes-master jujucharms 746 |
| ubuntu                                                               |
|                                                                      |
| kubernetes-worker 1.15.11 waiting 2 kubernetes-worker jujucharms 588 |
| ubuntu exposed                                                       |
|                                                                      |
| vsphere-integrator blocked 1 vsphere-integrator jujucharms 19 ubuntu |
|                                                                      |
| Unit Workload Agent Machine Public address Ports Message             |
|                                                                      |
| easyrsa/0\* active idle 0 10.0.1.144 Certificate Authority           |
| connected.                                                           |
|                                                                      |
| etcd/0\* active idle 0 10.0.1.144 2379/tcp Healthy with 1 known peer |
|                                                                      |
| kubeapi-load-balancer/0\* active idle 0 10.0.1.144 443/tcp           |
| Loadbalancer ready.                                                  |
|                                                                      |
| kubernetes-master/0\* waiting idle 0 10.0.1.144 6443/tcp Waiting for |
| cloud integration                                                    |
|                                                                      |
| calico/1 active idle 10.0.1.144 Calico is active                     |
|                                                                      |
| containerd/1 active idle 10.0.1.144 Container runtime available      |
|                                                                      |
| kubernetes-worker/0 waiting idle 1 10.0.1.143 80/tcp,443/tcp Waiting |
| for cloud integration                                                |
|                                                                      |
| calico/0 active idle 10.0.1.143 Calico is active                     |
|                                                                      |
| containerd/0 active idle 10.0.1.143 Container runtime available      |
|                                                                      |
| kubernetes-worker/1\* waiting idle 2 10.0.1.129 80/tcp,443/tcp       |
| Waiting for cloud integration                                        |
|                                                                      |
| calico/2\* active idle 10.0.1.129 Calico is active                   |
|                                                                      |
| containerd/2\* active idle 10.0.1.129 Container runtime available    |
|                                                                      |
| vsphere-integrator/0\* blocked executing 3 10.0.1.131                |
| (config-changed) missing credentials access; grant with: juju trust  |
|                                                                      |
| Machine State DNS Inst id Series AZ Message                          |
|                                                                      |
| 0 started 10.0.1.144 juju-cfeb1b-0 bionic poweredOn                  |
|                                                                      |
| 1 started 10.0.1.143 juju-cfeb1b-1 bionic poweredOn                  |
|                                                                      |
| 2 started 10.0.1.129 juju-cfeb1b-2 bionic poweredOn                  |
|                                                                      |
| 3 started 10.0.1.131 juju-cfeb1b-3 bionic poweredOn                  |
|                                                                      |
| hitp\@ubu-gw-ext:\~\$ juju status                                    |
|                                                                      |
| Model Controller Cloud/Region Version SLA Timestamp                  |
|                                                                      |
| default vsp-controller-03 vsp-cloud/Hewlett-Packard 2.7.5            |
| unsupported 03:30:04Z                                                |
|                                                                      |
| App Version Status Scale Charm Store Rev OS Notes                    |
|                                                                      |
| calico active 3 calico jujucharms 698 ubuntu                         |
|                                                                      |
| containerd active 3 containerd jujucharms 46 ubuntu                  |
|                                                                      |
| easyrsa 3.0.1 active 1 easyrsa jujucharms 289 ubuntu                 |
|                                                                      |
| etcd 3.2.10 active 1 etcd jujucharms 478 ubuntu                      |
|                                                                      |
| kubeapi-load-balancer 1.14.0 active 1 kubeapi-load-balancer          |
| jujucharms 695 ubuntu exposed                                        |
|                                                                      |
| kubernetes-master 1.15.11 waiting 1 kubernetes-master jujucharms 746 |
| ubuntu                                                               |
|                                                                      |
| kubernetes-worker 1.15.11 waiting 2 kubernetes-worker jujucharms 588 |
| ubuntu exposed                                                       |
|                                                                      |
| vsphere-integrator active 1 vsphere-integrator jujucharms 19 ubuntu  |
|                                                                      |
| Unit Workload Agent Machine Public address Ports Message             |
|                                                                      |
| easyrsa/0\* active idle 0 10.0.1.144 Certificate Authority           |
| connected.                                                           |
|                                                                      |
| etcd/0\* active idle 0 10.0.1.144 2379/tcp Healthy with 1 known peer |
|                                                                      |
| kubeapi-load-balancer/0\* active idle 0 10.0.1.144 443/tcp           |
| Loadbalancer ready.                                                  |
|                                                                      |
| kubernetes-master/0\* waiting executing 0 10.0.1.144 6443/tcp        |
| Waiting for cloud integration                                        |
|                                                                      |
| calico/1 active idle 10.0.1.144 Calico is active                     |
|                                                                      |
| containerd/1 active idle 10.0.1.144 Container runtime available      |
|                                                                      |
| kubernetes-worker/0 waiting executing 1 10.0.1.143 80/tcp,443/tcp    |
| Waiting for cloud integration                                        |
|                                                                      |
| calico/0 active idle 10.0.1.143 Calico is active                     |
|                                                                      |
| containerd/0 active idle 10.0.1.143 Container runtime available      |
|                                                                      |
| kubernetes-worker/1\* waiting executing 2 10.0.1.129 80/tcp,443/tcp  |
| Waiting for cloud integration                                        |
|                                                                      |
| calico/2\* active idle 10.0.1.129 Calico is active                   |
|                                                                      |
| containerd/2\* active idle 10.0.1.129 Container runtime available    |
|                                                                      |
| vsphere-integrator/0\* active idle 3 10.0.1.131 ready                |
|                                                                      |
| Machine State DNS Inst id Series AZ Message                          |
|                                                                      |
| 0 started 10.0.1.144 juju-cfeb1b-0 bionic poweredOn                  |
|                                                                      |
| 1 started 10.0.1.143 juju-cfeb1b-1 bionic poweredOn                  |
|                                                                      |
| 2 started 10.0.1.129 juju-cfeb1b-2 bionic poweredOn                  |
|                                                                      |
| 3 started 10.0.1.131 juju-cfeb1b-3 bionic poweredOn                  |
+----------------------------------------------------------------------+

\#\# Destroy Model/Controller

Remove model -\> Controller

+--------------------------------------------------------------------------+
| hitp\@ubu-gw-ext:\~\$ juju destroy-model default                         |
|                                                                          |
| WARNING! This command will destroy the \"default\" model.                |
|                                                                          |
| This includes all machines, applications, data and other resources.      |
|                                                                          |
| Continue \[y/N\]? y                                                      |
|                                                                          |
| Destroying model                                                         |
|                                                                          |
| Waiting for model to be removed, 2 machine(s), 7 application(s)\...\...  |
|                                                                          |
| Waiting for model to be removed, 2 machine(s), 5 application(s)\...      |
|                                                                          |
| Waiting for model to be removed, 2 machine(s)\....                       |
|                                                                          |
| Waiting for model to be removed, 1 machine(s)\.....                      |
|                                                                          |
| Model destroyed.                                                         |
|                                                                          |
| hitp\@ubu-gw-ext:\~\$ juju destroy-controller vsp-controller-01          |
|                                                                          |
| WARNING! This command will destroy the \"vsp-controller-01\" controller. |
|                                                                          |
| This includes all machines, applications, data and other resources.      |
|                                                                          |
| Continue? (y/N):y                                                        |
|                                                                          |
| Destroying controller                                                    |
|                                                                          |
| Waiting for hosted model resources to be reclaimed                       |
|                                                                          |
| All hosted models reclaimed, cleaning up controller machines             |
+--------------------------------------------------------------------------+

\#\# Ubuntu Proxy setting

+------------------------------+
| /etc/apt/apt.conf.d/         |
|                              |
| \+ 90curtin-aptproxy         |
|                              |
| \+ 95-juju-proxy-settings    |
+==============================+
| /etc/environmentcd           |
+------------------------------+
| /etc/juju-proxy.conf         |
+------------------------------+
| /etc/juju-proxy-systemd.conf |
+------------------------------+

\#\#\# 疎通確認

+----------------------------------------------------------------------+
| hitp\@ubu-gw01:\~\$ curl https://vcenter.vsphere.local/sdk \--head   |
| -v                                                                   |
|                                                                      |
| \* Trying 15.210.188.92\...                                          |
|                                                                      |
| \* TCP\_NODELAY set                                                  |
|                                                                      |
| \* Connected to vcenter.vsphere.local (15.210.188.92) port 443 (\#0) |
|                                                                      |
| \* ALPN, offering h2                                                 |
|                                                                      |
| \* ALPN, offering http/1.1                                           |
|                                                                      |
| \* successfully set certificate verify locations:                    |
|                                                                      |
| \* CAfile: /etc/ssl/certs/ca-certificates.crt                        |
|                                                                      |
| CApath: /etc/ssl/certs                                               |
|                                                                      |
| \* TLSv1.3 (OUT), TLS handshake, Client hello (1):                   |
|                                                                      |
| \* OpenSSL SSL\_connect: SSL\_ERROR\_SYSCALL in connection to        |
| vcenter.vsphere.local:443                                            |
|                                                                      |
| \* stopped the pause stream!                                         |
|                                                                      |
| \* Closing connection 0                                              |
|                                                                      |
| curl: (35) OpenSSL SSL\_connect: SSL\_ERROR\_SYSCALL in connection   |
| to vcenter.vsphere.local:443                                         |
+----------------------------------------------------------------------+

\#\# kubeflow deploy logs

+----------------------------------------------------------------------+
| hitp\@ubu-gw:\~/kubeflow-download\$ kfctl apply -V -f                |
| kfctl\_k8s\_istio.v1.0.2.yaml                                        |
|                                                                      |
| INFO\[0000\] No name specified in KfDef.Metadata.Name; defaulting to |
| kubeflow-download based on location of config file:                  |
| kfctl\_k8s\_istio.v1.0.2.yaml.                                       |
| filename=\"coordinator/coordinator.go:202\"                          |
|                                                                      |
| INFO\[0000\]                                                         |
|                                                                      |
| \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*         |
| \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* |
|                                                                      |
| Notice anonymous usage reporting enabled using spartakus             |
|                                                                      |
| To disable it                                                        |
|                                                                      |
| If you have already deployed it run the following commands:          |
|                                                                      |
| cd \$(pwd)                                                           |
|                                                                      |
| kubectl -n \${K8S\_NAMESPACE} delete deploy -l app=spartakus         |
|                                                                      |
| For more info:                                                       |
| https://www.kubeflow.org/docs/other-guides/usage-reporting/          |
|                                                                      |
| \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*         |
| \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* |
|                                                                      |
| filename=\"coordinator/coordinator.go:120\"                          |
|                                                                      |
| INFO\[0000\] Deleting cachedir .cache/manifests because              |
| Status.ReposCache is out of date filename=\"kfconfig/types.go:472\"  |
|                                                                      |
| INFO\[0000\] Fetching                                                |
| file:/home/hitp/kubeflow-download/v1.0-branch.tar.gz to              |
| .cache/manifests filename=\"kfconfig/types.go:493\"                  |
|                                                                      |
| INFO\[0000\] probing file path:                                      |
| /home/hitp/kubeflow-download/v1.0-branch.tar.gz                      |
| filename=\"kfconfig/types.go:543\"                                   |
|                                                                      |
| INFO\[0000\] updating localPath to                                   |
| .cache/manifests/manifests-1.0-branch                                |
| filename=\"kfconfig/types.go:552\"                                   |
|                                                                      |
| INFO\[0000\] Fetch succeeded; LocalPath                              |
| .cache/manifests/manifests-1.0-branch                                |
| filename=\"kfconfig/types.go:561\"                                   |
|                                                                      |
| INFO\[0000\] folder kustomize exists, skip kustomize.Generate        |
| filename=\"kustomize/kustomize.go:372\"                              |
|                                                                      |
| INFO\[0000\] .cache/manifests exists; not resyncing                  |
| filename=\"kfconfig/types.go:468\"                                   |
|                                                                      |
| INFO\[0000\] namespace: kubeflow filename=\"utils/k8utils.go:427\"   |
|                                                                      |
| INFO\[0000\] Creating namespace: kubeflow                            |
| filename=\"utils/k8utils.go:432\"                                    |
|                                                                      |
| INFO\[0000\] log cluster name into KfDef: juju-cluster               |
| filename=\"kustomize/kustomize.go:165\"                              |
|                                                                      |
| INFO\[0000\] Deploying application istio-crds                        |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/adapters.config.istio.io |
| created                                                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/apikeys.config.istio.io |
| created                                                              |
|                                                                      |
| customresour                                                         |
| cedefinition.apiextensions.k8s.io/attributemanifests.config.istio.io |
| created                                                              |
|                                                                      |
| customre                                                             |
| sourcedefinition.apiextensions.k8s.io/authorizations.config.istio.io |
| created                                                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/bypasses.config.istio.io |
| created                                                              |
|                                                                      |
| customres                                                            |
| ourcedefinition.apiextensions.k8s.io/certificates.certmanager.k8s.io |
| created                                                              |
|                                                                      |
| customr                                                              |
| esourcedefinition.apiextensions.k8s.io/challenges.certmanager.k8s.io |
| created                                                              |
|                                                                      |
| customr                                                              |
| esourcedefinition.apiextensions.k8s.io/checknothings.config.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/circonuses.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/cloudwatches.config.istio.io |
| created                                                              |
|                                                                      |
| customresou                                                          |
| rcedefinition.apiextensions.k8s.io/clusterissuers.certmanager.k8s.io |
| created                                                              |
|                                                                      |
| customreso                                                           |
| urcedefinition.apiextensions.k8s.io/clusterrbacconfigs.rbac.istio.io |
| created                                                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/deniers.config.istio.io |
| created                                                              |
|                                                                      |
| customresource                                                       |
| definition.apiextensions.k8s.io/destinationrules.networking.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/dogstatsds.config.istio.io |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/edges.config.istio.io  |
| created                                                              |
|                                                                      |
| customreso                                                           |
| urcedefinition.apiextensions.k8s.io/envoyfilters.networking.istio.io |
| created                                                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/fluentds.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/gateways.networking.istio.io |
| created                                                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/handlers.config.istio.io |
| created                                                              |
|                                                                      |
| customresourc                                                        |
| edefinition.apiextensions.k8s.io/httpapispecbindings.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/httpapispecs.config.istio.io |
| created                                                              |
|                                                                      |
| cus                                                                  |
| tomresourcedefinition.apiextensions.k8s.io/instances.config.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/issuers.certmanager.k8s.io |
| created                                                              |
|                                                                      |
| customre                                                             |
| sourcedefinition.apiextensions.k8s.io/kubernetesenvs.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/kuberneteses.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/listcheckers.config.istio.io |
| created                                                              |
|                                                                      |
| custo                                                                |
| mresourcedefinition.apiextensions.k8s.io/listentries.config.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/logentries.config.istio.io |
| created                                                              |
|                                                                      |
| cus                                                                  |
| tomresourcedefinition.apiextensions.k8s.io/memquotas.config.istio.io |
| created                                                              |
|                                                                      |
| customresource                                                       |
| definition.apiextensions.k8s.io/meshpolicies.authentication.istio.io |
| created                                                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/metrics.config.istio.io |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/noops.config.istio.io  |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/opas.config.istio.io   |
| created                                                              |
|                                                                      |
| cus                                                                  |
| tomresourcedefinition.apiextensions.k8s.io/orders.certmanager.k8s.io |
| created                                                              |
|                                                                      |
| customreso                                                           |
| urcedefinition.apiextensions.k8s.io/policies.authentication.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/prometheuses.config.istio.io |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/quotas.config.istio.io |
| created                                                              |
|                                                                      |
| customresou                                                          |
| rcedefinition.apiextensions.k8s.io/quotaspecbindings.config.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/quotaspecs.config.istio.io |
| created                                                              |
|                                                                      |
| cus                                                                  |
| tomresourcedefinition.apiextensions.k8s.io/rbacconfigs.rbac.istio.io |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/rbacs.config.istio.io  |
| created                                                              |
|                                                                      |
| custo                                                                |
| mresourcedefinition.apiextensions.k8s.io/redisquotas.config.istio.io |
| created                                                              |
|                                                                      |
| customre                                                             |
| sourcedefinition.apiextensions.k8s.io/reportnothings.config.istio.io |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/rules.config.istio.io  |
| created                                                              |
|                                                                      |
| customresour                                                         |
| cedefinition.apiextensions.k8s.io/serviceentries.networking.istio.io |
| created                                                              |
|                                                                      |
| customresou                                                          |
| rcedefinition.apiextensions.k8s.io/servicerolebindings.rbac.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/serviceroles.rbac.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/sidecars.networking.istio.io |
| created                                                              |
|                                                                      |
| cus                                                                  |
| tomresourcedefinition.apiextensions.k8s.io/signalfxs.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/solarwindses.config.istio.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/stackdrivers.config.istio.io |
| created                                                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/statsds.config.istio.io |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/stdios.config.istio.io |
| created                                                              |
|                                                                      |
| cus                                                                  |
| tomresourcedefinition.apiextensions.k8s.io/templates.config.istio.io |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/tracespans.config.istio.io |
| created                                                              |
|                                                                      |
| customresourc                                                        |
| edefinition.apiextensions.k8s.io/virtualservices.networking.istio.io |
| created                                                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/zipkins.config.istio.io |
| created                                                              |
|                                                                      |
| INFO\[0003\] Successfully applied application istio-crds             |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0003\] Deploying application istio-install                     |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| namespace/istio-system created                                       |
|                                                                      |
| mutatingwebh                                                         |
| ookconfiguration.admissionregistration.k8s.io/istio-sidecar-injector |
| created                                                              |
|                                                                      |
| serviceaccount/istio-citadel-service-account created                 |
|                                                                      |
| serviceaccount/istio-cleanup-secrets-service-account created         |
|                                                                      |
| serviceaccount/istio-egressgateway-service-account created           |
|                                                                      |
| serviceaccount/istio-galley-service-account created                  |
|                                                                      |
| serviceaccount/istio-grafana-post-install-account created            |
|                                                                      |
| serviceaccount/istio-ingressgateway-service-account created          |
|                                                                      |
| serviceaccount/istio-mixer-service-account created                   |
|                                                                      |
| serviceaccount/istio-multi created                                   |
|                                                                      |
| serviceaccount/istio-pilot-service-account created                   |
|                                                                      |
| serviceaccount/istio-security-post-install-account created           |
|                                                                      |
| serviceaccount/istio-sidecar-injector-service-account created        |
|                                                                      |
| serviceaccount/kiali-service-account created                         |
|                                                                      |
| serviceaccount/prometheus created                                    |
|                                                                      |
| role.rbac.authorization.k8s.io/istio-ingressgateway-sds created      |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/istio-citadel-istio-system     |
| created                                                              |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system |
| created                                                              |
|                                                                      |
| cl                                                                   |
| usterrole.rbac.authorization.k8s.io/istio-egressgateway-istio-system |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/istio-galley-istio-system      |
| created                                                              |
|                                                                      |
| clusterro                                                            |
| le.rbac.authorization.k8s.io/istio-grafana-post-install-istio-system |
| created                                                              |
|                                                                      |
| clu                                                                  |
| sterrole.rbac.authorization.k8s.io/istio-ingressgateway-istio-system |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/istio-mixer-istio-system       |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/istio-pilot-istio-system       |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/istio-reader created           |
|                                                                      |
| clust                                                                |
| errole.rbac.authorization.k8s.io/istio-sidecar-injector-istio-system |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kiali created                  |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kiali-viewer created           |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/prometheus-istio-system        |
| created                                                              |
|                                                                      |
| clusterrol                                                           |
| e.rbac.authorization.k8s.io/istio-security-post-install-istio-system |
| created                                                              |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/istio-ingressgateway-sds       |
| created                                                              |
|                                                                      |
| clu                                                                  |
| sterrolebinding.rbac.authorization.k8s.io/istio-citadel-istio-system |
| created                                                              |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/istio-cleanup-secrets-istio-system |
| created                                                              |
|                                                                      |
| clusterro                                                            |
| lebinding.rbac.authorization.k8s.io/istio-egressgateway-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.rb                                                |
| ac.authorization.k8s.io/istio-galley-admin-role-binding-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.autho                                        |
| rization.k8s.io/istio-grafana-post-install-role-binding-istio-system |
| created                                                              |
|                                                                      |
| clusterrol                                                           |
| ebinding.rbac.authorization.k8s.io/istio-ingressgateway-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.r                                                 |
| bac.authorization.k8s.io/istio-kiali-admin-role-binding-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.r                                                 |
| bac.authorization.k8s.io/istio-mixer-admin-role-binding-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/istio-multi created     |
|                                                                      |
| c                                                                    |
| lusterrolebinding.rbac.authorization.k8s.io/istio-pilot-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.authori                                      |
| zation.k8s.io/istio-sidecar-injector-admin-role-binding-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/prometheus-istio-system |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.author                                       |
| ization.k8s.io/istio-security-post-install-role-binding-istio-system |
| created                                                              |
|                                                                      |
| configmap/istio created                                              |
|                                                                      |
| configmap/istio-galley-configuration created                         |
|                                                                      |
| configmap/istio-grafana created                                      |
|                                                                      |
| configmap/istio-grafana-configuration-dashboards-galley-dashboard    |
| created                                                              |
|                                                                      |
| c                                                                    |
| onfigmap/istio-grafana-configuration-dashboards-istio-mesh-dashboard |
| created                                                              |
|                                                                      |
| configma                                                             |
| p/istio-grafana-configuration-dashboards-istio-performance-dashboard |
| created                                                              |
|                                                                      |
| conf                                                                 |
| igmap/istio-grafana-configuration-dashboards-istio-service-dashboard |
| created                                                              |
|                                                                      |
| confi                                                                |
| gmap/istio-grafana-configuration-dashboards-istio-workload-dashboard |
| created                                                              |
|                                                                      |
| configmap/istio-grafana-configuration-dashboards-mixer-dashboard     |
| created                                                              |
|                                                                      |
| configmap/istio-grafana-configuration-dashboards-pilot-dashboard     |
| created                                                              |
|                                                                      |
| configmap/istio-grafana-custom-resources created                     |
|                                                                      |
| configmap/istio-security-custom-resources created                    |
|                                                                      |
| configmap/istio-sidecar-injector created                             |
|                                                                      |
| configmap/kiali created                                              |
|                                                                      |
| configmap/prometheus created                                         |
|                                                                      |
| secret/kiali created                                                 |
|                                                                      |
| service/grafana created                                              |
|                                                                      |
| service/istio-citadel created                                        |
|                                                                      |
| service/istio-egressgateway created                                  |
|                                                                      |
| service/istio-galley created                                         |
|                                                                      |
| service/istio-ingressgateway created                                 |
|                                                                      |
| service/istio-pilot created                                          |
|                                                                      |
| service/istio-policy created                                         |
|                                                                      |
| service/istio-sidecar-injector created                               |
|                                                                      |
| service/istio-telemetry created                                      |
|                                                                      |
| service/jaeger-agent created                                         |
|                                                                      |
| service/jaeger-collector created                                     |
|                                                                      |
| service/jaeger-query created                                         |
|                                                                      |
| service/kiali created                                                |
|                                                                      |
| service/prometheus created                                           |
|                                                                      |
| service/tracing created                                              |
|                                                                      |
| service/zipkin created                                               |
|                                                                      |
| deployment.apps/grafana created                                      |
|                                                                      |
| deployment.apps/istio-citadel created                                |
|                                                                      |
| deployment.apps/istio-egressgateway created                          |
|                                                                      |
| deployment.apps/istio-galley created                                 |
|                                                                      |
| deployment.apps/istio-ingressgateway created                         |
|                                                                      |
| deployment.apps/istio-pilot created                                  |
|                                                                      |
| deployment.apps/istio-policy created                                 |
|                                                                      |
| deployment.apps/istio-sidecar-injector created                       |
|                                                                      |
| deployment.apps/istio-telemetry created                              |
|                                                                      |
| deployment.apps/istio-tracing created                                |
|                                                                      |
| deployment.apps/kiali created                                        |
|                                                                      |
| deployment.apps/prometheus created                                   |
|                                                                      |
| poddisruptionbudget.policy/istio-egressgateway created               |
|                                                                      |
| poddisruptionbudget.policy/istio-galley created                      |
|                                                                      |
| poddisruptionbudget.policy/istio-ingressgateway created              |
|                                                                      |
| poddisruptionbudget.policy/istio-pilot created                       |
|                                                                      |
| poddisruptionbudget.policy/istio-policy created                      |
|                                                                      |
| poddisruptionbudget.policy/istio-telemetry created                   |
|                                                                      |
| horizontalpodautoscaler.autoscaling/istio-egressgateway created      |
|                                                                      |
| horizontalpodautoscaler.autoscaling/istio-ingressgateway created     |
|                                                                      |
| horizontalpodautoscaler.autoscaling/istio-pilot created              |
|                                                                      |
| horizontalpodautoscaler.autoscaling/istio-policy created             |
|                                                                      |
| horizontalpodautoscaler.autoscaling/istio-telemetry created          |
|                                                                      |
| job.batch/istio-cleanup-secrets-1.1.6 created                        |
|                                                                      |
| job.batch/istio-grafana-post-install-1.1.6 created                   |
|                                                                      |
| job.batch/istio-security-post-install-1.1.6 created                  |
|                                                                      |
| attributemanifest.config.istio.io/istioproxy created                 |
|                                                                      |
| attributemanifest.config.istio.io/kubernetes created                 |
|                                                                      |
| handler.config.istio.io/kubernetesenv created                        |
|                                                                      |
| handler.config.istio.io/prometheus created                           |
|                                                                      |
| handler.config.istio.io/stdio created                                |
|                                                                      |
| kubernetes.config.istio.io/attributes created                        |
|                                                                      |
| logentry.config.istio.io/accesslog created                           |
|                                                                      |
| logentry.config.istio.io/tcpaccesslog created                        |
|                                                                      |
| metric.config.istio.io/requestcount created                          |
|                                                                      |
| metric.config.istio.io/requestduration created                       |
|                                                                      |
| metric.config.istio.io/requestsize created                           |
|                                                                      |
| metric.config.istio.io/responsesize created                          |
|                                                                      |
| metric.config.istio.io/tcpbytereceived created                       |
|                                                                      |
| metric.config.istio.io/tcpbytesent created                           |
|                                                                      |
| metric.config.istio.io/tcpconnectionsclosed created                  |
|                                                                      |
| metric.config.istio.io/tcpconnectionsopened created                  |
|                                                                      |
| rule.config.istio.io/kubeattrgenrulerule created                     |
|                                                                      |
| rule.config.istio.io/promhttp created                                |
|                                                                      |
| rule.config.istio.io/promtcp created                                 |
|                                                                      |
| rule.config.istio.io/promtcpconnectionclosed created                 |
|                                                                      |
| rule.config.istio.io/promtcpconnectionopen created                   |
|                                                                      |
| rule.config.istio.io/stdio created                                   |
|                                                                      |
| rule.config.istio.io/stdiotcp created                                |
|                                                                      |
| rule.config.istio.io/tcpkubeattrgenrulerule created                  |
|                                                                      |
| destinationrule.networking.istio.io/istio-policy created             |
|                                                                      |
| destinationrule.networking.istio.io/istio-telemetry created          |
|                                                                      |
| INFO\[0012\] Successfully applied application istio-install          |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0012\] Deploying application cluster-local-gateway             |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| namespace/istio-system configured                                    |
|                                                                      |
| serviceaccount/cluster-local-gateway-service-account created         |
|                                                                      |
| serviceaccount/istio-multi configured                                |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cluster-local-gateway-istio-system |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/istio-reader configured        |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cluster-local-gateway-istio-system |
| created                                                              |
|                                                                      |
| configmap/cluster-local-gateway-parameters-tbbdb2842d created        |
|                                                                      |
| service/cluster-local-gateway created                                |
|                                                                      |
| deployment.apps/cluster-local-gateway created                        |
|                                                                      |
| poddisruptionbudget.policy/cluster-local-gateway created             |
|                                                                      |
| horizontalpodautoscaler.autoscaling/cluster-local-gateway created    |
|                                                                      |
| INFO\[0012\] Successfully applied application cluster-local-gateway  |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0012\] Deploying application kfserving-gateway                 |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| service/kfserving-ingressgateway created                             |
|                                                                      |
| deployment.apps/kfserving-ingressgateway created                     |
|                                                                      |
| horizontalpodautoscaler.autoscaling/kfserving-ingressgateway created |
|                                                                      |
| INFO\[0012\] Successfully applied application kfserving-gateway      |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0012\] Deploying application istio                             |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-istio-admin created   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-istio-edit created    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-istio-view created    |
|                                                                      |
| configmap/istio-parameters-t6hhgfg9k2 created                        |
|                                                                      |
| gateway.networking.istio.io/kubeflow-gateway created                 |
|                                                                      |
| serviceentry.networking.istio.io/google-api-entry created            |
|                                                                      |
| serviceentry.networking.istio.io/google-storage-api-entry created    |
|                                                                      |
| virtualservice.networking.istio.io/google-api-vs created             |
|                                                                      |
| virtualservice.networking.istio.io/google-storage-api-vs created     |
|                                                                      |
| virtualservice.networking.istio.io/grafana-vs created                |
|                                                                      |
| clusterrbacconfig.rbac.istio.io/default created                      |
|                                                                      |
| INFO\[0012\] Successfully applied application istio                  |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0012\] Deploying application add-anonymous-user-filter         |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| envoyfilter.networking.istio.io/add-user-filter created              |
|                                                                      |
| INFO\[0012\] Successfully applied application                        |
| add-anonymous-user-filter filename=\"kustomize/kustomize.go:209\"    |
|                                                                      |
| INFO\[0012\] Deploying application application-crds                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/applications.app.k8s.io |
| created                                                              |
|                                                                      |
| INFO\[0012\] Successfully applied application application-crds       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0012\] Deploying application application                       |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/application-controller-service-account created        |
|                                                                      |
| clust                                                                |
| errole.rbac.authorization.k8s.io/application-controller-cluster-role |
| created                                                              |
|                                                                      |
| clusterrolebinding.r                                                 |
| bac.authorization.k8s.io/application-controller-cluster-role-binding |
| created                                                              |
|                                                                      |
| configmap/application-controller-parameters created                  |
|                                                                      |
| service/application-controller-service created                       |
|                                                                      |
| statefulset.apps/application-controller-stateful-set created         |
|                                                                      |
| application.app.k8s.io/kubeflow created                              |
|                                                                      |
| INFO\[0014\] Successfully applied application application            |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0014\] Deploying application cert-manager-crds                 |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourc                                                        |
| edefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io |
| created                                                              |
|                                                                      |
| customres                                                            |
| ourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io |
| created                                                              |
|                                                                      |
| customre                                                             |
| sourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io |
| created                                                              |
|                                                                      |
| c                                                                    |
| ustomresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io |
| created                                                              |
|                                                                      |
| custo                                                                |
| mresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io |
| created                                                              |
|                                                                      |
| INFO\[0015\] Successfully applied application cert-manager-crds      |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0015\] Deploying application                                   |
| cert-manager-kube-system-resources                                   |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| r                                                                    |
| ole.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection |
| created                                                              |
|                                                                      |
| role.rbac.authorization.k8s.io/cert-manager:leaderelection created   |
|                                                                      |
| rolebind                                                             |
| ing.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection |
| created                                                              |
|                                                                      |
| rolebinding.rbac.aut                                                 |
| horization.k8s.io/cert-manager-webhook:webhook-authentication-reader |
| created                                                              |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection    |
| created                                                              |
|                                                                      |
| configmap/cert-manager-kube-params-parameters created                |
|                                                                      |
| INFO\[0015\] Successfully applied application                        |
| cert-manager-kube-system-resources                                   |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0015\] Deploying application cert-manager                      |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| namespace/cert-manager created                                       |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| created                                                              |
|                                                                      |
| serviceaccount/cert-manager created                                  |
|                                                                      |
| serviceaccount/cert-manager-cainjector created                       |
|                                                                      |
| serviceaccount/cert-manager-webhook created                          |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit created      |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view created      |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| created                                                              |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| created                                                              |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| created                                                              |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| created                                                              |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| created                                                              |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| created                                                              |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| created                                                              |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| created                                                              |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| created                                                              |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| created                                                              |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| created                                                              |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| created                                                              |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| created                                                              |
|                                                                      |
| configmap/cert-manager-parameters created                            |
|                                                                      |
| service/cert-manager created                                         |
|                                                                      |
| service/cert-manager-webhook created                                 |
|                                                                      |
| deployment.apps/cert-manager created                                 |
|                                                                      |
| deployment.apps/cert-manager-cainjector created                      |
|                                                                      |
| deployment.apps/cert-manager-webhook created                         |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| created                                                              |
|                                                                      |
| application.app.k8s.io/cert-manager created                          |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| created                                                              |
|                                                                      |
| WARN\[0017\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout611930369\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server could not    |
| find the requested resource filename=\"kustomize/kustomize.go:202\"  |
|                                                                      |
| WARN\[0017\] Will retry in 3 seconds.                                |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0020\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout386750206\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0020\] Will retry in 6 seconds.                                |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0027\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout764103359\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0027\] Will retry in 8 seconds.                                |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0035\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout872353812\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0035\] Will retry in 9 seconds.                                |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0045\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout220337805\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0045\] Will retry in 14 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0059\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout171817210\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0059\] Will retry in 27 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0086\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout841722987\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0086\] Will retry in 17 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0104\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout158826288\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0104\] Will retry in 20 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0123\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout335666649\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0123\] Will retry in 18 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0142\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout594448438\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0142\] Will retry in 24 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0166\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout425632855\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0166\] Will retry in 30 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0197\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout735716748\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0197\] Will retry in 39 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0237\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout095700325\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0237\] Will retry in 21 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0258\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout596626098\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0258\] Will retry in 26 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0285\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout096161411\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0285\] Will retry in 25 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0309\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout307999016\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0309\] Will retry in 29 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0339\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout151400753\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0339\] Will retry in 23 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager unchanged                        |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0362\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout842713710\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0362\] Will retry in 24 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0386\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout162548463\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0386\] Will retry in 35 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0422\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout853389828\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0422\] Will retry in 22 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0444\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout957935421\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0444\] Will retry in 21 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0465\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout143992170\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0465\] Will retry in 26 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0491\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout557459867\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0491\] Will retry in 32 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| WARN\[0523\] Encountered error applying application cert-manager:    |
| (kubeflow.error): Code 500 with message: Apply.Run Error error when  |
| creating \"/tmp/kout081752608\": Internal error occurred: failed     |
| calling webhook \"webhook.cert-manager.io\": the server is currently |
| unable to handle the request filename=\"kustomize/kustomize.go:202\" |
|                                                                      |
| WARN\[0523\] Will retry in 41 seconds.                               |
| filename=\"kustomize/kustomize.go:203\"                              |
|                                                                      |
| namespace/cert-manager unchanged                                     |
|                                                                      |
| mutatingwe                                                           |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| serviceaccount/cert-manager unchanged                                |
|                                                                      |
| serviceaccount/cert-manager-cainjector unchanged                     |
|                                                                      |
| serviceaccount/cert-manager-webhook unchanged                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-edit unchanged    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-view unchanged    |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-webhook:webhook-requester |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector        |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| ole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-certificates |
| unchanged                                                            |
|                                                                      |
| clusterrole                                                          |
| binding.rbac.authorization.k8s.io/cert-manager-controller-challenges |
| unchanged                                                            |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers |
| unchanged                                                            |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim |
| unchanged                                                            |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers |
| unchanged                                                            |
|                                                                      |
| cluster                                                              |
| rolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders |
| unchanged                                                            |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/cert-manager-webhook:auth-delegator |
| configured                                                           |
|                                                                      |
| configmap/cert-manager-parameters unchanged                          |
|                                                                      |
| service/cert-manager unchanged                                       |
|                                                                      |
| service/cert-manager-webhook unchanged                               |
|                                                                      |
| deployment.apps/cert-manager unchanged                               |
|                                                                      |
| deployment.apps/cert-manager-cainjector configured                   |
|                                                                      |
| deployment.apps/cert-manager-webhook configured                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.webhook.cert-manager.io    |
| unchanged                                                            |
|                                                                      |
| application.app.k8s.io/cert-manager configured                       |
|                                                                      |
| clusterissuer.cert-manager.io/kubeflow-self-signing-issuer created   |
|                                                                      |
| validatingwe                                                         |
| bhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook |
| configured                                                           |
|                                                                      |
| INFO\[0565\] Successfully applied application cert-manager           |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0565\] Deploying application metacontroller                    |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefini                                                 |
| tion.apiextensions.k8s.io/compositecontrollers.metacontroller.k8s.io |
| created                                                              |
|                                                                      |
| customresourcedefin                                                  |
| ition.apiextensions.k8s.io/controllerrevisions.metacontroller.k8s.io |
| created                                                              |
|                                                                      |
| customresourcedefini                                                 |
| tion.apiextensions.k8s.io/decoratorcontrollers.metacontroller.k8s.io |
| created                                                              |
|                                                                      |
| serviceaccount/meta-controller-service created                       |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/meta-controller-cluster-role-binding |
| created                                                              |
|                                                                      |
| statefulset.apps/metacontroller created                              |
|                                                                      |
| INFO\[0565\] Successfully applied application metacontroller         |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0565\] Deploying application argo                              |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/workflows.argoproj.io  |
| created                                                              |
|                                                                      |
| serviceaccount/argo created                                          |
|                                                                      |
| serviceaccount/argo-ui created                                       |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/argo created                   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/argo-ui created                |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/argo created            |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/argo-ui created         |
|                                                                      |
| configmap/workflow-controller-configmap created                      |
|                                                                      |
| configmap/workflow-controller-parameters created                     |
|                                                                      |
| service/argo-ui created                                              |
|                                                                      |
| deployment.apps/argo-ui created                                      |
|                                                                      |
| deployment.apps/workflow-controller created                          |
|                                                                      |
| application.app.k8s.io/argo created                                  |
|                                                                      |
| virtualservice.networking.istio.io/argo-ui created                   |
|                                                                      |
| INFO\[0565\] Successfully applied application argo                   |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0565\] Deploying application kubeflow-roles                    |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-admin created         |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-edit created          |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-kubernetes-admin      |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-kubernetes-edit       |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-kubernetes-view       |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-view created          |
|                                                                      |
| INFO\[0565\] Successfully applied application kubeflow-roles         |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0565\] Deploying application centraldashboard                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/centraldashboard created                              |
|                                                                      |
| role.rbac.authorization.k8s.io/centraldashboard created              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/centraldashboard created       |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/centraldashboard created       |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/centraldashboard        |
| created                                                              |
|                                                                      |
| configmap/parameters created                                         |
|                                                                      |
| service/centraldashboard created                                     |
|                                                                      |
| deployment.apps/centraldashboard created                             |
|                                                                      |
| application.app.k8s.io/centraldashboard created                      |
|                                                                      |
| virtualservice.networking.istio.io/centraldashboard created          |
|                                                                      |
| INFO\[0566\] Successfully applied application centraldashboard       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0566\] Deploying application bootstrap                         |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/admission-webhook-bootstrap-service-account created   |
|                                                                      |
| clusterrol                                                           |
| e.rbac.authorization.k8s.io/admission-webhook-bootstrap-cluster-role |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.a                                            |
| uthorization.k8s.io/admission-webhook-bootstrap-cluster-role-binding |
| created                                                              |
|                                                                      |
| configmap/admission-webhook-bootstrap-config-map created             |
|                                                                      |
| statefulset.apps/admission-webhook-bootstrap-stateful-set created    |
|                                                                      |
| application.app.k8s.io/bootstrap created                             |
|                                                                      |
| INFO\[0566\] Successfully applied application bootstrap              |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0566\] Deploying application webhook                           |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/poddefaults.kubeflow.org |
| created                                                              |
|                                                                      |
| mutatingwebhookconfiguration.admission                               |
| registration.k8s.io/admission-webhook-mutating-webhook-configuration |
| created                                                              |
|                                                                      |
| serviceaccount/admission-webhook-service-account created             |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/admission-webhook-cluster-role |
| created                                                              |
|                                                                      |
| clusterrole.rb                                                       |
| ac.authorization.k8s.io/admission-webhook-kubeflow-poddefaults-admin |
| created                                                              |
|                                                                      |
| clusterrole.r                                                        |
| bac.authorization.k8s.io/admission-webhook-kubeflow-poddefaults-edit |
| created                                                              |
|                                                                      |
| clusterrole.r                                                        |
| bac.authorization.k8s.io/admission-webhook-kubeflow-poddefaults-view |
| created                                                              |
|                                                                      |
| clusterrolebind                                                      |
| ing.rbac.authorization.k8s.io/admission-webhook-cluster-role-binding |
| created                                                              |
|                                                                      |
| configmap/admission-webhook-admission-webhook-parameters created     |
|                                                                      |
| service/admission-webhook-service created                            |
|                                                                      |
| deployment.apps/admission-webhook-deployment created                 |
|                                                                      |
| application.app.k8s.io/webhook created                               |
|                                                                      |
| INFO\[0567\] Successfully applied application webhook                |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0567\] Deploying application jupyter-web-app                   |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/jupyter-web-app-service-account created               |
|                                                                      |
| role.rbac.authorization.k8s.io/jupyter-web-app-jupyter-notebook-role |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/jupyter-web-app-cluster-role   |
| created                                                              |
|                                                                      |
| clusterrole.                                                         |
| rbac.authorization.k8s.io/jupyter-web-app-kubeflow-notebook-ui-admin |
| created                                                              |
|                                                                      |
| clusterrole                                                          |
| .rbac.authorization.k8s.io/jupyter-web-app-kubeflow-notebook-ui-edit |
| created                                                              |
|                                                                      |
| clusterrole                                                          |
| .rbac.authorization.k8s.io/jupyter-web-app-kubeflow-notebook-ui-view |
| created                                                              |
|                                                                      |
| rolebinding.rba                                                      |
| c.authorization.k8s.io/jupyter-web-app-jupyter-notebook-role-binding |
| created                                                              |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/jupyter-web-app-cluster-role-binding |
| created                                                              |
|                                                                      |
| configmap/jupyter-web-app-config created                             |
|                                                                      |
| configmap/jupyter-web-app-parameters created                         |
|                                                                      |
| service/jupyter-web-app-service created                              |
|                                                                      |
| deployment.apps/jupyter-web-app-deployment created                   |
|                                                                      |
| application.app.k8s.io/jupyter-web-app created                       |
|                                                                      |
| virtualservice.networking.istio.io/jupyter-web-app created           |
|                                                                      |
| INFO\[0568\] Successfully applied application jupyter-web-app        |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0568\] Deploying application spark-operator                    |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinition.                                            |
| apiextensions.k8s.io/scheduledsparkapplications.sparkoperator.k8s.io |
| created                                                              |
|                                                                      |
| customresourcede                                                     |
| finition.apiextensions.k8s.io/sparkapplications.sparkoperator.k8s.io |
| created                                                              |
|                                                                      |
| serviceaccount/spark-operatoroperator-sa created                     |
|                                                                      |
| serviceaccount/spark-operatorspark-sa created                        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/spark-operatoroperator-cr      |
| created                                                              |
|                                                                      |
| clusterr                                                             |
| olebinding.rbac.authorization.k8s.io/spark-operatorsparkoperator-crb |
| created                                                              |
|                                                                      |
| deployment.apps/spark-operatorsparkoperator created                  |
|                                                                      |
| application.app.k8s.io/spark-operator created                        |
|                                                                      |
| job.batch/spark-operatorcrd-cleanup created                          |
|                                                                      |
| INFO\[0569\] Successfully applied application spark-operator         |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0569\] Deploying application metadata                          |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/metadata-ui created                                   |
|                                                                      |
| role.rbac.authorization.k8s.io/metadata-ui created                   |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/metadata-ui created            |
|                                                                      |
| configmap/metadata-db-parameters created                             |
|                                                                      |
| configmap/metadata-grpc-configmap created                            |
|                                                                      |
| configmap/metadata-ui-parameters created                             |
|                                                                      |
| secret/metadata-db-secrets created                                   |
|                                                                      |
| service/metadata-db created                                          |
|                                                                      |
| service/metadata-envoy-service created                               |
|                                                                      |
| service/metadata-grpc-service created                                |
|                                                                      |
| service/metadata-service created                                     |
|                                                                      |
| service/metadata-ui created                                          |
|                                                                      |
| deployment.apps/metadata-db created                                  |
|                                                                      |
| deployment.apps/metadata-deployment created                          |
|                                                                      |
| deployment.apps/metadata-envoy-deployment created                    |
|                                                                      |
| deployment.apps/metadata-grpc-deployment created                     |
|                                                                      |
| deployment.apps/metadata-ui created                                  |
|                                                                      |
| application.app.k8s.io/metadata created                              |
|                                                                      |
| virtualservice.networking.istio.io/metadata-grpc created             |
|                                                                      |
| virtualservice.networking.istio.io/metadata-ui created               |
|                                                                      |
| persistentvolumeclaim/metadata-mysql created                         |
|                                                                      |
| INFO\[0570\] Successfully applied application metadata               |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0570\] Deploying application notebook-controller               |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/notebooks.kubeflow.org |
| created                                                              |
|                                                                      |
| serviceaccount/notebook-controller-service-account created           |
|                                                                      |
| clusterrole.rb                                                       |
| ac.authorization.k8s.io/notebook-controller-kubeflow-notebooks-admin |
| created                                                              |
|                                                                      |
| clusterrole.r                                                        |
| bac.authorization.k8s.io/notebook-controller-kubeflow-notebooks-edit |
| created                                                              |
|                                                                      |
| clusterrole.r                                                        |
| bac.authorization.k8s.io/notebook-controller-kubeflow-notebooks-view |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/notebook-controller-role       |
| created                                                              |
|                                                                      |
| clusterro                                                            |
| lebinding.rbac.authorization.k8s.io/notebook-controller-role-binding |
| created                                                              |
|                                                                      |
| configmap/notebook-controller-parameters created                     |
|                                                                      |
| service/notebook-controller-service created                          |
|                                                                      |
| deployment.apps/notebook-controller-deployment created               |
|                                                                      |
| application.app.k8s.io/notebook-controller created                   |
|                                                                      |
| INFO\[0571\] Successfully applied application notebook-controller    |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0571\] Deploying application pytorch-job-crds                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/pytorchjobs.kubeflow.org |
| created                                                              |
|                                                                      |
| application.app.k8s.io/pytorch-job-crds created                      |
|                                                                      |
| INFO\[0571\] Successfully applied application pytorch-job-crds       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0571\] Deploying application pytorch-operator                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/pytorch-operator created                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-pytorchjobs-admin     |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-pytorchjobs-edit      |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-pytorchjobs-view      |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/pytorch-operator created       |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/pytorch-operator        |
| created                                                              |
|                                                                      |
| service/pytorch-operator created                                     |
|                                                                      |
| deployment.apps/pytorch-operator created                             |
|                                                                      |
| application.app.k8s.io/pytorch-operator created                      |
|                                                                      |
| INFO\[0572\] Successfully applied application pytorch-operator       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0572\] Deploying application knative-crds                      |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| namespace/knative-serving created                                    |
|                                                                      |
| customresourcedefiniti                                               |
| on.apiextensions.k8s.io/certificates.networking.internal.knative.dev |
| created                                                              |
|                                                                      |
| customresour                                                         |
| cedefinition.apiextensions.k8s.io/configurations.serving.knative.dev |
| created                                                              |
|                                                                      |
| customresourc                                                        |
| edefinition.apiextensions.k8s.io/images.caching.internal.knative.dev |
| created                                                              |
|                                                                      |
| customresourcedefin                                                  |
| ition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev |
| created                                                              |
|                                                                      |
| customresourcedefi                                                   |
| nition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev |
| created                                                              |
|                                                                      |
| customresourcedefinition.                                            |
| apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev |
| created                                                              |
|                                                                      |
| customr                                                              |
| esourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev |
| created                                                              |
|                                                                      |
| cust                                                                 |
| omresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev |
| created                                                              |
|                                                                      |
| customresourcedefinition.api                                         |
| extensions.k8s.io/serverlessservices.networking.internal.knative.dev |
| created                                                              |
|                                                                      |
| custom                                                               |
| resourcedefinition.apiextensions.k8s.io/services.serving.knative.dev |
| created                                                              |
|                                                                      |
| application.app.k8s.io/knative-serving-crds created                  |
|                                                                      |
| INFO\[0573\] Successfully applied application knative-crds           |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0573\] Deploying application knative-install                   |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| mutatingwebhookco                                                    |
| nfiguration.admissionregistration.k8s.io/webhook.serving.knative.dev |
| created                                                              |
|                                                                      |
| serviceaccount/controller created                                    |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/custom-metrics-server-resources |
| created                                                              |
|                                                                      |
| cluste                                                               |
| rrole.rbac.authorization.k8s.io/knative-serving-addressable-resolver |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/knative-serving-admin created  |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/knative-serving-core created   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/knative-serving-istio created  |
|                                                                      |
| cl                                                                   |
| usterrole.rbac.authorization.k8s.io/knative-serving-namespaced-admin |
| created                                                              |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-edit |
| created                                                              |
|                                                                      |
| c                                                                    |
| lusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-view |
| created                                                              |
|                                                                      |
| clust                                                                |
| errole.rbac.authorization.k8s.io/knative-serving-podspecable-binding |
| created                                                              |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/custom-metrics-auth-reader     |
| created                                                              |
|                                                                      |
| clusterrolebi                                                        |
| nding.rbac.authorization.k8s.io/custom-metrics:system:auth-delegator |
| created                                                              |
|                                                                      |
| cluste                                                               |
| rrolebinding.rbac.authorization.k8s.io/hpa-controller-custom-metrics |
| created                                                              |
|                                                                      |
| clusterro                                                            |
| lebinding.rbac.authorization.k8s.io/knative-serving-controller-admin |
| created                                                              |
|                                                                      |
| configmap/config-autoscaler created                                  |
|                                                                      |
| configmap/config-defaults created                                    |
|                                                                      |
| configmap/config-deployment created                                  |
|                                                                      |
| configmap/config-domain created                                      |
|                                                                      |
| configmap/config-gc created                                          |
|                                                                      |
| configmap/config-istio created                                       |
|                                                                      |
| configmap/config-logging created                                     |
|                                                                      |
| configmap/config-network created                                     |
|                                                                      |
| configmap/config-observability created                               |
|                                                                      |
| configmap/config-tracing created                                     |
|                                                                      |
| secret/webhook-certs created                                         |
|                                                                      |
| service/activator-service created                                    |
|                                                                      |
| service/autoscaler created                                           |
|                                                                      |
| service/controller created                                           |
|                                                                      |
| service/webhook created                                              |
|                                                                      |
| deployment.apps/activator created                                    |
|                                                                      |
| deployment.apps/autoscaler created                                   |
|                                                                      |
| deployment.apps/autoscaler-hpa created                               |
|                                                                      |
| deployment.apps/controller created                                   |
|                                                                      |
| deployment.apps/networking-istio created                             |
|                                                                      |
| deployment.apps/webhook created                                      |
|                                                                      |
| apiservice.apiregistration.k8s.io/v1beta1.custom.metrics.k8s.io      |
| created                                                              |
|                                                                      |
| application.app.k8s.io/knative-serving-install created               |
|                                                                      |
| horizontalpodautoscaler.autoscaling/activator created                |
|                                                                      |
| image.caching.internal.knative.dev/queue-proxy created               |
|                                                                      |
| gateway.networking.istio.io/cluster-local-gateway created            |
|                                                                      |
| gateway.networking.istio.io/knative-ingress-gateway created          |
|                                                                      |
| servicerole.rbac.istio.io/istio-service-role created                 |
|                                                                      |
| servicerolebinding.rbac.istio.io/istio-service-role-binding created  |
|                                                                      |
| validatingwebhookconfigura                                           |
| tion.admissionregistration.k8s.io/config.webhook.serving.knative.dev |
| created                                                              |
|                                                                      |
| validatingwebhookconfiguration                                       |
| .admissionregistration.k8s.io/validation.webhook.serving.knative.dev |
| created                                                              |
|                                                                      |
| INFO\[0577\] Successfully applied application knative-install        |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0577\] Deploying application kfserving-crds                    |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcede                                                     |
| finition.apiextensions.k8s.io/inferenceservices.serving.kubeflow.org |
| created                                                              |
|                                                                      |
| application.app.k8s.io/kfserving-crds created                        |
|                                                                      |
| INFO\[0577\] Successfully applied application kfserving-crds         |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0577\] Deploying application kfserving-install                 |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kfserving-proxy-role created   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-kfserving-admin       |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-kfserving-edit        |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-kfserving-view        |
| created                                                              |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/manager-role created           |
|                                                                      |
| clus                                                                 |
| terrolebinding.rbac.authorization.k8s.io/kfserving-proxy-rolebinding |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/manager-rolebinding     |
| created                                                              |
|                                                                      |
| configmap/inferenceservice-config created                            |
|                                                                      |
| configmap/kfserving-parameters-dbdb8cm9t2 created                    |
|                                                                      |
| secret/kfserving-webhook-server-secret created                       |
|                                                                      |
| service/kfserving-controller-manager-metrics-service created         |
|                                                                      |
| service/kfserving-controller-manager-service created                 |
|                                                                      |
| statefulset.apps/kfserving-controller-manager created                |
|                                                                      |
| application.app.k8s.io/kfserving created                             |
|                                                                      |
| INFO\[0578\] Successfully applied application kfserving-install      |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0578\] Deploying application spartakus                         |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/spartakus created                                     |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/spartakus created              |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/spartakus created       |
|                                                                      |
| configmap/spartakus-parameters created                               |
|                                                                      |
| deployment.apps/spartakus-volunteer created                          |
|                                                                      |
| application.app.k8s.io/spartakus created                             |
|                                                                      |
| INFO\[0579\] Successfully applied application spartakus              |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0579\] Deploying application tensorboard                       |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| configmap/parameters-dgd4h256h5 created                              |
|                                                                      |
| service/tensorboard created                                          |
|                                                                      |
| deployment.apps/tensorboard created                                  |
|                                                                      |
| virtualservice.networking.istio.io/tensorboard created               |
|                                                                      |
| INFO\[0579\] Successfully applied application tensorboard            |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0579\] Deploying application tf-job-crds                       |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/tfjobs.kubeflow.org    |
| created                                                              |
|                                                                      |
| application.app.k8s.io/tf-job-crds created                           |
|                                                                      |
| INFO\[0579\] Successfully applied application tf-job-crds            |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0579\] Deploying application tf-job-operator                   |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/tf-job-dashboard created                              |
|                                                                      |
| serviceaccount/tf-job-operator created                               |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-tfjobs-admin created  |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-tfjobs-edit created   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-tfjobs-view created   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/tf-job-operator created        |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/tf-job-operator created |
|                                                                      |
| service/tf-job-operator created                                      |
|                                                                      |
| deployment.apps/tf-job-operator created                              |
|                                                                      |
| application.app.k8s.io/tf-job-operator created                       |
|                                                                      |
| INFO\[0579\] Successfully applied application tf-job-operator        |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0579\] Deploying application katib-crds                        |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/experiments.kubeflow.org |
| created                                                              |
|                                                                      |
| cu                                                                   |
| stomresourcedefinition.apiextensions.k8s.io/suggestions.kubeflow.org |
| created                                                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/trials.kubeflow.org    |
| created                                                              |
|                                                                      |
| application.app.k8s.io/katib-crds created                            |
|                                                                      |
| INFO\[0580\] Successfully applied application katib-crds             |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0580\] Deploying application katib-controller                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/katib-controller created                              |
|                                                                      |
| serviceaccount/katib-ui created                                      |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/katib-controller created       |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/katib-ui created               |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-katib-admin created   |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-katib-edit created    |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/kubeflow-katib-view created    |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/katib-controller        |
| created                                                              |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/katib-ui created        |
|                                                                      |
| configmap/katib-config created                                       |
|                                                                      |
| configmap/katib-parameters created                                   |
|                                                                      |
| configmap/trial-template created                                     |
|                                                                      |
| secret/katib-controller created                                      |
|                                                                      |
| secret/katib-mysql-secrets created                                   |
|                                                                      |
| service/katib-controller created                                     |
|                                                                      |
| service/katib-db-manager created                                     |
|                                                                      |
| service/katib-mysql created                                          |
|                                                                      |
| service/katib-ui created                                             |
|                                                                      |
| deployment.apps/katib-controller created                             |
|                                                                      |
| deployment.apps/katib-db-manager created                             |
|                                                                      |
| deployment.apps/katib-mysql created                                  |
|                                                                      |
| deployment.apps/katib-ui created                                     |
|                                                                      |
| application.app.k8s.io/katib-controller created                      |
|                                                                      |
| virtualservice.networking.istio.io/katib-ui created                  |
|                                                                      |
| persistentvolumeclaim/katib-mysql created                            |
|                                                                      |
| INFO\[0583\] Successfully applied application katib-controller       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0583\] Deploying application api-service                       |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/ml-pipeline created                                   |
|                                                                      |
| role.rbac.authorization.k8s.io/ml-pipeline created                   |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/ml-pipeline created            |
|                                                                      |
| configmap/ml-pipeline-config created                                 |
|                                                                      |
| service/ml-pipeline created                                          |
|                                                                      |
| deployment.apps/ml-pipeline created                                  |
|                                                                      |
| application.app.k8s.io/api-service created                           |
|                                                                      |
| INFO\[0583\] Successfully applied application api-service            |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0583\] Deploying application minio                             |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| configmap/pipeline-minio-parameters created                          |
|                                                                      |
| secret/mlpipeline-minio-artifact created                             |
|                                                                      |
| service/minio-service created                                        |
|                                                                      |
| deployment.apps/minio created                                        |
|                                                                      |
| application.app.k8s.io/minio created                                 |
|                                                                      |
| persistentvolumeclaim/minio-pv-claim created                         |
|                                                                      |
| INFO\[0584\] Successfully applied application minio                  |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0584\] Deploying application mysql                             |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| configmap/pipeline-mysql-parameters created                          |
|                                                                      |
| service/mysql created                                                |
|                                                                      |
| deployment.apps/mysql created                                        |
|                                                                      |
| application.app.k8s.io/mysql created                                 |
|                                                                      |
| persistentvolumeclaim/mysql-pv-claim created                         |
|                                                                      |
| INFO\[0584\] Successfully applied application mysql                  |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0584\] Deploying application persistent-agent                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/ml-pipeline-persistenceagent created                  |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/ml-pipeline-persistenceagent   |
| created                                                              |
|                                                                      |
| clust                                                                |
| errolebinding.rbac.authorization.k8s.io/ml-pipeline-persistenceagent |
| created                                                              |
|                                                                      |
| deployment.apps/ml-pipeline-persistenceagent created                 |
|                                                                      |
| application.app.k8s.io/persistent-agent created                      |
|                                                                      |
| INFO\[0585\] Successfully applied application persistent-agent       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0585\] Deploying application pipelines-runner                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/pipeline-runner created                               |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/pipeline-runner created        |
|                                                                      |
| clusterrolebinding.rbac.authorization.k8s.io/pipeline-runner created |
|                                                                      |
| application.app.k8s.io/pipelines-runner created                      |
|                                                                      |
| INFO\[0585\] Successfully applied application pipelines-runner       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0585\] Deploying application pipelines-ui                      |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| serviceaccount/ml-pipeline-ui created                                |
|                                                                      |
| role.rbac.authorization.k8s.io/ml-pipeline-ui created                |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/ml-pipeline-ui created         |
|                                                                      |
| configmap/ui-parameters-hb792fcf5d created                           |
|                                                                      |
| service/ml-pipeline-tensorboard-ui created                           |
|                                                                      |
| service/ml-pipeline-ui created                                       |
|                                                                      |
| deployment.apps/ml-pipeline-ui created                               |
|                                                                      |
| application.app.k8s.io/pipelines-ui created                          |
|                                                                      |
| virtualservice.networking.istio.io/ml-pipeline-tensorboard-ui        |
| created                                                              |
|                                                                      |
| virtualservice.networking.istio.io/ml-pipeline-ui created            |
|                                                                      |
| INFO\[0585\] Successfully applied application pipelines-ui           |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0585\] Deploying application pipelines-viewer                  |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/viewers.kubeflow.org   |
| created                                                              |
|                                                                      |
| serviceaccount/ml-pipeline-viewer-crd-service-account created        |
|                                                                      |
| clusterrole.rbac.aut                                                 |
| horization.k8s.io/ml-pipeline-viewer-kubeflow-pipeline-viewers-admin |
| created                                                              |
|                                                                      |
| clusterrole.rbac.au                                                  |
| thorization.k8s.io/ml-pipeline-viewer-kubeflow-pipeline-viewers-edit |
| created                                                              |
|                                                                      |
| clusterrole.rbac.au                                                  |
| thorization.k8s.io/ml-pipeline-viewer-kubeflow-pipeline-viewers-view |
| created                                                              |
|                                                                      |
| clus                                                                 |
| terrole.rbac.authorization.k8s.io/ml-pipeline-viewer-controller-role |
| created                                                              |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/ml-pipeline-viewer-crd-role-binding |
| created                                                              |
|                                                                      |
| deployment.apps/ml-pipeline-viewer-controller-deployment created     |
|                                                                      |
| application.app.k8s.io/pipelines-viewer created                      |
|                                                                      |
| INFO\[0586\] Successfully applied application pipelines-viewer       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0586\] Deploying application scheduledworkflow                 |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customres                                                            |
| ourcedefinition.apiextensions.k8s.io/scheduledworkflows.kubeflow.org |
| created                                                              |
|                                                                      |
| serviceaccount/ml-pipeline-scheduledworkflow created                 |
|                                                                      |
| role.rbac.authorization.k8s.io/ml-pipeline-scheduledworkflow created |
|                                                                      |
| clu                                                                  |
| sterrole.rbac.authorization.k8s.io/kubeflow-scheduledworkflows-admin |
| created                                                              |
|                                                                      |
| cl                                                                   |
| usterrole.rbac.authorization.k8s.io/kubeflow-scheduledworkflows-edit |
| created                                                              |
|                                                                      |
| cl                                                                   |
| usterrole.rbac.authorization.k8s.io/kubeflow-scheduledworkflows-view |
| created                                                              |
|                                                                      |
| cluste                                                               |
| rrolebinding.rbac.authorization.k8s.io/ml-pipeline-scheduledworkflow |
| created                                                              |
|                                                                      |
| deployment.apps/ml-pipeline-scheduledworkflow created                |
|                                                                      |
| application.app.k8s.io/scheduledworkflow created                     |
|                                                                      |
| INFO\[0587\] Successfully applied application scheduledworkflow      |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0587\] Deploying application pipeline-visualization-service    |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| service/ml-pipeline-ml-pipeline-visualizationserver created          |
|                                                                      |
| deployment.apps/ml-pipeline-ml-pipeline-visualizationserver created  |
|                                                                      |
| application.app.k8s.io/pipeline-visualization-service created        |
|                                                                      |
| INFO\[0587\] Successfully applied application                        |
| pipeline-visualization-service                                       |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0587\] Deploying application profiles                          |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinition.apiextensions.k8s.io/profiles.kubeflow.org  |
| created                                                              |
|                                                                      |
| serviceaccount/profiles-controller-service-account created           |
|                                                                      |
| cluste                                                               |
| rrolebinding.rbac.authorization.k8s.io/profiles-cluster-role-binding |
| created                                                              |
|                                                                      |
| configmap/profiles-profiles-parameters-5c86m8kfb8 created            |
|                                                                      |
| service/profiles-kfam created                                        |
|                                                                      |
| deployment.apps/profiles-deployment created                          |
|                                                                      |
| application.app.k8s.io/profiles created                              |
|                                                                      |
| virtualservice.networking.istio.io/kfam created                      |
|                                                                      |
| INFO\[0588\] Successfully applied application profiles               |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0588\] Deploying application seldon-core-operator              |
| filename=\"kustomize/kustomize.go:172\"                              |
|                                                                      |
| customresourcedefinit                                                |
| ion.apiextensions.k8s.io/seldondeployments.machinelearning.seldon.io |
| created                                                              |
|                                                                      |
| mutatingwebhookconfiguration.admissi                                 |
| onregistration.k8s.io/seldon-mutating-webhook-configuration-kubeflow |
| created                                                              |
|                                                                      |
| serviceaccount/seldon-manager created                                |
|                                                                      |
| role.rbac.authorization.k8s.io/seldon-leader-election-role created   |
|                                                                      |
| role.rbac.authorization.k8s.io/seldon-manager-cm-role created        |
|                                                                      |
| clusterrole.rbac.authorization.k8s.io/seldon-manager-role-kubeflow   |
| created                                                              |
|                                                                      |
| cl                                                                   |
| usterrole.rbac.authorization.k8s.io/seldon-manager-sas-role-kubeflow |
| created                                                              |
|                                                                      |
| role                                                                 |
| binding.rbac.authorization.k8s.io/seldon-leader-election-rolebinding |
| created                                                              |
|                                                                      |
| rolebinding.rbac.authorization.k8s.io/seldon-manager-cm-rolebinding  |
| created                                                              |
|                                                                      |
| clusterroleb                                                         |
| inding.rbac.authorization.k8s.io/seldon-manager-rolebinding-kubeflow |
| created                                                              |
|                                                                      |
| clusterrolebindi                                                     |
| ng.rbac.authorization.k8s.io/seldon-manager-sas-rolebinding-kubeflow |
| created                                                              |
|                                                                      |
| configmap/seldon-config created                                      |
|                                                                      |
| service/seldon-webhook-service created                               |
|                                                                      |
| deployment.apps/seldon-controller-manager created                    |
|                                                                      |
| application.app.k8s.io/seldon-core-operator created                  |
|                                                                      |
| certificate.cert-manager.io/seldon-serving-cert created              |
|                                                                      |
| issuer.cert-manager.io/seldon-selfsigned-issuer created              |
|                                                                      |
| validatingwebhookconfiguration.admission                             |
| registration.k8s.io/seldon-validating-webhook-configuration-kubeflow |
| created                                                              |
|                                                                      |
| INFO\[0590\] Successfully applied application seldon-core-operator   |
| filename=\"kustomize/kustomize.go:209\"                              |
|                                                                      |
| INFO\[0590\] Applied the configuration Successfully!                 |
| filename=\"cmd/apply.go:72\"                                         |
+----------------------------------------------------------------------+

\#\# Issue

-   ~~Controller作成時、vCenterが別セグメントにいると失敗する（？）~~

-   

-   ~~Bundleデプロイ時に指定するConstraints（ノードのリソース値指定）が効かず、1GBMem/10GB
    Diskで作成される。（テンプレートのリソースのままデプロイされる）~~

-   JujuでKubeflowをデプロイすると古いKubeflowのバージョンがデプロイされる。

-   GPUを同時にデプロイできない。

-   ~~GPUを付けた後検知できない（期待値は自動検知→ドライバ自動インストール）~~

-   ~~juju add-unit kubernetes-worker が動かない~~

\#\# troubleshooting

1.  

2.  Could not create VM template

+----------------------------------------------------------------------+
| 03:26:21 WARN juju.provider.vmware client.go:115 empty string passed |
| as vm-folder, using Datacenter root folder instead                   |
|                                                                      |
| \- creating template VM: streaming                                   |
| http://cloud-images.ubuntu.com/releases/server/rel                   |
| eases/bionic/release-20200407/ubuntu-18.04-server-cloudimg-amd64.ova |
| to                                                                   |
| https://pesxiknl0                                                    |
| 2.vsphere.local/nfc/52a405b1-a91d-57ef-a4c3-18a6fbfdd2b0/dis03:26:21 |
| ERROR juju.cmd.juju.commands bootstrap.go:776 failed to bootstrap    |
| model: cannot start bootstrap instance in availability zone          |
| \"Develop\": creating template VM: streaming                         |
| http://cloud-images.ubuntu.com/releases/server/rel                   |
| eases/bionic/release-20200407/ubuntu-18.04-server-cloudimg-amd64.ova |
| to                                                                   |
| https://pesxiknl02                                                   |
| .vsphere.local/nfc/52a405b1-a91d-57ef-a4c3-18a6fbfdd2b0/disk-0.vmdk: |
| Post                                                                 |
| https://pesxiknl02                                                   |
| .vsphere.local/nfc/52a405b1-a91d-57ef-a4c3-18a6fbfdd2b0/disk-0.vmdk: |
| EOF                                                                  |
+----------------------------------------------------------------------+

→ Check proxy or connectivity between juju client and vCenter/ESXi

\#\# Juju bootstrap log(Success)

+----------------------------------------------------------------------+
| juju bootstrap vsp-cloud vsp-controller \--config bootstrap.conf     |
| \--show-log \--verbose                                               |
|                                                                      |
| 03:54:04 INFO juju.cmd supercommand.go:83 running juju \[2.7.5 gc    |
| go1.10.4\]                                                           |
|                                                                      |
| Adding contents of                                                   |
| \"/home/hitp/.local/share/juju/ssh/juju\_id\_rsa.pub\" to            |
| authorized-keys                                                      |
|                                                                      |
| Adding contents of \"/home/hitp/.ssh/id\_rsa.pub\" to                |
| authorized-keys                                                      |
|                                                                      |
| Creating Juju controller \"vsp-controller\" on                       |
| vsp-cloud/Hewlett-Packard                                            |
|                                                                      |
| 03:54:08 INFO juju.cmd.juju.commands bootstrap.go:825 combined       |
| bootstrap constraints:                                               |
|                                                                      |
| Loading image metadata                                               |
|                                                                      |
| Looking for packaged Juju agent version 2.7.5 for amd64              |
|                                                                      |
| 03:54:08 INFO juju.environs.bootstrap tools.go:72 looking for        |
| bootstrap agent binaries: version=2.7.5                              |
|                                                                      |
| 03:54:11 INFO juju.environs.bootstrap tools.go:74 found 1 packaged   |
| agent binaries                                                       |
|                                                                      |
| Starting new instance for initial controller                         |
|                                                                      |
| 03:54:12 WARN juju.provider.vmware client.go:115 empty string passed |
| as vm-folder, using Datacenter root folder instead                   |
|                                                                      |
| Launching controller instance(s) on vsp-cloud/Hewlett-Packard\...    |
|                                                                      |
| 03:54:18 INFO juju.provider.vmware createvm.go:487 selecting         |
| datastore datastore2                                                 |
|                                                                      |
| 03:54:19 WARN juju.provider.vmware client.go:115 empty string passed |
| as vm-folder, using Datacenter root folder instead                   |
|                                                                      |
| \- creating template VM                                              |
| \"juju-template-68352f                                               |
| 4a74cf349e463eced27eed11a9617a3a14c63ed01a157f469cff120543\"03:54:21 |
| INFO juju.provider.vmware createvm.go:260 Streaming VMDK from        |
| http://cloud-images.ubuntu.com/releases/server/rel                   |
| eases/bionic/release-20200407/ubuntu-18.04-server-cloudimg-amd64.ova |
| to                                                                   |
| https://pesxiknl0                                                    |
| 2.vsphere.local/nfc/52644dd7-506c-d672-ff31-abefb14ee213/disk-0.vmdk |
|                                                                      |
| \- streaming vmdk: 2.98% (2.5MiB/s) - streaming vmdk: 5.73%          |
| (2.7MiB/s) - streaming vmdk: 7.96% (1.1MiB/s) - streaming vmdk:      |
| 10.14% (675.7KiB/s) - streaming vmdk: 12.42% (1.6MiB/s) - streaming  |
| vmdk: 15.54% (2.4MiB/s) - streaming vmdk: 18.30% (1.8MiB/s) -        |
| streaming vmdk: 21.17% (2.5MiB/s) - streaming vmdk: 23.72%           |
| (1.1MiB/s) - streaming vmdk: 26.19% (1.4MiB/s) - streaming vmdk:     |
| 27.97% (1.5MiB/s) - streaming vmdk: 30.92% (2.3MiB/s) - streaming    |
| vmdk: 32.29% (755.7KiB/s) - streaming vmdk: 34.39% (1.3MiB/s) -      |
| streaming vmdk: 37.45% (2.4MiB/s) - streaming vmdk: 40.82%           |
| (1.7MiB/s) - streaming vmdk: 43.79% (3.0MiB/s) - streaming vmdk:     |
| 46.59% (2.1MiB/s) - streaming vmdk: 48.40% (1.8MiB/s) - streaming    |
| vmdk: 51.36% (1.3MiB/s) - streaming vmdk: 54.16% (1.5MiB/s) -        |
| streaming vmdk: 57.90% (1.3MiB/s) - streaming vmdk: 61.08%           |
| (1.5MiB/s) - streaming vmdk: 63.13% (991.6KiB/s) - streaming vmdk:   |
| 65.45% (1.9MiB/s) - streaming vmdk: 68.46% (2.9MiB/s) - streaming    |
| vmdk: 71.41% (1.2MiB/s) - streaming vmdk: 74.21% (1.9MiB/s) -        |
| streaming vmdk: 76.66% (1.9MiB/s) - streaming vmdk: 80.74%           |
| (2.1MiB/s) - streaming vmdk: 84.07% (2.8MiB/s) - streaming vmdk:     |
| 87.52% (2.3MiB/s) - streaming vmdk: 89.96% (1.9MiB/s) - streaming    |
| vmdk: 93.28% (2.1MiB/s) - streaming vmdk: 96.26% (1.8MiB/s)          |
|                                                                      |
| \- streaming vmdk: 98.12% (349.7KiB/s)                               |
|                                                                      |
| \- cloning template                                                  |
|                                                                      |
| \- VM cloned                                                         |
|                                                                      |
| \- extending disk to 8.0GiB                                          |
|                                                                      |
| \- powering on                                                       |
|                                                                      |
| 03:57:33 INFO juju.provider.vmware environ\_broker.go:94 started     |
| instance \"juju-aa066d-0\"                                           |
|                                                                      |
| \- juju-aa066d-0 (arch=amd64 mem=3.5G)                               |
|                                                                      |
| 03:57:33 INFO juju.environs.bootstrap bootstrap.go:856 newest        |
| version: 2.7.5                                                       |
|                                                                      |
| 03:57:33 INFO juju.environs.bootstrap bootstrap.go:871 picked        |
| bootstrap agent binary version: 2.7.5                                |
|                                                                      |
| Installing Juju agent on bootstrap instance                          |
|                                                                      |
| No available Juju GUI archives found                                 |
|                                                                      |
| Waiting for address                                                  |
|                                                                      |
| Attempting to connect to 10.0.1.141:22                               |
|                                                                      |
| Attempting to connect to fe80::250:56ff:fe1d:f4e4:22                 |
|                                                                      |
| Connected to 10.0.1.141                                              |
|                                                                      |
| 03:58:39 INFO juju.cloudconfig userdatacfg\_unix.go:565 Fetching     |
| agent: curl -sSfw \'agent binaries from %{url\_effective}            |
| downloaded: HTTP %{http\_code}; time %{time\_total}s; size           |
| %{size\_download} bytes; speed %{speed\_download} bytes/s \'         |
| \--retry 10 -o \$bin/tools.tar.gz                                    |
| \<\[https://streams.                                                 |
| canonical.com/juju/tools/agent/2.7.5/juju-2.7.5-ubuntu-amd64.tgz\]\> |
|                                                                      |
| Running machine configuration script\...                             |
|                                                                      |
| Bootstrap agent now started                                          |
|                                                                      |
| 04:01:43 INFO juju.juju api.go:302 API endpoints changed from \[\]   |
| to \[10.0.1.141:17070\]                                              |
|                                                                      |
| Contacting Juju controller at 10.0.1.141 to verify accessibility\... |
|                                                                      |
| 04:01:43 INFO juju.juju api.go:67 connecting to API addresses:       |
| \[10.0.1.141:17070\]                                                 |
|                                                                      |
| 04:01:53 INFO juju.api apiclient.go:624 connection established to    |
| \"wss                                                                |
| ://10.0.1.141:17070/model/a960aa10-16a1-42e4-8427-7c0db6010fcc/api\" |
|                                                                      |
| Bootstrap complete, controller \"vsp-controller\" is now available   |
|                                                                      |
| Controller machines are in the \"controller\" model                  |
|                                                                      |
| Initial model \"default\" added                                      |
|                                                                      |
| 04:01:53 INFO cmd supercommand.go:525 command finished               |
|                                                                      |
| hitp\@ubu-gw-ext:\~\$ juju status                                    |
|                                                                      |
| Model Controller Cloud/Region Version SLA Timestamp                  |
|                                                                      |
| default vsp-controller vsp-cloud/Hewlett-Packard 2.7.5 unsupported   |
| 04:02:09Z                                                            |
|                                                                      |
| Model \"admin/default\" is empty.                                    |
+----------------------------------------------------------------------+

\#\# References

Bugs

https://launchpad.net/charmed-kubernetes

Kubeflow

https://github.com/kubeflow/manifests/tree/master/kfdef

juju Kubernetes Integrator with vsphere

https://jaas.ai/u/containers/vsphere-integrator

vSphere storage

https://blog.inkubate.io/use-vsphere-storage-as-kubernetes-persistant-volumes/

vsphere x juju x gpu

https://hackernoon.com/vmware-vsphere-kubernetes-and-gpus-of-course-a7a06c87c51d

juju no-proxy with cidr

https://bugs.launchpad.net/juju/+bug/1639494
