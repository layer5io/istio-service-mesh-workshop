# Lab 3 - Deploy Sample Bookinfo app
To play with Istio and demonstrate some of it's capabilities we will deploy the sample BookInfo application, which is included the Istio package.

## What is the BookInfo Application?

This application is a polyglot composition of microservices are written in different languages and sample BookInfo application displays information about a book, similar to a single catalog entry of an online book store. Displayed on the page is a description of the book, book details (ISBN, number of pages, and so on), and a few book reviews.

The end-to-end architecture of the application is shown [here](https://calcotestudios.com/talks/slides-velocity-london-2018-using-istio.html#/4/1).

It’s worth noting that these services have no dependencies on Istio, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.

Sidecars proxy can either manually or automatically injected into your pods.

Automatic sidecar injection requires that your Kubernetes api-server supports `admissionregistration.k8s.io/v1beta1` or `admissionregistration.k8s.io/v1beta2` APIs. Verify whether your Kubernetes deployment supports these APIs by executing:

```sh
kubectl api-versions | grep admissionregistration
```
If your environment supports these two APIs, then you may use [automatic sidecar injection](#injector). As part of Istio deployment in [Lab 2](../lab-2/README.md), we have deployed the sidecar injector, however, we will not use the automatic sidecar injector in this workshop.

<img src="../img/info.png" width="48" align="left" /> ***Please note:*** Our `PWK` environment will **HAVE** to use [manual injection](#manual) irrespective of the version of Istio because `PWK` comes with Kubernetes version 1.8 which does not support `admissionregistration.k8s.io/v1beta1` or `admissionregistration.k8s.io/v1beta2` APIs. See <a href="auto">Appendix 3.A</a> for instructions on automatic sidecar injection.

## <a name="manual"></a> Deploying Sample App with manual sidecar injection

To do a manual sidecar injection we will be using `istioctl` command:

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/bookinfo.yaml | istioctl kube-inject --debug -f - > newBookInfo.yaml
```

Observing the new yaml file reveals that additional container Istio Proxy has been added to the Pods with necessary configurations:

```
        image: docker.io/istio/proxyv2:1.0.2
        imagePullPolicy: IfNotPresent
        name: istio-proxy
```

We need to now deploy the new yaml using `kubectl`
```sh
kubectl apply -f newBookInfo.yaml
```

To do both in a single command:
```sh
kubectl apply -f <(curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/bookinfo.yaml | istioctl kube-inject --debug -f -)
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
## [Continue to Lab 4 - Expose BookInfo via Istio Ingress Gateway](../lab-4/README.md)
 
 
### <a name="auto"></a> Appendix 3.A: Deploying Sample App with Automatic sidecar injection

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
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/bookinfo.yaml
```
## [Continue to Lab 4 - Expose BookInfo via Istio Ingress Gateway](../lab-4/README.md)
