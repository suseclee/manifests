
##### git clone git@github.com:suseclee/manifests.git

# Deploy nfs server and storageClass in Kubernetes
```
kubectl create -f manifests/kubernetes/nfs
```

# Verify NFS
```
kubectl get sa
NAME              SECRETS   AGE
default           1         22m
nfs-provisioner   1         16s


kubectl get pvc
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfsclaim   Bound    pvc-f17f6915-1b2a-453b-9261-f8e20f33615a   4Gi        RWX            persistent     41s

kubectl get deployment
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nfs-provisioner   1/1     1            1           52s

kubectl get service
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                              AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP                              14m
nfs-provisioner   ClusterIP   10.100.155.103   <none>        2049/TCP,20048/TCP,111/TCP,111/UDP   111s

kubectl get pod
NAME                               READY   STATUS    RESTARTS   AGE
nfs-provisioner                    1/1     Running   0          2m21s
nfs-provisioner-0                  1/1     Running   0          2m20s
nfs-provisioner-56589bd555-pr6cv   1/1     Running   0          2m20s

kubectl get storageclass
NAME         PROVISIONER   AGE
persistent   nfs           2m58s

kubectl patch storageclass persistent -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

kubectl get storageclass
NAME                   PROVISIONER   AGE
persistent (default)   nfs           4m12s
```
# Test
```
kubectl create -f manifests/kubernetes/nfs/test/write-pod.yaml 

kubectl get pod
NAME                               READY   STATUS      RESTARTS   AGE
nfs-provisioner                    1/1     Running     0          74m
nfs-provisioner-0                  1/1     Running     0          74m
nfs-provisioner-56589bd555-pr6cv   1/1     Running     0          74m
write-pod                          0/1     Completed   0          9s

kubectl create -f manifests/kubernetes/nfs/test/read-pod.yaml

kubectl get pod
NAME                               READY   STATUS      RESTARTS   AGE
nfs-provisioner                    1/1     Running     0          76m
nfs-provisioner-0                  1/1     Running     0          76m
nfs-provisioner-56589bd555-pr6cv   1/1     Running     0          76m
read-pod                           0/1     Completed   0          46s
write-pod                          0/1     Completed   0          2m10s
```

# Delete  
```
kubectl delete -f manifests/kubernetes/nfs
```
