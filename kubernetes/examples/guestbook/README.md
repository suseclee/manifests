# Example: Guestbook application on Kubernetes

This directory contains the source code and Kubernetes manifests for PHP
Guestbook application.

Follow the tutorial at https://kubernetes.io/docs/tutorials/stateless-application/guestbook/.
```
1. kubectl apply -f redis-master-deployment.yaml
2. kubectl apply -f redis-slave-deployment.yaml
3. kubectl apply -f redis-slave-service.yaml
4. kubectl apply -f frontend-deployment.yaml
5. kubectl apply -f frontend-service.yaml
6. kubectl get service frontend
7. Copy the workerNode's IP address and port # from #6
8. load the page in your browser from #7.



```
