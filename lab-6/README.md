# Lab 6 - Fault Injection

In this lab we will learn how to test the resiliency of an application by injecting systematic faults.

Before we start let us reset the route rules:

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
```

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

## 6.1 Inject a route rule to create a fault using HTTP delay

To start, we will inject a 7s delay between the reviews v2 and ratings service for a user `jason`. reviews v2 service has a 10s hard-coded connection timeout for its calls to the ratings service configured globally.

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
```


To confirm the rule is in place:
```sh
kubectl get virtualservice ratings -o yaml
```

Output will be similar to:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"hosts":["ratings"],"http":[{"fault":{"delay":{"fixedDelay":"7s","percent":100}},"match":[{"headers":{"end-user":{"exact":"USER_NAME"}}}],"route":[{"destination":{"host":"ratings","subset":"v1"}}]},{"route":[{"destination":{"host":"ratings","subset":"v1"}}]}]}}
  creationTimestamp: 2018-10-26T15:21:42Z
  generation: 1
  name: ratings
  namespace: default
  resourceVersion: "14790"
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/ratings
  uid: d7d7347f-d932-11e8-88c5-0242f983c5dd
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
        end-user:
          exact: jason
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

Now we login to `/productpage` as your user and observe that the page loads but because of the induced delay between services the reviews section will show :

<pre>
        <b>Error fetching product reviews!</b>

Sorry, product reviews are currently unavailable for this book.
</pre>

If you logout or login as a different user, the page should load normally without any errors.

## 6.2 Inject a route rule to create a fault using HTTP abort

In this section, , we will introduce an HTTP abort to the ratings microservices for your user.

Now apply the change to the cluster:
```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-abort.yaml
```


To confirm the rule is in place:
```sh
kubectl get virtualservice ratings -o yaml
```

Output will be similar to:
```yaml
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"hosts":["ratings"],"http":[{"fault":{"abort":{"httpStatus":500,"percent":100}},"match":[{"headers":{"end-user":{"exact":"USER_NAME"}}}],"route":[{"destination":{"host":"ratings","subset":"v1"}}]},{"route":[{"destination":{"host":"ratings","subset":"v1"}}]}]}}
  creationTimestamp: 2018-10-26T15:21:42Z
  generation: 1
  name: ratings
  namespace: default
  resourceVersion: "22405"
  selfLink: /apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/ratings
  uid: d7d7347f-d932-11e8-88c5-0242f983c5dd
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
        end-user:
          exact: jason
    route:
    - destination:
        host: ratings
        subset: v1
  - route:
    - destination:
        host: ratings
        subset: v1
```

Now we login to `/productpage` as your user and observe that the page loads without any new delays but because of the induced fault between services the reviews section will show:

 `Ratings service is currently unavailable`.

### 6.3 Verify fault injection
Verify the fault injection rule by logging out (or logging in as a different user), the page should load normally without any errors.


## [Continue to Lab 7 - Circuit Breaking](../lab-7/README.md)
