# Lab 8 - Circuit Breaking

In this lab we will configure circuit breaking using Istio. Circuit breaking allows developers to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities. This task will show how to configure circuit breaking for connections, requests, and outlier detection.

## 8.1 Preparing for circuit breaking
Before we can configure circuit breaking, please try to access the `product page` app from within `Meshery` to ensure all the calls are making it through **without** errors.

![](img/meshery_initial_load_test.png)


## 8.2 Configure circuit breaking
Now that we have the needed services in place, it is time to configure circuit breaking using a destination rule:

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


## 8.3 Time to trip the circuit
In the circuit-breaker settings, we specified maxRequestsPerConnection: 1 and http1MaxPendingRequests: 1. This should mean that if we exceed more than one request per connection and more than one pending request, we should see the istio-proxy sidecar open the circuit for further requests/connections. 

Let us now use `Meshery` to make several calls to the `productpage` app using 5 concurrent connections from within `Meshery`.

![](img/meshery_cb_load_test.png)

As seen only a percentage of the requests succeeded and the rest were trapped by circuit breaker.

## [Continue to Lab 9 - Mutual TLS & Identity Verification](../lab-9/README.md)