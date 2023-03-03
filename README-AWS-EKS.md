# AWS-EKS

- KOPS - settings up AWS resources for Kubernetes (kubernetes operations)
- or a managed service AWS EKS (AWS elastic kubernetes service)

EKS vs ECS
- AWS elastic kubernetes service / elastic container service
- EKS is specific for kubernetes deployments
- ECS is a general service for container deployments

- auth-api/
- kubernetes/
- users-api/
- docker-compose.yaml


- docker build -t swagfinger/kub-dep-users .
- docker push swagfinger/kub-dep-users

- docker build -t swagfinger/kub-dep-auth . 
- docker push swagfinger/kub-dep-auth

---------------------------------------------------------------------------

### AWS EKS 

#### make a cluster - eg. kub-dep-demo
- add cluster service role - iam -> roles -> create a new role - AWS services -> use case EKS cluster
- create Role: eg. eksClusterRole
- back to EKS cluster configuration-> specify networking (see STEP: below on create a VPC)
- kubectl - commands are sent to minikube
    -  windows users folder/ there is a .kube folder and config file inside that kubectl to communicate with minikube
    -   but we want to change to talk to eks

#### CREATE A VPC - go to cloud formation
    - create stack
    - https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html#create-vpc
    - copy the url IPV4: https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
    - paste it in Amazon s3 url
    - give stack name: eg. eksvpc

- back on EKS - cluster configuration -> click VPC refresh -> select your VPC
- SET: Cluster endpoint access - public and private

---
### kubectl updates to communicate with AWS EKS
- update the config file inside .kube in users folder (windows) by using AWS CLI (easiest)

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority: C:\Users\clark\.minikube\ca.crt
    extensions:
    - extension:
        last-update: Thu, 02 Mar 2023 23:51:58 CST
        provider: minikube.sigs.k8s.io
        version: v1.29.0
      name: cluster_info
    server: https://172.31.130.11:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Thu, 02 Mar 2023 23:51:58 CST
        provider: minikube.sigs.k8s.io
        version: v1.29.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: C:\Users\clark\.minikube\profiles\minikube\client.crt
    client-key: C:\Users\clark\.minikube\profiles\minikube\client.key

```

- download AWS CLI
https://aws.amazon.com/cli/

- AWS -> user account -> security credentials -> access keys
- we use access keys to allow things like CLI to communicate directly within AWS

### use AWS CLI to configure access 
```
aws configure
```

### EKS
- check that status is ACTIVE
- aws eks --region us-east-1 update-kubeconfig --name <cluster name>

```powershell
aws eks --region us-east-1 update-kubeconfig --name kub-dep-demo
```

- RESULT: 
- updates kubeconfig file with new data to talk to aws cluster

```yaml
# update config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: C:\Users\clark\.minikube\ca.crt
    extensions:
    - extension:
        last-update: Thu, 02 Mar 2023 23:51:58 CST
        provider: minikube.sigs.k8s.io
        version: v1.29.0
      name: cluster_info
    server: https://172.31.130.11:8443
  name: minikube
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJek1ETXdNekF4TWpJek0xb1hEVE16TURJeU9EQXhNakl6TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBS21ZCmw1SmxCQTV2cWh0VVJzaUExQ0RaOUNVZjBjRU5vY0F1d01VejZsVHRKdXlrWHNSTjBSazY4M0ZSU0NLYmt4eUUKb21EdDIwNm10eWo2SW9qUUk0SVYwSHhIaWNTZDd1NjBjUXh3RG51elJTMVZPa2J4RTJ1UDNVRk1DSFZwQVVOOQptYko2K3NMaWRwRW5FdXlJbmhIMXd6V3pSL0JyN3E5UDluRFJFeTc4bHZhK0lVTWMvN0tES1ZIcW1BOE43Y2o2CkpYNDdPaVZiVU83VUdUQlJnVHFSSnJwTGNOQWhJa1plMksyczNFSWg1UnduWGx2VzB3K0NVVFRoQjg4WFJEV0sKSUVtbG9nZkRyVVNxQW5yaXZ2UHR5SmhqYlNFT1hOVWxQMTNNTzNoVWVtbUtPOXluUXBVWjFGc1c5UFl5Ymtsdwo1cWhyZ1l4SWZVcTNCakEwSXVzQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZKQ1E1V1o4Q3pjZzNodm1MSnYxclhjNmxIQjFNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBS2FLcysyUVh5S1pxWkszQXVvYQpFMDZoaDBsWENNVVNUa1lTUzBjQ1ZqWVhCQ1duVXVoTWcyb09zYmFINHR0TnFkRVlNSW9OUktBVm1CZkU5Q3NzCktZb1NRYXBvT21obktUK1B3eUt3SnA0QVVtV09sQVdlMWM0NnQ4QkJMM0dQN1lxL3dmcTl6cDNDSXVob1RycGcKUitsWjEvSVhaeTVqc0FaL3JYeUI3b2ovSHhJbGdJNmxVRkhlaUt3bmJWOXpMVjYvMm4xeml0SUhxSHFGc3FCMwpJVHZ6akZqTjFXSWM3TmxsZ1hVdkNJbGd3YnBXZlBjNlZpOG5URFBoN3MwMlU2bkVnRjFKT1RuS1FidC91ZElUCndlT0pTT2dNV09YU2pWaUdTdC90UE5pOHpucjFBQ29aU3RiQlZyYlpBM2pIMGtscW1lMGVxSml6ckphTWlCQU8KNkRJPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://BA449F9EEF88A54B7DD6F976CE4613B8.gr7.us-east-1.eks.amazonaws.com
  name: arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Thu, 02 Mar 2023 23:51:58 CST
        provider: minikube.sigs.k8s.io
        version: v1.29.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
