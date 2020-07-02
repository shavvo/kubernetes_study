# kubernetes_study

A place to put notes and test manifests


# Useful Commands


__Get current namespaces:__

```
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   110m
kube-node-lease   Active   110m
kube-public       Active   110m
kube-system       Active   110m
```


__Create a small deployment:__

```
$ kubectl create deployment mynginx --image=nginx:1.15-alpine
deployment.apps/mynginx created
```


__Get information about `mynginx` app deployment:__

```
$ kubectl get deploy,rs,po -l app=mynginx
```

__Scale out the deployment to 3 replicas:__

```
$ kubectl scale deploy mynginx --replicas=3
```


__More details about specific deployment:__

```
$ kubectl describe deploy mynginx
```


__Details about rollout for deployment mynginx__

```
$ kubectl rollout history deploy mynginx
```

__Details about specific revision:__
```
$ kubectl rollout history deploy mynginx --revision=1
```

__Upgrade image in mynginx deployment:__
```
$ kubectl set image deployment mynginx nginx=nginx:1.16-alpine
```

__Rollback to previous revision:__
```
$ kubectl rollout undo deployment mynginx --to-revision=1
```

__Get everything on a cluster__
```
$ kubectl get all
```

__Create a resource from a file__
```
$ kubectl create -f filename
```

__More detailed information__
```
$ kubectl get {something} -o wide
```

__Update resources with new information from definition file__
```
$ kubectl replace -f filename
```

__Edit existing resource__
```
$ kubectl edit {resource} {name}
```

__Get definitions of existing resource in yaml (can be used as a backup)__
```
$ kubectl get {resource} -o yaml
```

__Create a simple pod__
```
$ kubectl run nginx --image=nginx
```

__Generate a pod manifest yaml file__
```
$ kubectl run nginx --image=nginx --dry-run=client -o yaml > pod-definition.yaml
```

__Change namespace__
```
$ kubectl config set-context $(kubectl config current-context) --namespace=space-name
```
