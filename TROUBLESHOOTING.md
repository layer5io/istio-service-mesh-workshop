## Kubernetes Troubleshooting

The most important tool to know for debugging problems is the describe resource command.
It can be used to describe any Kubernetes resource.

`kubectl describe pod helloworld-service-v1-119sf27584-jwfzh`

`kubectl describe service helloworld-service`

kubectl top can be used to see resource utilization. A very common problem is the pod was not able to be scheduled.

`kubectl top nodes`

`kubectl top pods`

View the logs of a Pod by finding the full pod name and specify the main container, i.e.:

`kubectl logs guestbook-ui-18721af123-1lw81 -c guestbook-ui`
