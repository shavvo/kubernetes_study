# What is kubernetes?

* https://kubernetes.io/docs/concepts/overview/components/

Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

### kube-apiserver:

The API server is a component of the Kubernetes control plane that exposes the Kubernetes API. The API server is the front end for the Kubernetes control plane.

### etcd:

Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

### kube-scheduler:

Control plane component that watches for newly created Pods with no assigned node, and selects a node for them to run on.

### kube-controller-manager:

Control Plane component that runs controller processes.

## Node Components:

### kubelet: 

An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn't manage containers which were not created by Kubernetes.


### kube-proxy:

kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.



## Kubernetes Specific: 

### Assign a pod to a specific node


To assign a pod to a specific node the following can be added in the `spec` section of the definition file:

```yaml
nodeName: <name of node>
```

### kubelet service configuration:
`/etc/systemd/system/kubelet.service`

### Check kubelet logs:
`sudo journalctl -u kubelet`

### Verify kubelet certificates:
`openssl x509 -in /path/to/cert.crt -text`

### Get cluster info:
`kubectl cluster-info`

### Find api resources:
`kubectl api-resources`

### Get everything on the cluster:
`kubectl get all`

### create resource in kubernetes from the filename mentioned:
*-f = filename* 

`kubectl create -f /path/to/file`
```
Ex:
kubectl create -f /var/stuff/pod.yaml
```



### Get information about a resource within kubernetes. [pods, controllers, nodes, ect..]
`kubectl get {whatever}`
```
Ex:

kubectl get pod
```


### Get more detailed information about a resource:
`kubectl get {whatever} -o wide`
```
Example:

kubectl get pod -o wide
```

### Get all the information about a resource:
`kubectl describe {whatever} {name}`

```
Ex:

kubectl describe pod podname
```


### Update resources with new information from the definition file:
*this will pick up any new changes made to that file and create/update the resources*

`kubectl replace -f filename`


### Scale up resources (such as replicas) without modifying the definition file (this will not change the replicas parameter in the file)

`kubectl scale --replicas=6 -f filename`


### Get documentation for various resources. (Pods, Nodes, services, ect.)


`kubectl explain pods --recursive`

### Command for deploying multiple(all) yaml files within a dir:

  `kubectl create -f .`

## More Specific commands:

### Create an nginx pod:
`kubectl run nginx --image=nginx`



### Generate a Pod manifest YAML file (-o yaml).
```
Note: 
This will perform a dry run as the `--dry-run=client` is present in the command.
```
`kubectl run nginx --image=nginx --dry-run=client -o yaml`

### Create a namespace from cli
`kubectl create namespace name`

### Change namespace
`kubectl config set-context $(kubectl config current-context) --namespace=space-name`


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

# Services

* https://kubernetes.io/docs/concepts/services-networking/service/

An abstract way to expose an application running on a set of Pods as a network service.

### Create a named redis-service to type ClusterIP to expose pod redis on port 6379
*This will automatically use the pods labels as selectors*

``kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml``

This will not use the pods labels as selectors, instead it will assume selectors as `app=redis`. This will not work very well if your pod has a different label set. Best practice is to generate the file and modify the selectors before creating service.

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`

### Create a service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes. 

*This will automatically use the pods labels as selectors, but you cannot specify the node port. You must generate a definition file and then add the node port in manually before creating the service.*

`kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml`

*This will not use the pods labels as selectors*

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`



# Taints and Tolerations

* https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/

Node affinity, is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

### Taints happen on the Node
`kubectl taint nodes node-name key=value:taint-effect`
   ```
   - key=value: pair is used to specify the taint. Ex: app=blue
   - taint-effect: what happens to PODs that do not tolerate the taint
        *  NoSchedule: Pods will not be scheduled on node
        *  PreferNoSchedule: Try to not place POD on node
        *  NoExecute: New pods will not be scheduled and existing pods onnode
                      will be evicted if they do not tolerate the taint
```

*Example*

`kubectl taint nodes node1 app=blue:NoSchedule`



### Tolerations happen on the PODs 
Add tolerations to pod definitions as: (using example kubectl from above)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "example-key"
    operator: "Exists"
    effect: "NoSchedule"
      
```
# Node Selectors

* https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector

nodeSelector is the simplest recommended form of node selection constraint. nodeSelector is a field of PodSpec. It specifies a map of key-value pairs. 

*Can be used to identify which node to create the pod on
Add within the pod definition file.*

```yaml
  apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```
### Label Nodes:

*Used to label nodes for selectors to use within pods*

`kubectl label nodes <node-name> key=value`

*Example*

`kubectl label nodes node-1 size=Large`


# Node Affinity

* https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity

Ensure pods are hosted on particular nodes.
Add within definition file. Useful for a set of nodes with different labels. IE add pods to nodes that contain [x, y, z] labels

```yaml
    apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

*Example deployment with node affinity:*

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists

```

# Resources and Requests

* https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

*When a pod is created the containers are assigned a default CPU request of .5 and memory of 256Mi. For the POD to pick up those defaults you must have first set those as default values for request and limit by creating a LimitRange in that namespace.*

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container

#Resources are defined in the definitions file:

spec:
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
    limits: # change resource limits
      memory: "2Gi"
      cpu: 2
```

# Daemon Sets

* https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/

*A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to a cluster, Pods are added to those nodes. As nodes are removed from the cluster, the Pods are garbage collected. Deleting a DaemonSet will clean up the Pods created in it. *

** ds = daemonset**

`kubectl get ds`

