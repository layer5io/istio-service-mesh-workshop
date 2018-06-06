# lab 9 - Circuit Breaking

In this lab we will configure circuit breaking using Istio.

## Configure circuit breaking
Let us start by deploying a simpler application to test circuit breaking:
```sh
kubectl apply -f <(istioctl kube-inject --debug -f samples/httpbin/httpbin.yaml)
```

Let us then deploy a client which is capable of talking to the httpbin service:
```sh
kubectl apply -f <(istioctl kube-inject --debug -f samples/httpbin/sample-client/fortio-deploy.yaml)
```

It is time to configure circuit breaking using a destination rule:
```sh
cat <<EOF | istioctl create -f -
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      http:
        consecutiveErrors: 1
        interval: 1s
        baseEjectionTime: 3m
        maxEjectionPercent: 100
EOF
```

To confirm the rule is in place:
```sh
istioctl get destinationrule httpbin -o yaml
```

Output:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: httpbin
  ...
spec:
  host: httpbin
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
      tcp:
        maxConnections: 100
    outlierDetection:
      http:
        baseEjectionTime: 180.000s
        consecutiveErrors: 1
        interval: 1.000s
        maxEjectionPercent: 100
```

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

## Time to trip the circuit
In the circuit-breaking settings, we specified maxRequestsPerConnection: 1 and http1MaxPendingRequests: 1. This should mean that if we exceed more than one request per connection and more than one pending request, we should see the istio-proxy sidecar open the circuit for further requests/connections. 

Let us make 50 calls using 4 concurrent connections:
```sh
load -c 4 -qps 0 -n 50 -loglevel Warning http://httpbin:8000/get
```

To view the results of the test we can talk to the istio-proxy sidecar for some stats:
```sh
kubectl exec -it $(kubectl get pod | grep fortio | awk '{ print $1 }')  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```

The stats will give you the exact number of requests which overflew.

#### [Continue to lab 10 - Istio Mutual TLS](../lab-10/README.md)
