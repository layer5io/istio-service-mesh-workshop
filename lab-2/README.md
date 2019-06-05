# Lab 2 - Download and deploy Istio resources

Now that we have a Kubernetes cluster, we are ready to download and deploy Istio resources.

## Steps

* [1. Downloading Istio resources](#1)
* [2. Setup istioctl](#2)
* [3. Install Istio](#3)
* [4. Verify install](#4)
* [5. Confirming Add-ons](#5)

## <a name="1"></a> 1 - Installing Istio
You will download and deploy Istio 1.1.7 resources on your Kubernetes cluster. 

***Note to Docker Desktop users:*** please ensure your Docker VM has atleast 4GiB of Memory, which is required for all services to run.

On your local machine:
```sh
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.7 sh -
```

Move into the Istio package directory and add the `istioctl` client to your PATH environment variable.
```sh
cd istio-1.1.7
export PATH=$PWD/bin:$PATH
```

Deploy Istio custom resources:
```sh
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
```

If you see an error message like this:
```sh
error: unable to recognize "istio.yaml": no matches for admissionregistration.k8s.io/, Kind=MutatingWebhookConfiguration
```

You are likely running Kubernetes version 1.9 or earlier, which might NOT have support for mutating admission webhooks or might not have it enabled and is the reason for the error. You can continue with the lab without any issues.

## <a name="2"></a> 3 - Setting up istioctl
On a *nix system, you can setup istioctl by doing the following: 

The above command will get the Istio 1.1.7 package and untar it in the same folder.

In the Docker Desktop environment you are most probably working as user `root` and now have the `istio-1.1.7` folder under `/root`. With this pressumption, run the following command to set the `PATH` appropriately. If not, please update the command below with the correct location of the `istio-1.1.7` folder.

```sh
export PATH="$PATH:/root/istio-1.1.7/bin"
```

To verify `istioctl` is setup lets try to print out the command help
```sh
istioctl version
```
## <a name="3"></a> 2 - Install Istio

```sh
kubectl apply -f install/kubernetes/istio-demo.yaml
```

## <a name="4"></a> 2 - Verify install

Istio is deployed in a separate Kubernetes namespace `istio-system`. To check if Istio is deployed, and also, to see all the pieces that are deployed, execute the following:

```sh
kubectl get all -n istio-system
```
## <a name="5"></a> Confirming Add-ons
	
	`Istio`, as part of this workshop, is installed with several optional addons like:
	  1. [Prometheus](https://prometheus.io/)
	  2. [Grafana](https://grafana.com/)
	  3. [Zipkin](https://zipkin.io/)
	  4. [Jaeger](https://www.jaegertracing.io/)
	  5. [Service Graph](https://istio.io/docs/tasks/telemetry/servicegraph/)
	
	You will use Prometheus and Grafana for collecting and viewing metrics, while for viewing distributed traces, you can choose between [Zipkin](https://zipkin.io/) or [Jaeger](https://www.jaegertracing.io/). In this training, we will use Jaeger.
	
	Service graph is another add-on which can be used to generate a graph of services within an Istio mesh and is deployed as part of Istio in this lab.
  
## [Continue to Lab 3 - Deploy Sample Bookinfo app](../lab-3/README.md)
