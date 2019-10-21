# Lab 2 - Deploy example application
To play with Istio and demonstrate some of it's capabilities, you will deploy the example BookInfo application, which is included the Istio package.

## What is the BookInfo Application?

This application is a polyglot composition of microservices are written in different languages and sample BookInfo application displays information about a book, similar to a single catalog entry of an online book store. Displayed on the page is a description of the book, book details (ISBN, number of pages, and so on), and a few book reviews.

The end-to-end architecture of the application is shown [here](https://calcotestudios.com/talks/decks/slides-velocity-london-2018-using-istio-workshop.html#/6/1).

It’s worth noting that these services have no dependencies on Istio, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.

Sidecars proxy can be either manually or automatically injected into your pods.

Automatic sidecar injection requires that your Kubernetes api-server supports `admissionregistration.k8s.io/v1` or `admissionregistration.k8s.io/v1beta1` or `admissionregistration.k8s.io/v1beta2` APIs. Verify whether your Kubernetes deployment supports these APIs by executing:

```sh
kubectl api-versions | grep admissionregistration
```
If your environment **does NOT** supports either of these two APIs, then you may use [manual sidecar injection](./appendix-manual-injection.md) to deploy the sample app. 

As part of Istio deployment in [Lab 1](../lab-1/README.md), we have deployed the sidecar injector.

### <a name="auto"></a> Deploying Sample App with Automatic sidecar injection

Istio, deployed as part of this workshop, will also deploy the sidecar injector. Let us now verify sidecar injector deployment.


```sh
kubectl -n istio-system get deployment -l istio=sidecar-injector
```
Output:
```sh
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
istio-sidecar-injector   1         1         1            1           1d
```

NamespaceSelector decides whether to run the webhook on an object based on whether the namespace for that object matches the [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors).


```sh
kubectl get namespace -L istio-injection
```

Output:
```sh
NAME           STATUS    AGE       ISTIO-INJECTION
default        Active    1h        
istio-system   Active    1h        disabled
kube-public    Active    1h        
kube-system    Active    1h
```

Now in Meshery in the browser, navigate to the Istio adapter's management page from the left nav menu.

On the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Sample Application` card and select `Book Info Application` from the list.

This will do 3 things: 
1. Label `default` namespace for sidecar injection
1. Deploys all the Book info services in the `default` namespace
1. Deploys the virtual service and gateway needed to expose the Book info's productpage in the `default` namespace.

<small>Manual step for can be found [here](#appendix)</small>


### Verify the namespace is labelled

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

### <a name="verify"></a> Verify Bookinfo deployment

1. Verify that previous deployments are all in a state of AVAILABLE before continuing. **Do not proceed until they are up and running.**

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



## <a name="appendix"></a> Appendix

### Label namespace for injection
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
istio-system   Active    1h        disabled
kube-public    Active    1h        
kube-system    Active    1h
```

### Deploy Book info
Applying this yaml file included in the Istio package you collected in https://github.com/layer5io/istio-service-mesh-workshop/tree/master/lab-1#1 will deploy the Book info app in you cluster.


```sh
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

### Deploy Gateway and Virtual Service for Book info app

```sh
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
```

## [Continue to Lab 3 - Access BookInfo via Istio Ingress Gateway](../lab-3/README.md)
