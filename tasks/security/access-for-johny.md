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

