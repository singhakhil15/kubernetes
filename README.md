[TOC]



### Pod

##### PodDefinition

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
	labels: 
		app: my-app
		type: front-end
spec:
	containers:
		- name: nginx-container
		  images: nginx
```



##### Commands

```sh
# run help 
kubectl run --help

# creating pod
kubectl create -f pod-def.yaml

# get all running pod
kubectl get pods

# get details of pod
kubectl describe pod <pod-name>

# create a pod with image
kubectl run nginx --image=nginx

# to get node details for pods
kubectl get pods -o wide

# delete pod
kubectl delete pod <pod-name>

# creating pod definition file named redis.yaml
kubectl run redis --images=redis123 --dry-run=client -o yaml > redis.yaml

# edit pod and apply changes after updating redis.yaml file
kubectl apply -f redis.yaml

# extract pod definition in a file
kubectl get pod <pod-name> -o yaml > pod-definition.yaml

```



- Extract the definition to a file using the below command:

  **`kubectl get pod <pod-name> -o yaml > pod-definition.yaml`**

  Then edit the file to make the necessary changes, delete, and re-create the pod.

- To modify the properties of the pod, you can utilize the **`kubectl edit pod <pod-name> `** command. Please note that only the properties listed below are editable.

  - spec.containers[*].image
  - spec.initContainers[*].image
  - spec.activeDeadlineSeconds
  - spec.tolerations
  - spec.terminationGracePeriodSeconds



### Controllers

They are brain which monitor kubernet's object and response accordenly.

##### Replication Controller

To achive high availability we need replication of pod. Replication Controller is older technology replaced by Replica Set.



```yaml
apiVersion: v1
kind: ReplicationController
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: frontend
spec:
	template:
	## template contain object from pod defintion
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
				- name: nginx-container
				  image: nginx
	replicas: 3
```

##### Commands

```bash
# get replication controll
kubectl get replicationcontroller

```

Get pod command will retrun all pod created. The name of pod will start with replication controller name if it created trough replication.



##### ReplicaSet Controller

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: frontend
spec:
	template:
	## template contain object from pod defintion
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
				- name: nginx-container
				  image: nginx
	replicas: 3
	selector: 
		matchLables:
			type: front-end
```

ReplicaSet can manage pods which is not defined as a part of replicates definition. Selector helps replicaset to identify pods which defines as part of replicaset definition.

*How does Replicaset manage pre-existing pods?*
Labeling of pod helps here. In replicaset we have to matchLabels of pods. Lets we have defined a pod with labels as `tier: front-end`, while creating replicaset we will add pod labels under `matchLables` of selector.

```yaml
selector:
	matchLabels:
		tier: front-end
```



##### Scale 

1. We can scale Replicaset by updating definition file `replicas: 6` and replace the replicaset with following command.

`kubectl replace -f replicaset-def.yaml`

2. `kubectl scale --replicas=6 replicaset-def.yaml` by using scale command with replicaset definition file. Scaling replicasetset by providing file as input will not update file replicaset definition file.
3. `kubectl scale --replicas=6 replicaset myapp-rc` by using replicaset name.

##### Commands

```bash
kubectl get replicaset

kubectl delete replicaset myapp-replicaset

kubectl replace -f replicaset-definition.yaml

kubectl scale -replicas=6 -f replicaset-def

kubectl describe replicaset <replicaset-name>

kubectl explain replicaset

kubectl edit replicaset <replicaset-name>
```



### Deployments

Deployment will automatically create replicaset. 

##### Definition file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: frontend
spec:
	template:
	## template contain object from pod defintion
		metadata:
			name: myapp-pod
			labels:
				app: myapp
				type: front-end
		spec:
			containers:
				- name: nginx-container
				  image: nginx
	replicas: 3
	selector: 
		matchLables:
			type: front-end
```



##### Commands

```bash
kubectl get deployments

