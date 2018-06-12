# Lab 6 - Distributed Tracing

The sample Bookinfo application is configured to collect trace spans using Zipkin or Jaeger. Although Istio proxies are able to automatically send spans, it needs help from the application to tie together the entire trace. To do this applications need to propagate the appropriate HTTP headers so that when the proxies send span information to Zipkin or Jaeger, the spans can be correlated correctly into a single trace.

To do this the application collects and propagates the following headers from the incoming request to any outgoing requests:

- `x-request-id`
- `x-b3-traceid`
- `x-b3-spanid`
- `x-b3-parentspanid`
- `x-b3-sampled`
- `x-b3-flags`
- `x-ot-span-context`


## View Traces

If you have not set `INGRESS_HOST` environment variable, please do so by following [Lab 5](../lab-5/README.md).

Now, let us generate a small load on the sample app by using [fortio](https://github.com/istio/fortio):

```sh
docker run istio/fortio load -t 5m -qps 5 http://$INGRESS_HOST/productpage
```

Let us find the port Jaeger is exposed on by running the following command:
```sh
kubectl -n istio-system get svc tracing
```

You can click on the link at the top of the page which maps to the right port and it will open Jaeger UI in a new tab. 

![](img/jaeger.png)

![](img/jaeger_1.png)

![](img/jaeger_2.png)



#### [Continue to lab 7 - Request Routing and Canary Testing](../lab-7/README.md)
