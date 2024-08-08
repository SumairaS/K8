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

## Steps:

Switch to the root user
```bash
sudo su
```

Add node IPs to the hosts file
```bash
printf "\n 98.70.79.195 k8s-control\n20.235.246.223 k8s-1\n \n" >> /etc/hosts
```

Load necessary kernel modules
```bash
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
modprobe overlay
modprobe br_netfilter
```

Configure sysctl settings
```bash
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99-kubernetes-cri.conf
```
```bash
sysctl --system
```

Download and install containerd
```bash
wget https://github.com/containerd/containerd/releases/download/v1.7.13/containerd-1.7.13-linux-amd64.tar.gz -P /tmp/
```
```bash
tar Cxzvf /usr/local /tmp/containerd-1.7.13-linux-amd64.tar.gz
```
```bash
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/
```
```bash
systemctl daemon-reload
```
```bash
systemctl enable --now containerd
```

Download and install runc
```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64 -P /tmp/
```
```bash
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc
```

Download and install CNI plugins
```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.4.0/cni-plugins-linux-amd64-v1.4.0.tgz -P /tmp/
```
```bash
mkdir -p /opt/cni/bin
```
```bash
tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.4.0.tgz
```

Configure containerd
```bash
mkdir -p /etc/containerd
```
```bash
containerd config default | tee /etc/containerd/config.toml   # Manually edit and change SystemdCgroup to true (not systemd_cgroup)
```
```bash
vi /etc/containerd/config.toml
```
```bash
systemctl restart containerd
```

Disable swap
```bash
swapoff -a  # Just disable it in /etc/fstab instead ie. #swap.img
```

Update package lists and install necessary packages
```bash
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
mkdir -p -m 755 /etc/apt/keyrings
```

Download Google Cloud public signing key
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add the Kubernetes apt repository
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Reboot the system to apply changes
```bash
reboot
```

Switch to the root user again
```bash
sudo su
```

Update apt package index, install kubelet, kubeadm, and kubectl, and pin their versions
```bash
apt-get update
apt-get install -y kubelet=1.30.0-1.1 kubeadm=1.30.0-1.1 kubectl=1.30.0-1.1 --allow-change-held-packages
```

Prevent automatic updates
```bash
apt-mark hold kubelet kubeadm kubectl
```

Check swap configuration
```bash
free -m
```

Verify installed versions
```bash
kubelet --version
```

**ONLY on the Master Node**
Initialize the control plane
```bash
kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.30.0 --node-name k8s-control --ignore-preflight-errors=all
```

Set up kubeconfig for kubectl
```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

Add Calico CNI
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/tigera-operator.yaml
```
```bash
wget https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/custom-resources.yaml
```
# Edit the CIDR for pods if it's a custom CIDR: 10.10.0.0/16
```bash
vi custom-resources.yaml  
```
```bash
kubectl apply -f custom-resources.yaml
```

Get worker node join command
```bash
kubeadm token create --print-join-command
```

Label the worker node (example)
```bash
kubectl label node vm-sumaira-k8node1 node-role.kubernetes.io/worker=worker
```

**ONLY ON WORKER nodes**

Run the join command from the output of the above step
```bash
# Command from kubeadm token create output
```
