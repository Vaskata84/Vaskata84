<div align="center">

# 🚀 Kubernetes Bare Metal Installation Guide

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.26.1-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![containerd](https://img.shields.io/badge/containerd-v1.6.16-00ADD8?style=for-the-badge&logo=docker&logoColor=white)](https://containerd.io/)
[![Calico](https://img.shields.io/badge/Calico-v3.25-FF6C2C?style=for-the-badge&logo=canonical&logoColor=white)](https://www.tigera.io/project-calico/)
[![Flannel](https://img.shields.io/badge/Flannel-CNI-orange?style=for-the-badge&logo=linux&logoColor=white)](https://github.com/flannel-io/flannel)

**Complete step-by-step guide for setting up a production-ready Kubernetes cluster on bare metal**

[📖 Documentation](https://kubernetes.io/docs/) • [🌐 Visit SupportPC.org](https://supportpc.org) • [💬 Get Support](https://supportpc.org/#!/contact)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Architecture](#-architecture)
- [Installation Steps](#-installation-steps)
  - [Step 1: Disable Swap](#step-1-disable-swap)
  - [Step 2: Update /etc/fstab](#step-2-update-etcfstab)
  - [Step 3: Update /etc/hosts](#step-3-update-etchosts)
  - [Step 4: Enable Bridged Traffic in IPTABLES](#step-4-enable-bridged-traffic-in-iptables)
  - [Step 5: Install Container Runtime (containerd)](#step-5-install-container-runtime-containerd)
  - [Step 6: Installing runc](#step-6-installing-runc-for-containerd)
  - [Step 7: Installing CNI Plugin](#step-7-installing-cni-plugin)
  - [Step 8: Configure containerd](#step-8-configure-containerd)
  - [Step 9: Install Kubernetes Components](#step-9-adding-kubernetes-repo-to-install-kubectl-kubeadm-kubelet)
  - [Creating Control Panel](#creating-control-panel-on-master-node-only)
  - [Adding Calico 3.25 CNI](#adding-calico-325-cni-or-flannel)
  - [Install cri-dockerd (Optional)](#install-cri-dockerd-optional)
  - [Join Worker Nodes](#join-the-cluster-with-the-token)
- [Verification](#-verification)
- [Troubleshooting](#-troubleshooting)
- [Additional Resources](#-additional-resources)
- [Contributing](#-contributing)

---

## 🎯 Overview

This guide provides a comprehensive walkthrough for deploying a **production-ready Kubernetes cluster** on bare metal infrastructure. Perfect for DevOps engineers, system administrators, and anyone looking to build their own Kubernetes environment from scratch.

### 🌟 What You'll Learn

- 🔧 Complete bare metal Kubernetes setup
- 🐳 Container runtime (containerd) configuration
- 🌐 CNI plugin deployment (Calico AND Flannel options)
- 🐋 Docker support with cri-dockerd
- 🔐 Security best practices
- 📊 Cluster management basics

---

## ✅ Prerequisites

Before starting, ensure you have:

- 🖥️ **Ubuntu/Debian-based system** (tested on Ubuntu 20.04+)
- 💾 **Minimum 2GB RAM** per node (4GB+ recommended)
- 🔌 **2 CPU cores** per node minimum
- 🌐 **Network connectivity** between all nodes
- 👤 **Root or sudo access**
- 📡 **Static IP addresses** for all nodes

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│              Kubernetes Cluster Architecture        │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────┐      ┌──────────────────┐     │
│  │   Master Node    │      │   Worker Nodes   │     │
│  │  (k8s-control)   │◄────►│                  │     │
│  │                  │      │  - Node 1        │     │
│  │  - API Server    │      │  - Node 2        │     │
│  │  - Scheduler     │      │  - Node N        │     │
│  │  - Controller    │      │                  │     │
│  └──────────────────┘      └──────────────────┘     │
│           │                                         │
│           ▼                                         │
│  ┌──────────────────────────────────────────┐       │
│  │  Container Runtime: containerd v1.6.16   │       │
│  └──────────────────────────────────────────┘       │
│           │                                         │
│           ▼                                         │
│  ┌──────────────────────────────────────────┐       │
│  │  CNI Plugin (Choose one):                │       │
│  │  • Calico v3.25                          │       │
│  │  • Flannel (Recommended)                 │       │
│  └──────────────────────────────────────────┘       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 🚀 Installation Steps

### Step 1: Disable Swap

Kubernetes requires swap to be disabled for optimal performance.

```bash
sudo swapoff -a
```

> **Why?** Kubernetes scheduler needs to manage memory allocation precisely. Swap can interfere with this process.

---

### Step 2: Update /etc/fstab

Prevent swap from being re-enabled after system reboot.

```bash
sudo sed -i '/\sswap\s/s/^/#/' /etc/fstab
```

> **Explanation:** This comments out any swap entries in /etc/fstab

---

### Step 3: Update /etc/hosts

Configure hostname resolution for your cluster nodes.

```bash
sudo sed -i '/<IP_address>/d; $a\<IP_address>\t<hostname>' /etc/hosts
```

**Example:**
```bash
# For master node
sudo sed -i '/192.168.1.100/d; $a\192.168.1.100\tk8s-master' /etc/hosts

# For worker nodes
sudo sed -i '/192.168.1.101/d; $a\192.168.1.101\tk8s-worker1' /etc/hosts
sudo sed -i '/192.168.1.102/d; $a\192.168.1.102\tk8s-worker2' /etc/hosts
```

---

### Step 4: Enable Bridged Traffic in IPTABLES

Enable necessary kernel modules and network settings for Kubernetes networking.

```bash
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf >/dev/null && cat /etc/modules-load.d/containerd.conf

sudo modprobe overlay
sudo modprobe br_netfilter

echo -e "net.bridge.bridge-nf-call-iptables  = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\nnet.ipv4.ip_forward                 = 1" | sudo tee /etc/sysctl.d/k8s.conf >/dev/null && cat /etc/sysctl.d/k8s.conf
```

**Verify the configuration:**
```bash
# Check if modules are loaded
lsmod | grep br_netfilter
lsmod | grep overlay

# Apply sysctl parameters
sudo sysctl --system
```

---

### Step 5: Install Container Runtime (containerd)

Install containerd as the container runtime for Kubernetes.

```bash
wget https://github.com/containerd/containerd/releases/download/v1.6.16/containerd-1.6.16-linux-amd64.tar.gz -P /tmp/

tar Cxzvf /usr/local /tmp/containerd-1.6.16-linux-amd64.tar.gz

wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/

systemctl daemon-reload
systemctl enable --now containerd
```

**Verify installation:**
```bash
systemctl status containerd
containerd --version
```

---

### Step 6: Installing runc for (containerd)

runc is the container runtime that containerd uses to run containers.

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64 -P /tmp/

install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc
```

**Verify installation:**
```bash
runc --version
```

---

### Step 7: Installing CNI Plugin

Container Network Interface (CNI) plugins for network configuration.

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz -P /tmp/

mkdir -p /opt/cni/bin

tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.2.0.tgz
```

**Verify installation:**
```bash
ls -la /opt/cni/bin/
```

---

### Step 8: Configure containerd

Generate and customize containerd configuration.

```bash
mkdir -p /etc/containerd

containerd config default | tee /etc/containerd/config.toml
```

⚠️ **IMPORTANT:** Manually edit `/etc/containerd/config.toml` and change `SystemdCgroup` to `true`

```bash
# Edit the config file
sudo nano /etc/containerd/config.toml

# Find this line (around line 125):
#   SystemdCgroup = false
# Change it to:
#   SystemdCgroup = true
```

**Restart containerd:**
```bash
systemctl restart containerd
```

---

### Step 9: Adding Kubernetes Repo to Install kubectl, kubeadm, kubelet

Install Kubernetes components from official repository.

```bash
apt-get update

apt-get install -y apt-transport-https ca-certificates curl

curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list

apt-get update

sudo -s

apt-get install -y kubelet=1.26.1-00 kubeadm=1.26.1-00 kubectl=1.26.1-00

apt-mark hold kubelet kubeadm kubectl
```

**Verify installation:**
```bash
kubectl version --client
kubeadm version
kubelet --version
```

---

### Creating Control Panel on Master Node Only

Initialize the Kubernetes cluster on the master node.

#### Option 1: Command Line Initialization

```bash
kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.26.1 --node-name k8s-control
```

> **Note:** Change `10.10.0.0/16` to your network CIDR if needed

#### Option 2: Configuration File

```bash
kubeadm init --config=kubeadm-config.yaml --upload-certs
```

⚠️ **IMPORTANT:** Save the `kubeadm join` command from the output! You'll need it for worker nodes.

**Configure kubectl access:**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Verify cluster:**
```bash
kubectl cluster-info
kubectl get nodes
```

---

### Adding Calico 3.25 CNI or Flannel

Choose one CNI plugin for your cluster networking.

#### 🟢 Option A: Calico CNI (Recommended)

```bash
# Install Tigera Operator
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml

# Download and customize resources
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml

# Edit the CIDR for pods if it's custom
vi custom-resources.yaml

# Apply configuration
kubectl apply -f custom-resources.yaml
```

**Watch Calico pods deployment:**
```bash
watch kubectl get pods -n calico-system
```

#### 🔵 Option B: Flannel CNI

```bash
# Apply Flannel manifest
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**Alternative deployment:** Check [Flannel Manual Deployment](https://github.com/flannel-io/flannel#deploying-flannel-manually)

**Restart services if needed:**
```bash
sudo systemctl restart containerd
systemctl daemon-reload
```

---

### Install cri-dockerd (Optional)

If you need Docker support alongside containerd.

#### Check Docker Status

```bash
systemctl status docker
```

#### Install cri-dockerd on Debian-based Systems

```bash
# Update and install prerequisites
sudo apt update
sudo apt install git wget curl

# Get latest version
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')
echo $VER
```

#### For Intel 64-bit CPU

```bash
# Download cri-dockerd
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd-${VER}.amd64.tgz

# Extract and install
tar xvf cri-dockerd-${VER}.amd64.tgz
sudo mv cri-dockerd/cri-dockerd /usr/local/bin/

# Verify installation
cri-dockerd --version
```

#### Configure systemd Services

```bash
# Download service files
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket

# Move to systemd directory
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/

# Update service path
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

# Reload systemd
sudo systemctl daemon-reload

# Enable services
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
sudo systemctl enable cri-docker.service
sudo systemctl enable cri-docker.socket

# Start services
sudo systemctl start cri-docker.service
sudo systemctl start cri-docker.socket
```

**Check service status:**
```bash
journalctl -u cri-docker.service
sudo systemctl status cri-docker.service
```

---

### Join the Cluster with the Token

Add worker nodes to your Kubernetes cluster.

**On each worker node:**
```bash
# Use the join command from kubeadm init output
sudo kubeadm join <MASTER_IP>:6443 \
  --token <YOUR_TOKEN> \
  --discovery-token-ca-cert-hash sha256:<YOUR_HASH>
```

**If you lost the join command, generate a new one on master:**
```bash
kubeadm token create --print-join-command
```

---

## ✅ Verification

### Get Running Nodes

```bash
kubectl get nodes
```

**Expected output:**
```
NAME          STATUS   ROLES           AGE   VERSION
k8s-control   Ready    control-plane   15m   v1.26.1
k8s-worker1   Ready    <none>          10m   v1.26.1
k8s-worker2   Ready    <none>          10m   v1.26.1
```

### Additional Verification Commands

```bash
# Check all pods across all namespaces
kubectl get pods -A

# Check node details
kubectl get nodes -o wide

# Check cluster info
kubectl cluster-info

# Check component status
kubectl get componentstatuses

# Check namespaces
kubectl get namespaces

# Verify CNI plugin (for Calico)
kubectl get pods -n calico-system

# Verify CNI plugin (for Flannel)
kubectl get pods -n kube-flannel
```

---

## 🔧 Troubleshooting

### Common Issues and Solutions

<details>
<summary><b>⚠️ Nodes showing NotReady status</b></summary>

```bash
# Check kubelet status
sudo systemctl status kubelet

# View kubelet logs
sudo journalctl -u kubelet -n 100 --no-pager

# Check CNI installation
kubectl get pods -n kube-system

# Restart kubelet
sudo systemctl restart kubelet
```
</details>

<details>
<summary><b>⚠️ containerd not starting</b></summary>

```bash
# Check containerd status
sudo systemctl status containerd

# View containerd logs
sudo journalctl -u containerd -n 100 --no-pager

# Verify configuration
sudo containerd config dump

# Restart containerd
sudo systemctl restart containerd
```
</details>

<details>
<summary><b>⚠️ Pod network not working</b></summary>

```bash
# Verify network modules
lsmod | grep br_netfilter
lsmod | grep overlay

# Check sysctl settings
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.ipv4.ip_forward

# Reload network configuration
sudo sysctl --system
```
</details>

<details>
<summary><b>⚠️ CoreDNS pods pending</b></summary>

```bash
# Check CoreDNS status
kubectl get pods -n kube-system | grep coredns

# Describe CoreDNS pod
kubectl describe pod -n kube-system <coredns-pod-name>

# Restart CoreDNS
kubectl rollout restart deployment coredns -n kube-system
```
</details>

<details>
<summary><b>⚠️ Worker node join fails</b></summary>

```bash
# On worker node - reset kubeadm
sudo kubeadm reset

# Clean up
sudo rm -rf /etc/cni/net.d
sudo rm -rf $HOME/.kube/config

# Generate new token on master
kubeadm token create --print-join-command

# Try joining again
```
</details>

---

## 📚 Additional Resources

### 📖 Official Documentation

- **Kubernetes:** [https://kubernetes.io/docs/](https://kubernetes.io/docs/)
- **containerd:** [https://containerd.io/docs/](https://containerd.io/docs/)
- **Calico:** [https://docs.tigera.io/calico/latest/about/](https://docs.tigera.io/calico/latest/about/)
- **Flannel:** [https://github.com/flannel-io/flannel#deploying-flannel-manually](https://github.com/flannel-io/flannel#deploying-flannel-manually)

### 🛠️ Useful Commands Cheat Sheet

```bash
# Cluster Management
kubectl cluster-info                    # Display cluster info
kubectl get nodes                       # List all nodes
kubectl get pods -A                     # List all pods in all namespaces
kubectl get services -A                 # List all services

# Pod Management
kubectl get pods -n <namespace>         # List pods in namespace
kubectl describe pod <pod-name>         # Detailed pod info
kubectl logs <pod-name>                 # View pod logs
kubectl exec -it <pod-name> -- /bin/bash # Access pod shell

# Node Management
kubectl describe node <node-name>       # Detailed node info
kubectl cordon <node-name>              # Mark node unschedulable
kubectl uncordon <node-name>            # Mark node schedulable
kubectl drain <node-name>               # Drain node for maintenance

# Troubleshooting
kubectl get events -A                   # View cluster events
journalctl -u kubelet -f                # Follow kubelet logs
journalctl -u containerd -f             # Follow containerd logs
kubectl top nodes                       # Node resource usage
kubectl top pods -A                     # Pod resource usage
```

### 🔗 Helpful Links

- [Kubernetes Networking Guide](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- [kubeadm Installation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [Troubleshooting kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)

---

## 🌐 Need Help?

<div align="center">

### Visit **[SupportPC.org](https://supportpc.org)** for:

</div>

- 💬 **Community Support** - Get help from experienced users
- 📖 **Additional Guides** - More tutorials and documentation
- 🎯 **Professional Services** - Expert consulting and training
- 🔧 **Troubleshooting** - Dedicated assistance for your issues
- 📧 **Contact Us** - Direct support channels

<div align="center">

[![Visit SupportPC.org](https://img.shields.io/badge/Visit-SupportPC.org-blue?style=for-the-badge&logo=google-chrome&logoColor=white)](https://supportpc.org)

</div>

---

## 🤝 Contributing

We welcome contributions! Here's how you can help:

1. 🍴 **Fork** the repository
2. 🔧 **Create** a feature branch (`git checkout -b feature/AmazingFeature`)
3. 💡 **Commit** your changes (`git commit -m 'Add some AmazingFeature'`)
4. 📤 **Push** to the branch (`git push origin feature/AmazingFeature`)
5. 🎉 **Open** a Pull Request

### Guidelines

- Ensure all commands are tested
- Update documentation for any changes
- Follow existing formatting style
- Add comments for complex configurations

---

## 📝 Version History

- **v1.26.1** - Current stable release
  - containerd v1.6.16
  - Calico v3.25.0
  - Support for both Calico and Flannel CNI
  - Optional cri-dockerd support

---

## 📄 License

This guide is available under the [MIT License](LICENSE).

---

## ⭐ Show Your Support

If this guide helped you set up your Kubernetes cluster, please:

- ⭐ **Star** this repository
- 🔗 **Share** it with your colleagues and friends
- 📢 **Follow** us on [SupportPC.org](https://supportpc.org)
- 💬 **Leave** feedback and suggestions
- 📝 **Write** about your experience

---

<div align="center">

## 🎉 Thank You for Using This Guide! 🎉

### 🚀 Happy Kubernetes Clustering! 🚀

**Made with ❤️ by the [SupportPC.org](https://supportpc.org) Team**

---

[![Website](https://img.shields.io/badge/Visit-SupportPC.org-blue?style=for-the-badge&logo=google-chrome&logoColor=white)](https://supportpc.org)
[![GitHub](https://img.shields.io/badge/Follow-GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge&logo=opensourceinitiative&logoColor=white)](LICENSE)

---

### 📞 Contact & Support

For professional support and consulting services, visit our website:

🌐 **[SupportPC.org](https://supportpc.org)**

</div>
