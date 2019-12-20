1. kubectl apply -f nginx.yaml
2. kubectl describe deployment nginx-deployment
3. kubectl get pods -l app=nginx
4. kubectl describe pod <pod-name>
5. kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
6. kubectl delete deployment nginx-deployment
