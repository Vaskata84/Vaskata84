# Installation Steps

## Step 1: Disable Swap


### Run the following command to disable swap:


### Step 1: disable swap
```bash

$ sudo swapoff -a

```
### Step 2: Update /etc/fstab
```bash

$ sudo sed -i '/\sswap\s/s/^/#/' /etc/fstab

```
### Step 3: Update /etc/hosts
```bash

sudo sed -i '/<IP_address>/d; $a\<IP_address>\t<hostname>' /etc/hosts

```
### Step 4: Enable Bridged Traffic in IPTABLES
```bash

echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf >/dev/null && cat /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter
echo -e "net.bridge.bridge-nf-call-iptables  = 1\nnet.bridge.bridge-nf-call-ip6tables = 1\nnet.ipv4.ip_forward                 = 1" | sudo tee /etc/sysctl.d/k8s.conf >/dev/null && cat /etc/sysctl.d/k8s.conf


```
### Step 5: Install Container Runtime (containerd)
```bash

wget https://github.com/containerd/containerd/releases/download/v1.6.16/containerd-1.6.16-linux-amd64.tar.gz -P /tmp/
tar Cxzvf /usr/local /tmp/containerd-1.6.16-linux-amd64.tar.gz
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -P /etc/systemd/system/
systemctl daemon-reload
systemctl enable --now containerd
```

### Step 6: Installing runc for (containerd)
```bash
wget https://github.com/opencontainers/runc/releases/download/v1.1.4/runc.amd64 -P /tmp/
install -m 755 /tmp/runc.amd64 /usr/local/sbin/runc

```

### Step 7: Installing cni plugin
```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.2.0/cni-plugins-linux-amd64-v1.2.0.tgz -P /tmp/
mkdir -p /opt/cni/bin
tar Cxzvf /opt/cni/bin /tmp/cni-plugins-linux-amd64-v1.2.0.tgz

```

### Step 8: 
```bash
mkdir -p /etc/containerd
containerd config default | tee /etc/containerd/config.toml   <<<<<<<<<<<<<< manually edit and change systemdCgroup to true
systemctl restart containerd

```
### Step 9:adding kubernetes repo to  Install kubectl, kubeadm,kubelet
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

### creating control panel on master node only
```bash 
kubeadm init --pod-network-cidr 10.10.0.0/16 --kubernetes-version 1.26.1 --node-name k8s-control     ///change to you network cidr

```

### Adding Calico 3.25 CNI or flannel
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/tigera-operator.yaml
wget https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/custom-resources.yaml
vi custom-resources.yaml <<<<<< edit the CIDR for pods if its custom
kubectl apply -f custom-resources.yaml

kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
https://github.com/flannel-io/flannel#deploying-flannel-manually

sudo systemctl restart containerd coredns - pending all server
systemctl daemon-reload 


```
#### Join the cluster with the token provided by the cluster 
### Get running nodes 
```bash
kubecl get nodes

