# Scheduling

## Manual Scheduling

How a scheduler works in Kubernetes and what happens when there is no scheduler to monitor and schedule nodes.  

When a pod is created, it has a field called `NodeName` that is not set by default. The scheduler goes through all the pods and looks for those that do not have this property set. It then identifies the right node for the pod by running the scheduling algorithm. 

If there is no scheduler to monitor and schedule nodes, the pods continue to be in a pending state. 

To manually assign pods to nodes, you can set the `nodeName` field to the name of the node in your pod specification file while creating the pod. 

#### [nginx-pod-definition.yaml](nginx-pod-definition.yaml)  

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  lables:
    env: dev
    app: frontend
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
      containerPort: 8080
  nodeName: Node2

```

If the pod is already created and you want to assign the pod to a node, you can create a binding object and send a POST request to the pod's binding API with the data set to the binding object in a JSON format.

#### [pod-bind-definition.yaml](pod-bind-definition.yaml)

```
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: Node2
```
Convert the above to json and pass it in curl command as shown below.

`curl --header "Content-Type:application/json" --request POST --data '{ "apiVersion": "v1","kind": "Binding" ...]}} http://#SERVER/api/v1/namespace/default/pods/$PODNAME/binding`

