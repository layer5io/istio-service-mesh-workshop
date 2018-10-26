# Lab 7 - Request Routing and Canary Testing

In this lab we are going to get our hands on some of the traffic management capabilities of Istio.

## 7.1 Configure the default route for all services to V1

As part of the bookinfo sample app, there are multiple versions of reviews service. When we load the `/productpage` in the browser multiple times we have seen the reviews service round robin between v1, v2 or v3. As the first exercise, let us first restrict traffic to just V1 of all the services.

Set the default version for all requests to v1 of all service using :

```sh
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/virtual-service-all-v1.yaml 
```

This creates a bunch of `virtualservice` and `destinationrule` entries which route calls to v1 of the services.

To view the applied rule:
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


## 7.2 Content based routing

Let's replace our first rules with a new set. Enable the `ratings` service for your user by routing `productpage` traffic to `reviews` v2.

Let us first grab the yaml file we will be working with:
```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/virtual-service-reviews-test-v2.yaml > user-v2.yaml
```


Now, find the user name of the user you are logged into the Bookinfo app from the browser:
![](img/)

Once you have found out the user name, we can update the `user-v2.yaml` file with the right user name in the placeholder `USER_NAME` we have in place:

```sh
vi user-v2.yaml
```

After you have updated the user name in the right place, let us save the changes and then apply it on the cluster:

```sh
kubectl apply -f user-v2.yaml
```



To view the applied rule:
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
          exact: USER_NAME
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

Now if we login as your user, you will be able to see data from reviews v2. While if you NOT logged in or logged in as a different user, you will see data from `reviews` v1.


## 7.3 Canary Testing - Traffic Shifting

### 7.3.1 Reset rules
Before we start the next exercise, lets first reset the routing rules back to our 7.1 rules:

```sh
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/virtual-service-all-v1.yaml 
```

Once again, all traffic will be routed to `v1` of all the services. 

### 7.3.2 Canary testing w/50% load
To start canary testing, let's begin by transferring 50% of the traffic from reviews:v1 to reviews:v3 with the following command:

```sh
kubectl apply -f  https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/virtual-service-reviews-50-v3.yaml
```

To confirm the rule was applied:
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


### 7.3.3 Shift 100% to v3
When version v3 of the reviews microservice is considered stable, we can route 100% of the traffic to reviews:v3:

```sh
kubectl apply -f https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.2/virtual-service-reviews-v3.yaml
```

To confirm the rule was applied:
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

## [Continue to lab 8 - Fault Injection and Rate Limiting](../lab-8/README.md)
