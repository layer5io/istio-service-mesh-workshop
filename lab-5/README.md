## lab 5 - Telemetry

#### Generate Bookinfo Telemetry data

Let us get the first accessible ingress port and store it in a variable:
```sh
export INGRESS_PORT=$(kubectl get service istio-ingressgateway -n istio-system --template='{{(index .spec.ports 0).nodePort}}')
```

Once we have the port, we can either append `localhost` or IP of one of the nodes to get the host:
```sh
export INGRESS_HOST="localhost:$INGRESS_PORT"
```

Now, let us generate a small load on the sample app by using [fortio](https://github.com/istio/fortio) which is a load testing library created by the `Istio` team:

The command below will run load test by making 5 calls per second for 5 minutes:
```sh
docker run istio/fortio load -t 5m -qps 5 http://$INGRESS_HOST/productpage
```

### Grafana

Establish port forward from local port 3000 to the Grafana instance:
```sh
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana \
  -o jsonpath='{.items[0].metadata.name}') 3000:3000
```

If you are in Cloud Shell, you'll need to use Web Preview and Change Port to `3000`.

Browse to http://localhost:3000 and navigate to Istio Dashboard

### Prometheus
```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') \
  9090:9090
```

If you are in Cloud Shell, you'll need to use Web Preview and Change Port to `9090`.  

Browse to http://localhost:9090/graph and in the “Expression” input box enter: `istio_request_count`. Click the Execute button.

### Service Graph

```sh
kubectl -n istio-system port-forward \
  $(kubectl -n istio-system get pod -l app=servicegraph -o jsonpath='{.items[0].metadata.name}') \
  8088:8088
```

If you are in Cloud Shell, you'll need to use Web Preview and Change Port to `9090`. Once opened, you'll see `404 not found` error. This is normal because `/` is not handled. Append the URI with `/dotviz`, e.g.: `http://8088-dot-...-dot-devshell.appspot.com/dotviz`

Browse to http://localhost:8088/dotviz

#### [Continue to lab 6 - Distributed Tracing](../lab-6/README.md)
