# API access from POD

That's task is about using service-aacount in order to access kube-api

## Create service account and bind that account to some clusterole 

```
k create serviceaccount pv-viewer
k create clusterrolebinding pv-viewer-role-binding --clusterrole --serviceaccount=pv-viewer
```

## Create pod 

Now let's create pod which will use created service account

```
k run curl --image=curlimages/curl --serviceaccount=pv-viewer --command -- sleep 4800
```

## Access api from pod

Actually that's not very tricky as all required information is mounted to /va/run/secrets/kubernetes.io/serviceaccount. See [doc](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)

Let's do some actions from the pod
```
k exec -it curl -- sh
```

And invoke following to access the api

```
curl "https://kubernetes/api/v1" --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
```
