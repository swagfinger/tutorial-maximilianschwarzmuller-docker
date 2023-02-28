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