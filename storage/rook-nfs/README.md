# NOT Working

#Prerequites
```
sudo zypper in lvm2 nfs-client
sudo modprobe rbd
sudo lsmod| grep -i rbd
```

```
kubectl apply -f common.yaml
kubectl apply -f provisioner.yaml
kubectl apply -f operator.yaml
kubectl apply -f nfs.yaml
```

### Error 
```
kubectl -n rook-nfs get nfsservers.nfs.rook.io
No resources found in rook-nfs namespace.

# This output should show the following
kubectl -n rook-nfs get nfsservers.nfs.rook.io
NAME       AGE   STATE
rook-nfs   32s   Running
```

```
kubectl apply -f sc.yaml
kubectl apply -f pvc.yaml

kubectl get pvc
NAME                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
rook-nfs-pv-claim   Pending                                      nfs            91s
```
