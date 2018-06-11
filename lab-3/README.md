# Lab 3 - Deploy Sample Bookinfo app

To play with Istio and demonstrate some of it's capabilities we will deploy the sample BookInfo application which comes as part of the Istio package.



## What is the BookInfo Application?

The sample bookinfo application displays information about a book, similar to a single catalog entry of an online book store. Displayed on the page is a description of the book, book details (ISBN, number of pages, and so on), and a few book reviews.

The Bookinfo application is broken into four separate microservices:

1. productpage. The productpage microservice calls the details and reviews microservices to populate the page.
2. details. The details microservice contains book information.
3. reviews. The reviews microservice contains book reviews. It also calls the ratings microservice.
4. ratings. The ratings microservice contains book ranking information that accompanies a book review.

There are 3 versions of the reviews microservice:

* Version v1 doesn’t call the ratings service.
* Version v2 calls the ratings service, and displays each rating as 1 to 5 black stars.
* Version v3 calls the ratings service, and displays each rating as 1 to 5 red stars.


The end-to-end architecture of the application is shown below.

<iframe src="http://calcotestudios.com/talks/slides-dockercon-18-using-istio.html#/4/1" frameborder="0"></iframe>


This application is polyglot, i.e., the microservices are written in different languages. It’s worth noting that these services have no dependencies on Istio, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.


To run the sample with Istio requires no changes to the application itself. Instead, we simply need to configure and run the services in an Istio-enabled environment, with Envoy sidecars injected along side each service. The needed commands and configuration vary depending on the runtime environment although in all cases the resulting deployment will look like this:

![](https://istio.io/docs/guides/img/bookinfo/withistio.svg)

All of the microservices will be packaged with an Envoy sidecar that intercepts incoming and outgoing calls for the services, providing the hooks needed to externally control, via the Istio control plane, routing, telemetry collection, and policy enforcement for the application as a whole.


We can inject the proxy sidecars either manually or automatically. 

For automatic sidecar injection kubernetes we need api server to support `admissionregistration.k8s.io/v1beta1` or `admissionregistration.k8s.io/v1beta2` apis. To verify that run:

```sh
kubectl api-versions | grep admissionregistration
```

If we get back any of the 2 apis then we can proceed with [automatic sidecar injection](#injector). 

As part of Istio deployment in [Lab 2](../lab-2/README.md), we have deployed the sidecar injector.
If not, we can proceed with [manual injection](#manual).

<img src="../img/info.png" width="48" align="left" /> ***Please note:*** Folks using `PWK` environment will **HAVE** to use [manual injection](#manual) irrespective of the version of Istio because `PWK` comes with Kubernetes version 1.8 which does not support `admissionregistration.k8s.io/v1beta1` or `admissionregistration.k8s.io/v1beta2` apis. 



## <a name="auto"></a> Deploying Sample App with Automatic sidecar injection

Istio, deployed as part of this workshop, will also deploy the sidecar injector. If you are using `PWK`, please proceed to [Deploying Sample App with manual sidecar injection](#manual)

Let us now verify sidecar injector deployment & label namespace for automatic sidecar injection.


```sh
kubectl -n istio-system get deployment -listio=sidecar-injector
```
Output:
```sh
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
istio-sidecar-injector   1         1         1            1           1d
```

NamespaceSelector decides whether to run the webhook on an object based on whether the namespace for that object matches the [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).

Label the default namespace with istio-injection=enabled

```sh
kubectl label namespace default istio-injection=enabled
```

```sh
kubectl get namespace -L istio-injection
```

Output:
```sh
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    1h        enabled
istio-system   Active    1h        
kube-public    Active    1h        
kube-system    Active    1h
```

Now that we have the sidecar injector with mutating webhook in place and the namespace labelled for automatic sidecar injection, we can proceed to deploy the sample app:

```sh
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-0.8.0/bookinfo.yaml
```


## <a name="manual"></a> Deploying Sample App with manual sidecar injection

To do a manual sidecar injection we will be using `istioctl` command:

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-0.8.0/bookinfo.yaml | istioctl kube-inject --debug -f - > newBookInfo.yaml
```

Observing the new yaml file reveals that additional container Istio Proxy has been added to the Pods with necessary configurations:

```
        image: docker.io/istio/proxyv2:0.8.0
        imagePullPolicy: IfNotPresent
        name: istio-proxy
```

We need to now deploy the new yaml using `kubectl`
```sh
kubectl apply -f newBookInfo.yaml
```

To do both in a single command:
```sh
kubectl apply -f <(curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-0.8.0/bookinfo.yaml | istioctl kube-inject --debug -f -)
```

## Verify Bookinfo deployment

1. Verify that previous deployments are all in a state of AVAILABLE before continuing. **Do not procede until they are up and running.**

    ```sh
    watch kubectl get deployment
    ```

2. Inspect the details of the pods

    Let us look at the details of the pods:
    ```sh
    watch kubectl get po
    ```

    Let us look at the details of the services:
    ```sh
    watch kubectl get svc
    ```

    Now let us pick a service, for instance productpage service, and view it's sidecar configuration:
    ```sh
    kubectl get po

    kubectl describe pod productpage-v1-.....
    ```

#### [Continue to lab 4 - Expose Bookinfo site through Istio Ingress Controller/Gateway](../lab-4/README.md)