kubectl create deployment <deployment-name> --image=<image-name> --replicas=<replica-desire-number>
```



### Namespaces

Namespace is a machanism that enable us to organize resources. It is like a virtual cluster inside a cluster.

##### Initial Namespaces

- ***kube-system:*** System processes like Master and kubectl processes are deployed in this namespace; thus, it is advised not to create or modify the namespace.

- ***kube-public:*** This namespace contains publicly accessible data like a [configMap](https://www.geeksforgeeks.org/kubernetes-configmap/) containing cluster information.

- ***kube-node-lease:*** This namespace is the heartbeat of [nodes](https://www.geeksforgeeks.org/kubernetes-node/). Each node has its associated lease object. It determines the availability of a node.

- ***default:*** This is the namespace that you use to create your resources by default.

  

##### Connection between Namespaces

Resources within same namespace will refer eachother with their names. For example, we have pods name as `web-app` , `db-service` and `web-deployment`

Then `web-app` can reffer `db-service` as 

```js
mysql.connect("db-service")

// application in deifferent namespace dev
mysql.connect("db-service.dev.svc.cluster.local")

<application-name>.<namespace-name>.svc.cluster.local
<application-name>.<namespace-name>.<service>.<domain>
```

##### Commands

```sh
# get pods in given namespace
kubectl get pods --namespace=kube-system

# create a pod in given namespace
kubectl create -f pod-definition.yaml --namespace=dev

#create a new namespace
kubectl create namespace dev

# set dev namespace to current context
kubectl config set-context $(kubectl config current-context) --namespace=dev

# get all pods in all namespaces
kubectl get pods --all-namespaces
```



##### Definition File

Add namespace definition into pod definition file under `metadata` section.

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: front-end
	################
	namespace: dev
	################
	labels:
		type: front-end
		app: myapp
spec:
	container:
		- name: nginx-container
		  image: nginx
```



##### Creating new Nmaespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
	name: dev
```



##### Resource Quota

To limit resources in a namespace create Resource Quota.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
	name: compute-quota
	namespace: dev
spec:
	hard:
		pods: "10"
		requests.cpu: "4"
		requests.memory: 5Gi
		limits.cpu: "10"
		limits.memory: 10Gi
```



### Imperative Commands

`--dry-run`: By default, as soon as the command is run, the resource will be created. If you simply want to test your command, use the `--dry-run=client` option. This will not create the resource. Instead, tell you whether the resource can be created and if your command is right.

`-o yaml`: This will output the resource definition in YAML format on the screen.

```sh
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

##### POD

**Create an NGINX Pod**

```
kubectl run nginx --image=nginx
```



**Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)**

```
kubectl run nginx --image=nginx --dry-run=client -o yaml
```



##### Deployment

**Create a deployment**

```
kubectl create deployment --image=nginx nginx
```



**Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**

```
kubectl create deployment --image=nginx nginx --dry-run -o yaml
```



**Generate Deployment with 4 Replicas**

```
kubectl create deployment nginx --image=nginx --replicas=4
```



You can also scale deployment using the `kubectl scale` command.

```
kubectl scale deployment nginx --replicas=4
```



##### POD

**Create an NGINX Pod**

```
kubectl run nginx --image=nginx
```



**Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)**

```sh
kubectl run nginx --image=nginx --dry-run=client -o yaml
```



##### Deployment

**Create a deployment**

```sh
kubectl create deployment --image=nginx nginx
```



**Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**

```sh
kubectl create deployment --image=nginx nginx --dry-run -o yaml
```



**Generate Deployment with 4 Replicas**

```sh
kubectl create deployment nginx --image=nginx --replicas=4
```



You can also scale deployment using the `kubectl scale` command.

```sh
kubectl scale deployment nginx --replicas=4
```

**Another way to do this is to save the YAML definition to a file and modify**

```sh
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
```



##### Service

**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**

```sh
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors)

Or

```sh
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

**Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**

```sh
kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```sh
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```



##### Commands

