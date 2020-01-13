```
cd /home/chang/Workspace/manifests/kubernetes/network/nginx-ingress-controller
kubectl apply -f sa.yaml
kubectl apply -f default-server-secret.yaml
kubectl apply -f nginx-config.yaml
kubectl apply -f custom-resource-definitions.yaml
kubectl apply -f rbac.yaml
kubectl apply -f nginx-ingress.yaml
kubectl create -f service-nodeport.yaml
```

I could not verify this is working because I could not fin dhow to access inggress to service.
```
cd /home/chang/Workspace/manifests/kubernetes/network/ingress
kubectl apply -f apple-app.yaml
kubectl apply -f banaba-app.yaml
kubectl apply -f ingress.yaml

kubectl get ingress
NAME              HOSTS                      ADDRESS   PORTS   AGE
example-ingress   chang.apple,chang.banana             80      69m

```

