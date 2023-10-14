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

## Taint and Tolerations

- We have a cluster with three worker nodes (one, two, and three) and four pods (A, B, C, and D).  

- Initially, the scheduler places the pods across all nodes to balance them out equally.  

- We want to dedicate node one to a specific application, so we add a taint (frontend) to the node.  

    `kubectl taint nodes node-name key=value:taint-effect`  

    `kubectl taint nodes node1 app=frontend:noschedule`  

- None of the pods can tolerate the taint by default, so none of them can be placed on node one.  

- We add a toleration to pod D, making it tolerant to the taint frontend.

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
  tollerations:
    - key: "app"  #we can either use "" or without that also it will work.
      operator: "Equal"
      value: "frontend"
      effect: "NoSchedule"
```


- Now, when the scheduler tries to place pod D on node one, it can be scheduled because it is tolerant to the taint.

> Taint is for Nodes and Toleration is for Pods.  

### Taint effects:
1. No schedule: Pods will not be scheduled on the node.

2. Prefer no schedule: The system will try to avoid placing a pod on the node, but it's not guaranteed.

3. No execute: New pods will not be scheduled on the node, and existing pods on the node may be evicted if they do not tolerate the taint.

### Taints and tolerations configuration:
- Use the kubectl taint nodes command to taint a node.  
  
  `kubectl taint nodes node-name key=value:taint-effect` 

- How to remove the taints.  

  `kubectl taint nodes node-name key=value:taint-effect-`

- Tolerations are added to pods in the pod definition file as shown in above file.  

> Taints and Tolerations does not tell the pod to go to a particular node. Instead, it tells the node to only accept pods with certain tolerations. If your requirement is to restrict a pod to certain nodes, it is achieved through another concept called as node affinity.

### Master nodes:
- Master nodes in the cluster have a taint that prevents any pods from being scheduled on them by default.

- To see master node taint run the below command.  

  `kubectl get node kubemaster| grep Taint`  

  `>> Taints:  node-role.kubernetes.io/master:NoSchedule`  

## Node Selectors  
 
In the three-node cluster, you have two smaller nodes with lower hardware resources and one larger node with higher resources. You have different types of workloads running in the cluster, and you want to dedicate the data processing workloads that require higher resources to the larger node. However, in the current setup, any pods can go to any nodes, which is not desired. To solve this, you can set a limitation on the pods so that they only run on specific nodes.  

There are two ways to do this:  

1. **Node Selectors**: This is the simpler and easier method. You can use labels and selectors to specify which nodes the pods should run on. First, label your nodes with key-value pairs. For example, you can label the larger node as "size=large". Then, in the pod definition file, add a new property called "nodeSelector" to the "spec" section and specify the size as "large". This way, the scheduler will match the labels and place the pods on the appropriate nodes.  

    `kubectl label nodes node-name label-key=label-value`  

    `kubectl label nodes node01 size=large`

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
      nodeSelector:
        size: large
    ```


2. **Node Affinity and Anti-Affinity**: Node selectors have limitations when the requirements are more complex. In such cases, you can use node affinity and anti-affinity features. These allow you to specify more advanced rules for pod placement, such as placing the pod on a large or medium node or placing the pod on any node that is not small.

For more content, Please refer the official documentaion of [NodeSelectors](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector)

## Node Affinity

- The primary purpose of the node affinity feature is to ensure that pods are hosted on particular nodes, such as placing a large data processing pod on node1.  

- Node selectors are a simple way to achieve this, but they don't support advanced expressions like "or" or "not".  

- Node affinity provides advanced capabilities for limiting pod placement on specific nodes, although it can be more complex to use.  

- Under the pod's specification, you have the "affinity" section, which contains the "node affinity" property.  

- The "node affinity" property has a sentence-like sub-property called "required during scheduling, ignored during execution", which defines the behavior of the scheduler with respect to node affinity and the stages in the pod's life cycle.  

- There are currently two types of node affinity available: "required during scheduling, ignored during execution" and "preferred during scheduling, ignored during execution".  

- During scheduling, if a pod's affinity rules cannot be matched to any available nodes, the "required" type will prevent the pod from being scheduled, while the "preferred" type will allow the scheduler to place the pod on any available node.  

