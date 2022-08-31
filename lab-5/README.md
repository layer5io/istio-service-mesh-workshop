# Lab 5 - Request Routing and Canary Testing

In this lab, we are going to get our hands on some of the traffic management capabilities of Istio.

## 5.1 Configure the default route for all services to V1

As part of the bookinfo sample app, there are multiple versions of reviews service. When we load the `/productpage` in the browser multiple times we have seen the reviews service round robin between v1, v2 or v3. As the first exercise, let us first restrict traffic to just V1 of all the services.

Using Meshery, navigate to the designs page under configuration and import the below design. Make sure Istio adapter is running.
**Patternfile**:
```yaml
name: Lab5
services:
  vs:
    settings:
      hosts:
        - reviews
      http:
        - route:
            - destination:
                host: reviews
                subset: v1
    type: VirtualService.Istio
    name: vs
    namespace: default

---
```
<!-- 
```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml 
``` -->

To view the applied rule:
```sh
kubectl get virtualservice
```

To take a look at a specific one:
```sh
kubectl get virtualservice reviews -o yaml
```

*Please note:* In the place of the above command, we can either use kubectl or istioctl.



Now when we reload the `/productpage` several times, we will ONLY be viewing the data from v1 of all the services, which means we will not see any ratings (any stars).


## 5.3 Content-based routing

Let's replace our first rules with a new set. Enable the `ratings` service for a user `jason` by routing `productpage` traffic to `reviews` v2:

```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
``` 
Using Meshery, navigate to the Istio management page:


Using Meshery, navigate to the designs page under configuration and import the below design. Make sure Istio adapter is running.
**Patternfile**:
```yaml
name: V2ForJason
services:
  vs:
    settings:
      hosts:
        - reviews
      http:
        - corsPolicy: {}
          match:
            - headers:
                end-user:
                  exact: jason
          route:
            - destination:
                host: reviews
                subset: v2
        - corsPolicy: {}
          route:
            - destination:
                host: reviews
                subset: v1
    type: VirtualService.Istio
    name: vs
    namespace: default
---
```
<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for reviews to route all traffic for user `jason` to review V2.

In a few, we should be able to verify the virtual service by using the command below:
```sh
kubectl get virtualservice reviews -o yaml
```



Now if we login as your `jason`, you will be able to see data from `reviews` v2. While if you NOT logged in or logged in as a different user, you will see data from `reviews` v1.


## 5.4 Canary Testing - Traffic Shifting

### 5.4.1 Canary testing w/50% load
To start canary testing, let's begin by transferring 50% of the traffic from reviews:v1 to reviews:v3 with the following command:

<!-- ```sh
kubectl apply -f  samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
``` -->


Using Meshery, navigate to the designs page under configuration and import the below design. Make sure Istio adapter is running.
**Patternfile**:
```yaml
name: CanaryV1V3
services:
  vs:
    settings:
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
    type: VirtualService.Istio
    name: vs
    namespace: default

---
```
<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for reviews to route 50% of all traffic to review V3.

In a few, we should be able to verify the virtual service by using the command below:
```sh
kubectl get virtualservice reviews -o yaml
```


Now, if we reload the `/productpage` in your browser several times, you should now see red-colored star ratings approximately 50% of the time.


### 5.4.2 Shift 100% to v3
When version v3 of the reviews microservice is considered stable, we can route 100% of the traffic to reviews:v3:

<!-- ```sh
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
``` -->


Using Meshery, navigate to the designs page under configuration and import the below design. Make sure Istio adapter is running.
**Patternfile**:
```yaml
name: ShiftAllTrafficToV3
services:
  vs:
    settings:
      hosts:
      - reviews
      http:
      - route:
        - destination:
            host: reviews
            subset: v3
    type: VirtualService.Istio
    name: vs
    namespace: default

---
<small>Manual step for can be found [here](#appendix)</small>

This will update the existing virtual service definition for reviews to route 100% of all traffic to review V3.

In a few, we should be able to verify the virtual service by using the command below:

```sh
kubectl get virtualservice reviews -o yaml
```



Now, if we reload the `/productpage` in your browser several times, you should now see red-colored star ratings 100% of the time.


## [Continue to lab 6 - Fault Injection and Rate Limiting](../lab-6/README.md)

<hr />
Alternative, manual installation steps below. No need to execute, if you have performed the steps above.
<hr />

## <a name="appendix"></a> Appendix - Manual Instructions

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
