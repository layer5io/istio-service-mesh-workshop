# Prerequisites
You will need each of the following in order to complete the workshop:

1. Docker (installed locally)
1. Kubernetes (installed locally or have remote access to a cluster)
1. Meshery (installed locally)

## Install Docker and Create a Kubernetes Cluster<a name="1"></a>

This training requires access to a Kubernetes cluster and uses Docker Desktop and Minikube as the example Kubernetes platforms. Alternatively, you may choose to use any of the other [supported Kubernetes platform](https://meshery.layer5.io/docs/installation/platforms).

### Setup Docker Desktop (MacOS and Windows)

1. Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop).
   1. Ensure 4GB is allocated to your Docker Desktop VM in Docker Desktop preferences ([see screenshot](https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/docker-desktop-memory.png)).
1. Create Kubernetes cluster:Enable Kubernetes in Docker Desktop preferences ([see screenshot](https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/docker-desktop-kube.png)).
1. Please open `~/.kube/config` and check the `docker-desktop` cluster under `clusters` section and ensure you see something like the image below:
  ![](img/docker-desktop-config.png)

    __Note__: If you see `https://localhost:6443` as the value for server, please get the IP address of your host and replace `localhost` with the the IP address. The end result should look like this `https://1.2.3.4:6443`.

- Mac and Windows users may continue this workshop with Kubernetes on Docker Desktop.


### Or... Setup Minikube (MacOS, Windows, Linux)
1. [Install minikube](https://minikube.sigs.k8s.io).
1. Create Kubernetes cluster: `minikube start`.
1. Export kubeconfig (to be used with Meshery):
```kubectl config view --minify --flatten > config_minikube.yaml```

### Check Cluster Status
Check the status of the nodes. Ensure `Ready` state.
```sh
[node1 ~]$ kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
node1     Ready     master    1h        v1.15.2
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

If all pods are in a `Running` state, you have an operational Kubernetes cluster. Please continue to download and run Meshery. 

## Download `mesheryctl`<a name="3"></a>

### Meshery Architecture
In this workshop, Meshery will run "off-cluster", which is to say it will be running outside of the Kubernetes cluster. You will be running Meshery in your local Docker instance. 

<img src="img/meshery-architecture.svg" alt="Meshery Architecture" style="float: left; margin-right: 10px;" width="60%" />

Install Meshery on your local machine (running Docker) by executing the following:

#### Install on MacOS with Homebrew
```
brew tap layer5io/tap
brew install mesheryctl
mesheryctl start
```

#### Or... Install on MacOS and Linux with bash script: 

```
curl -L https://git.io/meshery | bash -  
```

#### Or.... Install on Windows with `mesheryctl` binary

### [Windows](https://meshery.layer5.io/docs/installation#using-docker-on-windows)
1. Download and unzip `mesheryctl` from the [Meshery releases](https://github.com/layer5io/meshery/releases/latest) page.
1. Add `mesheryctl` to your PATH for ease of use. Then, execute:

```
./mesheryctl start
```

Upon starting Meshery successfully, instructions to access Meshery will be printed on the sceen.

## Run Meshery

Meshery will automatically launch in your browser.

Sign into Meshery ([see screenshot](https://raw.githubusercontent.com/layer5io/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/sign-into-meshery.png)) using either Twitter, Linkedin, GitHub or Google authentication.

Meshery attempts to automatically connect with your Kubernetes cluster by loading the kubeconfig found in your `$HOME/.kube` folder and connecting existing service mesh adapters ([see screenshot](https://raw.githubusercontent.com/layer5io/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/meshery_landing_page.png)).

If your kubeconfig is in a different location (i.e. if you're using minikube, not Docker Desktop), point Meshery to your kubeconfig location by navigating to the Settings page. Navigate to Settings by clicking the gear icon on the right top of the screen ([see screenshot](https://raw.githubusercontent.com/layer5io/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/meshery_landing_page_settings_icon.png)). 

This will take the user to the `Settings` page and here you can load up your new config file and select the context to use ([see screenshot](https://raw.githubusercontent.com/layer5io/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/meshery_settings_page.png)).

In a similar fashion, if you don't see the Istio adapter loaded, you should be able to switch to the `Service Meshes` tab in the `Settings` page and connect to existing adapters from the drop down ([see screenshot](https://raw.githubusercontent.com/layer5io/istio-service-mesh-workshop/feature/blend-in-meshery/prereq/img/meshery_settings_page-service-meshes.png)).

Once an adapter is connected, you will also see it added to the nav menu on the left ([see screenshot](https://github.com/layer5io/istio-service-mesh-workshop/raw/feature/blend-in-meshery/prereq/img/meshery_settings_page-service_meshes_with_menu.png)).

In the labs, you will use a combination of Meshery's UI and your terminal. We suggest splitting the view on your display between your terminal and your web browser, so that you don't have to switch between apps frequently.

<!-- ### <span style="color:gray">Alternatively - Run Meshery In-Cluster</span>
 
<p style="color:gray">
Or to install on Kubernetes by cloning the Meshery repo:</p>

 ```
 $ git clone https://github.com/layer5io/meshery.git; cd meshery        
 $ kubectl create ns meshery
 $ kubectl -n meshery apply -f deployment_yamls/k8s
``` -->

# [Continue to Lab 1 - Deploy Istio](../lab-1/README.md)
