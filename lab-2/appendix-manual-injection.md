### <a name="manual"></a> Lab 3 Appendix: Deploying Sample App with manual sidecar injection

To do a manual sidecar injection we will be using `istioctl` command:

```sh
curl https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/platform/kube/bookinfo.yaml | istioctl kube-inject -f - > newBookInfo.yaml
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

```sh
kubectl apply -f <(curl https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/platform/kube/bookinfo.yaml | istioctl kube-inject -f -)
```

Now continue to [Verify Bookinfo deployment](README.md#verify).