# Docker and Kubernetes
- [Kubernetes](http://kubernetes.io) is a framework/tool concepts/standard to assist in container orchestration and largescale deployments (independent of cloud provider we use)
  - automate deployment
  - scaling
  - management of containerized applications
  - not locked into a specific cloud provider - standardized way of describing deployments
  - like docker compose for deployment

- solves
 - crashes replacing down instances
 - traffic spikes - more container instances
 - distribute incoming traffic evenly

- AWS ECS
- container health checks + auto redeployment
- autoscaling
- load balancer

- AWS EKS allows you to setup own kubernetes

# Kubernetes architecture

-> services (group of pods with unique independent IP addresses - allows exposing services to outside world with certain IP)   
-> containers   
-> pods (container(s) + reqiored resources eg. volumes)   
-> worker node(s) (a machine cpu/memory) + proxy/config (configure traffic)   
-> master node (control plane / interacts with worker nodes)   
-> cluster   

- the master node sends instructions to cloud provider api to tell it to create resources to replicate desired big picture (end state)

### what you setup
- cluster and node instances (worker + master)
- api server, kubelet, kubernetes services/software on nodes
- other cloud provider resources (load balancers, file systems)

### what kubernetes manages
- create objects (pods) and manage them
- monitor + recreate them, scale pods
- kubernetes cloud resources to apply your configuration/goals

### worker nodes
- one computer/machine/virtual instance
- managed by master node
- have pods - which host application containers and their resources (volumes/ip/run config) or duplicate pods
- Kubelet - communication device between worker and master node
- kube-proxy - handling incoming/outgoing traffic /traffic restrictions

### master node
- API server - counter point for Kubelets - communication between worker and master node 
- scheduler - watching pods, selects worker nodes to run pods on
- Kube-Controller-Manager ensures correct number of pods - watches and controls worker nodes
- Cloud-Controller-Manager (like Kube-controller-manager) but for a specific cloud provider - knows how to interact + work with cloud provider services


## Kubernetes does not manage inrastructure
- responsible for managing deployed application
- DOES NOT manage infrastructure

- what do you need to do?:  
  - Create the cluster, and the node instances (worker + master nodes)
  - setup API Server, kublets, kubernetes services / software on nodes
  - Kubermatic - automatation tooling for creating infrastructure / remote machines/ instances
  - AWS EKS - allows your own kubernetes configuration

## installation of Kubernetes
- need: 

1. Cluster - Master node (api server, scheduler) + Worker nodes (docker, kubelet )
2. Kube control tool (Kubectl)- communication with cluster - send instructions to cluster (master node) - master node applies commands

deployed in cloud?
cloud: cluster
localmachine: kubectl

testing env?
localmachine: cluster , kubectl

3. minikube (optional) - tool set up locally to play around with kubernetes and testing - uses a virtual machine to hold cluster (single master+worker node)

### windows setup
```shell
systeminfo
```
expect output: 'your system already has a hypervisor installed and you can skip the next step'
otherwise install hypervisor like virtualBox - cant have both installed hypervisor and virtualbox

1. - install kubectl
  - check it works: kubectl version --client
  - then under user folder in windows, create .kube folder
  - inside create 'config' file (tells kubectl which Cluster it should connect - auto populated)

2. - install minikube (sets up virtual machine)
  - needs hypervisor (if you have hypervisor you dont need virtualbox)
  - confirm installation: 
  
```
minikube start --driver=hyperv
minikube status
minikube dashboard
```

### 185. Kubernetes objects
- Objects: Pods, deployments, services, volume, etc
- imperative / declarative : way to create objects

#### Pod Object
- smallest unit kubernetes works with
- contains one/more container
- create pod object and send to kubernetes
- can communicate with other pods
- has cluster IP address
- multiple containers in same pods can communicate with each other through localhost
- AWS ECS task is similar to a pod in kubernetes
- ephemeral - lose state if removed/replaced

#### deployment Object (controller obeject)
- create deploy object which you give instructions to control the number of pods/containers it should manage for you.
- can control one or multiple pods
- the controller - we set desired state, kubernetes will get us to the desired state
- pause/delete/rollback deployments
- autoscaling
- multiple instances of a pod is possible

### imperative approach - deployment

- run Dockerfile  
- build image

```shell
docker build -t kub-first-app .  

```
### create repo on dockerhub
swagfinger/kub-first-app

### rename/re-tag image on local

```shell
docker build -t swagfinger/kub-first-app .

```

### push to dockerhub
```shell
docker push swagfinger/kub-first-app
```
---------------------------------------------------

### send to kubernetes cluster - as a pod OR part of a deployment
- as part of a deployment

1. start cluster

```powershell
minikube status

# start cluster (ADMIN PRIVELEDGE REQUIRED)
minikube start --driver=hyperv
```

2. send image/instruction to create deployment to cluster

- using kubectl
- might need powershell administration 
- using kubectl to create objects

```shell
<!-- create deployment object and send to kubernetes cluster-->


# this actually causes an error as when we create a deployment and specify image=kub-first-app, its looking for the image in the kubernetes cluster and the cluster will look for this image (inside virtual machine) - NOT FROM LOCAL MACHINE 
kubectl create deployment <name> --image=<specify image -image to be used for the container of the pod created by this deployment -remote eg. dockerhub>
# ERROR: `kubectl create deployment first-app --image=kub-first-app`

```
### need to call it from something like dockerhub repository

```shell
kubectl create deployment first-app --image=swagfinger/kub-first-app

```

### see deployments in cluster 
-- wait a bit while cluster starts up

```
<!-- confirst status of cluster -->
kubectl get deployments
```


### get further insight
kubectl get pods


### powershell after starting minikube
minikube dashboard

-------------------------------------------------------------------------------

## Service Object 
- to reach a pod, exposing pods to other pods or the outside
- pods have internal IP (changes when pod is replaced)
- service groups pods together and give them a shared IP address (IP Unchangeable)
- Service can expose this IP to outside the cluster

#### creating a Service
- use 'expose' to expose a pod created by deployment to outside
- specify types of exposure: ClusterIP, NodePort
   - ClusterIP - means internally you gain access to pod
   - NodePort - deployment should be exposed with help of IP address of worker node on which it is runnign
   - LoadBalancer - uses load balancer (generates unique IP address) only available if cluster/infrastructure supports it (eg. aws/minikube).


```
kubectl expose deployment first-app --type=LoadBalancer --port=8080

kubectl get services

```

- minikube gives us command which gives us access to Service which maps a port to an ip which we can reach from local machine
- result: IP address to view running node.js application -> running in container-> pod-> created by kubernetes -> based on deployment we sent to kubernetes cluster. and the service we created allows us to view it on IP address from minikube

```powershell
# view deployment (with admin rights)
minikube service first-app
```

### scaling Kubernetes cluster
`--replicas=3` specifies how many istances we want to have

```shell
# scale up
kubectl scale deployment/first-app --replicas=3

# scale down
kubectl scale deployment/first-app --replicas=1
kubectl get pods          
```
--------------------------------------------------------------------------------------------------

## Updating Deployments
- source code changes...
- rebuild image (remember to Tag it with version number)
- NOTE: new images are only downloaded if they have a different tag (so we specify it)


```shell
docker build -t swagfinger/kub-first-app:2 .
```

### push to dockerhub
- (remember to Tag it with version number)

```shell
docker push swagfinger/kub-first-app:2
```

### update the deployment
- update deployment using 'set' command
- NOTE: new images are only downloaded if they have a different tag (so we specify it)
- check status of deployment with: kubectl rollout status deployment/first-app

```shell
kubectl set image deployment/first-app kub-first-app=swagfinger/kub-first-app:2

# check status of deployment
kubectl rollout status deployment/first-app

minikube dashboard

# view deployment (with admin rights)
minikube service first-app
```
------------------------------------------------------------------------------------------------------

## Deployment rollbacks & history
- rolling updates strategy - because kubernetes doesnt shutdown old pods before a healthy one is running, you may end up with an error that leaves the cmd with an error like below, if the update encounters error when deploy:

  ` waiting for deployment "first-app" rollout to finish: 1 old replicas are pending termination...`


#### view status of pods
```shell
kubectl get pods
```

#### rollback a deployment - note the stuck pods and rollback

```shell
kubectl rollout undo deployment/first-app
kubectl get pods
```

#### rollback to even older deployment
```shell
# list deployment history (gives REVISION NO's)
kubectl rollout history deployment/first-app

# view details about deployment
kubectl rollout history deployment/first-app --revision=3

# rollback to specific revision
kubectl rollout undo deployment/first-app --to-revision=1

```


### cleanup / delete service

```shell
kubectl delete service first-app

kubectl delete deployment first-app

```
---------------------------------------------------------------------------------
 
## Kubernetes - Declarative Approach (concept similar to Docker compose )

- Resource definition files declarative approach to kubernetes commands via yaml
- so instead of running kubectl commands - we point to a yaml file (kubectl apply -f config.yaml)


```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: second-app-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: second-app
      tier: backend
  template:
    metadata: 
      labels:
        app: second-app
        tier: backend
    spec: 
      containers:
        - name: second-node
          image: swagfinger/kub-first-app:2
        # - name: ...
        #   image: ...

```

### deployment of yaml file
ERROR: missing required filed "selector"

```yaml
  selector:
    matchLabels:
      app: second-app
      tier: backend
```

```shell
kubectl apply -f=deployment.yml
```