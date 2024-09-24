+++
title = "Certified Kubernetes Administrator: Core Concepts"
date = "2024-09-23T19:44:48-05:00"
author = "glewis"
authorTwitter = "" #do not include @
cover = ""
tags = ["cka", "kubernetes"]
keywords = ["certifications", "kubernetes", "concepts"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

# Cluster Architecture
## Control Plane
**Master Node**: Manage, Plan, Schedule, and Monitor Nodes
**ETCD Cluster**: Highly available key/value database
**kube-scheduler**: Identifies the right node to place a container on
**Node-Controller**: Manages nodes
**Replication-Controller**: Ensures desired number of containers are running in a replication group
**kube-apiserver**: Orchestrates all operations within the cluster and exposes the API
## Workers
**Worker Nodes**: Host applications as Containers
A container runtime engine, like Docker, must be installed and available on all nodes
**kubelet**: Agent that runs on each node in a cluster and listens for instructions from the API as well as monitors status of all the other nodes
**kube-proxy**: Service ensures that the rules are in place on the worker nodes so the containers can communicate between each other
# Docker vs. Containerd
Docker support was removed from Kubernetes, but Docker images still work because they adhere to imagespec
**containerd**: Container runtime used by Docker that can be ran by itself
`ctr`: Debugging tool bundled with containerd
`nerdctl`: Provides a Docker-like for containerd
	Supports docker compose
	Supports newest features in containerd
`crictl`: Provides a CLI for Container Runtime Interface (CRI) compatible container runtimes
	Installed separately
	Used to inspect and debug container runtimes
	Works across different runtimes
# ETCD for Beginners
ETCD is a distributed reliable key-value store
**Key-Value Store**: Stores information as documents and all information on an item is contained in the corresponding document
To install ETCD, download and extract the binary then run the service
It will run on port 2379 and clients can be attached
`etcdctl`: Command client
	`etcdctl put key1 value1`: Sets a key
	`ectdctl get key1`: Gets a key value
# ETCD in Kubernetes
Stores information on the Nodes, PODs, Configs, Secrets, etc.
All changes are updated in the ETCD server, and is not considered completed until changed
## Setup
### Manual
The binary is downloaded, then manually set as a service by executing the `etcd` command with the required options
## kubeadm
ETCD is deployed as a pod in the clusted
`kubectl exec etcd-master -n kube-system etcdctl get / --prefic -keys-only`: Get keys stored in the ETCD pod

In a HA Environment, there are multiple ETCD masters
# Kube-API Server
When running a command, it is sent to the kube-apiserver
POST requests can also be sent in place of commands:
`curl -X POST /api/v1/namespaces/default/pods ...`: Creates a Pod
	The scheduler detects there is a pod with no node assigned, then chooses a pod and communicates to the kube-apiserver
	The apiserver instructs the kubelet to create a pod
	The kubelet communicates the change to the apiserver, which updates ETCD
# Kube Controller Manager
A controller is a process that continuously monitors the system and takes necessary actions to keep the system in a specified state
**Node Controller**: Responsible for monitoring the status of the nodes and takes action to keep applications running
	Checks status every 5 second
	After 40 seconds of no heartbeat, a node is marked unreachable
	After 5 minutes, the node is evicted and pods are moved to healthy nodes
**Replication Controller**: Responsible for monitoring the status of replicasets and ensures the number of pods are available within a set
	If a pod dies, it creates another pod
There are many types of controllers in a Kubernetes Cluster, and are all packaged in the **Kube-Controller-Manager**
kubeadm deploys the kube-controller-manager as a pod
# Kube Scheduler
Responsible which pod goes on which node
Does not actually place the pod, the kubelet does
The scheduler looks at each pod and finds the correct node for it based on requirements (CPU, Memory, Labels, etc.)
The scheduler ranks the nodes and assigns on a rank from 0-10
kubeadm deploys the scheduler as a pod
# Kubelet
* Registers nodes
* Creates pods
* Monitors nodes and pods
The kubelet service is not installed by kubeadm and must be manually installed on each node
# Kube Proxy
Every pod can reach every pod
A pod network is an internal network each pod communicates with
Applications are deployed as services so they can be available to the cluster
The service has an assigned IP
**kube-proxy**: Process that runs on each that looks for new services, then creates the appropriate rules to route traffic through the pods
kubeadm deploys kube-proxy as a pod
# Pods
**Pod**: A single instance of an application and is the smallest object that can be created in Kubernetes
Containers are not added to existing pods to scale
Pods can container multiple containers, but the containers are not of the same image
`kubectl run nginx --image nginx`: Deploys a docker container by creating a pod using the NGINX Docker Image
`kubectl get pods`: Lists Pods
# Pods with YAML
Kubernetes uses YAML files as inputs for creation of objects
A Kubernetes definition file always contains the following four top-level fields:
```
apiVersion: v1
kind: Pod
metadata:
	name: myapp-prod
	labels:
		app: myapp
		type: front-end
spec:
	containers:
		- name: nginx-container
		- image: nginx
```
- `apiVersion`: The version of the API the object will be created with
- `kind`: Type of object being created
- `metadata`: Data about the object in the form of a dictionary
- `spec`: Specifications pertaining to the object
`kubectl create -f pod-definition.yml`: Creates the object listed in the specified file
`kubectl describe pod myapp-pod`: Lists information on the specified pod
# ReplicaSets
**Replication Controller**: Runs multiple instances of a single Pod in a cluster
A replication controller also brings up new pods when a pod fails and ensures the necessary number of pods are running
Replication assists with load balancing and scaling by deploying multiple nodes in the cluster across nodes
**ReplicaSet**: New recommended utility to manage replication
To create a replication controller:
`rc-definition.yml`:
```
apiVersion: v1
kind: ReplicationController
metadata:
	name: myapp-rc
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
			name: myapp-prod
			labels:
				app: myapp
				type: front-end
	spec:
		containers:
		- name: nginx-container
	      image: nginx
	replicas: 3
```
`template`: Pod template to be created by the replication controller, which will contain the contents of a Pod YAML file
`kubectl create -f rc-definition.yml`: Creates the replication controller
`kubectl get replicationcontroller`: Lists replication controller

To create a replicaset:
`replicaset-definition.yml`
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-replicaset
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
			name: myapp-prod
			labels:
				app: myapp
				type: front-end
	spec:
		containers:
		- name: nginx-container
	      image: nginx
	replicas: 3
	selector:
		matchLabels:
			type: front-end
```
`selector`: Identifies applicable pods, as ReplicaSets can manage pods that were not created by it. It is not a required field
`kubectl create -f replicaset-definition.yml`: Creates the replicaset
`kubectl get replicaset`: Lists replicasets
ReplicaSets can begin monitoring existing pods upon creation if said pods match the selector criteria
`kubectl delete replicaset myapp-replicaset`: Deletes the specified replicaset and the underlying pods
## Scale
To scale:
* Manually edit the `replicas` value in the definition file
* Use the `kubectl scale --replicas=6 -f replicaset-definition.yml` command
* Use the `kubectl scale --replicas=6 replicaset myapp-replicaset` command
# Deployments
**Rolling Updates**: Updating pods one by one to limit impact to accessibility
**Deployment**: Object that provides capability to upgrade with rolling updates, as well pause, and resume changes
`deployment-definition.yml`:
```
apiVersion: apps/v1
kind: Deployment
metadata:
	name: myapp-deployment
	labels:
		app: myapp
		type: front-end
spec:
	template:
		metadata:
			name: myapp-prod
			labels:
				app: myapp
				type: front-end
	spec:
		containers:
		- name: nginx-container
	      image: nginx
	replicas: 3
	selector:
		matchLabels:
			type: front-end
```
`kubectl create -f deployment-definition.yml`: Creates the deployment
`kubectl get deployments`: Lists deployments
`kubectl get all`: Shows all objects created
# Services: Node Port
Services enable connectivity between groups of pods and external sources
Services are deployed as an object
**Node Port Service**: Listens to a port on the node and forward requests to a port running on the application
	**Target Port**: Port on the pod that requests are forwarded to
	**Port**: The port on the service
	**Node Port**: Physical port on the Node that ranges from 30008-32767
**Cluster IP**: Virtual IP inside the cluster to allow communication between different services
**LoadBalancer**: Distributes load based on set requirements
`service-definition.yml`:
```
apiVersion: v1
kind: Service
metadata:
	name: myapp-service
	
spec:
	type: NodePort
	ports:
		- targetPort: 80
		port: 80
		nodePort: 30008
	selector:
		app: myapp
		type: front-end
```
`selector`: Selects the applicable pods based on labels
	The service will be applied to all pods with the label `app: myapp` 
`kubectl get services`: Lists services
# Services: Cluster IP
Pods are assigned non-static IP addresses when created
A service can group like pods together and provide a single interface for access purposes
`service-definition.yml`:
```
apiVersion: v1
kind: Service
metadata:
	name: back-end
	
spec:
	type: ClusterIP
	ports:
		- targetPort: 80
		port: 80
	selector:
		app: myapp
		type: back-end
```
# Services: LoadBalancer
Kubernetes supports integration with cloud provider LoadBalancers
Uses the same format as `NodePort` YAML files, except type is set to `LoadBalancer`:
```
apiVersion: v1
kind: Service
metadata:
	name: myapp-service
	
spec:
	type: LoadBalancer
	ports:
		- targetPort: 80
		port: 80
		nodePort: 30008
	selector:
		app: myapp
		type: front-end
```
# Namespaces
Objects are created in the `default` namespace that was made available during the creation of the cluster
Another namespace called `kubesystem` was also created for internal services
A third namespace, `kubepublic`, was created in which resources meant to made public to users are held
Resources can be isolated in namespaces, i.e. Development vs. Production
Resources within the same namespace can reach other by the object name:
`mysql.connect("db-service)`
To reach a service within another namespace, the name of the namespace needs to be appended to the service:
`mysql.connect("db-service.dev.svc.cluster.local")`
	When the service was created, a DNS entry was created in this format:
	`db-service`: Sevice Name
	`dev`: Namespace
	`svc`: Service
	`cluster.local`: Domain
`kubectl get pods -n [namespace]`: List pods in a specified namespace
`kubectle create -f definition.yml --namespace=[namespace]`: Creates the object in the specified namespace
The namespace can be applied in the `metadata` section of a definition YAML file:
```
apiVersion: v1
kind: Pod
metadata:
	name: myapp-prod
	namespace: dev
	labels:
		app: myapp
		type: front-end
spe
	containers:
		- name: nginx-container
		- image: nginx
```
To create a new namespace:
`namespace-dev.yml`:
```
apiVersion: v1
kind: Namespace
metadata:
	name: dev
```
A namespace can also be created with `kubectl create namespace dev`
To set the default namespace `kubectl` reads from:
```
kubectl config set-context $(kubectl config current-context) --namespace=namespace
```
To limit resources in a namespace, create a Resource Quota:
`compute-quota.yml`:
```
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