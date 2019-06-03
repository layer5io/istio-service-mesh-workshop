# Lab 2 - Deploy Istio

Now that we have a Kubernetes cluster, we are ready to deploy Istio.

## Steps

* [1. Installing Istio](#1)
* [2. Seting up istioctl](#2)
* [3. Verify install](#3)
* [4. Configuring Add-ons](#4)

## <a name="1"></a> 1 - Installing Istio
You will download and deploy Istio 1.1.7 resources on your Kubernetes cluster. 

***Note to Docker Desktop users:*** please ensure your Docker VM has atleast 4GiB of Memory, which is required for all services to run.

On Docker for Desktop:
```sh
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.7 sh -
```
Move into the Istio package directory.

Add the `istioctl` client to your PATH environment variable.
```sh
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

```sh
kubectl apply -f install/kubernetes/istio-demo.yaml
```

## <a name="2"></a> 2 - Verify install

Istio is deployed in a separate Kubernetes namespace `istio-system`. To check if Istio is deployed, and also, to see all the pieces that are deployed, execute the following:

```sh
kubectl get all -n istio-system
```

Also on PWK you will notice several ports being exposed as shown in the image below once Istio deployment yaml is applied on the cluster:
![](img/exposed_ports.png)

## <a name="3"></a> 3 - Setting up istioctl
On a *nix system, you can setup istioctl by doing the following: 

```sh
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.0.4 sh -
```
The above command will get the Istio 1.0.4 package and untar it in the same folder.

In the Docker Desktop environment you are most probably working as user `root` and now have the `istio-1.1.7` folder under `/root`. With this pressumption, run the following command to set the `PATH` appropriately. If not, please update the command below with the correct location of the `istio-1.1.7` folder.

```sh
export PATH="$PATH:/root/istio-1.1.7/bin"
```

To verify `istioctl` is setup lets try to print out the command help
```sh
istioctl version
```

## Configuring Add-ons

`Istio`, as part of this workshop, is installed with several optional addons like:
  1. [Prometheus](https://prometheus.io/)
  2. [Grafana](https://grafana.com/)
  3. [Zipkin](https://zipkin.io/)
  4. [Jaeger](https://www.jaegertracing.io/)
  5. [Service Graph](https://istio.io/docs/tasks/telemetry/servicegraph/)

You will use Prometheus and Grafana for collecting and viewing metrics, while for viewing distributed traces, you can choose between [Zipkin](https://zipkin.io/) or [Jaeger](https://www.jaegertracing.io/). In this training, we will use Jaeger.

Service graph is another add-on which can be used to generate a graph of services within an Istio mesh and is deployed as part of Istio in this lab.

### Exposing services

Istio add-on services are deployed by default as `ClusterIP` type services. We can expose the services outside the cluster by either changing the Kubernetes service type to `NodePort` or `LoadBalancer` or by port-forwarding or by configuring Kubernetes Ingress. In this lab, we will briefly demonstrate the `NodePort` and port-forwarding ways of exposing services.

#### Option 1: Expose services with NodePort
To expose them using NodePort service type, we can edit the services and change the service type from `ClusterIP` to `NodePort`

```sh
kubectl -n istio-system edit svc prometheus
```

```sh
kubectl -n istio-system edit svc grafana
```

```sh
kubectl -n istio-system edit svc servicegraph
```

For Jaeger, either of `tracing` or `jaeger-query` can be exposed.
```sh
kubectl -n istio-system edit svc tracing
```


Once this is done the services will be assigned dedicated ports on the hosts. 

To find the assigned ports for Grafana:
```sh
kubectl -n istio-system get svc grafana
```

To find the assigned ports for Prometheus:
```sh
kubectl -n istio-system get svc prometheus
```

To find the assigned ports for Servicegraph:
```sh
kubectl -n istio-system get svc servicegraph
```

To find the assigned ports for Jaeger:
```sh
kubectl -n istio-system get svc tracing
```

#### Option 2: Expose services with port-forwarding
To port-forward Grafana:
```sh
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana \
  -o jsonpath='{.items[0].metadata.name}') 3000:3000 &
```

To port-forward Prometheus:
```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') \
  9090:9090 &
```

To port-forward Service Graph:
```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') \
  8088:8088 &
```

To port-forward Jaeger:
```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=jaeger -o jsonpath='{.items[0].metadata.name}') \
  16686:16686 &
```


Port-forwarding runs in the foreground. We have appeneded `&` to the end of the above 2 commands to run them in the background. If you donot want this behavior, please remove the `&` from the end.


### Accessing Exposed Services

In `PWK`, once a port is exposed it will appear on top of the page as shown below as clickable hyperlinks:

![](img/exposed_ports.png)

Click each new links now and navigate to the respective add-ons web UI, if available. 


## [Continue to Lab 3 - Deploy Sample Bookinfo app](../lab-3/README.md)
