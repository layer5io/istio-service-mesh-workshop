### <a name="manual"></a> Lab 3 Appendix: Deploying Sample App with manual sidecar injection

To do a manual sidecar injection we will be using `istioctl` command:

With twitter auth:
```sh
curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.4/bookinfo-twitter-auth.yaml | istioctl kube-inject --debug -f - > newBookInfo.yaml
```

Without twitter auth:
```sh
kubectl apply -f sample/bookinfo/platforms/kube/bookinfo.yaml | istioctl kube-inject --debug -f
```

Observing the new yaml file reveals that additional container Istio Proxy has been added to the Pods with necessary configurations:

```
        image: docker.io/istio/proxyv2:1.1.7
        imagePullPolicy: IfNotPresent
        name: istio-proxy
```

We need to now deploy the new yaml using `kubectl`
```sh
kubectl apply -f newBookInfo.yaml
```

To do both in a single command:


Without twitter auth:
```sh
kubectl apply -f <(curl https://raw.githubusercontent.com/leecalcote/istio-service-mesh-workshop/master/deployment_files/istio-1.0.4/bookinfo.yaml | istioctl kube-inject --debug -f -)
```

Now continue to [verify sample app deployment](#verify).