*Creating a daemonset is interesting as there is no interpretive command.
The best(?) way to do this is to generate a deployment yaml file first
and then edit that to suit your needs. Using the test question data of:*

```
name: elasticsearch
namespace: kube-system
image: whatever image it says (will use nginx for simplicity)
```

`kubectl create deployment elasticsearch --namespace=kube-system --image=nginx --dry-run=client -o yaml > ds.yaml`

*Edit the kind to `DaemonSet` and change the name and namespace to what is specified. Make sure selectors and labels are correct and modify the image to the correct image. Remove anything else.* 

* https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/controllers/daemonset.yaml


# Static Pods

* https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/

Kubelet can manage a node indepenently. If no API server is available to provide pod creation instructions, the kubelet can be configured to look for manifests on the server. Place POD manifests within `/etc/kubernetes/manifests` as a default. However, the directory can be configured to be anything. 

These pods that are created without any intervention (by the kubelet) on its own from the
API server or master node are known as `static pods`. Can only create pods.

Static pods are created relative to the node you are currently on. To find the pod directory or setup on a particular node
ssh into the node itself and look at the kubelet process. 

### Look at all nodes and get the ip of the node in question

  `kubectl get nodes -o wide `

### ssh into the node:

  `ssh {node ip}`

To check for the pod path or how the static pod is configured, look at the kubelet process.

  `ps -aux |grep kubelet`

The process will have the path to the config file.

This can be configured a couple ways. Once with the ExecStart (in the kubelet service file) option with the following option:

  `--pod-manifest-path=/ect/kubernetes/manifests`  
  
  or whatever dir you want or you can specify a path to the config yaml.

  `--config=kubeconfig.yaml`

In kubeconfig.yaml
  
  `staticPodPath: /etc/kubetnetes/manifests`

Generally, anything setup as a master is a static pod. 

 
Creating a Static Pod on whatever node you are on:

`kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > static-busybox.yaml`


# Scheduler

* https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/

A scheduler watches for newly created Pods that have no Node assigned. For every Pod that the scheduler discovers, the scheduler becomes responsible for finding the best Node for that Pod to run on. The scheduler reaches this placement decision taking into account the scheduling principles described below.


### How to find where the kube-scheduler.yaml lives:

Find the kubelet service config file

  `cd /etc/systemd/system/kubelet.service.d`

Find the config arguments for kubelet. 

  `cat {kubeadmn.conf} (look for kubelet_config_args)`

grep out the config path for staticPods

`grep -i static /var/lib/kubelet/config.yaml`

  ```
  staticPodPath: /etc/kubernetes/manifests/
  ```

cd into that dir and the `kube-scheduler.yaml` should be in there

To create a custom scheduler, can copy the default kube scheduler
and modify the scheduler name. Move this file to its own dir.
Also under the `command` section add in an additional command that tells the scheduler where to look. Also need to set the `leader-elect` option to false. Can also pass in additional parameter if there are multiple master nodes. This will set a `lock object` name.

  ```
  - --lock-object-name=my-custom-scheduler
  - --scheduler-name=my-custom-scheduler
  ```

*An example of creating a custom scheduler with a custom port and name:*

```yaml
spec:
  containers:
    - command:
      - kube-scheduler
      - --address=127.0.0.1
      - --kubeconfig=/etc/kubernetes/scheduler.conf
      - --leader-elect=false
      - --port=10282
      - --scheduler-name=my-scheduler

```

In order for new pods to use the new scheduler, the proper ident 
needs to be set in the definition file. 

```yaml
spec:
  schedulerName: my-custom-scheduler 
```

To verify that the scheduler was created, you can look at the events.

`  kubectl get events`

Look for the `Reason` and `Source` that should have some of the custom
schedulers details in it. 

To look at the logs for the scheduler run the following:

  `kubectl logs my-custom-scheduler -n kube-system`




# Monitoring

* https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/

Mertics Server:

One metrics server per cluster. Only an in memory monitoring
does not store metrics on disk. Runs on the kubelet as a sub component
called the `cAdvisor`. This makes the metrics available to the API
for the metrics server to injest. 

Once data is processed, performance can be viewed by looking at the following
command. For node

  `kubectl top node`

View metrics for pods:

  `kubectl top pod`


# Logging

View logs for a pod

  `kubectl logs {pod name}` 

See logs as they happen

  `kubectl logs -f {pod name}`

If there are multiple images{containers} on a pod you can view logs
for an individual one by using the following:

  `kubectl logs -f {pod name} (image name)`



# Updates and Rollouts

* https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment

Rollouts are versions or revisions of an application

See the status of a rollout:

  `kubectl rollout status deployment/myapp-deployment`

To view the history of a rollout use the following:

  `kubectl rollout history deployment/myapp-deployment`


### Two types of deployment strategies:

```
Recreate:
destroy all instances and create new (application is down)
```

```
Rolling (default strategy)
Take down old version and spin up new version one at a time
```



### Two ways of updating:


```
Modify image in definition file
```

```
run the kubectl set command:
```

  `kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`


*To undo a changed, run the following:*

  `kubectl rollout undo deployment/myapp-deployment`



# Config Maps:

* https://kubernetes.io/docs/concepts/configuration/configmap/

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

Two phases of creating a config map. 

### Create the config map:

#### Imperative:
  `kubectl create configmap <config-name> --from-literal=<key>=<value>`

The `--from-literal` option is used to specify the key/value pair in the command itself. To specify multiple pairs, you must use `--from-literal` for each pair. 

