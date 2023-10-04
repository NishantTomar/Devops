# Services  

Services enable the frontend application to be made available to end users. It helps communication between backend and frontend Pods, and helps in establishing connectivity to an external data source.

Few main services are listed below:  

1. NodePort
2. ClusterIp
3. LoadBalancer 

## NodePort  

This service listens to a port on the node and forwards request to the Pods.
NodePort where the service makes an internal port accessible on a port on the node.
The service is, in fact, like a virtual server inside the node. Inside the cluster it has its own IP address, and that IP address is called the ClusterIP of the service.
Node ports can only be in a valid range which by default is from 30,000 to 32,767.


### [nginx-pod-definition.yaml](../1_Pods/nginx-pod-definition.yaml)  

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

### [nodeport-service.yaml](nodeport-service.yaml)  
 
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  ports:
    - targetPort: 80 #port on the pod
      port: 80  #Service port
      nodePort: 3008  #node port
  selector: 
    name: app 
    env: stage

```

1. How to create a service using .yaml.  

    `kubectl create -f nodeport-service.yaml`  

2. How to get the service.  

    `kubectl get service`  

3. How to get the information of the service.  

    `kubectl describe service`


## ClusterIp

This Service is used for internal communication within the cluster.

The pods all have an IP address assigned to them, but these IPs, as we know, are not static. These pods can go down anytime and new pods are created all the time and so you cannot rely on these IP addresses for internal communication between the application. Thats where we need clusterIp service for internal or backend communication between the pods.  

### [clusterip-service.yaml](clusterip-service.yaml)  

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    name: app 
    env: stage

```

## LoadBalancer

With nodeport our application is accessible at particular port of a node. And if we have multiple nodes then there are many ips:port to access the application. But end user just need a url to access our application as they cannot rember the multiple combinations of ips and ports. So, This is where we use LoadBalancer service to provision the load balancer on the cloud and configure it to expose the application to external world. 
By using the domain name in route53 to route traffic to the loadbalancer and load balancer to the kubernetes cluster pods.


### [loadbalancer-service.yaml](loadbalancer-service.yaml)  

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80
      port: 80
      nodePort: 3008
  selector:
    name: app 
    env: stage
```

> If we dont specify any type of service under spec it creates **ClusterIp** service by default.