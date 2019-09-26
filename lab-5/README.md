# Lab 5 - Request Routing and Canary Testing

In this lab we are going to get our hands on some of the traffic management capabilities of Istio.

## 5.1 Apply default destination rules

Before we start playing with Istio's traffic management capabilities we need to define the available versions of the deployed services. They are called subsets, in destination rules.

<!-- Run the following command to create default destination rules for the Bookinfo services:
```sh
kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
``` -->

Now in Meshery in the browser, navigate to the Istio adapter's management page from the left nav menu again.

On the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Configure` card and select `Default Book info destination rules (defines subsets)` from the list. 

<small>Manual step for can be found [here](#appendix)</small>

This will deploy the destination rules for all the Book info services defining their subsets.

In a few seconds we should be able to verify the destination rules created by using the command below:

```sh
kubectl get destinationrules


kubectl get destinationrules -o yaml
```

## 5.2 Configure the default route for all services to V1

As part of the bookinfo sample app, there are multiple versions of reviews service. When we load the `/productpage` in the browser multiple times we have seen the reviews service round robin between v1, v2 or v3. As the first exercise, let us first restrict traffic to just V1 of all the services.

In Meshery on the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Configure` card and select `Route traffic to V1 of all Book info services` from the list. 

<small>Manual step for can be found [here](#appendix)</small>

<!-- 
```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
``` -->

In a few, this will creates a bunch of `virtualservice` entries which will route all requests to ONLY V1 of the services.

To view the applied rule:
```sh
kubectl get virtualservice
```

To take a look at a specific one:
```sh
kubectl get virtualservice reviews -o yaml
```

*Please note:* In the place of the above command, we can either use kubectl or istioctl.


Output:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"}}]}]}}
  creationTimestamp: null
  name: reviews
  namespace: default
  resourceVersion: "11595"
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
---
```

Now when we reload the `/productpage` several times, we will ONLY be viewing the data from v1 of all the services, which means we will not see any ratings (any stars).


## 5.3 Content-based routing

Let's replace our first rules with a new set. Enable the `ratings` service for a user `jason` by routing `productpage` traffic to `reviews` v2:

<!-- ```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
``` -->
In Meshery on the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Configure` card and select `Route traffic to V2 of Book info reviews service for user Jason` from the list. 

<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for reviews to route all traffic for user `jason` to review V2.

In a few, we should be able to verify the virtual service by using the command below:
```sh
kubectl get virtualservice reviews -o yaml
```

Output will be similar to the one below:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"match":[{"headers":{"end-user":{"exact":"USER_NAME"}}}],"route":[{"destination":{"host":"reviews","subset":"v2"}}]},{"route":[{"destination":{"host":"reviews","subset":"v1"}}]}]}}
  creationTimestamp: null
  name: reviews
  namespace: default
  resourceVersion: "10366"
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
---
```

Now if we login as your `jason`, you will be able to see data from `reviews` v2. While if you NOT logged in or logged in as a different user, you will see data from `reviews` v1.


## 5.4 Canary Testing - Traffic Shifting

### 5.4.1 Reset rules
Before we start the next exercise, lets first reset the routing rules back to our 5.1 rules:

<!-- ```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
``` -->

In Meshery on the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Configure` card and select `Route traffic to V1 of all Book info services` from the list. 

<small>Manual step for can be found [here](#appendix)</small>

Once again, all traffic will be routed to `v1` of all the services. 

### 5.4.2 Canary testing w/50% load
To start canary testing, let's begin by transferring 50% of the traffic from reviews:v1 to reviews:v3 with the following command:

<!-- ```sh
kubectl apply -f  samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
``` -->

In Meshery on the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Configure` card and select `Route 50% of the traffic to Book info reviews V3` from the list. 

<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for reviews to route 50% of all traffic to review V3.

In a few, we should be able to verify the virtual service by using the command below:
```sh
kubectl get virtualservice reviews -o yaml
```

Output will be similar to:
```yaml
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["r
eviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":50},{"destination":{"host":"reviews","subset":"v3"},"weight":50}]}]}}
  creationTimestamp: null
  name: reviews
  namespace: default
  resourceVersion: "11904"
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
---
```

Now, if we reload the `/productpage` in your browser several times, you should now see red-colored star ratings approximately 50% of the time.


### 5.4.3 Shift 100% to v3
When version v3 of the reviews microservice is considered stable, we can route 100% of the traffic to reviews:v3:

<!-- ```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
``` -->

In Meshery on the Istio adapter's management page, please enter `default` in the `Namespace` field.
Then, click the (+) icon on the `Configure` card and select `Route 100% of the traffic to Book info reviews V3` from the list. 

<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for reviews to route 100% of all traffic to review V3.

In a few, we should be able to verify the virtual service by using the command below:

```sh
kubectl get virtualservice reviews -o yaml
```
Output:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["r
eviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v3"}}]}]}}
  creationTimestamp: null
  name: reviews
  namespace: default
  resourceVersion: "12157"
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v3
---
```

Now, if we reload the `/productpage` in your browser several times, you should now see red-colored star ratings 100% of the time.

## <a name="appendix"></a> Appendix

### Default destination rules
Run the following command to create default destination rules for the Bookinfo services:
```sh
kubectl apply -f samples/bookinfo/networking/destination-rule-all-mtls.yaml
```

### Route all traffic to version V1 of all services

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
```

### Route all traffic to version V2 of reviews for user Jason

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
```

### Route 50% of traffic to version V3 of reviews service

```sh
kubectl apply -f  samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
```

### Route 100% of traffic to version V3 of reviews service

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
```


## [Continue to lab 6 - Fault Injection and Rate Limiting](../lab-6/README.md)
