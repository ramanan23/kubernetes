# kubernetes


# Kubernetes Home Lab Setup to play around - Not the hard way
_____________
This tutorial walks you through setting up Kubernetes at your PC/ Laptop and you would love if you hate your AWS bills after running EKS. The results of this tutorial should not be viewed as production ready.
_____________


I am using VMware Workstation 17 Pro to spin up VMs

Used the below mirror to download Centos 10 iso

https://www.centos.org/download/      ##choose the desired version

https://mirrors.centos.org/mirrorlist?path=/10-stream/BaseOS/x86_64/iso/CentOS-Stream-10-latest-x86_64-dvd1.iso&redirect=1&protocol=https

Recommendation : Its recommended to install Centos server without GUI for better performance on homelab setup

# Installation of Kubernetes Cluster using kubeadm
Follow this documentation to set up a Kubernetes cluster on __CentOS Stream release 10(Coughlan)__.

This documentation guides you in setting up a cluster with one master node and one worker node.

## My cluster config
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|master1.web.com|192.168.136.130|CentOS 10|4G|2|
|Worker1|worker1.web.com|192.168.136.131|CentOS 10|4G|2|

## On both Master and Worker1
Perform all the commands as root user unless otherwise specified
##### Disable Firewall
```
systemctl disable firewalld; systemctl stop firewalld
```
##### Disable swap space
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install Docker engine 
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-19.03.12 
systemctl enable --now docker
```
### Kubernetes Setup
##### Add Kubernetes yum repository
```
Note: Please verify whether you are using latest kubernetes repo because repos has been depricated from packages.cloud.google.com so we shold use pkgs.k8s.io
Reference: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/

cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.31/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.31/rpm/repodata/repomd.xml.key
EOF
```
##### Install Kubernetes components
```
Note: Make sure that you are using same version of below packages 

yum install -y kubeadm kubelet kubectl
```
##### Enable and Start kubelet service
```
systemctl enable --now kubelet
```
## On master node
##### Initialize Kubernetes Cluster
```
Note:Its recommended to use to different network for API server and pod network

kubeadm init --apiserver-advertise-address=192.168.136.130 --pod-network-cidr=172.16.0.0/16
```
##### Deploy Calico network (CNI)
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.1/manifests/tigera-operator.yaml
```
##### Cluster join command
```
kubeadm token create --print-join-command

Expected output: kubeadm join 192.168.136.130:6443 --token 2chnqv.5ta9izid20ib \
        --discovery-token-ca-cert-hash sha256:ad3f9d004aeb2602ce10025114dbeacc8357c47ceb25da
```
##### To be able to run kubectl commands as non-root user
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
## On worker1 node
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```
kubectl get nodes
```
##### Get component status
```
kubectl get cs

Expected Output:
NAME                 STATUS    MESSAGE   ERROR
controller-manager   Healthy   ok
etcd-0               Healthy   ok
scheduler            Healthy   ok

```

Let us kube !
