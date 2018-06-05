# lab 4 - Expose Bookinfo site through Istio Ingress Controller/Gateway

The components deployed on the service mesh by default are not exposed outside the cluster. External access to individual services so far has been provided by creating an external load balancer on each service.

In Istio-0.7.1, a Kubernetes Ingress rule can be created that routes external requests through the Istio Ingress Controller to the backing services.
In Istio-0.8.0, things are a little different. An ingress gateway service is deployed as a LoadBalancer service. For making Bookinfo accessible from outside, we create an `Istio Gateway` for the service and also define a `Istio VirtualService` for Bookinfo with the routes we need.

## Inspecting the Istio Ingress controller/gateway

The ingress controller/gateway gets expossed as a normal kubernetes service load balancer:

Istio-0.7.1
```sh
kubectl get svc istio-ingress -n istio-system -o yaml
```

Istio-0.8.0
```sh
kubectl get svc istio-ingressgateway -n istio-system -o yaml
```

Because the Istio Ingress Controller/Gateway is an Envoy Proxy you can inspect it using the admin routes.  First find the name of the istio ingress proxy:

For Istio 0.7.1:
```sh
kubectl get pods -n istio-system
kubectl -n istio-system exec -it istio-ingress-... bash
```


For Istio 0.8.0:
```sh
kubectl get pods -n istio-system
kubectl -n istio-system exec -it istio-ingressgateway-... bash
```

You can view the statistics, listeners, routes, clusters and server info for the envoy proxy by forwarding the local port:

```sh
curl localhost:15000/help
curl localhost:15000/stats
curl localhost:15000/listeners
curl localhost:15000/routes
curl localhost:15000/clusters
curl localhost:15000/server_info
```

See the [admin docs](https://www.envoyproxy.io/docs/envoy/latest/operations/admin) for more details.

Also it can be helpful to look at the log files of the Istio ingress controller to see what request is being routed.  First find the ingress pod and output the log files:

```sh
kubectl logs istio-ingressgateway-... -n istio-system
```

## View Bookinfo Ingress Routes (Istio 0.7.1)

1 - Routes for Bookinfo app have already been deployed as part of the Bookinfo deployment.

In the bookinfo ingress file notice that the ingress class is specified as   `kubernetes.io/ingress.class: istio` which routes the request to Istio.

```sh
kubectl describe ingress
```

2 - Find the external port of the Istio Ingress controller by running:

```sh
kubectl get service istio-ingress -n istio-system -o wide
```

To just get the first port of istio-ingress service, we can run this:
```sh
kubectl get service istio-ingress -n istio-system --template='{{(index .spec.ports 0).nodePort}}'
```

3 - Browse to the website of the Bookinfo: On `PWK` the exposed ingress ports are available as hyperlinks at the top of the page. Clicking on a valid ingress port will open a page.

To view the product page, you will have to append
`/productpage` to the url.


4 - Now, reload the page multiple times and notice how it round robins between v1, v2 and v3 of the reviews service:


## Configure Bookinfo Ingress Routes with the Istio Ingress Controller (Istio 0.8.0)


1 - Configure the Bookinfo route with the Istio Ingress gateway:

We can create a virtualservice & gateway for bookinfo app in the ingress gateway by running the following:

```sh
istioctl create -f samples/bookinfo/routing/bookinfo-gateway.yaml
```

2 - Viewing the gateway and virtualservices

Check the created gateway and virtualservice:
```sh
istioctl get gateway
istioctl get gateway -o yaml

istioctl get virtualservices
istioctl get virtualservices -o yaml
```

3 - Find the external port of the Istio Ingress controller by running:

```sh
kubectl get service istio-ingressgateway -n istio-system -o wide
```

To just get the first port of istio-ingressgateway service, we can run this:
```sh
kubectl get service istio-ingressgateway -n istio-system --template='{{(index .spec.ports 0).nodePort}}'
```

3 - Browse to the website of the Bookinfo: On `PWK` the exposed ingress ports are available as hyperlinks at the top of the page. Clicking on a valid ingress port will open a page.

To view the product page, you will have to append
`/productpage` to the url.


4 - Now, reload the page multiple times and notice how it round robins between v1, v2 and v3 of the reviews service:


## Inspecting the Istio proxy of the productpage pod

To better understand the istio proxy, let's inspect the details.  exec into the productpage pod to find the proxy details.  First find the full pod name and then exec into the istio-proxy container:

```sh
kubectl get pods
kubectl exec -it productpage-v1-... -c istio-proxy  sh
```

Once in the container look at some of the envoy proxy details:

```sh
ps aux
ls -l /etc/istio/proxy
cat /etc/istio/proxy/envoy-rev0.json
```

You can also view the statistics, listeners, routes, clusters and server info for the envoy proxy by forwarding the local port:

```sh
curl localhost:15000/stats
curl localhost:15000/listeners
curl localhost:15000/routes
curl localhost:15000/clusters
curl localhost:15000/server_info
```

See the [admin docs](https://www.envoyproxy.io/docs/envoy/v1.5.0/operations/admin) for more details.

#### [Continue to lab 5 - Telemetry](../lab-5/README.md)
