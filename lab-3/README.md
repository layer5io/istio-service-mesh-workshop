# lab 3 - Deploy Sample app

To play with Istio and demonstrate some of it's capabilities we will deploy the sample BookInfo application which comes as part of the Istio package.



#### <a name="injector"></a> Bookinfo application

The bookinfo application that displays information about a book, similar to a single catalog entry of an online book store. Displayed on the page is a description of the book, book details (ISBN, number of pages, and so on), and a few book reviews.

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

![](https://istio.io/docs/guides/img/bookinfo/noistio.svg)


This application is polyglot, i.e., the microservices are written in different languages. It’s worth noting that these services have no dependencies on Istio, but make an interesting service mesh example, particularly because of the multitude of services, languages and versions for the reviews service.


To run the sample with Istio requires no changes to the application itself. Instead, we simply need to configure and run the services in an Istio-enabled environment, with Envoy sidecars injected along side each service. The needed commands and configuration vary depending on the runtime environment although in all cases the resulting deployment will look like this:

![](https://istio.io/docs/guides/img/bookinfo/withistio.svg)

All of the microservices will be packaged with an Envoy sidecar that intercepts incoming and outgoing calls for the services, providing the hooks needed to externally control, via the Istio control plane, routing, telemetry collection, and policy enforcement for the application as a whole.
























We can inject the proxy sidecars either manually or automatically. 

For automatic sidecar injection kubernetes we need api server to support `admissionregistration.k8s.io/v1beta1` apis. To verify that run:

```sh
kubectl api-versions | grep admissionregistration
```

If we get back `admissionregistration.k8s.io/v1beta1` then we can proceed with [automatic sidecar injection](#injector). If not, we can proceed with [manual injection](#manual).

#### <a name="injector"></a> Deploying sidecar injector with mutating webhook

#TBD

Then you can proceed to deploying Bookinfo app with [auto sidecar injection](#auto)

#### <a name="manual"></a> With manual sidecar injection

To do a manual sidecar injection we will be using `istioctl` command:
```sh
istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml > newBookInfo.yaml
```

Observing the new yaml file reveals that additional container Istio Proxy has been added to the Pods with necessary configurations:

```
image: docker.io/istio/proxy:0.7.1
imagePullPolicy: IfNotPresent
name: istio-proxy
```

We need to now deploy the new yaml using `kubectl`
```sh
kubectl apply -f newBookInfo.yaml
```

To do both in a single command:
```sh
kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
```

#### <a name="auto"></a> With Automatic sidecar injection

```sh
kubectl apply -f samples/bookinfo/kube/bookinfo.yaml
```

#### Verify Bookinfo deployment

1. Verify that previous deployments are all in a state of AVAILABLE before continuing. **Do not procede until they are up and running.**

    ```sh
    watch kubectl get deployment
    ```

2. Inspect the details of the pods

Look at the details of the pod and then inspect the envoy config:

```sh
watch kubectl get po
```

```sh
watch kubectl get svc
```

```sh
kubectl describe pod productpage-v1-.....
kubectl exec -it productpage-v1-..... -c istio-proxy bash
cd /etc/istio/proxy
more envoy-rev0.json
exit
```

#### [Continue to lab 4 - Istio Ingress controller](../lab-7/README.md)
