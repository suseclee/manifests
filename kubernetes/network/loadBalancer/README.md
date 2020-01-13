1. kubectl apply -f hello.yaml
2. kubectl get deployments hello-world
3. kubectl get replicasets
4. kubectl expose deployment hello-world --type=LoadBalancer --name=lb-service
5. kubectl get services lb-service
6. kubectl describe services my-service
7. kubectl get pods --output=wide
8. Use the external IP address (LoadBalancer Ingress) to access the Hello World application:
>```
>curl http://<external-ip>:<port>
>where <external-ip> is the external IP address (LoadBalancer Ingress) of your Service, and <port> is the value of Port in your Service description. If you are using minikube, typing minikube service my-service will automatically open the Hello World application in a browser.
>
>The response to a successful request is a hello message:
>
>Hello Kubernetes!
>```
9. kubectl delete services lb-service
10. kubectl delete deployment hello-world

