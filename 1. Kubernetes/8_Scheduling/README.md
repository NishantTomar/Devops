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

## Labels And Selectors

- Labels and selectors are a standard method to group and filter items based on different criteria. Labels are properties attached to each item, and selectors help filter these items based on the specified criteria.  

- In Kubernetes, labels and selectors are used to group and select different objects, such as pods, services, replica sets, and deployments.  

- To specify labels in Kubernetes, you can add a "labels" section under the metadata of a pod definition file and define the labels in a key-value format.  

- Kubernetes objects use labels and selectors internally to connect different objects together. For example, a replica set can be created by labeling pod definitions and using a selector in the replica set to group the pods.

1. How to select a pod with specific labels.  

  `kubectl get pods --selector app=front-end`  

2. How to select a pod with multiple labels.

  `kubectl get pods --selector env=dev,app=payment,tier=backend`  

3. How to get all the objects with specific or multiple labels.

  `kubectl get all --selector env=dev`  

  `kubectl get all --selector env=dev,app=payment,tier=backend`

## Annotations  

- Annotations in Kubernetes are used to record additional details for informational purposes, such as tool details or contact information.  

#### Example of Labels and Selectors and Annotations
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    env: dev
    appname: myapp
    annotations:  
      buildversion: 1.2.0
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        env: dev
        appname: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      env: dev
```