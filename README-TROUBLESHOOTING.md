# Troubleshoot

first cleanup deployments (start clean)

```powershell
kubectl get deployment
kubectl delete deployment (...name of deployment)
kubectl get services
kubectl delete services (...name of service)
```

- ONLY 'kubernetes' cluster should be running - this is started automatically when you start

```
minikube start --driver=hyperv
```
make sure your container images in .yaml point to YOUR dockerhub repo

what also made a difference is to run powershell with admin rights and all commands in same powershell window. meaning dont run half the commands that dont need admin rights in VScode and the other half from another powershell window (not from vs code).

make sure you are in the correct folder:
when you run - it must be from your kubernetes folder

```powershell
kubectl apply -f ./users-deployment.yaml
kubectl apply -f ./users-serice.yaml
```

minikube service users-service
it must be from your kubernetes folder

when you build an image, it must be from that folder