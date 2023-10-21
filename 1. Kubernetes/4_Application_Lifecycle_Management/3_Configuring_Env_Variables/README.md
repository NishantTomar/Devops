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
      secretKeyRef: 

```

In most cases, the types of data defined using environment variables in Kubernetes can also be configured using other methods. However, environment variables play an important role in Kubernetes, and you can use them not only to provide basic information about the operating system to your application but also as the main.

## ConfigMaps

- ConfigMaps are used to pass configuration data in the form of key-value pairs in Kubernetes.
- They allow you to manage environment-specific configuration data centrally, rather than within individual pod definition files.
- There are two phases involved in configuring ConfigMaps: creating the ConfigMap and injecting it into the pod.
- You can create a ConfigMap in two ways: imperatively without using a ConfigMap definition file, or declaratively by using a ConfigMap definition file.
- To create a ConfigMap imperatively, you can use the `kubectl create configmap` command and specify the key-value pairs in the command line.  

  ```
  kubectl create configmap \
  app-config --from-literal=APP_COLOR=green
  
  #Using a file

  kubectl create configmap \
  app-config --from-file=app_config.properties
  ```
- To create a ConfigMap declaratively, you can create a definition file with the `apiVersion`, `kind`, `metadata`, and `data` fields.  

  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: myapp-config
  data:
    APP_COLOR: green
    APP_MODE: auto
  ```

- Once you have created a ConfigMap, you can inject it into a pod by adding a new property to the container called `envFrom`.

  ```
  envFrom:
  - configMapRef:
      name: myapp-config
  ```  
  
- The `envFrom` property is a list, and each item in the list corresponds to a ConfigMap item.
- You can view ConfigMaps using the `kubectl get configmaps` command.
- There are other ways to inject configuration data into pods, such as as a single environment variable or as files in a volume.  

  **Single env variable**  
  
  ```
  env:
    - name: APP_COLOR
      valuesFrom:
        configMapKeyRef:
          name: app-config
          key: App_COLOR
  ```

  **As Files in volume**  

  ```
  volume:
  - name: config-map-volume
    configMap:
      name: app-config  
  ```
## Secrets

1. Create the secret: Secrets are used to store sensitive information like passwords or keys. They're similar to ConfigMaps except that they're stored in an encoded format. There are two ways of creating a secret:

   - Imperative way: This method does not use a secret definition file. You can directly specify the key value pairs in the command line itself. To create a secret of the given values, run the `kubectl create secret generic` command. The command is followed by the secret name and the option `--from-literal`. The `--from-literal` option is used to specify the key value pairs in the command itself. For example, to create a secret by the name `app-secret` with a key value pair `DB_host=MySQL`, run the command 
   `kubectl create secret generic app-secret --from-literal=DB_host=MySQL`.  
   
   - Declarative way: This method uses a secret definition file. Create a definition file with API version, kind, metadata, and data. The API version is V1, kind is secret. Under metadata, specify the name of the secret. Under data, add the secret data in a key value format. However, while creating a secret with a declarative approach, you must specify the secret values in an encoded format. To encode the data, run the command `echo -n <text> | base64`. Replace `<text>` with the text you're trying to convert. For example, to encode `MySQL`, run the command `echo -n MySQL | base64`. Then, specify the encoded data in the definition file. To create the secret, run the command `kubectl apply -f <filename>`. Replace `<filename>` with the name of the definition file.

2. Inject the secret into a pod: To inject an environment variable, add a new property to the container called `envFrom`. The `envFrom` property is a list, so you can pass as many environment variables as required. Each item in the list corresponds to a secret item. Specify the name of the secret you created earlier. Creating the pod definition file now makes the data in the secret available as environment variables for the application.

#### secret-defination.yaml
```
  apiVersion: v1
  kind: Secret
  metadata:
    name: myapp-secret
  data:
    MYSQL_USER: root
    MYSQL_PASSWD: root@123
```

```
  envFrom:
  - secretRef:
      name: myapp-secret
```


3. Other ways to inject secrets into pods:
   - Inject as single environment variables  

    ```
    env:
      - name: MYSQL_PASSWD
        valuesFrom:
          secretKeyRef:
            name: myapp-secret
            key: MYSQL_PASSWD
    ```

   - Inject the whole secret as files in a volume. If you mount the secret as a volume in the pod, each attribute in the secret is created as a file with the value of the secret as its content.  
  ```
  volume:
  - name: secret-volume
    secret:
      secretName: myapp-secret  
  ```


4. Things to keep in mind when working with secrets:
   - Secrets are not encrypted, they're only encoded. Anyone can look up the file that you created for secrets or get the secret object and then decode it using the methods discussed before to see the confidential data. So, do not check in your secret definition files along with your code when you push to GitHub or something.
   - Secrets are not encrypted in etcd. Consider enabling encryption at rest.
   - Anyone able to create pods or deployments in the same namespace can access the secrets as well. Consider configuring role-based access control to restrict access.
   - Consider third-party secret providers, such as AWS provider or Azure provider or GCP provider or the vault provider. This way, the secrets are stored not in etcd but in an external secret provider, and those providers take care of most of the security.