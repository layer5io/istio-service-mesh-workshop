# Lab 10 - Circuit Breaking

In this lab we will configure circuit breaking using Istio. Circuit breaking allows developers to write applications that limit the impact of failures, latency spikes, and other undesirable effects of network peculiarities. This task will show how to configure circuit breaking for connections, requests, and outlier detection.

## 10.1 Preparing for circuit breaking

Based on our testing circuit breaker configuration in Istio 1.0.2 has issues with mTLS turned on.

### 10.1.1 Turn off mTLS
Based on our testing circuit breaker configuration in Istio has issues with mTLS turned on.

Let us turn off mTLS now. To turn off mTLS we will have to edit the configmap:
```sh
kubectl edit configmap -n istio-system istio
```

Please **comment** out 2 lines as below:
```sh
    # authPolicy: MUTUAL_TLS
    ...
    # controlPlaneAuthPolicy: MUTUAL_TLS
```

Now save the file and exit the editor.

Then, lets kill istio-pilot pods allowing it to reload:
```sh
kubectl delete pods -n istio-system -l istio=pilot
```

Please give it a few minutes to restart and get back to running state.


### 10.1.2 <a name="#deploy"></a> Deploy a simple application
Now, let us start by deploying a simpler application to test circuit breaking:


***With manual sidecar injection:***

```sh
kubectl apply -f <(curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/httpbin.yaml | istioctl kube-inject --debug -f -)
```

***With automatic sidecar injector:***

```sh
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/httpbin.yaml
```


### 10.1.3 Deploy a client for the app
Let us then deploy a client which is capable of talking to the httpbin service:

***With manual sidecar injection:***

```sh
kubectl apply -f <(curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/fortio-deploy.yaml | istioctl kube-inject --debug -f -)
```

***With automatic sidecar injector:***

```sh
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/fortio-deploy.yaml
```


### 10.1.4 Initial test calls from client to server
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

### 10.1.5 Configure Circuit Breaking
Now that we have the needed service in place, it is time to configure circuit breaking using a destination rule:

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/circuit-breaking.yaml | istioctl create -f - 
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


## 10.2 Time to trip the circuit
In the circuit-breaking settings, we specified maxRequestsPerConnection: 1 and http1MaxPendingRequests: 1. This should mean that if we exceed more than one request per connection and more than one pending request, we should see the istio-proxy sidecar open the circuit for further requests/connections. 

Let us make 50 calls using 4 concurrent connections:
```sh
load -c 4 -qps 0 -n 50 -loglevel Warning http://httpbin:8000/get
```

To view the results of the test we can talk to the istio-proxy sidecar for some stats:
```sh
kubectl exec -it $(kubectl get pod | grep fortio | awk '{ print $1 }')  -c istio-proxy  -- sh -c 'curl localhost:15000/stats' | grep httpbin | grep pending
```


This wraps up this lab and the workshop. Thank you for attending our workshop!

---

# Layer5.io
For future updates and additional resources, see [layer5.io](http://layer5.io).
