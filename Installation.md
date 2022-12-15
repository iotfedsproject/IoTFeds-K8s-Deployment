[[_TOC_]]

Instructions on installing Kubernetes 1.25.03  
And additional information to install it on Intracom premises


This guide is refering to a cluster with 
- Ubuntu 22.04 LTS 
- Kubernetes 1.25.3 
- Containerd (crt)  
- Calico 

## Preapare the System

### Turn Swapp Off

As a first step we have to disable swapp because Kubernetes can't tolerate it.

```bash
sudo swapoff -a
```

### Permanetly delete swapp

After that you need to open the ‘fstab’ file and comment out the line which has mention of swap partition, to permantly disable it.

```bash
sudo vim /etc/fstab
```

You can also use sed

```bash
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Host names

**Note** : In the case of Intracom premises the VM's have already a hostname

To change the hostname of all machines, run the below command to open the file and subsequently rename the master and the nodes at the name that you desire. The regex for validation for all Kubernetes related names is `'[a-z0-9]([-a-z0-9]*[a-z0-9])?'`

```bash
sudo nano /etc/hostname
```

Run the following command on all machines to note the IP addresses of each.

```bash
ifconfig
```

Make a note of the IP address from the output of the above command. The IP address which has to be copied should be under `ens160`.


Edit the `host` file on both the master and the nodes and add an entry specifying their respective IP addresses along with their names. This is used for referencing them in the cluster.

```bash
sudo vim /etc/hosts
```

e.g.
```bash
XXX.XXX.XXX.XXX kmaster
XXX.XXX.XXX.XXX knode1
XXX.XXX.XXX.XXX knode2
```

## Containerd

Installation of prerequisites required (Run on every Master and Node):

Letting iptables see bridged traffic:
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
Installation of Containerd (CRI)
[Documentation](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

```bash
sudo apt-get update && sudo apt-get install -y containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```


## Kubernetes related packages

### Set up repos

This packages have to be installed before the installation

```bash
sudo apt-get install openssh-server  

sudo apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
deb http://packages.cloud.google.com/apt/ kubernetes-xenial main
EOF

sudo apt-get update
```

### Installation of tools

```bash
sudo apt-get install -y kubelet=1.25.3-00 kubeadm=1.25.3-00 kubectl=1.25.3-00
sudo apt-mark hold kubelet kubeadm kubectl
```

**Note** : It is mandatory to hold the updates of the kubernetes related tools automatically because the process of the update is different.

## Proxy

### Set up Proxy to environment 

You have to set proxy setting to your environment and add the to the services. The are not configured dynamically from the `/etc/environmet` when it is changed.


## Cluster Init
Only on master

```bash
sudo kubeadm init --apiserver-advertise-address=146.124.106.215 --pod-network-cidr=10.240.0.0/16 --v=5
```

After the completion of the above command keep the command with the token and do the following.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Network

Before intiliazation of the nodes we have to create the network.

**Note**: If you are using pod CIDR 192.168.0.0/16, skip to the next step. If you are using a different pod CIDR with kubeadm, no changes are required - Calico will automatically detect the CIDR based on the running configuration. For other platforms, make sure you uncomment the **CALICO_IPV4POOL_CIDR** variable in the manifest and set it to the same value as your chosen pod CIDR.

Customize the manifest as necessary.
Apply the manifest using the following command.


```bash
curl https://docs.projectcalico.org/manifests/calico.yaml -O
vim calico.yaml

kubectl apply -f calico.yaml
```

Watch the deployment of the network pods

```bash
kubectl get pods -n kube-system -w
```

## Join Nodes

After the containers of calico are on `Running` state join the nodes.


As you copied it from earlier.

```bash
sudo kubeadm join 146.124.106.181:6443 --token p8id6o.hs6r6zdmpjhtb461 --discovery-token-ca-cert-hash sha256:afe0e385b4c70beaea2078233d875c28ddd4043c8b206cd0ce62dbbefa43de8c --cri-socket /run/containerd/containerd.sock
```
