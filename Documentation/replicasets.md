# ReplicaSet

* https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/

A replicasets purpose is to maintain a stable set of replica Pods running at any given time. It is often used to guarantee the availability of a specified number of pods. 

A replicaSet ensures that a specified number of pod replicas are running at any given time. However, a *deployment* is a higher-level concept that manages ReplicaSets and provides declarative updates to pods along with a lot of other useful features. Therefore, using Deployments is recommended
over using ReplicaSets, unless a custom update orchestration is required. 



### Delete a resource (such as a replica set):
*This will also delete the pods under the replica set*

`kubectl delete replicaset {name of replicaset}` 


### Edit existing image (such as a replicaset):
`kubectl edit replicaset {name}`


### Grab output of a resource in yaml. As a backup if you dont have the definitions:

 *-o = output*
 
`kubectl get resource {resourceName} -o yaml`

Example:
```
kubectl get replicaset frontend -o yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3

```