```shell
kubectl run --help

kubectl run redis --image=redis:alpine --labels="tier=db,env=prod"

kubectl expose --help

kubectl expose pod redis --port 6379 --name redis-service

#kubectl create service --help
#kubectl create service clusterip redis --tcp=6379:6379

kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3 

# creat a new pod called `custom-nginx` using the `nginx` image and expose at port 8080

kubectl run custom-nginx --image=nginx --port 8080

# create nasmespace dev-ns
kubectl create namespace dev-ns

# create a new deployment called `redis-deploy` in `dev-ns` namespace with `redis` image and 2 replicas 
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns

#create a pod called httpd using the image `httpd:alpine` in the default namespace. Next create a service  of type `ClusterIP` by the same name `httpd`. Thr target port for service should be 80.

kubectl run http --image=httpd:alpine --port=80 --expose=true
```



### CMD And Entrypoint 

```dockerfile
FROM Ubuntu

CMD sleep 5
```

```sh
docker run ubuntu-sleeper sleep 10
```



```dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]
```

```sh
docker run ubuntu-sleeper 10
```



```dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["10"]
```

```sh
docker run ubuntu-sleeper 

## replace entry point at run time
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```





### Commands and Arguments in Kubernetes

Dockerfile
```dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["10"]
```



Pod-Definition.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: ununtu-sleeper-pod
spec:
	container:
		- name: ubuntu-sleeper
			images: ubuntu-sleeper
			command: ["sleep2.0"]
			argd: ["10"]
## Notes:- command will replace `ENTRYPOINT` and args will replace `CMD` 			
```





```sh
kubectl run ubuntu-sleeper --image=ubuntu-sleeper -- sleep 20

kubectl run webapp-green --image=kodekloud/webapp-color -- --color green

kubectl run webapp-green --image=kodekloud/webapp-color --command -- python2.py --color green
```



### Edit a POD

Remember, you CANNOT edit specifications of an existing POD other than the below.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod.

We have 2 options 

1. Create a copy of pod definition and save in temporary location. Then delete pod and create pod with temporary file.
   ```sh
   kubectl edit pod ubuntu-sleeper
   
   kubectl replace --force -f /tmp/kubectl-edit-1234.yaml
   ```

   

2. The second option is to extract the pod definition in YAML format to a file using the command

   ```sh
   kubectl get pod webapp -o yaml > my-new-pod.yaml
   ```

   Then make the changes to the exported file using an editor (vi editor). Save the changes

   ```sh
   vi my-new-pod.yaml
   ```

   Then delete the existing pod

   ```sh
   kubectl delete pod webapp
   ```

   Then create a new pod with the edited file

   ```sh
   kubectl create -f my-new-pod.yaml
   ```

### Edit Deployments

With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification, with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

```sh
kubectl edit deployment my-deployment
```



### ENV Variables in Kubernetes

```sh
docker run -e APP_COLOR=pink dimple-web-color
```



```yaml
apiVersion: v1
kind: Pod
metadata: 
	name: simple-webapp-color
spec:
	containers:
		- name: simple-webapp-color
			image: simple-webapp-image
			ports:
				- containerPort: 8080
			env:
      	- name: APP_COLOR
      		value: pink
```

There 3 ways of setting ENV vars

1. ```yaml
   env:
   	- name: APP_COLOR
   	  value: pink
   ```

   

2. ```yaml
   env:
   	- name: APP_COLOR
   	  valueFrom:
   	  	configMapKeyRef:
   ```

3. ```yaml
   env:
   	- name: APP_COLOR
   	  valueFrom:
   	  	secretKeyRef:
   ```



### ConfigMaps

ConfigMap are used to pass configuration data in key-value pairs in kubernets. When pod is created and injected ConfigMap to pod. The key value pairs will be available as environment variables for application inside the container in the pod. There are 2 steps involved in ConfigMap. 

1. Create ConfigMap
   a. Imperative

   ```sh
   kubeconfig create configmap <config-map-name> \
   --from-literal=APP_COLOR=blue \
   --from-literal=APP_MOD=prod 
   
   
   # creating from configmap file
   kubectl create configmap <configmap-name> \
   --from-file=<path-to-file>
   
   #
   kubectl create configmap app-config \
   --from-file=app_config.properties
   
   ```

   Confimap file (app_config.properties)
   ```yaml
   APP_COLOR: blue
   APP_MOD: prod 
   ```

   

   b. Declarative 

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata: 
   	name: app-config
   data:
   	APP_COLOR: red
   	APP_MOD: blue
   ```

   

