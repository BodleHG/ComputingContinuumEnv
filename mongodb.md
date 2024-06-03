```bash
kubectl create namespace mongodb-replica
```

```bash
openssl rand -base64 741 > ./mongo-replica-sets-key.txt
```

```bash
kubectl create secret generic mongo-shared-bootstrap-data -n mongodb-replica --from-file=internal-auth-mongodb-keyfile=./mongo-replica-sets-key.txt
```

