# Grant access to johny =)

## Task

You're a superadmin in screwtech company and you onboard new developer johny into the team. 
Johny needs to perform create, update, delete actions with pods, services deployments in development ns.

## Solution

Well, let's start from generating new key and CSR for Johny using openssl

```
openssl req -new -x509 -newkey rsa:2048 -keyout johny.key -out johny.csr -days 365 -subj '/CN=johny' -nodes
```
