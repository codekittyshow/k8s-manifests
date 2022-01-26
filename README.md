# K8s Manifests 

First Create the namespace:
```shell
kubectl create ns prod -o yaml --dry-run=client > prod-namespace.yaml
```

First Create the Code Kitty API Deployment:
```shell
kubectl create deployment code-kitty-api --image=ghcr.io/codekittyshow/code-kitty-api:latest -n prod -o yaml --dry-run=client > code-kitty-api-deploy.yaml
```

Pod logs are like below

```shell
➜  k8s-manifests git:(main) ✗ kubectl logs code-kitty-api-55b68b7b54-96nww -n prod

> @codekitty/code-kittty-api@1.0.0 start /app
> tsc && node dist/index.js

Server running on port 4000
Unable to connect to MongoDB! Cannot read property 'match' of undefined
```

DB is not connected since we have not provide env variable for it.

```yaml
env:
  - name: PORT
    value: "8080"
  - name: DB_PASSWORD
    value: ""
```


Create Secret

```shell
kubectl create secret generic mongodb-secret -o yaml --dry-run=client > mongodb/mongodb-secret.yaml
```

Change the namespace to prod
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
  namespace: prod
type: Opaque
data:
  db-username: Y2hhbW9k
  db-password: MTIzNDU=
  mongodb-replica-set-key: bXlzdHJvbmdwYXNzd29yZA==
```

Base 64 encode the DB_PASSWORD

```shell
echo -n "12345" | base64
```

```shell
kubectl apply -f mongodb/mongodb-secret.yaml
```