2. Injecting ConfigMap
   Pod-definition.yaml

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
   	name: myapp
   	labels:
   		tier: front-end
   spec:
   	containers:
   		- name: my-app
   			image: nginx
   			port: 8080
   		envFrom:
   			- configMapRef:
   					name: app-config
   		
   ```

   Passing single key-value from configmap 


   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
   	name: myapp
   	labels:
   		tier: front-end
   spec:
   	containers:
   		- name: my-app
   			image: nginx
   			port: 8080
   		env:
   			- name: APP_COLOR
   				valueFrom:
   					configMapKeyRef:
   						name: app-config
   						key: APP_COLOR
   ```

   

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
   	name: myapp
   	labels:
   		tier: front-end
   spec:
   	containers:
   		- name: my-app
   			image: nginx
   			port: 8080
   		volumes:
   			- name: app-config-volume
   				configMap:
   					name: app-config
   ```

   

### Secrets

1. Create Secrets

   1. Imperative

      ```sh
      kubectl create secret generic \
          <secret-name> --from-literal=<key>=<value>
          
      kubectl create secret generic \
          app-secret --from-literal=BD_HOST=mysql \
                     --from-literal=BD_PASS=admin 
      
      ## Using file to stor secrets
      kubectl create secret genric \
        <secret-name> --from-file=<path-to-file>
      ```

   2. Declaritive

      ```yaml
      apiVersion: v1
      kind: Secret
      metadata:
      	name: app-secret
      data:
      	DB_HOST: mysql
      	DB_PASSWORD: admin
      	DB_USER: root
      
      ```

      

      Encoding secrets 

      ```sh
      echo -n 'mysql' | base64
      echo -n 'root' | base64
      ```

      Decoding 

      ```sh
      echo -n '==89bdnf' | base64 --decode
      echo -n 'cm34hj--' | base64 --decode
      ```

      

      Commands

      ```sh
      kubectl get secrets
      
      ## Describe secrets will hide value 
      kubectl describe secrets
      
      # To see secrets values
      kubectl get secret app-secret -o yaml
      ```

   

2. Injecting Secrets

   Pod-definition.yaml
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata: 
   	name: web-app
   	labels:
   		name: simple-web-app
   spec:
   	containers:
   		- name: simple-web-app
   		  image: nginx
   		  ports:
   		  	- containerPort: 8080
   		  envFrom:
   		  	- secretRef:
   		  			name: app-secret
   ```

   

   Injecting Single secrets

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata: 
   	name: web-app
   	labels:
   		name: simple-web-app
   spec:
   	containers:
   		- name: simple-web-app
   		  image: nginx
   		  ports:
   		  	- containerPort: 8080
   		  env:
   		  	- name: DB_PASS
   		  		valueFrom:
   		  			secretKeyRef:
   		  				name: app-secret
   		  				key: DB_PASS
   ```

   

   Injecting Secret as Volume

   When we Inject a secret as volume in a Pod, each attribute will create a seprate file and stroe value in it.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata: 
   	name: web-app
   	labels:
   		name: simple-web-app
   spec:
   	containers:
   		- name: simple-web-app
   		  image: nginx
   		  ports:
   		  	- containerPort: 8080
   		  volume:
   		  	- name: DB_PASS
   		  		secret:
   		  			secretName: app-secret
   ```

   

   **Note:-**

   1. Secrets are not encrypted, they are encoded. 
   2. Secrets are not encrypted in ETCD.          

   