- context:
    cluster: arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo
    user: arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo
  name: arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo
current-context: arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: C:\Users\clark\.minikube\profiles\minikube\client.crt
    client-key: C:\Users\clark\.minikube\profiles\minikube\client.key
- name: arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - --region
      - us-east-1
      - eks
      - get-token
      - --cluster-name
      - kub-dep-demo
      command: aws
```

```
Added new context arn:aws:eks:us-east-1:147149888005:cluster/kub-dep-demo to C:\Users\clark\.kube\config
```
- now kubectl will talk to aws cluster and not minikube cluster
- running minikube delete will delete local vpn and if you then type 'kubectl get pods' that will be results from AWS and not minikube

```
kubectl get pods
```

### 248. adding worker nodes

#### STEP1: CONFIGURE NODE GROUP
- EKS - compute section -> add node group
- give it a name: eg. demo-dep-node
- IAM create role for nodes (EC2 instances) 
  - IAM ROLE - create - use cases -> EC2 
  - search 'EKSworker' -> select checkbox
  - search 'cni' -> select CNI policy ('AmazonEKS_CNI_Policy') 
  - search 'ec2containerreg' -> select 'AmazonEC2ContainerRegistryReadOnly'
- next -> give role a name: eg. 'eksNodeGroup'
- back on Cluster Node group configuration -> REFRESH -> select NODE IAM Role 'eksNodeGroup'

#### STEP2: Set compute and scaling configuration
- instance type: t3.small (MAXMILLIAN: scheduling pods on t3.micro can fail - leaving it in pending state)
- next-> next -> next-> create
- this spins up a couple of ec2 instances and add them to this cluster
- EKS will also install all kubernetes software: kubelets, kubeproxy

- EC2 instance page - you can see your EC2 instances
  - note: there are no load balancers at the moment

-------------------------------------
- the cluster is now set up 
- everything is same as minikube but running on AWS
- so we can still send commands with 'kubectl'

### 249. Apploying our Kubernetes config
- can now apply our kubernetes/ yaml config 
- from kubernetes/

```powershell
# make sure our images were created and pushed to dockerhub
-this step was done earlier 

# from kubernetes/ folder
kubectl apply -f ./auth.yaml -f ./users.yaml
kubectl get pods
```

- before services on minikube always had pending IP address, 
- on AWS EKS - service gives url you can send requests to (eg. our users service) can be tested in POSTMAN
   - test URL /signup or /login

- cluster internal communication with service names and 'default' namespaces work!

```powershell
kubectl get namespaces
```

### 250. Volumes
- csi (flexible volume type where 3rd party can add own integrations - AWS has EFS)
- hostPath (good for single node set up - local development - created path on node and used that as volume - survives pod restarts)
- emptyDir (local development - empty directory for each pod - data lost on pod restart)

- to persist data we used a volume in docker-compose
- kubernetes 
  - 1. add volumes in containers setup in yaml
  - 2. set up persistence volume resource with a persistence volume claim

#### CSI integration support AWS EFS (3rd party)
- allows AWS EFS as volume driver as a volume type
- https://github.com/kubernetes-sigs/aws-efs-csi-driver
  - installation: 
    deploy the driver: - installs driver in kubernetes cluster

```
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.5"

