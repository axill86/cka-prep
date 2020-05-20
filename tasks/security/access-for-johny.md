# Grant access to johny =)

## Task

You're a superadmin in screwtech company and you onboard new developer johny into the team. 
Johny needs to perform create, update, delete actions with pods, services deployments in development ns.

## Solution

Well, let's start from generating new key and CSR for Johny using openssl

```
openssl req -new -newkey rsa:2048 -keyout johny.key -out johny.csr -days 365 -subj '/CN=johny' -nodes
```

Okay. Our next step will be creating CSR for that guy

```
cat <<EOF | kubectl apply -f
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: johny-request
spec:
  request: $(cat johny.csr | base64 | tr -d '\n')
  usages:
  - client auth
EOF
```
That actually copy-pasted from following [doc entry](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/#create-a-certificate-signing-request)
**Note**: Valid usages should be set

Now that CSR can be viewed and approved

```
kubectl get csr johny-request
```

```
kubectl certificate approve johny-request
```

It's time to send the certificate to johny so he can use it. Get generated cert with following command

```
kubectl get csr johny-request -o jsonpath="{.status.certificate}" | base64 -Dd > johny.cert
```

Johny can use that to do proper setup =)

Actually, now we need to add role and rolebinding for johny in order to make it possible for heam to do stuff

```
kubectl create role developer --verb=create,delete,get,list,patch,update,watch,deletecollection --resource=pod,deployment,service -n development

kubectl create rolebinding johny-developer --user=johny --role=developer -n development
```

Let's check that johny actually can do stuff

```
kubectl auth can-i get pods -n development --as johny
```

Answer should be yes


### Instruction for johny to setup the context
