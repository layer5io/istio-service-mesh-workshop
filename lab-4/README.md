# Lab 4 - Expose Bookinfo site through Istio Ingress Gateway

The components deployed on the service mesh by default are not exposed outside the cluster. External access to individual services so far has been provided by creating an external load balancer on each service.

An ingress gateway service is deployed as a LoadBalancer service. For making Bookinfo accessible from outside, we have to create an `Istio Gateway` for the service and also define an `Istio VirtualService` for Bookinfo with the routes we need.

## 4.1 Inspecting the Istio Ingress gateway

The ingress gateway gets expossed as a normal kubernetes service load balancer:
```sh
kubectl get svc istio-ingressgateway -n istio-system -o yaml
```

Because the Istio Ingress Gateway is an Envoy Proxy you can inspect it using the admin routes.  First find the name of the istio-ingressgateway:

```sh
kubectl get pods -n istio-system
```
Copy and paste your ingress gateway's pod name. Execute:
```sh
kubectl -n istio-system exec -it <istio-ingressgateway-...> bash
```

You can view the statistics, listeners, routes, clusters and server info for the Envoy proxy by forwarding the local port:

```sh
curl localhost:15000/help
curl localhost:15000/stats
curl localhost:15000/listeners
curl localhost:15000/routes
curl localhost:15000/clusters
curl localhost:15000/server_info
```

See the [admin docs](https://www.envoyproxy.io/docs/envoy/latest/operations/admin) for more details.

Also it can be helpful to look at the log files of the Istio ingress controller to see what request is being routed. We should also be able to view the `curl` calls we just made from inside the ingressgateway. Let us first find the ingress pod and output the log files:


```sh
kubectl logs istio-ingressgateway-... -n istio-system
```

## 4.2 Configure Istio Ingress Gateway for Bookinfo

### 4.2.1 - Configure the Bookinfo route with the Istio Ingress gateway:

We can create a virtualservice & gateway for bookinfo app in the ingress gateway by running the following:

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/bookinfo-gateway.yaml | istioctl create -f - 
```

### 4.2.2 - View the Gateway and VirtualServices

Check the created `Istio Gateway` and `Istio VirtualService` to see the changes deployed:
```sh
istioctl get gateway
istioctl get gateway -o yaml

istioctl get virtualservices
istioctl get virtualservices -o yaml
```

### 4.2.3 - Find the external port of the Istio Ingress Gateway by running:

```sh
kubectl get service istio-ingressgateway -n istio-system -o wide
```

To just get the first port of istio-ingressgateway service, we can run this:
```sh
kubectl get service istio-ingressgateway -n istio-system --template='{{(index .spec.ports 0).nodePort}}'
```

### 4.2.4 - Browse to Bookinfo
Browse to the website of the Bookinfo: On `PWK` the exposed ingress ports are available as hyperlinks at the top of the page. Clicking on a valid ingress port will open a page.

To view the product page, you will have to append
`/productpage` to the url.


### 4.2.5 - Reload Page
Now, reload the page multiple times and notice how it round robins between v1, v2 and v3 of the reviews service.


## 4.3 Inspect the Istio proxy of the productpage pod

To better understand the istio proxy, let's inspect the details.  Let us `exec` into the productpage pod to find the proxy details.  To do so we need to first find the full pod name and then `exec` into the istio-proxy container:

```sh
kubectl get pods
kubectl exec -it productpage-v1-... -c istio-proxy  sh
```

Once in the container look at some of the envoy proxy details by inspecting it's config file:

```sh
ps aux
ls -l /etc/istio/proxy
cat /etc/istio/proxy/envoy-rev0.json
```


For more details on envoy proxy please check out their [admin docs](https://www.envoyproxy.io/docs/envoy/v1.5.0/operations/admin) for more details.

## [Continue to lab 5 - Telemetry](../lab-5/README.md)