```
EFS - create a new file system
EC2 - create a security groups 
  -> create security group -> eg. eks-efs  
  -> vpc -> select EKS VPC (we created this for our kubernetes cluster)
    - Inbound rules -> NFS -> source -> custom -> (OPEN AWS services -> VPC in new tab -> EKS VPC-> copy IPV4 CIDR)
    -> create security group

EFS - create a file system -> select VPC created for cluster (same network as cluster)
  -> network access page -> 
    1. remove the security groups 
    2. add the security groups we created in previous step (EKS security groups) do this for all AZ
    3. create

### adding our EFS as a volume 
- add EFS as a volume to our kubernetes 
- this EFS volume currently only works with persistent volumes
- note: in users.yaml sibling of containers create volumes:

this is the configuration needed to add this CSI EFS driver as a volume to this deployment.
- created a StorageClass,
- created a PersistentVolume,
- a PersistentVolumeClaim,
- connected all of that to our pod
- connected to the container.

```yaml
 volumes:
        - name: efs-vol
          persistentVolumeClaim: 
            claimName: efs-pvc
```

```yaml
# kubernetes/users.yaml



kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity: 
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc # https://github.com/kubernetes-sigs/aws-efs-csi-driver/examples/kubernetes/static_provisioning/specs/storageclass.yaml
  csi:
    driver: efs.csi.aws.com #from examples: https://github.com/kubernetes-sigs/aws-efs-csi-driver
    volumeHandle: fs-59d14521 #file system id for created AWS EFS: eks-efs filesystem id
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
        - name: users-api
          image: academind/kub-dep-users:latest
          env:
            - name: MONGODB_CONNECTION_URI
              value: 'mongodb+srv://maximilian:wk4nFupsbntPbB3l@cluster0.ntrwp.mongodb.net/users?retryWrites=true&w=majority'
            - name: AUTH_API_ADDRESSS
              value: 'auth-service.default:3000'
          volumeMounts:
            - name: efs-vol
              mountPath: /app/users
      volumes:
        - name: efs-vol
          persistentVolumeClaim: 
            claimName: efs-pvc
```
----------------------------------------------------
# 253 using EFS Volume
- kubernetes/ configuration updated to reflect efs volume / persistentVolumeClaim
- AWS EFS is set up.

- update images for containers to reflect dockerhub images
- from users-api rebuild and push to dockerhub

```powershell
# build
docker build -t swagfinger/kub-dep-users .

# push
docker push swagfinger/kub-dep-users

# from kubernetes/ folder
kubectl delete deployment users-deployment

# apply
kubectl apply -f ./users.yaml

kubectl get pods

```

- you can test via postman
  - use the public ipv4 url given from AWS EKS to test urls in routes/

```
router.post('/signup', userActions.createUser);
router.post('/login', userActions.verifyUser);
router.get('/logs', userActions.getLogs);
```

### 255. Tasks

```yaml
# kubernetes/tasks.yaml

apiVersion: v1
kind: Service
metadata:
  name: tasks-service
spec:
  selector:
    app: task
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tasks-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: task
  template:
    metadata:
      labels:
        app: task
    spec:
      containers:
        - name: tasks-api
          image: swagfinger/kub-dep-tasks
          env:
            - name: MONGODB_CONNECTION_URI
              value: 'mongodb+srv://clarkcookie:1v1u2Tp2c7jNPOzo@cluster0.wlkvg7z.mongodb.net/?retryWrites=true&w=majority'
            - name: AUTH_API_ADDRESS
              value: 'auth-service.default:3000'

```
- from tasks-api/ folder: 
```
build and push image to dockerhub
```

- from kubernetes/ folder
- this creates a url
```
kubectl apply -f ./tasks.yaml

```

- rebuild and push:
auth-api: swagfinger/kub-dep-auth  
users-api: swagfinger/kub-dep-users  

- 3 deployments:
  1. auth service
  2. tasks service has an external IP
  3. users service has an external IP

- POSTMAN /signup -> body {email, password} -> then login
- get a token - this token come from /login
- to GET tasks from externalIP/tasks - add Headers {"Authorization": "Bearer <TOKEN>"}