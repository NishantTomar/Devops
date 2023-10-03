# Replication Controller  

Replication controller is used to create the replicas of the defined pod in the spec section. And to make sure that desired number of replicas of the pod is running everytime.

### [replication-controller-definition.yaml](replication-controller-definition.yaml)
```
apiVersion: v1
kind: ReplicationController
metadeta:
  name: myapp-rc
  labels:
    env: dev
    appname: demorc
spec:
  template:
    metadata:
      name: myapp_pod
      labels:
        name: app
        env: dev
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```  

1. Create a replication controller from .yaml file.  

    `kubectl create -f replication-controller.yml`  

2. To see the information about replication controller.  

    `kubectl get replicationcontroller`  
      