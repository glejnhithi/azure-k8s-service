#Kubernetes - Environment Variables 

##Step-01: Introduction
- Environment variables are a common way for developers to move application and infrastructure configuration into an external source outside of application code.
- A common reason to do this is to enable easier switching between environments.
- When an application runs on a machine, the variable becomes part of the environment a process runs in.
- Typically you can set these variables directly from a terminal, from a configuration file in a home directory, or using other tools

## Step-02: Set Environment variables to file
```
conatiners:
- name:
  image: 
  env: 
  - name: <name of Variable>
    value: <data>
```
## Step-03: Create & Test 
```
# Create All Objects
kubectl apply -f kube-manifests-env/

# List Pods
kubectl get pods

# Display the environment variables within the pod
kubectl exec <pod name> -- printenv 
or 
# Login to pod and exec command ENV
kubectl exec -it <pod name> -- bash 
execute "env"

# Display the environment variables within the pod
kubectl logs <pod name> / kubectl logs -f <pod name>
```
-----------------------------------------------------------------
#Kubernetes - ConfigMaps

## Step-01: Introduction
- Kubernetes Secrets let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. 
- Storing confidential information in a Secret is safer and more flexible than putting it directly in a Pod definition or in a container image. 
- A ConfigMap is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed 1 MiB. If you need to store settings that are larger than this limit, you may want to consider mounting a volume.

## Step-02: Create Config Map (alias cm)
### Create Kubernetes ConfigMap manifest
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-pod
data:
   # property-like keys; each key maps to a simple value
  player_nr: "3"
  ui_properties_file_name: "user-interface.properties"

  # file-like keys
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true   
```
### Create ConfigMap with interactive method
```
# Create from-literal

kubectl create configmap <name> --from-literal=<key=pair> --from-literal=<key=pair>
kubectl create configmap first-cm --from-literal=kye=2 --from-literal=user=cm.test

# Create from-file 
kubectl create configmap <name> --from-file=/file/path --from-file=/file/path
kubectl create configmap <cm-file> --from-literal=<key=pair> --from-literal=<key=pair>
```
### Update ConfigMap as Environment variables
```
containers:
	env: 
	- name: SPECIAL_LEVEL_KEY
	  valueFrom:
		configMapKeyRef:
		  name: special-config  # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY                   
          key: special.how # Specify the key associated with the value

```
### Add ConfigMap data to a Volume
```
  containers:
    - name: test-container
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
```
### Configure all ConfigMap key-value as container environment variables
```
 containers:
    - name: test-container
      envFrom:
      - configMapRef:
            name: special-config
```
## Step-03: Create & Test
```
# Create ConfigMap with kubectl
kubectl create configmap <cm name> --from-literal=<key=value data>
kubectl create configmap first-config --from-literal=low.config=level2
#Create the pod and check
kubectl apply -f 00-cm-demo-pod.yml
# Check CM & POD ENV
kubectl logs pod-name

# Create multi ConfigMaps and config to Pod
kubectl apply -f 01-multi-cm-demo.yml
kubectl apply -f 01-multi-cm-demo-pod.yml
#check Pod and Logs
kubectl logs pod-name


```
-----------------------------------------------------------------
# Kubernetes - Secrets

## Step-01: Introduction
- Kubernetes Secrets let you store and manage sensitive information, such as passwords, OAuth tokens, and ssh keys. 
- Storing confidential information in a Secret is safer and more flexible than putting it directly in a Pod definition or in a container image. 

## Step-02: Create Secret for MySQL DB Password
### 
```
# Mac
echo -n 'dbpassword11' | base64

# URL: https://www.base64encode.org
```
### Create Kubernetes Secrets manifest
```yml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-password
#type: Opaque means that from kubernetes's point of view the contents of this Secret is unstructured.
#It can contain arbitrary key-value pairs. 
type: Opaque
data:
  # Output of echo -n 'Redhat1449' | base64
  db-password: ZGJwYXNzd29yZDEx
```
## Step-03: Update secret in MySQL Deployment for DB Password
```yml
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-db-password
                  key: db-password
```

## Step-04: Update secret in UWA Deployment
- UMS means User Management Microservice
```yml
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-db-password
                  key: db-password
```

## Step-05: Create & Test
```
# Create All Objects
kubectl apply -f kube-manifests/

# List Pods
kubectl get pods

# Get Public IP of Application
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
Username: admin101
Password: password101
```

## Step-06: Clean-Up
- Delete all k8s objects created as part of this section
```
# Delete All
kubectl delete -f kube-manifests/

# List Pods
kubectl get pods

# Verify sc, pvc, pv
kubectl get sc,pvc,pv
```