*Example:*

  `kubectl create configmap app-config --from-literal=APP_COLOR=blue`

This can also be done from a file. 

  `kubectl create configmap <config-name> --from-file=<path-to-file>`

*Example:*

  `kubectl create configmap app-config --from-file=app_config.properties`

#### Declarative:

Create a config-map definition file:

config-map.yaml:

```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    APP_COLOR: blue
    APP_MODE: prod
```

Then create the map with:

  `kubectl create -f config-map.yaml`



### Inject it into the pod. 

To inject a configmap into a pod, add an additional section to the
definition file. 

*Example:*

pod-definition.yaml

```yaml
  apiVersion: v1
  kind: Pod
  metadata: 
    name: simple-webapp
    labels:
      name: simple-webapp
  spec:
    containers:
    - name: simple-webapp
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```


# Secrets:

* https://kubernetes.io/docs/concepts/configuration/secret/

Kubernetes Secrets let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. Storing confidential information in a Secret is safer and more flexible than putting it verbatim in a Pod definition or in a container image.

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in an image.


Two ways of creating a secret:

#### Imperative:

  `kubectl create secret generic <secret-name> --from-literal=<key>=<value>`

*Example:*

  `kubectl create secret generic app-secret --from-literal=DB_Host=mysql`

The same logic applies for secrets as it does for configmaps. If you want to 
specify multiple key/value pairs in the same Imperative command, you must state `--from-literal` for each pair. 


Can also use a file to pass in secrets using the `--from-file` option to pass in a lot of secrets at once.

  `kubectl create secret generic <secret-name> --from-file<file-path>`

  *Example* 
  
  `kubectl create secret generic app-secret --from-file=app_secret.properties`


#### Declarative:

Create a secrets definition file and house the secrets there. 

```yaml
  secret-data.yaml
    apiVersion: v1
    kind: Secret
    metadata: 
      name: my_app
    data:
      DB_Host: mysql
      DB_User: root
      DB_Password: paswrd
```
** Note: the key/value pairs must be encoded. **

How to encode:

  *Can be run in a terminal:*
  
  `echo -n 'mysql' |base64 (returns the base64 value of the string provided)`


Getting the secrets and their values is done through the `-o yaml` return

  `kubectl get secret app-secret -o yaml`

Injecting the secret into a pod is done through the pod definition file:

```yaml
  spec:
    containers:
    - image: 
        envFrom:
          - secretRef:
              name: app-secret
```

If a secret is injected as a volume, the volume will contain files for each
secret. The name of the file will be the key and the contents of the file will be the value.

```
  IE:
    DB_Host - filename
      mysql - contents
```

# Multi container pods

* https://kubernetes.io/docs/concepts/workloads/pods/#how-pods-manage-multiple-containers

A Pod can encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. 


Modify the spec to have multiple containers

```yaml
  spec:
    containers:
    - name: app
      image: kodekloud/event-simulator
      volumeMounts:
      - mountPath: /log
        name: log-volume

    - name: sidecar
      image: kodecloud/filebeat-configured
      volumeMounts:
      - mountPath: /var/log/event-simulator/
        name: log-volume

    volumes:
    - name: log-volume
      hostPath:
      # directory location on host
      path: /var/log/webapp
      # optional field
      type: DirectoryOrCreate
```


# init containers

* https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

An init container is configured in a pod like all other containers.
However, it is specified within the `initContainers` section.

```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
  spec:
    containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    initContainers:
    - name: init-myservice
      image: busybox
      command: ['sh', '-c', 'git clone <some-repository-to-be-used-by-app> ; done;']
```

When a pod is first created, the `initContainer` is run. The process in the 
init container must run to completion before the real container hosting the app starts. 

Multiple initContainers can be created as well. In this case, each init container is run one at a time in sequential order. 

```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
  spec:
    containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
    - name: init-mydb
      image: busybox:1.28
      command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

If any of the initContainers fail to complete, kubernetes will restart the pod until the initContainer succeeds. However, if the Pod has a restartPolicy of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.



# Cluster Maintenance:

To drain a specific node: (note, these pods are terminated and re-created on the new node). The node is also cordoned (marked as unscheduleable)

  `kubectl drain node-1`

When you reboot and bring the broken node back up, it will need to be uncordoned:

  `kubectl uncordon node-1`


To make a node unscheduleable, you can cordon the node:

`kubectl cordon node-1`



# Cluster Upgrade process


https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/


The control plane components can be different versions.

The kube-apiserver is the catalyst:

Command to see current versions:

  `kubectl version --short`

*Example:*

  `kube-apiserver can be v1.10`

  `controller-manager` and `kube-scheduler` can be one version lower than apiserver

```
  controller-manager: v1.9 or v1.10
  kubve-scheduler: v1.9 or 1.10

  kubelet and kube-proxy can be two versions lower:

  kubelet: v1.8, v1.9, or v1.10
  kube-proxy: v1.8, v1.9 or v1.10

  kubectl can be one version higher or lower than apiserver:

  kubectl: v1.9, v1.10 or v1.11
```

Recommended version upgrade is one minor version at a time. 


Upgrade strategies:

* All at once: all nodes go down and upgrade
* One node at a time: workloads move to different nodes, and first node upgrades. 
* Provision new nodes and upgrade: spin up new node, upgrade it and move workload to it


To upgrade:

First, you must upgrade the version of kubeadm:

**Note: this applies to the node you are on IE master or worker nodes**

```
  apt install kubeadm=<version number>
  apt install kubeadm=1.12.0-00
