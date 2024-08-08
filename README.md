# Kubernetes installation and initalizing cluster using kudeadm on Ubuntu 24.04 LTS   6.8.0-1010-azure 

# UBUNTU SERVER LTS  - https://ubuntu.com/download/server

# KUBERNETES 1.30.0	- https://kubernetes.io/releases/
# CONTAINERD 1.7.13 	- https://containerd.io/releases/
# RUNC 1.1.12		- https://github.com/opencontainers/runc/releases
# CNI PLUGINS 1.4.0	- https://github.com/containernetworking/plugins/releases
# CALICO CNI 3.27.2         - https://docs.tigera.io/calico/3.27/getting-started/kubernetes/quickstart
*********************************************************************************************************************************
**# 2 NODES, x64 Standard D2s v3 (2 vcpus, 8 GiB memory) EACH
**below are my IP address you can replace it with yours
**# k8s-control   98.70.79.195
**# k8s-1         20.235.246.223

# Steps: 
```bash
sudo su
printf "\n 98.70.79.195 k8s-control\20.235.246.223 k8s-1\n \n" >> /etc/hosts

printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf

modprobe overlay
modprobe br_netfilter

printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99-kubernetes-cri.conf

sysctl --system

wget https://github.com/containerd/containerd/releases/download/v1.7.13/containerd-1.7.13-linux-amd64.tar.gz -P /tmp/
tar Cxzvf /usr/local /tmp/containerd-1.7.13-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now containerd

wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64 -P /tmp/
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc

wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz -P /tmp/
mkdir -p /opt/cni/bin

tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.4.0.tgz

mkdir -p /etc/containerd

containerd config default | tee /etc/containerd/config.toml   <<<<<<<<<<< manually edit and change SystemdCgroup to true (not systemd_cgroup)

vi /etc/containerd/config.toml

systemctl restart containerd

swapoff -a  # <<<<<<<< just disable it in /etc/fstab instead ie. add a line #swap.img

apt-get update

apt-get install -y apt-transport-https ca-certificates curl gpg
mkdir -p -m 755 /etc/apt/keyrings

# Download the Google Cloud public signing key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add the Kubernetes apt repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

#Very IMP for the above update’s changes need Reboot
reboot 

sudo su
# Update apt package index, install kubelet, kubeadm and kubectl, and pin their version
apt-get update

apt-get install -y kubelet=1.30.0-1.1 kubeadm=1.30.0-1.1 kubectl=1.30.0-1.1 --allow-change-held-packages

#To avoid auto update, set version on hold
apt-mark hold kubelet kubeadm kubectl

# check swap config, ensure swap is 0
free -m

# Verifying Installed Versions
kubelet --version

#******* ONLY on the Master Node *************

# control plane install:

kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.30.0 --node-name k8s-control --ignore-preflight-errors=all

export KUBECONFIG=/etc/kubernetes/admin.conf

# add Calico 3.27.2 CNI 
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/custom-resources.yaml
vi custom-resources.yaml <<<<<< edit the CIDR for pods if its custom cidr: 10.10.0.0/16
kubectl apply -f custom-resources.yaml

# get worker node commands to run to join additional nodes into cluster
kubeadm token create --print-join-command

# kubectl label node k8s-
kubectl label node vm-sumaira-k8node1 node-role.kubernetes.io/worker=worker

### ONLY ON WORKER nodes
Run the command from the token create output above Master node 
 

 # Kubernetes Commands

To get detailed information about your pods, use the following command:
kubectl get pods -o wide
