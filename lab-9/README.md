# Lab 9 - Circuit Breaking

In this lab we will configure circuit breaking using Istio. Circuit breaking allows developers to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities. This task will show how to configure circuit breaking for connections, requests, and outlier detection.

## 9.1 Preparing for circuit breaking
Before we can configure circuit breaking, please try to access the `product page` app from within `Meshery` to ensure all the calls are making it through **without** errors.

![](img/meshery_initial_load_test.png)


### 9.1.3 Initial test calls from client to server
Now that we have the needed service in place, it is time to configure circuit breaking using a destination rule:

```sh
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - labels:
      version: v1
    name: v1
  trafficPolicy:
    tls:
      mode: ISTIO_MUTUAL
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
kubectl get destinationrule productpage -o yaml
```

Output will be similar to:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"DestinationRule","metadata":{"annotations":{},"name":"productpage","namespace":"bookinfo"},"spec":{"host":"productpage","subsets":[{"labels":{"version":"v1"},"name":"v1"}],"trafficPolicy":{"connectionPool":{"http":{"http1MaxPendingRequests":1,"maxRequestsPerConnection":1},"tcp":{"maxConnections":1}},"outlierDetection":{"baseEjectionTime":"3m","consecutiveErrors":1,"interval":"1s","maxEjectionPercent":100},"tls":{"mode":"ISTIO_MUTUAL"}}}}
  creationTimestamp: "2019-06-05T13:44:17Z"
  generation: 4
  name: productpage
  namespace: bookinfo
  resourceVersion: "2094075"
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/bookinfo/destinationrules/productpage
  uid: 037a4fae-8798-11e9-aa76-00505698648f
spec:
  host: productpage
  subsets:
  - labels:
      version: v1
    name: v1
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
    tls:
      mode: ISTIO_MUTUAL
```


## 9.2 Time to trip the circuit
In the circuit-breaking settings, we specified maxRequestsPerConnection: 1 and http1MaxPendingRequests: 1. This should mean that if we exceed more than one request per connection and more than one pending request, we should see the istio-proxy sidecar open the circuit for further requests/connections. 

Let us make several calls using 5 concurrent connections from within `Meshery`

In the output you will see a section similar to this one:

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
