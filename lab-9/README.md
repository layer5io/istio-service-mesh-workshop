# Lab 9 - Circuit Breaking

In this lab we will configure circuit breaking using Istio. Circuit breaking allows developers to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities. This task will show how to configure circuit breaking for connections, requests, and outlier detection.

## 9.1 Preparing for circuit breaking


### 9.1.1 <a name="#deploy"></a> Deploy a simple application
Now, let us start by deploying a simpler application to test circuit breaking:


***With automatic sidecar injector:***

```sh
kubectl apply -f samples/httpbin/httpbin.yaml
```

***With manual sidecar injection:***

```sh
kubectl apply -f <(istioctl kube-inject --debug -f samples/httpbin/httpbin.yaml)
```


### 9.1.2 Deploy a client for the app
Let us then deploy a client which is capable of talking to the httpbin service:

***With automatic sidecar injector:***

```sh
kubectl apply -f samples/httpbin/sample-client/fortio-deploy.yaml
```

***With manual sidecar injection:***

```sh
kubectl apply -f <(istioctl kube-inject --debug -f samples/httpbin/sample-client/fortio-deploy.yaml)
```

Before we proceed further, lets ensure the services are up and running:

```sh
kubectl get deployments

kubectl get po
```


### 9.1.3 Initial test calls from client to server
To make testing easier, let us create an alias to execute `load` inside the httpbin client we created above:
```sh
alias load="kubectl exec -it $(kubectl get pod | grep fortio | awk '{ print $1 }') -c fortio /usr/local/bin/fortio -- load"
```

Let us make a call to the httpbin service from the client using the `alias` we just created:
```sh
load -curl  http://httpbin:8000/get
```

You should see an output similar to this:
```sh
HTTP/1.1 200 OK
server: envoy
date: Tue, 16 Jan 2018 23:47:00 GMT
content-type: application/json
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 445
x-envoy-upstream-service-time: 36

{
  "args": {},
  "headers": {
    "Content-Length": "0",
    "Host": "httpbin:8000",
    "User-Agent": "istio/fortio-0.6.2",
    "X-B3-Sampled": "1",
    "X-B3-Spanid": "824fbd828d809bf4",
    "X-B3-Traceid": "824fbd828d809bf4",
    "X-Ot-Span-Context": "824fbd828d809bf4;824fbd828d809bf4;0000000000000000",
    "X-Request-Id": "1ad2de20-806e-9622-949a-bd1d9735a3f4"
  },
  "origin": "127.0.0.1",
  "url": "http://httpbin:8000/get"
}
```

### 9.1.4 Configure Circuit Breaking
Now that we have the needed service in place, it is time to configure circuit breaking using a destination rule:

```sh
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
EOF
```

To confirm the rule is in place:
```sh
kubectl get destinationrule httpbin -o yaml
```

Output will be similar to:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"httpbin","namespace":"default"},"spec":{"host":"httpbin","trafficPolicy":{"connectionPool":{"http":{"http1MaxPendingRequests":1,"maxRequestsPerConnection":1},"tcp":{"maxConnections":1}},"outlierDetection":{"baseEjectionTime":"3m","consecutiveErrors":1,"interval":"1s","maxEjectionPercent":100}}}}
  creationTimestamp: 2018-10-26T17:14:00Z
  generation: 1
  name: httpbin
  namespace: default
  resourceVersion: "27074"
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/httpbin
  uid: 87be07db-d942-11e8-88c5-0242f983c5dd
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
      tcp:
        maxConnections: 1
    outlierDetection:
      baseEjectionTime: 3m
      consecutiveErrors: 1
      interval: 1s
      maxEjectionPercent: 100
```


## 9.2 Time to trip the circuit
In the circuit-breaking settings, we specified maxRequestsPerConnection: 1 and http1MaxPendingRequests: 1. This should mean that if we exceed more than one request per connection and more than one pending request, we should see the istio-proxy sidecar open the circuit for further requests/connections. 

Let us make 50 calls using 4 concurrent connections:
```sh
load -c 4 -qps 0 -n 50 -loglevel Warning http://httpbin:8000/get
```

In the output you will see a section similar to this one:
```sh
Code 200 : 13 (26.0 %)
Code 503 : 37 (74.0 %)
```
As seen only a percentage of the requests succeeded and the rest were trapped by circuit breaking.

To verify the results of the test we can also talk to the istio-proxy sidecar for some stats:
```sh
kubectl exec -it $(kubectl get pod | grep fortio | awk '{ print $1 }')  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```
Output will be similar to this:
```sh
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_active: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_failure_eject: 0
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_overflow: 37
cluster.outbound|8000||httpbin.default.svc.cluster.local.upstream_rq_pending_total: 13
```
You can see the numbers we got from sidecar and fortio match. `upstream_rq_pending_overflow` specifies the number of calls flagged for circuit breaking.

## [Continue to Lab 10 - Mutual TLS & Identity Verification](../lab-10/README.md)

---

# Layer5.io
For future updates and additional resources, check out [layer5.io](https://layer5.io).
