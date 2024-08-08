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
```markdown 

2 NODES, x64 Standard D2s v3 (2 vcpus, 8 GiB memory) EACH  
k8s-control   98.70.79.195  
k8s-1         20.235.246.223  

## Steps:

# Switch to the root user
```bash
sudo su
```

# Add node IPs to the hosts file
```bash
printf "\n 98.70.79.195 k8s-control\n20.235.246.223 k8s-1\n \n" >> /etc/hosts
```

# Load necessary kernel modules
```bash
printf "overlay\nbr_netfilter\n" >> /etc/modules-load.d/containerd.conf
modprobe overlay
modprobe br_netfilter
```

# Configure sysctl settings
```bash
printf "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\n" >> /etc/sysctl.d/99
