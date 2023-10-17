# Configuring Env Variables

To set environment variables in Kubernetes, you can use the `env` or `envFrom` field in the configuration file. Here are the steps to set an environment variable in Kubernetes:

1. Create a pod definition file that uses the same image as the Docker command.
2. Use the `env` property to set an environment variable. `env` is an array, so every item under the `env` property starts with a dash indicating an item in the array. Each item has a name and a value property. The name is the name of the environment variable made available with the container, and the value is its value.

#### [pod-with-env.yaml](pod-with-env.yaml)
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
    - name: my-webapp-container
      image: my-webapp
  env:
    - name: APP_COLOR
      value: pink
```

3. Alternatively, you can set environment variables using config maps and secrets. Instead of specifying a value, you say "value from" and then a specification of config map or secret.

Here are the different ways to set environment variables in Kubernetes:

- Using the `env` field: This is the direct way of specifying environment variables using your plain key-value pair format.
- Using `envFrom` field: This allows you to set environment variables for a container by referencing either a ConfigMap or a Secret.
- Using dependent environment variables: You can use `$(VAR_NAME)` in the value of `env` in the configuration file to set dependent environment variables.

**Env variable**
```
env:
- name: APP_COLOR
    value: pink
```

**ConfigMaps**
```
env:
- name: APP_COLOR
    valueFrom:
    configMapKeyRef: 
```

**ConfigSecrets**
```
env:
- name: APP_COLOR
    valueFrom:
    SecretKeyRef: 

```

In most cases, the types of data defined using environment variables in Kubernetes can also be configured using other methods. However, environment variables play an important role in Kubernetes, and you can use them not only to provide basic information about the operating system to your application but also as the main.

