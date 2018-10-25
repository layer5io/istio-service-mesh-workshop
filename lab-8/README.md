# Lab 8 - Fault Injection and Rate-Limiting

In this lab we will learn how to test the resiliency of an application by injecting systematic faults.

Before we start let us reset the route rules:
```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/route-rule-all-v1.yaml | istioctl replace -f - 
 
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/route-rule-reviews-test-v2.yaml | istioctl replace -f - 
```

## 8.1 Inject a route rule to create a fault using HTTP delay

To start, we will inject a 7s delay between the reviews v2 and ratings service for user `jason`. reviews v2 service has a 10s hard-coded connection timeout for its calls to the ratings service.

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/route-rule-ratings-test-delay.yaml | istioctl replace -f - 
```


To confirm the rule is in place:
```sh
istioctl get virtualservice ratings -o yaml
```
Output:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
  ...
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        fixedDelay: 7s
        percent: 100
    match:
    - headers:
        cookie:
          regex: ^(.*?;)?(user=jason)(;.*)?$
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

Now we login to `/productpage` as user `jason` and observe that the page loads but because of the induced delay between services the reviews section will show :

<pre>
        <b>Error fetching product reviews!</b>

Sorry, product reviews are currently unavailable for this book.
</pre>

If you logout or login as a different user, the page should load normally without any errors.

## 8.2 Inject a route rule to create a fault using HTTP abort

In this section, , we will introduce an HTTP abort to the ratings microservices for the user `jason`.

```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/route-rule-ratings-test-abort.yaml | istioctl replace -f - 
```

To confirm the rule is in place:
```sh
istioctl get virtualservice ratings -o yaml
```
Output:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
  ...
spec:
  hosts:
  - ratings
  http:
  - fault:
      abort:
        httpStatus: 500
        percent: 100
    match:
    - headers:
        cookie:
          regex: ^(.*?;)?(user=jason)(;.*)?$
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

Now we login to `/productpage` as user `jason` and observe that the page loads without any new delays but because of the induced fault between services the reviews section will show:

 `Ratings service is currently unavailable`.

### 8.3 Verify fault injection
Verify the fault injection rule by logging out (or logging in as a different user), the page should load normally without any errors.


## [Continue to Lab 9 - Mutual TLS & Identity Verification](../lab-9/README.md)