```
Run kubeadm upgrade plan to see the versioning and current version. 

  `kubeadm upgrade plan`

Upgrade by applying the new version:

  `kubeadm upgrade apply v1.12.0`

If you have kubelets on master node, you need to upgrade those as well:

```
  apt-get upgrade -y kubelet=1.12.0-00
  systemctl restart kubelet
```

Next, upgrade worker nodes. 

Drain the nodes:

  `kubectl drain node-01`

Upgrade kubeadm and kubelet:

```
  apt-get upgrade -y kubeadm=1.12.0-00
  apt-get upgrade -y kubelet=1.12.0-00
```
Upgrade node: {note: you must be ON the worker node (ssh node01)}

  `kubeadm upgrade node config --kubelet-version v1.12.0`

Restart the kubelet:

  `systemctl restart kubelet`

Uncordon upgraded node:

  `kubectl uncordon node01`



# Backup and Restore methods


Reference Links:
* https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
* https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md
* https://www.youtube.com/watch?v=qRPNuT080Hk


`etcdctl` is the cli for etcd. Before using it, you need to make sure the correct version is in place. This can be done by exporting the variable `ECTDCTL_API`:

  `export ETCDCTL_API=3`

`etcdctl` has some mandatory options that need to be included:

* --cacert: verify certificates of TLS-enabled secure servers using this CA bundle
* --cert: identify secure client using this TLS cert
* --endpoints=[127.0.0.1:2379]: This is the default as ETCd running on master and exposed on localhost 2379
* --key: identify secure client using this TLS key file

**Useful command for etcdctl is member list:**

`ETCDCTL_API=3 etcdctl member list 
--endpoints=https://[127.0.0.1]:2379
--cacert=/etc/kubernetes/pki/etcd/ca.crt
--cert=/etc/kubernetes/pki/etcd/server.crt
--key=/etc/kubernetes/pki/etcd/server.key`



**Example of a snapshot command:**

`  ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379
  --cacert=/etc/kubernetes/pki/etcd/ca.crt
  --cert=/etc/kubernetes/pki/etcd/server.crt
  --key=/etc/kubernetes/pki/etcd/server.key snapshot save /tmp/snapshot-pre-boot.db`


Get a full view of all resources:

  `kubectl get all --all-namespaces -o yaml > path-to-file.yaml`

Create a snapshot of etcd by using the ectdctl command: When using the command, must specify

* --endpoints=
* --cacert=
* --cert=
* --key=

`ectdctl snapshot save <name>.db`

View status of an etcd snapshot

`ectdctl snapshot status <name>.db`


Restore a snapshot:

stop kube-apiserver:

  `service kube-apiserver stop`

  `etcdctl snapshot restore <name>.db
  --data-dir /var/lib/ectd-from-backup 
  --initial-cluster-token etcd-cluster-1`

  `systemctl daemon-reload`
  
  `service etcd restart`
  
  `service kube-apiserver start`

Full restore command

  `ETCDCTL_API=3 etcdctl snapshot restore --endpoints=127.0.0.1:2379
  --cacert="/etc/kubernetes/pki/etcd/ca.crt"
  --cert="/etc/kubernetes/pki/etcd/server.crt"
  --key="/etc/kubernetes/pki/etcd/server.key"
  --data-dir="/var/lib/etcd-from-backup"
  --initial-advertise-peer-urls="http://127.0.0.1:2380"
  --initial-cluster="master=http://127.0.0.1:2380"
  --initial-cluster-token="etcd-cluster-1"
  --name="master"
  /tmp/snapshot-pre-boot.db`

After the restore has completed, the etcd.yaml needs to be modified to update some variables:

  `/etc/kubernetes/manifests`

  modify datadir to new data directory specified in the restore command.
  add the `inition-cluster-token` as an option underneath the data dir
  modify the mountPath to the same as the datadir (for etcd-data)
  modify the hostPath to the same as the datadir (for etcd-data)
  
  
  
  

# Security




### Authentication


https://kubernetes.io/docs/reference/access-authn-authz/authentication/


Static password and token files: (not recommended)

Static and Password files(can also add a group option for more granular access):

Can be stored in a csv
		`password,user,userid,group`

Pass the file to the kubeservice or update the kube-apiserver.yaml
	
  `--basic-auth-file=user-details.csv`

```yaml
	spec:
	  containers:
 	  -  command:
	     - -- basic-auth-file=user-details.csv
```

Static token files (The group column is optional for more granular access via groups):
	
  Also stored in csv:
		`token,user,userid,group`

Passed directly:

	`--token-auth-file=user-details.csv`


Basic Setup instructions:

* Create  a local file /tmp/users/user-details.csv
* Edit the kube-apiserver static pod configured by kubeadm to pass in the user details. The file is usually located at /etc/kubernetes/manifests/kube-apiserver.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: kube-apiserver
 namespace: kube-system
spec:
 containers:
 - command:
   - kube-apiserver
     <content-hidden>
   image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
   name: kube-apiserver
   volumeMounts:
   - mountPath: /tmp/users
     name: usr-details
     readOnly: true
 volumes:
 - hostPath:
     path: /tmp/users
     type: DirectoryOrCreate
   name: usr-details
```
Modify the kube-apiserver startup options to include the basic-auth file:

```yaml
apiVersion: v1
kind: Pod
metadata:
 creationTimestamp: null
 name: kube-apiserver
 namespace: kube-system
spec:
 containers:
 - command:
   - kube-apiserver
   - --authorization-mode=Node,RBAC
     <content-hidden>
   - --basic-auth-file=/tmp/users/user-details.csv
```
Create the appropriate roles and role bindings for the users:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 namespace: default
 name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
 resources: ["pods"]
 verbs: ["get", "watch", "list"]
