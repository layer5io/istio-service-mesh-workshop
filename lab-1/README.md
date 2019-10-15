# Lab 1 - Download and deploy Istio resources

Now that we have a Kubernetes cluster and Meshery, we are ready to download and deploy Istio resources.

## Steps

* [1. Download Istio resources](#1)
* [2. Setup `istioctl`](#2)
* [3. Install Istio](#3)
* [4. Verify install](#4)
* [5. Confirm Add-ons](#5)


## <a name="3"></a> 3 - Install Istio

In Meshery, select the deployed Istio adapter in the left nav menu under the `Management` section ([see screenshot](https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/feature/blend-in-meshery/lab-1/img/meshery_management_istio.png)).

On the Istio adapter's management page, on the `Install` card, you can click on the (+) icon and select `Latest Istio with mTLS` to install the latest version of Istio with mTLS ([see screenshot](https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/feature/blend-in-meshery/lab-1/img/meshery_management_istio-install.png)).

<small>For manual steps go [here](#appendix)</small>

## <a name="4"></a> 4 - Verify install

Istio is deployed in a separate Kubernetes namespace `istio-system`. To check if Istio is deployed, and also, to see all the pieces that are deployed, execute the following:

```sh
kubectl get all -n istio-system
```
## <a name="5"></a> 5 - Confirming Add-ons
	
Istio, as part of this workshop, is installed with several optional addons like:
1. [Prometheus](https://prometheus.io/)
2. [Grafana](https://grafana.com/)
3. [Zipkin](https://zipkin.io/)
4. [Jaeger](https://www.jaegertracing.io/)
5. [Kiali](https://www.kiali.io/)
	
You will use Prometheus and Grafana for collecting and viewing metrics, while for viewing distributed traces, you can choose between [Zipkin](https://zipkin.io/) or [Jaeger](https://www.jaegertracing.io/). In this training, we will use Jaeger.
	
Kiali is another add-on which can be used to generate a graph of services within an Istio mesh and is deployed as part of Istio in this lab.


## <a name="appendix"></a> Appendix - Manual Install

### <a name="1"></a> 1 - Download Istio
You will download and deploy the latest Istio resources on your Kubernetes cluster. 

***Note to Docker Desktop users:*** please ensure your Docker VM has atleast 4GiB of Memory, which is required for all services to run.

On your local machine:
```sh
curl -L https://git.io/getLatestIstio | sh -
```

### <a name="2"></a> 2 - Setting up istioctl
On a *nix system, you can setup istioctl by doing the following: 

The above command will get the latest Istio package and untar it in the same folder.

Change into the Istio package directory and add the `istioctl` client to your PATH environment variable.
```sh
cd istio-*
export PATH=$PWD/bin:$PATH
```

To verify `istioctl` is setup lets try to print out the command help
```sh
istioctl version
```

To verify `istioctl` is setup lets try to print out the command help
```sh
istioctl version
```

We can use a new feature in istioctl to check if the cluster is ready for install:

```sh
istioctl verify-install
```

### Install istio:

Deploy Istio custom resources:
```sh
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
```

If you see an error message like this:
```sh
error: unable to recognize "istio.yaml": no matches for admissionregistration.k8s.io/, Kind=MutatingWebhookConfiguration
```

You are likely running Kubernetes version 1.9 or earlier, which might NOT have support for mutating admission webhooks or might not have it enabled and is the reason for the error. You can continue with the lab without any issues.

```sh
kubectl apply -f install/kubernetes/istio-demo-auth.yaml
```

## [Continue to Lab 2 - Deploy Sample Bookinfo app](../lab-2/README.md)
