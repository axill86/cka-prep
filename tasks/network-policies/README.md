# Play with network policies
Let's do some fancy things with network policies.

## Aliases
```
alias k=kubectl
```
## Busybox pod
First, let's do some preparations. Let's create busybox pod, which we will use further
```
k run busybox --image=busybox --command -- sleep 4800
```
Let's try to nslookup for kube-api, which exposed as a service.

```
k exec -it busybox -- nslookup $(k get svc kubernetes -o jsonpath="{.spec.clusterIP}")
```

That command should produce some output like
```
Server:         10.96.0.10
Address:        10.96.0.10:53

1.0.96.10.in-addr.arpa  name = kubernetes.default.svc.cluster.local
```

## Network policies
[Network Policiess](https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors)
Now, let's crreate default deny network policy
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

```

After we apply that, let's do nslookup again.

```
k exec -it busybox -- nslookup $(k get svc kubernetes -o jsonpath="{.spec.clusterIP}")

;; connection timed out; no servers could be reached

command terminated with exit code 1
```
Why did that happen? Well, actually because we denied all the outcoming request for all pods in our namespace. Let' fix that.

## Enabling dns service requests from pods

Solution is pretty simple. The only thing which we need to do is to enable outcoming UDP connections to port 53 of kubernetes dns service.
But, let's do some preparation here first =)

```
k label ns kube-system name=kube-system
```
That's needed for setting up our network policy later.
Also, note that dns server pods have k8s-app=kube-dns label set, which will allow to write selector for our network policy. You can check that via command

```
k get pods -n kube-system --show-labels
```

Now, let's construct our network policy

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  egress:
  - to:
    - namespaceSelector:
       matchLabels:
         name: kube-system
    - podSelector:
       matchLabels:
         k8s-app: kube-dns
    ports:
      - port: 53
        protocol: UDP

```
Now, after we applied that, let's check our dns resolving again
```
k exec -it busybox -- nslookup $(k get svc kubernetes -o jsonpath="{.spec.clusterIP}")
Server:         10.96.0.10
Address:        10.96.0.10:53

1.0.96.10.in-addr.arpa  name = kubernetes.default.svc.cluster.local

```
Hooray =) that works.

