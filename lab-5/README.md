# lab 5 - Telemetry


## Inspecting Mixer

Envoy proxies call Mixer to report statistics and check for route rules. We can get an idea on what it is collecting:

```sh
kubectl get pods -n istio-system
```

Istio 0.7.1:
```sh
kubectl -n istio-system exec -it istio-mixer-... -c istio-proxy -- sh
```

Istio 0.8.0:
```sh
kubectl -n istio-system exec -it istio-policy-... -c istio-proxy -- sh
```

```sh
curl localhost:9093/metrics

exit
```

## Generate Bookinfo Telemetry data

Let us get the first accessible ingress port and store it in a variable:

Istio 0.7.1:
```sh
export INGRESS_PORT=$(kubectl get service istio-ingress -n istio-system --template='{{(index .spec.ports 0).nodePort}}')
```

Istio 0.8.0:
```sh
export INGRESS_PORT=$(kubectl get service istio-ingressgateway -n istio-system --template='{{(index .spec.ports 0).nodePort}}')
```

Once we have the port, we can append the IP of one of the nodes to get the host. in `PWK` we can get the ip from the host list on the left.
```sh
export INGRESS_HOST="<IP>:$INGRESS_PORT"
```

***Please note:*** If using Docker for Mac or Windows for Istio, INGRESS_HOST should be set to `localhost:80`

Now, let us generate a small load on the sample app by using [fortio](https://github.com/istio/fortio) which is a load testing library created by the `Istio` team:

The command below will run load test by making 5 calls per second for 5 minutes:
```sh
docker run istio/fortio load -t 5m -qps 5 http://$INGRESS_HOST/productpage
```

## Appoptics
If you had followed [optional lab-2](../lab-2/optional.md), created or have an Appoptics account, created a dashboard, obtained a valid Appoptics API token and deployed Istio with [solarwinds mixer adapter](https://github.com/solarwinds/istio-adapter), you will be able to view the metrics data from Istio in the Appoptics Dashboard.

![](img/AO_Dashboard.png)

## Loggly
If you had followed [optional lab-2](../lab-2/optional.md), created or have an Loggly account, obtained a valid Loggly API token and deployed Istio with [solarwinds mixer adapter](https://github.com/solarwinds/istio-adapter), you will be able to view the access logs from Istio in Loggly.

![](img/Loggly.png)


## Grafana

If you have not already deployed and exposed grafana, please follow [lab-2](../lab-2/README.md). 
In `PWK`, once you have exposed grafana on a port by using any of the specified methods, it will appear at the top of the page as a hyperlink. You can click on the link at the top of the page which maps to the right port and it will open grafana in new tab. You can then navigate to the `Istio Dashboard`.

![](img/Grafana_Istio_Dashboard.png)

## Prometheus
If you have not already deployed and exposed prometheus, please follow [lab-2](../lab-2/README.md). 
In `PWK`, once you have exposed prometheus on a port by using any of the specified methods, it will appear at the top of the page as a hyperlink. You can click on the link at the top of the page which maps to the right port and it will open prometheus in new tab. 
Browse to `/graph` and in the `Expression` input box enter: `istio_request_count`. Click the Execute button.
![](img/Prometheus.png)

## Service Graph

If you have not already deployed and exposed servicegraph, please follow [lab-2](../lab-2/README.md). 
In `PWK`, once you have exposed servicegraph on a port by using any of the specified methods, it will appear at the top of the page as a hyperlink. You can click on the link at the top of the page which maps to the right port and it will open a new tab but will show an error page with `404 not found`. 
Update the URI to `/dotviz` and you will see the generated service graph.
![](img/servicegraph.png)


#### [Continue to lab 6 - Distributed Tracing](../lab-6/README.md)
