#POD
Pod is an smallest unit or entity in the Kubernetes world which may have single or multiple containers. Based on the requirement we may need a helper contianer for our main application container that might be doing some processing task for it.

In general we may have multiple containerized applications that shares common dependencies like
1. network
2. resources file/mounts/volumes
3. lifecycle (start/stop)

If there is a need of scaling up of our application we create new pods, and to scale down, we delete existing pod.

1. How to spin up a new pod?
```kubectl run nginx --image nginx```
2. How to see the pods on the cluster?
```kubectl get pods -n namespace```

Lets consider the below pod definition file.
####[nginx-pod-definition.yaml](nginx-pod-definition.yaml)
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
      image: nginx
```  
3. How to create a pod from .yaml file?
```kubectl create -f nginx-pod-definition.yaml```
4. To see the information about pod.
```
kubectl describe pod <pod_name>
kubectl describe pod myapp_pod
```
5. how to acces the logs of the running pod on the cluster?
```kubectl logs <pod_name>```
6. how to get more information about the pods on the cluster?
```kubectl get pods -o wide -n namespace```

7. how to delete a pod?
```
kubectl delete -f nginx-pod-definition.yaml 
or
kubectl delete <pod_name>
```
    
       


