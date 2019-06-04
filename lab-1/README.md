# Lab 1 - Create a Kubernetes Cluster

Access to a Kubernetes cluster is required. You may use any Kubernetes platform of your choice. The "Introduction to Istio" training uses Docker Desktop the example Kubernetes platform. If you would like to use a different Kubernetes cluster (like your lab cluster or Minikube), you can skip lab-1 (this lab).

## Docker Desktop Setup

1. Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop).
1. Ensure 4GB is allocated to your Docker Desktop VM in Docker Desktop preferences ([see screenshot](img/docker-desktop-memory.png)).
1. Enable Kubernetes in Docker Desktop preferences ([see screenshot](img/docker-desktop-kube.png)).

- Mac and Windows users should be able to continue with Kubernetes on Docker Desktop.
- Linux users should be able to install kubeadm and kubelet version 1.14.1 with your respective package managers on your machines and continue with the labs. [Here](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/) is a link which might be helpful in this regard.

### Check Cluster Status
Check the status of the nodes. Ensure `Ready` state.
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

We can see all the pods are in `Running` state. If you have a running Kubernetes clsuter, please [continue to Lab 2 - Deploy Istio](../lab-2/README.md) 

# [Continue to Lab 2 - Deploy Istio](../lab-2/README.md)