```


This role binding allows "jane" to read pods in the "default" namespace.

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 name: read-pods
 namespace: default
subjects:
- kind: User
 name: user1 # Name is case sensitive
 apiGroup: rbac.authorization.k8s.io
roleRef:
 kind: Role #this must be Role or ClusterRole
 name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
 apiGroup: rbac.authorization.k8s.io
```


Once created, you can authenticate to the kube-api server using the users credentials:

`curl -v -k https://localhost:6443/api/v1/pods -u “user1:password123”`




### TLS

https://kubernetes.io/docs/reference/access-authn-authz/authentication/#x509-client-certs


Basic OpenSSL generation commands:

* Generate a key: 
		`openssl genrsa -out ca.key 2048`
* Generate a CSR: 
	* Note: on kubernetes there is a specific group used to id admin users. This needs to be used during the generation of an admin key in the CSR. That group is `system:masters`.
	* Along those lines nodes also have a specific group that need to be added: This group is `system:nodes`
	
		`openssl req -new -key ca.key -subj “/CN=kube-ca” -out ca.csr`
	* AdminKey:
		`openssl req -new -key ca.key -subj “CN=kube-admin/O=system:masters” -out admin.csr`
    
* Sign a cert (self signed):
		`openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt`
* Sign a cert with another cert:
		`openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt`
	

In order to specify alternate names on a certificate, a `.cnf` file should be used. 

	

	openssl.cnf 
		[req]
		req_extentions = v3_req
		[ v3_req ]
		basicConstraints = CA:FALSE
		keyUsage = nonRepudiation
		subjectAltName = @alt_names
		[alt_names]
		DNS.1 = kubernetes
		DNS.2 = kubernetes.default
		DNS.3 = kubernetes.default.svc
		DNS.4 = kubernetes.default.svc.cluster.local
		IP.1	= 10.96.0.1
		IP.2	= 172.17.0.87


Use the openssl.cnf to generate the csr:

`openssl req -new -key apiserver.key -subj “/CN=kube-apiserver” -out apiserver.csr -config openssl.cnf`

	

Viewing Certificates:

The kube-apiserver.yaml will have all the certificate file references for the apiserver in it.

`cat /etc/kubernetes/manifests/kube-apiserver.yaml`

Same for the etcd server:
	
`cat /etc/kubernetes/manifests/etcd/yaml`

To view a specific certificate:
	
`Openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout`






Generate a Certificate through kubernetes cert API:

Make a csr.yaml and use that to make the certificate:

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
Metadata:
    name: jane
spec:
    groups:
      system:authenticated
    usages:
      digital signature
      key encipherment
      server auth
    request:
        (use the base64 encoded version of the csr here)
        Cat whatever.csr |base64
