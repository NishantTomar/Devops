# Deployment  

The deployment provides us with the capability to upgrade the underlying instances seamlessly using rolling updates, undo changes, and pause, and resume changes as required.

### [deployment-definition.yml](deployment-definition.yaml)  

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
  labels:
    env: dev
spec:
  template:
    metadata:
      labels:
        env: dev
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      env: dev
```

1. How to create a deployment using .yaml file.  

    `kubectl create -f deployment-definition.yml`  

2. How to see the deployment.  

    `kubectl get deployment`  

3. How to get all(deployments,replicaset,pods...) in one command.  

    `kubectl get all`  

4. How to create a .yaml file using a command for deployment.  

    `kubectl create deployment --image=<image-name> <deployment-name> --dry-run=client -o yaml > <deployment-filename.yaml>`  

    Now you can edit it and then create the deployment using the below command.  
    
    `kubectl create -f <deploymentfilename.yaml>`  
    
