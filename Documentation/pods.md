# Pods

* https://kubernetes.io/docs/concepts/workloads/pods/

A Pod (as in a pod of whales or pea pod) is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers. Pods are the smallest deployable units of computing that you can create and manage in Kubernetes.

### Get information about pods on a certain namespace (general information)
`kubectl get pods -n space-name`

### Describe a pod (more detailed information)
`kubectl describe po podName -n namespace` 

### Get a pod based off of label/selector 

`kubectl get pod --selector app=App1`

### Create a pod from definition file in certain namespace
`kubectl create -f definition-file.yaml --namespace=space-name`

### Assign a pod to a node if the pod is already created. Create a binding object and send a POST request to the pods binding API. 

*First thing is to create a pod binding definition file and then convert that file to json. Then send the curl request to the API* 

`curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1"....} http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/`


It is rare to create an individual pod directly in kubernetes. Typically a controller is used with a `pod template` to create a pod. Examples of a workload resource (controller for the resource) are `Deployment`, `StatefulSet`, and `DaemonSet`.

Controllers for a workload resource create Pods from a *pod template* and manage those pods on your behalf. 

*EXAMPLE simple job with a template that creates a busybox container, echos a message and sleeps*

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

Modifying the pod template or switching to a new pod template has no direct effect on Pods that already exist. If you change the pod template for a workload resource, that resource needs to create replacement pods that use the updated template. 