```

Once the certificate request has been made you can view it by using kubectl:

`kubectl get csr`

You can approve or deny it from kubectl as well:

`kubectl certificate approve <name>`

The certificate contents can be viewed with -o yaml:

`kubectl get csr jane -o yaml`

The output will give the certificate in the base64 encoded form. It will still need to be decoded using base64:

`echo “<base64 certificate>” | base64 --decode`

  
  
  # KubeConfig:
  
  
  https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
  
  
  By default `kubeconfig` looks for a configuration file in the users home directory. `$home/.kube/config`. The config file will contain certain variables such as which server to run on, certificate, key and ca cert.
  
  
  The config file is separated into 3 different sections:
  
  ### Clusters:
  
  These are the various k8s clusters you need access to.
  
  *Example*:
  
  ```
   Dev
   Production
   Staging
  ```
  ### Users:
  
  The user accounts with which you have access to these clusters.
  
  *Example*:
  
  ```
  Admin
  Dev user
  Prod User
  ```
 
 ### Contexts:
  
  Contexts put the cluster and users together. Such as admin@production.
  
  
  `$home/.kube/config`
 
 In the file you can specify which cluster to use by adding in the `current-context` line in your config. Can specify a certain namespace within the context field using `namspace` 
  
 ```yaml
  apiVersion: v1
  kind: Config
  
  current-context: my-kube-admin@my-kube-playground
  
  clusters:
  - name: my-kube-playground
    cluster:
      certificate-authority: ca.crt
      server: https://my-kube-playground:6443
     
  contexts:
  - name: my-kube-admin@my-kube-playground
    context:
      cluster: my-kube-playground
      user: my-kube-admin
      namespace: finance
      
  users:
  - name: my-kube-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
 
 ```
  
  
  
  View the current config file through kubectl:
  
  `kubectl config view`
  
  View a config file outside of the default:
  
  `kubectl config view --kubeconfig=my-custom-config`
  
  
  To change contexts from default to a different context:
  
  `kubectl config use-context prod-user@production`
  
  Change contexts from default to a different context using a different config file:
  
  `kubectl config --kubeconfig=/path/to/file use-context my-context`
  
  *Example*:
  
  `kubectl config --kubeconfig=/root/my-kube-config use-context research`
  
  
  
  
  
  # API GROUPS:
  
  
  https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning
  
  
  *Note*: You cannot access these apis (aside from smaller ones like version) without authenticating first. You can either pass in your certificate details with each command, or you can start up a `kubectl proxy service` which will load up the details of your certificates from your kubeconfig file. The proxy launches on localhost on port 8001.
  
  `kubectl proxy`
  
  
  *Note:* `kubectl proxy` and `kube proxy` are not the same. `kubectl proxy` is used to access the kubeapi server. `kube proxy` is used to enable connectivity between nodes and services on the cluster. 
  
  Using kubeproxy to access master node group apis:
  
  `curl http://localhost:8001 -k`
  
  
  Default address to view the master node version:
  
  `curl https://kube-master:6443/version`
  
  View the master node pods:
  
  `curl https://kube-master:6443/api/v1/pods`
  
  
  
  
  ### /API : Core
  
  Responsible for all core functionality
  
  ```
  namespaces
  events
  bindings
  configmaps
  pods
  endpoints
  secrets
  nodes
  services
  ect...
  ```
  
  
  ### /APIS: named
  
  More organized grouping of apis. Has api groups ordered in a better way with resources under those groups:
  
  
  
  ```
  /apps
  	/v1
      /deployments
      /replicasets
      /statefulsets
      
  /networking.k8s.io
  	/v1
      /networkpolicies
      
  /certificates.k8s.io
  	/v1
      /certificatesigningrequests
  ```
  
  Each of these has their own set of sub commands (verbs) that can be used to perform certain actions:
  
  ```
  list
  get
  create
  delete
  update
  watch
  ```
  
  To get a list of all the API groups from the master node:
  
  `curl http://localhost:6443 -k`
  
  To get a list of resources for any of those groups:
  
  `curl http://localhost:6443/{grpup} -k |grep "name"`
  
  *Example*:
  
  `curl http://localhost:6443/apis -k |grep "name"`
  
  
  
  
  # RBAC:
  
  Create a yaml file for roles:
  
  
  ### Role:
  Create the role and actions for the role
  
  *developer-role.yaml*
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: blue # specify namespace for role
    name: developer # name of the role
  rules: # rules for each resource that the role will access
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
    resourceNames: ["blue", "orange"]
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["create"]
  - apiGroups: ["apps", "extentions"]
    resources: ["deployments"]
    verbs: ["create"]
  ```
  
  *Note:* If the role is for one of the core groups, the `apiGroups` rule can be left blank
  
  *Note:* can also restrict further down to specific pods (as in the example above by adding in `resourceNames`
  
  
  ### Role Binding
  
  Link a user object to a role
  
  *devuser-developer-binding.yaml*
  
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
  	namespace: blue # Specify namespace
  	name: devuser-developer-binding
  subjects:
  -	kind: User
  	name: dev-user
  	apiGroup: rbac.authorization.k8s.io
  roleRef:
  	kind: Role
  	name: developer
  	apiGroup: rbac.authorization.k8s.io
  ```
  
  
  Get roles in default ns:
  
  `kubectl get roles`
  
  Get role bindings in default ns:
  
  `kubectl get rolebindings`
  
  More details on specific role:
  
  `kubectl describe role {role name}`
  
  Check to see if you have permissions for a certain role/verb
  
  `kubectl auth can-i create deployments`
  
  Can also impersonate a user:
  
  `kubectl auth can-i create deployments --as dev-user`
  
  
  Check API server `kube-api` for current auth methods set
  
  `kubectl describe pod kube-apiserver-master -n kube-system`
  
  
# Securing Images:


  Whenever an image is specified within a pod definition file:
  
  ```yaml
    image: nginx
  ```
  
  It is assumed the image will be pulled from docker.io. The full Url would be
  
  `docker.io/nginx/nginx`
  
  registry/user/image
  
  to specify a private regitry for images, the full path to the registry will need to be added to the image:
  
  ```yaml
    image: private-registry.io/app/internal-app
  ```
  
  
  
 To authenticate against the private registry, a secret will need to be created:
 
 `kubectl create secret docker-registry regcred --docker-server=private-registry.io --docker-username=registry-user --docker-password=registry-password --docker-email=registry-user@email.org `
 
 
  After the secret is created, add to the definition file:
  
  ```yaml
  spec:
    containers:
      - name: foo
        image: private-registry.io/app/internal-app
    imagePullSecrets:
      - name: regcred
  ```
  
  
  
  
# Cluster roles and bindings


https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding


There are a couple different bindings `namespaced` and `non-namespaced` 

```
Namespaced: bindings that are specific to a namespace
	IE: pods, replicasets, deployments, services
  
Non-namespaced: bindings that are cluster wide. 
	IE: nodes, certificatesigningrequests, namespaces
```

To see a full list of namespaced and non-namespaced resources run:

`kubectl api-resources --namespace=true` or `--namespace=false`
  
  
### clusterroles:

similar to `roles` however, they are used for cluster scoped resources such as nodes or namespaces.
  
  
  cluser-admin-role.yaml
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
  	name: cluster-administrator
  rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["list", "get", "create", "delete"]
  ```
  
  
### clusterrolebinding:

Links the user to the role:


cluser-admin-role-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io

```
  
  
# Security Contexts


https://kubernetes.io/docs/tasks/configure-pod-container/security-context/


As a general note, if you cannot get the information about a certain container setting or feature, you can always `exec` into the container and try to get it. 

IE: to see what user a container is running commands as:

`kubectl exec ubuntu-sleeper -- whoami`

```
master $ k exec ubuntu-sleeper -- whoami
root
master $
```

Can configure security settings at either a pod or container level. If settings are configured on both pod and container levels, the container settings will override the pod level settings. 

Pod level applies settings to all containers within the pod.

