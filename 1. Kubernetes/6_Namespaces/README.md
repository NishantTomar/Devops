# Namespaces  

It is just creating different Namespace to isolate the resources.  

By default all the resources that we create are created in **default** Namespace.

Kubernetes creates a set of pods and services for its internal purpose, such as those required by the networking solution, the DNS service, etc. To isolate these from the user and to prevent you from accidentally deleting or modifying these services, Kubernetes creates them under another name space created at cluster startup named **kube-system**.

A third name space created by Kubernetes automatically is called **kube-public**.
This is where resources that should be made available to all users are created.  

Within namespace we use the names for resources to reffer each other.

Suppose we have below resources in the **DEFAULT** Namespace.
- web-pod
- db-service
- web-deployment

And below resources in the **DEV** Namespace.
- web-pod
- db-service
- web-deployment

1. So if web-pod needs to connnect to db-service within same namespace it uses.  
    `mysql.connect("db-service")`

2. If web-pod (DEFAULT Namespace) needs to connnect to db-service that is in DEV namespace it uses.  

    `mysql.connect("db-service.dev.svc.cluster.local")`  

    `               <service-name>.<Namespace>.<service>.<domain> ----> "cluster.local" is the domain here.`

3. Get the pods of default namespace.  

    `kubectl get pods`

4. Get the pods of another namespace.  

    `kubectl get pods --namespace=<namespacename>`   

5. How to create a pod in a particular namespace.  
    
    `kubectl create -f pod-defnation.yaml --namespace=<namespacename>`

    **OR**

    *write the namespace in the pod definition file only under metadata and use the command*
    
    `kubectl create -f pod-definition.yaml` 

    #### [pod-definition.yaml](pod-definition.yaml)  

    ```
    apiVersion: v1
    kind: Pod
    metadata:
    name: myapp-pod
    namespace: dev
    labels:
        type: front-end
    spec:
    containers:
        - name: nginx-container
        image: nginx
    ```

6. Create namespace.  

    `kubectl create namespace dev`  

    `kubectl create namespace <namespacename>`  

    *By using a file*

    #### [namespace-definition.yaml](namespace-definition.yaml)
    ```
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
    ```  

    `kubectl create -f namespace-definition.yaml`

7. How to switch to a particular namespace permanently, so that we no need to specify namespace option in all the commands.  

    `kubectl config set-context $(kubectl config current-context) --namespace=dev`

8. To get the pods of all the namespaces.  

    `kubectl get pods --all-namespaces`

9. How to list all the namespaces.  

    `kubectl get namespaces`