- During execution, if a change in the environment affects node affinity (e.g., a label change on a node), the "ignored" value for the two existing types means that running pods will not be impacted by the change.  

- In the future, new types of node affinity are planned, with the main difference being their behavior during the execution phase. For example, a new option called "required during execution" could evict pods that are running on nodes that no longer meet the affinity rules.  


#### [node-affinity-definition.yaml](node-affinity-definition.yaml)  

```
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
  containers:
  - name: with-node-affinity
    image: nginx
```

For more content, Please refer the official documentaion of [NodeAffinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

## Combination of Taints/Tolleration and NodeAffinity

- We have three nodes and three pods each in three colors: blue, red, and green.  

- The ultimate aim is to place the blue pod in the blue node, the red pod in the red node, and likewise for green.  

- We are sharing the same Kubernetes cluster with other teams, so there are other pods in the cluster as well as other nodes.  

- We do not want any other pod to be placed on our node, neither do we want our pods to be placed on their nodes.  

1. Let us first try to solve this problem using taints and tolerations.  

    1. Apply a taint to the nodes marking them with their colors (blue, red, and green).  

    2. Set a toleration on the pods to tolerate the respective colors.  

    3. When the pods are created, the nodes ensure they only accept the pods with the right toleration.  

    4. The green pod ends up on the green node, and the blue pod ends up on the blue node.  

    5. However, taints and tolerations do not guarantee that the pods will only prefer these nodes.  

    6. The red pod ends up on one of the other nodes that do not have a taint or toleration set. This is not desired.  

2. Let us try to solve the same problem with node affinity.  

    1. Label the nodes with their respective colors (blue, red, and green).  

    2. Set node selectors on the pods to tie the pods to the nodes.  

    3. The pods end up on the right nodes.  

    4. However, this does not guarantee that other pods are not placed on these nodes.  

    5. There is a chance that one of the other pods may end up on our nodes. This is not something we desire.  

3. A combination of taints, tolerations, and node affinity rules can be used together to completely dedicate nodes for specific pods.  

    1. Use taints and tolerations to prevent other pods from being placed on our nodes.  

    2. Use node affinity to prevent our pods from being placed on their nodes.  


## Resource Requirements and Limits

- Each node has a set of CPU and memory resources available.  

- Every pod requires a set of resources to run.  

- The Kubernetes scheduler decides which node a pod goes to based on resource availability.  

- The scheduler schedules a new pod on a node with sufficient resources.  

- If there are no sufficient resources available on any of the nodes, the scheduler holds back scheduling the pod, and the pod remains in a pending state.  


### Resource Requirements for Pods

- You can specify the amount of CPU and memory required for a pod when creating one.  

- This is known as the resource request for a container, which is the minimum amount of CPU or memory requested by the container.  

- The scheduler uses these numbers to identify a node with sufficient resources when placing the pod.  

- You can set limits for the resource usage on pods to prevent them from consuming more resources than necessary.  

### CPU and Memory Units

- One count of CPU is equivalent to one vCPU in AWS, one core in GCP or Azure, or one hyperthread on other systems.  

- You can specify CPU values as low as 0.1, which can also be expressed as 100m (milli).  

- Memory values can be specified in mebibytes (Mi), gigabytes (G), or gibibytes (Gi).

### Resource Limits

- By default, a container has no limit to the resources it can consume on a node.  

- You can set limits for CPU and memory usage on containers using the limits section in your pod-definition file.  

- If a pod exceeds its specified limit for memory, it will be terminated with an OOM (out of memory) error.

#### [resource-limit-nginx-pod-definition.yaml](resource-limit-nginx-pod-definition.yaml)

```
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels:
    name: myapp
spec:
  containers:
  - name: myapp
    image: nginx
    resources:
      requests:
        memory: "52Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80

```


### Default Configuration

- By default, Kubernetes does not have a CPU or memory request or limit set for pods.  

- This means that any pod can consume as much resources as required on any node, potentially suffocating other pods or processes.  

### CPU Requests and Limits

- Without a resource or limit set, one pod can consume all the CPU resources on a node and prevent other pods from getting the required resources.  

- If only limits are specified, Kubernetes automatically sets requests to the same as limits.  

- If both requests and limits are set, each pod gets a guaranteed number of CPU requests and can go up to the defined limits, but not more.  

- The ideal scenario is to set requests but no limits, allowing pods to use available CPU cycles as long as they are not needed by other pods.

### Limit Ranges

- Limit ranges can be used to define default values for containers in pods that are created without a request or limit specified in the pod-definition files.  

- Limit ranges are applicable at the namespace level and are defined as an object with the apiVersion set to v1, kind set to LimitRange, and a name specified.  

- The default limit and default request can be set for CPU and memory, along with a maximum and minimum limit for CPU and memory.  

- These limits are enforced when a pod is created, and if a limit range is created or updated, it only affects newer pods that are created after the limit range is created or updated.  

#### [cpu-limit.yaml](cpu-limit.yaml)
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
    - default:
        cpu: "500m"
      defaultRequest: 
        cpu: "500m" 
      max:
        cpu: "1"
      min:
        cpu: "500m"
      type: Container
```
#### [memory-limit.yaml](memory-limit.yaml)

```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
    - default:
        memory: 1Gi
      defaultRequest: 
        memory: 1Gi 
      max:
        memory: 1Gi
      min:
        memory: 500Mi
      type: Container
```

How to get the yaml file of a current running pod.  
  
  `kubectl get pod <podname> -o yaml > my-new-pod.yaml`

### Resource Quotas

- Resource quotas can be created at a namespace level to limit the CPU and memory consumption of all pods in a cluster.  

- A resource quota is a namespace level object that sets hard limits for requests and limits.  

- In the example, a resource quota is created to limit the total requested CPU to 4 and memory to 4 gibibytes in the current namespace.  

- The resource quota also defines a maximum limit of 8 CPU and 10 gibibytes of memory to be consumed by all the pods together.  

#### [resource-quotas.yaml](resource-quotas.yaml)

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 8
    limits.memory: 10Gi
    
```

## Daemon Sets

- **Deploying Pods with ReplicaSets and Deployments:**
  - We have deployed various pods on different nodes in our cluster using ReplicaSets and deployments.
  - These tools ensure that multiple copies of our applications are available across different worker nodes.

- **Introduction to DaemonSets:**
  - DaemonSets are similar to ReplicaSets but have a different deployment strategy.  

  - They help deploy multiple instances of pods, running one copy of the pod on each node in the cluster.
  
  - When a new node is added, a replica of the pod is automatically added to that node.
  
  - When a node is removed, the pod is automatically removed.
  
  - The DaemonSet ensures that one copy of the pod is always present on all nodes in the cluster.

- **Use Cases of DaemonSets:**
  
  - Deploying a monitoring agent or log collector on each node in the cluster for better cluster monitoring.
  
  - Deploying essential components like kube-proxy on every node in the cluster.
  
  - Networking solutions like Vivenet, which require an agent to be deployed on each node in the cluster.

- **Creating a DaemonSet:**
  
  - Creating a DaemonSet is similar to creating a ReplicaSet.
  
  - The DaemonSet definition file has a similar structure, with the kind set to DaemonSet instead of ReplicaSet.
  
  - The DaemonSet has a selector and a pod specification template, just like the ReplicaSet.
  
  - Ensure that the labels in the selector match the ones in the pod template.
  
  - Use the `kubectl create daemonset` command to create the DaemonSet.
  
  - Use the `kubectl get daemonset` command to view the created DaemonSet.
  
  - Use the `kubectl describe daemonset` command to view more details about the DaemonSet.

    #### [daemonset-definition.yaml](daemonset-definition.yaml)
    ```
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: my-monitoring-agent
    spec:
      selector:
        matchLabels:
          app: my-monitoring-agent
      template:
        metadata:
          labels:
            app: my-monitoring-agent
        spec:
          containers:
          - name: monitoring-agent
            image: checkmk
    ```


- **How Does a DaemonSet Work?**
  - In earlier versions of Kubernetes, you could set the node name property on the pod to schedule it on a specific node.
  - However, from version 1.12 onwards, DaemonSets use the default scheduler and node affinity rules to schedule pods on nodes.

## Static Pods

- The kubelet can manage a node independently, even without a Kubernetes cluster.

- To create pods, you need a pod definition file, but without a kube API server, you can configure the kubelet to read the pod definition files from a designated directory on the server.
    Example: `/etc/kubernetes/mainifest`

- The kubelet periodically checks this directory for files, reads them, and creates pods on the host. It can also ensure that the pod stays alive by attempting to restart it if the application crashes.

- If you make changes to any of the files within this directory, the kubelet recreates the pod for those changes to take effect. If you remove a file from this directory, the pod is deleted automatically.

- These pods created by the kubelet on its own, without the intervention from the API server or the rest of the Kubernetes cluster components, are known as static pods.

- You can only create pods this way, not replica sets, deployments, or services, as those require other cluster plane components.

- The location of the designated directory can be passed to the kubelet as an option while running the service. Add this option in **kubelet.service** file `--pod-mainifest-path=/etc/kubernetes/mainifest`. Or you can provide a pod to another config file using the config option and define the directory path as a static pod path in that file. like add this option to **kubelet.service** file `--config=kubeconfig.yaml`. And create a kubeconfig.yaml file with `staticPodPath: /etc/kubernetes/mainifest`
  

- Once the static pods are created, you can view them by running the Docker ps command, as we don't have the rest of the Kubernetes cluster yet to use the kube control command.

- The kubelet can create both static pods and pods from the API server at the same time, as it can take in requests for creating pods from different inputs.

- If the kubelet creates a static pod as part of a cluster, it also creates a mirror object in the kube API server, which you can view with the kube control get pods command on the master node.

1. How to view static pods of all namespaces.  

    `kubectl get pods -A`

    the pod names with the suffix of node names are the static pods

2. Where is the kubelet config.yaml file placed in ubuntu by default.

    `/var/lib/kubelet/config.yaml`

## Schedulers

- The default scheduler in Kubernetes distributes pods evenly across nodes and considers various conditions like taints, tolerations, and node affinity. However, if you have a specific application that requires custom scheduling, you can write your own scheduler program, package it, and deploy it as the default scheduler or as an additional scheduler in the Kubernetes cluster.

- Your Kubernetes cluster can have multiple schedulers at a time, and when creating a pod or a deployment, you can instruct Kubernetes to use a specific scheduler. Each scheduler must have a different name to be identified as a separate scheduler.

- The default scheduler is named "default scheduler," and its name is configured in a kube-scheduler configuration file. Other schedulers can have separate configuration files with their own names.

- To deploy an additional scheduler, you can use the same kube-scheduler binary as the default scheduler and point it to a custom configuration file. Alternatively, you can deploy the scheduler as a pod or a deployment, specifying the custom configuration file and the scheduler name.

- When running multiple copies of the same scheduler on different master nodes for high availability, you can use the leader elect option to choose a leader for scheduling activities.

- To use a custom scheduler, add a new field called "scheduler name" to your pod or deployment definition file and specify the name of the custom scheduler. This way, the right scheduler will be picked up during the scheduling process.

Read more on official Kubernetes documentation about [configuring multiple schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

#### Configuring Scheduler profiles

The Kubernetes scheduler is responsible for matching Pods to Nodes so that Kubelet can run them. Here is a simple example of how the Kubernetes scheduler works:

- A Pod definition file is created and the Pod is waiting to be scheduled on one of the available Nodes in the Kubernetes cluster.
- The Pod has a resource requirement of 10 CPU, so it can only be scheduled on a Node that has 10 CPU remaining.
- The Pods are sorted based on the priority defined on the Pods, and Pods with higher priority get to the beginning of the queue to be scheduled first.
- The Nodes that cannot run the Pod are filtered out.
- Nodes are scored with different weights based on the free space that it will have after reserving the CPU required for that Pod.
- The Node with the highest score is picked up, and the Pod is finally bound to that Node.

All of these operations are achieved with certain plugins. To set a priority, you must first create a priority class. The scheduling framework is a pluggable architecture for the Kubernetes scheduler that adds a new set of "plugin" APIs to the existing scheduler. You can customize the behavior of the kube-scheduler by writing a configuration file and passing its path as a command-line argument. You can even run multiple schedulers simultaneously alongside the default scheduler and instruct Kubernetes what scheduler to use for each of your Pods.