*Note:* 
If modifying an existing pod, make sure to edit the existing `securityContext` and not add a new one. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```


Container level applies settings to that specific container. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

  
  

# Networking Policies:


https://kubernetes.io/docs/concepts/services-networking/network-policies/


Two types of traffic:

### Ingress:

Incoming traffic from the user to the server

### Egress:

Outgoing traffic from server to API server or back to user. 


By default kubernetes has an `all allow` rule for networking that allows all pods/services to talk to one another throughout the cluster.

Create a network policy:

```yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: internal
  policyTypes:
  - Ingress
  - Egress
  ingress: # if no ingress use -{} or exclude ingress from yaml file
  - from: # Specifies where the traffic is allowed to come from
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to: # Specifies where the traffic is allowed to go
    - podSelector:
        matchLabels:
          name: mysql
        
    ports:
    - protocol: TCP
      port: 3306
      
  - to:
    - podSelector:
        matchLabels:
          name: payroll
          
    ports:
    - protocol: TCP
      port: 8080
      
  - ports: # This is added for dns resolution
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```
  
  
See network policies:

`kubectl get networkpolicy`


  
  
# Storage:


https://kubernetes.io/docs/concepts/storage/



### Docker Storage:

Docker stores data on the local file system by creating a folder structure at `/var/lib/docker` . There are multiple folders underneath this main folder. 
  
  
  
  Create a volume in docker:
  	`docker volume create data_volume`
    
  This will create a volume directory in `/var/lib/docker/volumes` called `data_volume`
  
  
  Run a container with the volume created:
  
  `docker run -v data_volume:/var/lib/mysql mysql`
    
  This will mount the volume in the dockers container read/write layer. It will mount it to `/var/lib/mysql` within the container. Now all data written to `/var/lib/mysql` in the container will be written to the volume. 
  
  
  Two types of mounting in docker:
  
  Volume Mounting:
  	mounts a volume from the volumes directory. 
  
  Bind Mounting:
    mounts a directory from any location on the docker host. 
    
    
  There is a more verbose way of mounting by using `--mount`
  
  `docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql`
  
  
  
  
### Volumes:

https://kubernetes.io/docs/concepts/storage/volumes/


Simple volume mount:

The command listed in the container will write to the data-volume in the file `/opt/number.out`  on the container. This is a volume mount to `/data` on the host. 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
      
  volumes:
  - name: data-volume
    hostPath: # This will configure a dir on the host for storage
      path: /data
      type: Directory
```
  
The `hostPath` option is fine for a single node setup, but for a multinode setup it is better to use a distributed storage option. These include `aws ebs`, `gcp persistent storage`..ect...
  
  
Configure a volume to use `aws ebs`

```yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```
  

### Persistent Volumes:

https://kubernetes.io/docs/concepts/storage/persistent-volumes/

A persistent volume is a cluster wide pool of volumes configured by an admin to be used by users deploying applications on the cluster. 

View persistentVolumes:

`kubectl get persistentvolumes`


*pv-definition.yaml*

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes: 
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
  
```



There are a few supported `accessModes`:

*The access modes of the volume and volume claim must match*

https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes

```
ReadOnlyMany (ROX) - Volume can be mounted read-only by many nodes
ReadWriteOnce (RWO)- Volume can be mounted as read-write by a single node
ReadWriteMany (RWX)- Volume can be mounted as read-write by many nodes
```

*Important! A volume can only be mounted using one access mode at a time, even if it supports many. For example, a GCEPersistentDisk can be mounted as ReadWriteOnce by a single node or ReadOnlyMany by many nodes, but not at the same time*




### Persistent Volume Claims


https://kubernetes.io/docs/concepts/storage/persistent-volumes/#lifecycle-of-a-volume-and-claim


Persistent Volume Claims (PVC) are separate from Persistent Volumes (PV) themselves.

Every PVC is bound to a single PV there cannot be multiple claims on the same PV. 

View PVC:
`kubectl get persistentvolumeclaim`

*pvc-definition.yaml*

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Delete a PVC:

`kubectl delete persistentvolumeclaim myclaim`

When a PVC is deleted the PV is still intact. There are options that can be added within the definition file to instruct on what to do. The default behavior is `Retain`

`persistentVolumeReclaimPolicy: Retain` 

An interesting command to change the reclaim polity:

`kubectl patch pv <your-pv-name> -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'`


Different Types:

```
Retain - When the PersistentVolumeClaim is deleted, the PersistentVolume still exists and the volume is considered "released".
Recycle - (deprecated) a basic scrub of the volume after claim deletion. (rm -rf /thevolume/*
Delete - deletion removes both the PersistentVolume object from Kubernetes, as well as the associated storage asset in the external infrastructure.
```

Once the PVC is created, use it in a pod definition file by specifying the PVC claim under `persistentVolumeClaim`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
  
  
### Storage classes:


https://kubernetes.io/docs/concepts/storage/storage-classes/


*Static Provisioning*:

When PVs are defined within manifests, the storage disk must be manually created first. 

*Dynamic Provisioning*

Define a provisioner such as gcp that can automatically provision storage and attach to pods when a claim is made. 

To see the storage classes:

`kubectl get sc`


*sc-definition.yaml*

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd

parameters:
  type: pd-standard
  replication-type: none
