### <a name="manual"></a> Lab 3 Appendix: Deploying Sample App with manual sidecar injection

To do a manual sidecar injection we will be using `istioctl` command:

```sh
istioctl kube-inject -f samples/bookinfo/platform/kube/bookinfo.yaml > newBookInfo.yaml
```

Observing the new yaml file reveals that additional container Istio Proxy has been added to the Pods with necessary configurations:

```
        image: docker.io/istio/proxyv2:1.3.0
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
kubectl apply -f <(istioctl kube-inject --debug -f samples/bookinfo/platform/kube/bookinfo.yaml)
```

Now continue to [Verify Bookinfo deployment](README.md#verify).
