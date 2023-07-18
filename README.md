# k8s_installation

## **Pre-Requistes:**

1. Ubuntu instance with 4 GB RAM - Master Node 
2. Ubuntu instance with at least 2 GB RAM - Worker Node 

Links refered:

* [coachdevops](https://www.coachdevops.com/2020/06/how-to-setup-kubernetes-cluster-in.html)

* [stackoverflow](https://stackoverflow.com/questions/52119985/kubeadm-init-shows-kubelet-isnt-running-or-healthy)

## **Kubernetes Setup using Kubeadm**

### **Commands to be executed in both Master and worker nodes**

Login to instances and execute the below commands:
```
sudo apt-get update -y  && sudo apt-get install apt-transport-https -y
```

Change to root user
```
sudo su -
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```
```
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
```
sudo apt-get update
```
Disable swap memory for better performance

```
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Enable IP tables

We need to enable IT tables for pod to pod communication.

```
modprobe br_netfilter
sysctl -p
sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

Install Docker on both Master and Worker nodes
```
apt-get install docker.io -y
```

Add ubuntu user to Docker group
```
usermod -aG docker ubuntu
```
```
systemctl restart docker
systemctl enable docker.service
```

Exit to come out of root user.
```
exit
```

Install Kubernetes Modules
```
sudo apt-get install -y kubelet=1.22.2-00 kubeadm=1.22.2-00 kubectl=1.22.2-00 kubernetes-cni
```
```
sudo systemctl daemon-reload
sudo systemctl start kubelet
sudo systemctl enable kubelet.service
sudo systemctl status docker
```

Edit file **/etc/docker/daemon.json**
```
sudo nano /etc/docker/daemon.json
```
Paste the below content in **/etc/docker/daemon.json**
```
{
    "exec-opts": ["native.cgroupdriver=systemd"]
}
```

```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

### **End of the execution in both Master and worker nodes**


### **Initialize Kubeadm on Master Node(only on Master Node)**

**Execute the below command as root user to initialize Kubernetes Master node.**
```
sudo su -
kubeadm init
```
Copy the "kubeadm join"command which is to be executed in worker nodes for joining with the master node

Make sure you see the master node is up.

Now exit from root user and execute below commands as normal user
```
exit
```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Installing the Weave Net Add-On
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
It make take a few mins to execute the above command.

Now execute the below command to see the pods.
```
kubectl get pods  --all-namespaces
```


## **Now login to Worker Node**

## **Join worker node to Master Node**
Paste the copied command which will join worker node to master node, execute this a normal user by putting sudo before:
```
sudo kubeadm join <master_node_ip>:6443 --token xxxxxxxxxxxxxx \
    --discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```


Go to Master and type the below command
```
kubectl get nodes
```
The above command should display both Master and worker nodes.

**```It means Kubernetes Cluster - both Master and worker nodes are setup successfully and up and running!!!```**
