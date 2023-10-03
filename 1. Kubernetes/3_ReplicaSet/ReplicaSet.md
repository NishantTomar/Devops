# ReplicaSet  

ReplicaSet also works similar to the ReplicaController. The only one change is that it uses "selector" keyword to match the "lables" to filter out the pods that it will monitor for replicas.
And it may even monitor the pods that were created before the creation of replicaset, if the labels matches with them.
The pod spec defination is required to be defined in the replicaset definition file because replicaset must know which pods it need to spin up once they are destroyed.

### [replicaSet-definition.yml](replicaset-definition.yaml)  

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    env: dev
    appname: myapp
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

1. Create replicaset using .yaml file.  

    `kubectl create -f replicaset-definition.yaml`  

2. How to see the replicaset.  

    `kubectl get replicaset`  

3. How to scale up the replicas?  

   - Edit the replicas in the replicaset-definition.yaml file and use below command to replace the replicaset defination file to have the redefined replicas.  

        `kubectl replace -f replicaset-definition.yaml`  

   - Use the scale command to update the replicas but this will not update the replicas value in the replicaset-defination.yaml file.  

        `kubectl scale --replicas=6 -f replicaset-definition.yaml`  
    
        or  

        `kubectl scale --replicas=6 replicaset myapp-rs`
                                      type      Name(of replicaset)  

4. How to delete replicaset?  

    `kubectl delete replicaset myapp-rs` *also deletes underlying pods*  

5. How to see the information about replicaset.  

    `kubectl describe replicaset <replicaset-name>`  

6. How to edit replicaset.  

    `kubectl edit replicaset <replicaset-name>`  
    
