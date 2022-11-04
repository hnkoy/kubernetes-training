# Kubernetes-training

This repository gathers all my Kubernetes works done during my DevOps training at EazyTraining 

# Mini project

In this project, we are going to deploy WordPress cms and a MySQL server using Kubernetes
# steps
1. create a Namespace 
2. create a Cluster Ip service 
3. create a NodePort service
4. create a Secret to store secret information
5. create a Mysql deployment manifest
6. create a WordPress deployment manifest



![enter image description here](https://raw.githubusercontent.com/hnkoy/kubernetes-training/master/k8s_infra.jpg)

# NameSpace

In Kubernetes, _namespaces_ provide a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects _(e.g. Deployments, Services, etc)_ and not for cluster-wide objects _(e.g. StorageClass, Nodes, PersistentVolumes, etc)_.

 ```
 apiVersion: v1
** specify k8s object type
kind: Namespace
**
** specify namespace name 
metadata:
  name: wordpress
**
** set status on active
status:
phase: Active
**
  ```

make sure you are in your manifest directory and run this command
 ```
kubectl apply -f app-wordpress-namespace.yml
  ```
 and run this command to show the namespace
  ```
  kubectl get ns
   ```
   
# Services

### 1.  NodePort service

 ```
apiVersion: v1
** specify k8s object type
kind: Service
**
metadata:
** set the service name
  name: wordpress
**
** set a label 
  labels:
    app: wordpress
**
** set the service into the namespace created 
  namespace: wordpress
**
spec:
** set service type 
  type: NodePort
**
** set service selector
  selector:
    app: wordpress
    tier: frontend
**
** set the listen ports
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30008
  ```

### 2.  NodePort service

 ```
apiVersion: v1
** specify k8s object type
kind: Service
**
metadata:
** set the service name
  name: wp-mysql
**
** set a label 
  labels:
    name: wordpress
**
** set the service to the namespace wordpress
  namespace: wordpress
**
** set service type ClusterIP
spec:
  type: ClusterIP
**
** set service selector will select all objects in the selector
  selector:
    app: wordpress
    tier: mysql
** set the listen ports
  ports:
   - protocol: TCP
     port: 3306
     targetPort: 3306
  ```

make sure you are in your manifest directory and run this command
 ```
kubectl apply -f service-clusterip-mysql.yml service-nodeport-wordpress.yml
  ```

 and run this command to show the services don't forget  the option -n  to select only the services in the specific namespace
  ```
  kubectl get svc -n wordpress
 ```

# Secret
```
apiVersion: v1
data:
** specify secret keys and values
  wordpress_db_password: dG90bwo=
  mysql_password: dG90bwo=
  mysql_random_root_password: MQo=
**
** specify k8s object type
kind: Secret
**
metadata:
** set the service name
  name: app-wordpress-secret
**
** set the service the namespace wordpress
  namespace: wordpress
**
type: Opaque
```
make sure you are in your manifest directory and run this command
 ```
kubectl apply -f app-wordpress-secret.yml
  ```

 and run this command to show the services don't forget  the option -n  to select only the secrets in the specific namespace
  ```
  kubectl get secret -n wordpress
 ```

# Deployments

### 1. Mysql Deployment

```
apiVersion: apps/v1
** specify k8s object 
kind: Deployment
**
metadata:
** name our deployment
  name: wp-mysql
**
** Labels can be used to organize and to select subsets of objects.
  labels:
    app: wordpress
**
** set the deployment into our namespace created
  namespace: wordpress
**
spec:
** declare a replicas with 1 pod
  replicas: 1
  selector:
** match label
    matchLabels:
      app: wordpress
      tier: mysql
 ** recreate strategy in update case
 define
  strategy:
    type: Recreate
** 
** template pod
template:
  metadata:
    labels:
    app: wordpress
    tier: mysql

  spec:
** container specifications 
    containers:
** specify the image and container name
      - image: mysql:5.7
        name: mysql
**
** specify environment variables and get some values into our secret
        env:
          - name: MYSQL_DATABASE
            value: wordpress
          - name: MYSQL_USER
            value: toto
          - name: MYSQL_PASSWORD
            valueFrom:
              secretKeyRef:
                name: app-wordpress-secret
                key: mysql_password
         - name: MYSQL_RANDOM_ROOT_PASSWORD
           valueFrom:
             secretKeyRef:
                name: app-wordpress-secret
                key: mysql_random_root_password
** specify the container listen port
         ports:
         - containerPort: 3306
           name: mysql
***
** mount a volume
         volumeMounts:
         - name: mysql-persistent-storage
           mountPath: /var/lib/mysql
**
** declare the volume used 
volumes:
 - name: mysql-persistent-storage
   hostPath:
     path: /data/mysql
```
  