```

The `parameters` option is very specific to the provisioner so you need to look it up based on which provisioner is used. 


In the claim definition file, use the `storageClassName` to use the storage class.

```yaml
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```

  
  
  
# Networking:


https://kubernetes.io/docs/concepts/services-networking/network-policies/


Kubernetes networking model:

```
1. Every Pod should have an IP address
2. Every Pod should be able to communicate with every other Pod in the same node
3. Every Pod should be able to communicate with every other Pod on other nodes without NAT
```
  
  
Check to see which network plugin is used on the node:

`ps aux |grep network-plugin`

(CNI): Important Directories

`/etc/cni/net.d` contains configuration file

`/opt/cni/bin` contains executable files
  
  
  
  
### CoreDns file location

`/etc/coredns/Corefile`


  
# Ingress


https://kubernetes.io/docs/concepts/services-networking/ingress/


An api object that manages external access to services within a cluster. (Typically HTTP)

Ingress may provide loadbalancing, SSL termination and name-based virtual hosting. Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controller by rules defined on the ingress resource. 

### Ingress Controller:


https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/


In order for the ingress resource to work, the cluster must have an ingress controller running. unlike other typs of controllers which run as a part of the `kube-controller-manager` binary, Ingress controllers are not started automatically with a cluster. 


Ingress controller needs a few things to operate correctly:

```
Deployment
Servic
ConfigMap
Auth
```


`ingress controller example`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
      args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
      
      env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
      
      ports:
        - name: http
          containerPort: 80
        -name: https:
          containerPort: 443
```

ConfigMap to store any variables:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
```

  
Ingress service to expose the ingress controller to outside world:

`ingress-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
  namespace: whatever-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

Can also use: 

`kubectl -n {namespace} expose deployment {deploymentName} --name name --port 80 --target-port 80 NodePort --dry-run=client -o yaml >ingress-svc.yaml`

This will create a service yaml file that exposes the deployment on port 80 with a target port of 80.
  
Service Account with the right set of permissions is needed for the controller to assign and perform all tasks needed to configure the underlying nginx server when something has changed. 
  
  
  
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```
  


### Ingress Resources


https://kubernetes.io/docs/concepts/services-networking/ingress/#resource-backend


A set of rules and configurations applied on the ingress controller. This is where you can configure different portions of the site to point to different resources on the cluster. It is done through a definition file.

To separate traffic to go to a certain place: IE example.com/app goes to pod 1, a rule will need to be defined.


`basic ingress file`:
```yaml
apiVersion: extentions/v1beta1
kind: Ingress
metadata: 
  name: ingress-example
spec:
  backend:
    serviceName: example-service
    servicePort: 80
```
  
`ingress file with rules added`:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-example-app
spec:
  rules:
  - http:
      paths: 
      - path: /app
        backend:
          serviceName: example-app
          servicePort: 80
```
  
`ingress file that uses domain names`:


```yaml
apiVersion: extentions/v1beta1
kind: Ingress
metadata:
  name: ingress-example-app
spec:
  rules:
  - host: example.com/app
    http:
      paths: 
      - backend:
          serviceName: example-app
          servicePort: 80
```
  

Using a rewrite-target:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```
  
  
  
# Cluster setup via kubeadm

https://kubernetes.io/docs/reference/setup-tools/kubeadm/
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/


1. Designate vms for the cluster. IE: master and workers:
2. install the container runtime engine (Docker):
3. install kubeadm: This needs to be done on both master and worker nodes.: 
	*https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/*
  
  a. Find which OS the system is on `cat /etc/os-release`
  
  b. Follow install instructions for that particular OS
  
  c. verify kubeadm install `kubeadm version -o short`
  
  d. verify version of kubelet `kubelet --version`
    
4. initialize the master node:
	*https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/*
  
  a. initialize with `kubeadm init`
  
  b. Once completed it will give you a command to run that copies the kube config file to the default directory
  	
    `mkdir -p $HOME/.kube`
    
    `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
    
    `sudo chown $(id -u):$(id -g) $HOME/.kube/config`
  
5. configure POD network:
	*https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network*
  
  
  *https://kubernetes.io/docs/concepts/cluster-administration/addons/*
  
	a. Follow the instructions for whatever addon you want to put on. 
6. join worker nodes to master:
 
  a. the output of the kubeadm init command will usually give you a join command. Otherwise a new token can be generated to be able to join. 
  
  b. `kubeadm token create --print-join-command`
  
  
  
  
# JSON Path


https://kubernetes.io/docs/reference/kubectl/jsonpath/

  
  get information on resources in json format
  
  `kubectl get pods -o json`
  
  use json path query language to get specific elements from a get command
  
  `kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'`
  
  Get all node names:
  
  `kubectl get nodes -o=jsonpath='{.items[*].metadata.name}'`
  
  Get node architecture:
  
  `kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}'`
  
  Get node cpu capacity:
  
  `kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}'`
  
  Can use two queries in one command:
  
  `kubectl get nodes -o=jsonpath='{items[*].metadata.name}{.itmes[*].status.capacity.cpu}'`
  
  Can also format the line to look better:
  
  `kubectl get nodes -o=jsonpath='{items[*].metadata.name} {"\n"} {.itmes[*].status.capacity.cpu}'`
  
  A more advanced method of iterating through objects and formatting them:
  
  `kubectl get nodes -o=jsonpath=` This is assumed
  
  `'{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\t"}` This is appended to the `-o=jsonpath=`
  
  
  Can also use `-o custom-columns=` to format in a column style
  
  `kubectl get nodes -o=custom-columns=<columnName>:<jsonPath>`
  
  Example:
  
  `kubectl get nodes -o=custom-columns=NODE:metadata.name, CPU:status.capacity.cpu`
  
  
  Sorting is an option:
  
  `kubectl get nodes --sort-by=metadata.name`
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
