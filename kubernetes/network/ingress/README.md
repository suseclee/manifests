How to Use Nginx Ingress Controller

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nginx-ingress-with-cert-manager-on-digitalocean-kubernetes
https://matthewpalmer.net/kubernetes-app-developer/articles/kubernetes-ingress-guide-nginx-example.html

1. Start by creating the “mandatory” resources for Nginx Ingress in your cluster.
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```
2. minikube addons enable ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/cloud-generic.yaml

```
3. Check that it’s all set up correctly.
```
kubectl get pods --all-namespaces -l app=ingress-nginx

```

4. deploy apps
```
kubectl apply -f apple-app.yaml
kubectl apply -f banana-app.yaml
```

5. Create the Ingress in the cluster
```
kubectl create -f ingress.yaml
kubectl get ingress
```

6. Let’s check that it’s working. If you’re using Minikube, you might need to replace localhost with 192.168.99.100.

```
$ curl -kL http://localhost/apple
apple

$ curl -kL http://localhost/banana
banana

$ curl -kL http://localhost/notfound
default backend - 404
````


