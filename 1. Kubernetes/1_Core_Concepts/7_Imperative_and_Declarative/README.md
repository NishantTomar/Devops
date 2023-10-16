# Imperative Commands  

Create objects: 

`kubectl run nginx --image=nginx`  

`kubectl create deployment --image=nginx nginx`  

`kubectl expose deployment nginx --port=80`

Update objects:  

`kubectl scale deployment nginx --replicas=4`  

`kubectl edit deployment nginx`  

`kubectl set image deployment nginx nginx=nginx:1.18`


- --dry-run: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run=client option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

- -o yaml: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

1. ### POD

    1. Create an NGINX Pod.  

        `kubectl run nginx --image=nginx`

    2. Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run).  

        `kubectl run nginx --image=nginx --dry-run=client -o yaml`


2. ### Deployment

    1. Create a deployment.  

        `kubectl create deployment --image=nginx nginx`

    2. Generate Deployment YAML file (-o yaml). Don't create it(--dry-run).  

        `kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

    3. Generate Deployment with 4 Replicas.  

        `kubectl create deployment nginx --image=nginx --replicas=4`

    4. You can also scale a deployment using the kubectl scale command.  
        
        `kubectl scale deployment nginx --replicas=4`

    5. Another way to do this is to save the YAML definition to a file and modify.  

        `kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`

        You can then update the YAML file with the replicas or any other field before creating the deployment.


3. ### Service

    1. Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379.

        `kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

        This will automatically use the pod's labels as selectors.

        OR  
        
        `kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml`  


        This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service.

    2. Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:  

        `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`

        This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.

        Or

        `kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`

        This will not use the pods labels as selectors.

        Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

Reference:
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

https://kubernetes.io/docs/reference/kubectl/conventions/


# Declarative Commands

The apply command takes into consideration 
- the local configuration file,
- the live object definition on Kubernetes,
- and the last applied configuration
before making a decision on what changes are to be made.

So when you run the apply command, if the object does not already exist the object is created.

### nginx-pod.yaml (LocalFile)
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp_pod 
  labels: 
    name: app 
    env: stage
spec:
  containers:
    - name: nginx-container
      image: nginx:18.0.9
```  

`kubectl apply -f nginx-pod.yaml`

When the object is created, an object configuration, similar to what we created locally is created within Kubernetes but with additional fields to store status of the object. This is the live configuration of the object on the Kubernetes cluster.

### LiveConfigurationFile
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp_pod 
  labels: 
    name: app 
    env: stage
spec:
  containers:
    - name: nginx-container
      image: nginx:18.0.19
status:
  conditions:
  - lastProbeTime: null
    status: "True"
    type: Initialized

```

This is how Kubernetes internally stores information about an object, no matter what approach you use to create the object. But when you use the kubectl apply command to create an object, it does something a bit more.

The YAML version of the local object configuration file we wrote is converted to a json format, and it is then stored as the last applied configuration.

### LastAppliedConfiguration

```
{
"apiVersion": "v1",
"kind": "Pod",
"metadata": {
    "name": "myapp_pod",
    "annotations" : {}, 
  "labels": {
    "name": "app",
    "env": "stage"
  } 
},
"spec": {
  "containers": [
    {
    "name": "nginx-container",
    "image": "nginx:18.0.9"
    }
  ]
}
}
```

Going forward, for any updates to the object, all the three are compared to identify what changes are to be made on the live object.

For example, say when the nginX image is updated to 1.19 in our local file and we are on the kubectl apply command, this value is compared with the value in the live configuration.

And if there is a difference, the live configuration is updated with the new value.

After any change, the last applied json format is always updated to the latest so that it's always up to date.

So, why do we then really need the last applied configuration, right?

So if a field was deleted, say for example the type label was deleted, and now when we run the kubectl apply command, we see that the last applied configuration had a label but it's not present in the local configuration.

This means that the field needs to be removed from the live configuration.

So if a field was present in the live configuration and not present in the local or the last applied configuration, then it will be left as is.

But if a field is missing from the local file and it is present in the last applied configuration, so that means that in the previous step, or whenever the last time we ran the kubectl apply command, that particular field was there and it is now being removed.

So the last applied configuration helps us figure out what fields have been removed from the local file, right?

So that field is then removed from the actual, the live configuration.

The live object configuration is in the Kubernetes memory.

But where is this json file that has the last applied configuration stored?

Well, it's stored on the live object configuration on the Kubernetes cluster itself as an annotation named a last applied configuration. So remember that this is only done when you use the apply command.

### LiveConfigurationFile
```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configurations: {"apiVersion": "v1","kind": "Pod","metadata": {    "name": "myapp_pod","annotations" : {}, "labels": {"name": "app","env": "stage"}},"spec": {"containers": [{"name": "nginx-container","image": "nginx:18.0.9"}]}}
  name: myapp_pod 
  labels: 
    name: app 
    env: stage
spec:
  containers:
    - name: nginx-container
      image: nginx:18.0.19
status:
  conditions:
  - lastProbeTime: null
    status: "True"
    type: Initialized

```

The kubectl create or replace command do not store the last applied configuration like this. So you must bear in mind not to mix the imperative and declarative approaches while managing the Kubernetes objects.

So once you use the applied command, going forward, whenever a change is made the apply command compares all three sections. The local part definition file, the live object configuration, and the last applied configuration stored within the live object configuration file, for deciding what changes are to be made
to the live configuration.