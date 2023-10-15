# Monitoring

- Monitoring a Kubernetes cluster involves tracking resource consumption and performance metrics at the node and pod levels. 

- Node-level metrics include the number of nodes in the cluster, the number of healthy nodes, and performance metrics such as CPU, memory, network, and disk utilization.

- Pod-level metrics include the number of pods and their performance metrics, such as CPU and memory consumption.

- Kubernetes does not have a built-in monitoring solution, but there are open-source and proprietary options available, such as Metrics Server, Prometheus, the Elastic Stack, Datadog, and Dynatrace.

- Heapster, an original monitoring project for Kubernetes, is now deprecated, and a slimmed-down version called Metrics Server is used instead.

- The Metrics Server is an in-memory monitoring solution that retrieves metrics from nodes and pods, aggregates them, and stores them in memory. It does not store metrics on disk, so historical performance data is not available.

- The kubelet, an agent running on each node, is responsible for retrieving performance metrics from pods through the cAdvisor (Container Advisor) subcomponent and exposing them through the kubelet API for the Metrics Server.

- To deploy the Metrics Server, use the appropriate command for your environment (e.g., `minikube addons enable metrics-server` for minikube or clone the Metrics Server deployment files from the GitHub repository and deploy them using `kubectl create` for other environments).

- Once the Metrics Server is deployed and has collected and processed data, you can view cluster performance by running the command `kubectl top node` to see CPU and memory consumption for each node and `kubectl top pod` to view performance metrics for pods in Kubernetes.

# Logging

- I have a Docker container called event-simulator that generates random events, simulating a web server. The events are streamed to the standard output by the application.

- If I run the Docker container in the background using the -d option, I won't see the logs. To view the logs, I can use the docker logs command followed by the container ID. The -f option helps me see the live log trail.

- In Kubernetes, I create a pod with the same Docker image using the pod definition file. Once the pod is running, I can view the logs using the kubectl logs command with the pod name. Again, the -f option can be used to stream the live logs.
    `kubectl logs -f podname`

- The logs are specific to the container running inside the pod. Kubernetes pods can have multiple Docker containers in them. If I modify my pod definition file to include an additional container called image-processor, I need to specify the name of the container explicitly in the kubectl logs command to view its logs.
    `kubectl logs -f podname containername`
