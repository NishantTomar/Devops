Pod is an smallest unit or entity in the Kubernetes world which may have single or multiple containers. Based on the requirement we may need a helper contianer for our main application container that might be doing some processing task for it.

In general we may have multiple containerized applications that shares common dependencies like
1. network
2. resources file/mounts/volumes
3. lifecycle (start/stop)

If there is a need of scaling up of our application we create new pods, and to scale down, we delete existing pod.


1. How to spin up a new pod?
    kubectl run nginx --image nginx
2. How to see the running pods?
    kubectl get pods

[nginx-pod-definition.yaml](nginx-pod-definition.yaml)

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
3. Create a pod from .yml file
    kubectl create -f pod-defination.yml
4. To see details about pod
    kubectl describe pod <pod_name>
    kubectl describe pod myapp_pod
    
       


