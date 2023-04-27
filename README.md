# k8s-kubeadm-calico
Installing and configuring kubernetes clusters on ubuntu using kubeadm, calico, etc.

## 1. Check ubuntu version

lsb_release -a

## 2. Clean up minikube if installed

minikube delete --all --purge

sudo rm -r /usr/local/bin/minikube

restart machine

## 3. Installing kubeadm, kubelet and kubectl

### (1). Update the apt package index and install packages needed to use the Kubernetes apt repository:

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

### (2). Download the Google Cloud public signing key:

sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

### (3). Add the Kubernetes apt repository:

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list


### (4). Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl

## 4. Reset and/or uninstall k8s

sudo kubeadm reset

sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube*   

sudo apt-get autoremove  

sudo rm -rf ~/.kube

## 5. Create a single-host Kubernetes cluster with kubeadm

sudo kubeadm init --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

## 6. Install Calico

### (1). Install the Tigera Calico operator and custom resource definitions.

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/tigera-operator.yaml

### (2). Install Calico by creating the necessary custom resource.

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/custom-resources.yaml

### (3). Confirm that all of the pods are running with the following command

watch kubectl get pods -n calico-system

![screen-shot-k8s-calico-pods](screen-shot/calico-pods.png)

### (4). Remove the taints on the control plane so that you can schedule pods on it.

kubectl taint nodes --all node-role.kubernetes.io/control-plane-

kubectl taint nodes --all node-role.kubernetes.io/master-

kubectl get nodes -o wide

![screen-shot-k8s-calico-pods](screen-shot/taint-node.png)

### For TLS certificate errors, overwrite the existing kubeconfig for the "admin" user:

mv  $HOME/.kube $HOME/.kube.bak

mkdir $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config
