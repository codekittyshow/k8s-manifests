# K8s Manifests 

First Create the namespace:
```shell
kubectl create ns prod -o yaml --dry-run=client > prod-namespace.yaml
```

First Create the Code Kitty API Deployment:
```shell
kubectl create deployment code-kitty-api --image=ghcr.io/codekittyshow/code-kitty-api:latest -n prod --port 8080 -o yaml --dry-run=client > api/code-kitty-api-deploy.yaml
```

Run below command to get logs
```shell
kubectl logs --selector=app=code-kitty-api -n prod --tail -1
```

Pod logs are like below

    > @codekitty/code-kittty-api@1.0.0 start /app
    > tsc && node dist/index.js

    Server running on port 4000
    Connected to MongoDB!

DB is not connected since we have not provide env variable for it.
```yaml
env:
    - name: PORT
      value: "8080"
    - name: DB_USERNAME
      value: "chamod"
    - name: DB_HOST
      value: "codekitty.sacfe.mongodb.net"
    - name: DB_NAME
      value: "CodeKitty"
```

Create Secret
```shell
kubectl create secret generic mongodb-secret -n prod -o yaml --dry-run=client > mongodb/mongodb-secret.yaml
```

Base 64 encode the DB_PASSWORD
```shell
echo -n "12345" | base64
```

Add that encoded password to our secret
```yaml
type: Opaque
data:
  db-password: MTIzNDU=
```

Ref secret to the env variable
```yaml
- name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mongodb-secret
        key: db-password
```

Apply yaml we created so far
```shell
kubectl apply -f prod-namespace.yaml
kubectl apply -f mongodb
kubectl apply -f api
```

Create service for Code Kitty API Deployment
```shell
kubectl expose deployment/code-kitty-api -n prod --type=LoadBalancer
```
Make sure you have a public IP address for the service

```shell
curl <EXTERNAL_IP>:8080/api/v1
```

Create the Code Kitty Frontend Deployment:
```shell
kubectl create deployment code-kitty --image=ghcr.io/codekittyshow/code-kitty:latest -n prod --port 80 -o yaml --dry-run=client > frontend/code-kitty-deploy.yaml
```

Let's create frontend deployment
```shell
kubectl apply -f frontend
```

Create service for Code Kitty Frontend Deployment
```shell
kubectl expose deployment/code-kitty -n prod --port 80 --type=LoadBalancer
```

```shell
kubectl get svc -n prod
```