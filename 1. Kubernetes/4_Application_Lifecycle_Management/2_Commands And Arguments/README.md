# Commands And Arguments
- We created a simple Docker image that sleeps for a given number of seconds and named it Ubuntu sleeper.
- We ran it using the Docker command: `docker run ubuntu-sleeper`. By default, it sleeps for five seconds, but you can override it by passing a command line argument.
- We will now create a pod using this image. We start with a blank pod definition template, input the name of the pod, and specify the image name.
- When this pod is created, it creates a container from the specified image, and the container sleeps for five seconds before exiting.
- If you need the container to sleep for 10 seconds, you can specify the additional argument in the pod definition file.
- Anything that is appended to the docker run command will go into the args property of the pod definition file, in the form of an array.
- The command field in the pod definition file overrides the cmd instruction in the Docker file.
- The command field corresponds to the entry point instruction in the Docker file.
- The args field in the pod definition file overrides the command instruction in the Docker file.
- To summarize, there are two fields that correspond to two instructions in the Docker file: the command field overrides the entry point instruction, and the args field overrides the command instruction in the Docker file.
- Remember, it is not the command field that overrides the cmd instruction in the Docker file.

In summary, to specify commands and arguments in a Kubernetes pod, you can use the command and args fields in the pod definition file. The command field overrides the entry point instruction in the Docker file, while the args field overrides the command instruction in the Docker file.
