# Lab 9 - Mutual TLS & Identity Verification

Istio provides transparent mutal TLS to services inside the service mesh where both the client and the server authenticate each others certificates as part of the TLS handshake. As part of this workshop we have deployed Istio with mTLS.

## 9.1 Verify mTLS
To verify mTLS is enabled:
```sh
kubectl get configmap istio -o yaml -n istio-system | grep authPolicy | head -1
```

If it is enabled you will see output similar to the below from above command:
```sh
    authPolicy: MUTUAL_TLS
```

To experiment with mTLS, let us get into the sidecar proxy of productpage page by running the command below:
```sh
kubectl exec -it $(kubectl get pod | grep productpage | awk '{ print $1 }') -c istio-proxy -- /bin/bash
```

We are now in the sidecar of the productpage pod. Let us check all the ceritificates loaded in the sidecar:
```sh
ls /etc/certs/
```

You should see 3 entries:
```sh
cert-chain.pem  key.pem  root-cert.pem
```

Let us try to make a curl call to the details service over HTTP
```sh
curl http://details:9080/details/0
```

Since, we have TLS between the sidecar's, an HTTP call will not work. You will see an error like the one below:
```sh
curl: (56) Recv failure: Connection reset by peer
```

Let us try to make a curl call to the details service over HTTPS but **WITHOUT** certs:
```sh
curl https://details:9080/details/0 -k
```

The request will be denied and you will see an error like the one below:
```sh
curl: (35) gnutls_handshake() failed: Handshake failed
```

Now, let us use curl over HTTPS with certificates to the details service:
```sh
curl https://details:9080/details/0 -v --key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem --cacert /etc/certs/root-cert.pem -k
```

Output will be similar to this:
```sh
*   Trying 10.107.35.26...
* Connected to details (10.107.35.26) port 9080 (#0)
* found 1 certificates in /etc/certs/root-cert.pem
* found 0 certificates in /etc/ssl/certs
* ALPN, offering http/1.1
* SSL connection using TLS1.2 / ECDHE_RSA_AES_128_GCM_SHA256
*        server certificate verification SKIPPED
*        server certificate status verification SKIPPED
* error fetching CN from cert:The requested data were not available.
*        common name:  (does not match 'details')
*        server certificate expiration date OK
*        server certificate activation date OK
*        certificate public key: RSA
*        certificate version: #3
*        subject: O=#1300
*        start date: Thu, 07 Jun 2018 14:36:56 GMT
*        expire date: Wed, 05 Sep 2018 14:36:56 GMT
*        issuer: O=k8s.cluster.local
*        compression: NULL
* ALPN, server accepted to use http/1.1
> GET /details/0 HTTP/1.1
> Host: details:9080
> User-Agent: curl/7.47.0
> Accept: */*
>
< HTTP/1.1 200 OK
< content-type: application/json
< server: envoy
< date: Thu, 07 Jun 2018 15:19:46 GMT
< content-length: 178
< x-envoy-upstream-service-time: 1
< x-envoy-decorator-operation: default-route
<
* Connection #0 to host details left intact
{"id":0,"author":"William Shakespeare","year":1595,"type":"paperback","pages":200,"publisher":"PublisherA","language":"English","ISBN-10":"1234567890","ISBN-13":"123-1234567890"}
```

This proves the existence of mTLS between the services on the Istio mesh.


## 9.2 [Secure Production Identity Framework for Everyone (SPIFFE)](https://spiffe.io/)

Istio uses [SPIFFE](https://spiffe.io/) to assert the identify of workloads on the cluster. SPIFFE consists of a notion of identity and a method of proving it. A SPIFFE identity consists of an authority part and a path. The meaning of the path in spiffe land is implementation defined. In k8s it takes the form `/ns/$namespace/sa/$service-account` with the expected meaning. A SPIFFE identify is embedded in a document. This document in principle can take many forms but currently the only defined format is x509. Let's see what a SPIFFE x509 looks like.


To start our investigation, let us create a local `tmp` directory, grab the certs from the productpage sidecar & place it in `tmp`:
```sh
mkdir ~/tmp
cd ~/tmp
fs=(key.pem cert-chain.pem root-cert.pem)
productProxy=$(kubectl get pod | grep productpage | awk '{ print $1 }')
for f in ${fs[@]}; do kubectl exec -c istio-proxy $productProxy /bin/cat -- /etc/certs/$f >$f; done
```

Let us now investigate the ceritificate using openssl. `PWK` hosts don't come installed with openssl. Let us first install it:
```
yum install -y openssl
```

Now that we have openssl installed and the certificate files in place, let us view the certificate:
```
openssl x509 -in cert-chain.pem -text
```

There are a few things which are interesting. To name a few, the subject isn't what you'd normally expect, URI SAN extension has a `spiffe` URI.

```sh
    X509v3 Subject Alternative Name:
        URI:spiffe://cluster.local/ns/default/sa/default
```



There is one more part to SPIFFE identity, and that's a signing authority. This a CA certificate with a SPIFFE identify with _no_ path component. We can use openssl to verify the certificate with the CA using the command below:

```
openssl verify -CAfile root-cert.pem cert-chain.pem
```

The verification should succeed.



## [Continue to Lab 10 - Circuit Breaking](../lab-10/README.md)
