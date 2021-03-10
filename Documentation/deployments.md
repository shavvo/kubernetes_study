# Deployments:

* https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

A Deployment provides declarative updates for pods replicasets. A desired state is described in a Deployment and the Deployment Controller changes the current state to the desired state in a controller rate. 


### Create a deployment
`kubectl create deployment --image=imagename name`
```
Ex:
kubectl create deployment --image=nginx nginx
```

### Generate a deployment yaml file from a dry run
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deploy.yaml`


Example output:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
