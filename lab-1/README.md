# lab 1 - Create a Kubernetes Cluster

Throughout this workshop, we will use Play with Kubernetes (PWK) as our hosted lab environment. For this DockerCon18 workshop only, a temporarily-provisioned space has been provided [workshop.play-with-k8s.com](https://workshop.play-with-k8s.com). If you would like to use a different Kubernetes cluster (like your lab cluster or Docker for Desktop or Minikube), you can skip lab-1 (this lab).

## Setup Steps

1. [Set up your Kubernetes master node](#1)
1. [Install overlay networking](#2)
1. [Add more nodes to the cluster](#3)

## <a name="1"></a> 1 - Set up your Kubernetes master node

<h3> 1.1 Visit https://workshop.play-with-k8s.com.</h2>
<img src="img/pwk_start.png" width="250" />

Once you start the session, you will have your own lab environment.<br />
<img src="img/pwk_main.png" width="550" />

<br />

### 1.1 Add first node
Now add one instance by clicking the `ADD NEW INSTANCE` button on the left. When you create your first instance, it will have the name `node1`. Each instance has [Docker Community Edition (CE)](https://www.docker.com/community-edition) and [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) preinstalled. 
<br />
<img src="img/pwk_instance1.png" width="85%" />

<img src="img/warning.png" width="48" align="left" />**Warning:** Please donot follow the instructions as it is. We will be following similar but slightly different instructions described below.

We will use `node1` as the master node for our cluster. While we will create a multi-__node__ cluster in this lab, creating a multi-__master__ cluster is out of the scope of this workshop.

### 1.2 Configure external name server (DNS)
Before we start bootstrapping the cluster, first update DNS settings on the node. This step is needed to get external network connectivity from within the Kubernetes cluster we are about to setup. Let's use one of Google's public name servers:

```sh
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```
_(any public name servers will work)_ 

### 1.3 Bootstrap cluster
Next, bootstrap the Kubernetes cluster by initializing the master (`node1`) node:
```sh
kubeadm init --apiserver-advertise-address $(hostname -i)
```

Sample output from initialization:
```sh
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 0c6e9e.607906dbdcacbf64 192.168.0.8:6443 --discovery-token-ca-cert-hash sha256:b8116ec1b224d82983b10353498d222f6f2e8fcbdf5d1075b4eece0f37df5896

Waiting for api server to startup.........
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
daemonset "kube-proxy" configured
No resources found
```
### 1.4 What happened?
As part of the initialization `kubeadm` has written config files needed, setup RBAC and deployed Kubernetes control plane components (like `kube-apiserver`, `kube-dns`, `kube-proxy`, `etcd`, etc.). Control plane components are deployed as Docker containers. `kubectl` is also configured for the `root` user's account.

<img src="img/info.png" width="48" align="left" /> Please copy and save the `kubeadm join` command from the previous output for later use. This command will be used to join other nodes to your cluster. The command should look like the one below (do not use this example output):
```sh
kubeadm join --token 0c6e9e.607906dbdcacbf64 192.168.0.8:6443 --discovery-token-ca-cert-hash sha256:b8116ec1b224d82983b10353498d222f6f2e8fcbdf5d1075b4eece0f37df5896
```
### 1.5 Check cluster status
Check the status of the nodes and then the pods. To check the status of the nodes:
```sh
kubectl get nodes
```

Output of the previous command:
```sh
[node1 ~]$ kubectl get nodes
NAME      STATUS     ROLES     AGE       VERSION
node1     NotReady   master    1h        v1.10.2
```

To check the status of the pods:
```sh
kubectl get pods --all-namespaces
```

Output from the previous command:
```sh
[node1 ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY     STATUS    RESTARTS   AGE
kube-system   etcd-node1                      1/1       Running   0          1h
kube-system   kube-apiserver-node1            1/1       Running   0          59m
kube-system   kube-controller-manager-node1   1/1       Running   0          1h
kube-system   kube-dns-545bc4bfd4-nnbwn       0/3       Pending   0          1h
kube-system   kube-proxy-pxq27                1/1       Running   0          1h
kube-system   kube-scheduler-node1            1/1       Running   0          1h
```

We can see that the master node is `NotReady` state. Next, we need to install a network plugin, so that our pods and nodes can communicate with each other.

<img src="img/info.png" width="48" align="left" /> Also, `kube-dns` will not start up before a network is installed. The general recommendation is to install a Container Network Interface (CNI)-based network driver. For this workshop, we will use [Weave Net](https://www.weave.works/oss/net/).

## <a name="2"></a> 2 - Install overlay networking

### 2.1 To install weave net, execute:
```sh
kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"
```

Output from previous command:
```sh
[node1 ~]$ kubectl apply -n kube-system -f \
>     "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"
serviceaccount "weave-net" created
clusterrole "weave-net" created
clusterrolebinding "weave-net" created
role "weave-net" created
rolebinding "weave-net" created
daemonset "weave-net" created
```

Re-check the status of the nodes, we will see it is now in `Ready` state.
```sh
[node1 ~]$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
node1     Ready     master    1h        v1.10.2
```

Check the status of the pods next:
```sh
[node1 ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY     STATUS    RESTARTS   AGE
kube-system   etcd-node1                      1/1       Running   0          1h
kube-system   kube-apiserver-node1            1/1       Running   0          1h
kube-system   kube-controller-manager-node1   1/1       Running   0          1h
kube-system   kube-dns-545bc4bfd4-nnbwn       3/3       Running   0          1h
kube-system   kube-proxy-pxq27                1/1       Running   0          1h
kube-system   kube-scheduler-node1            1/1       Running   0          1h
kube-system   weave-net-wq5t5                 2/2       Running   0          2m
```

We can see all the pods are in `Running` state.

## <a name="3"></a> 3 - Adding more nodes to the cluster

We will build a 3-node cluster. 

### 3.1 Add two more instances
Click the `ADD NEW INSTANCE` button on the left.

As before, on each of the instances, first update the DNS settings:
```sh
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```
### 3.2 Join new nodes to the cluster
Now we can make the new nodes join the Kubernetes cluster by executing the `kubeadm join` you previously copied and saved. Run that command on each of the two new nodes:
```sh
kubeadm join --token 0c6e9e.607906dbdcacbf64 192.168.0.8:6443 --discovery-token-ca-cert-hash sha256:b8116ec1b224d82983b10353498d222f6f2e8fcbdf5d1075b4eece0f37df5896
```

Output from previous command:
```sh
[node2 ~]$ kubeadm join --token 0c6e9e.607906dbdcacbf64 192.168.0.8:6443 --discovery-token-ca-cert-hash sha256:b8116ec1b224d82983b10353498d222f6f2e8fcbdf5d1075b4eece0f37df5896
Initializing machine ID from random generator.
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[preflight] Skipping pre-flight checks
[discovery] Trying to connect to API Server "192.168.0.8:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://192.168.0.8:6443"
[discovery] Requesting info from "https://192.168.0.8:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "192.168.0.8:6443"
[discovery] Successfully established connection with API Server "192.168.0.8:6443"
[bootstrap] Detected server version: v1.8.13
[bootstrap] The server supports the Certificates API (certificates.k8s.io/v1beta1)

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```
#### What happened?
Go back to the master node `node1` terminal and check the status of the nodes:
```sh
watch kubectl get nodes
```

Initially, the new nodes will be in `Not Ready` state and will eventually become `Ready`

Output from previous command:
```sh
Every 2.0s: kubectl get nodes                                                                Mon May 21 03:27:34 2018

NAME      STATUS    ROLES     AGE       VERSION
node1     Ready     master    2h        v1.10.2
node2     Ready     <none>    22m       v1.10.2
node3     Ready     <none>    55s       v1.10.2
```

We now have a 3-node Kubernetes cluster ready for an Istio deployment.


# [Continue to lab 2 - Deploy Istio](../lab-2/README.md)
