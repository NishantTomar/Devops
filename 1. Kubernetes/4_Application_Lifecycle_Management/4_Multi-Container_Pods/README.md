# Multi-Container Pods  

The idea of breaking down a large monolithic application into sub-components known as microservices can help us develop and deploy independent, small, and reusable code. This architecture can help us scale up, down, and modify each service as required, as opposed to modifying the entire application. However, at times, you may need two services to work together, such as a web server and a logging service. Here are the steps to create multi-container pods that share the same life cycle:

1. You need one agent instance per web server instance paired together.
2. You don't want to merge and load the code of the tool services, as each of them targets different functionalities and you would still like them to be developed and deployed separately.
3. You only need the two functionalities to work together.
4. You need one agent per web server instance paired together that can scale up and down together.
5. Create multi-container pods that share the same life cycle, which means they are created together and destroyed together.
6. They share the same network space, which means they can refer to each other as localhost, and they have access to the same storage volumes.
7. To create a multi-container pod, add the new container information to the pod definition file.
8. The container section under the spec section in a pod definition file is an array, and the reason it is an array is to allow multiple containers in a single pod.
9. In this case, we add a new container named log agent to our existing pod.

By following these steps, you can create multi-container pods that share the same life cycle and work together to provide the required functionalities.
