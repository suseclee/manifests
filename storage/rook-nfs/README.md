# WIP

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
NAME       AGE   STATE
rook-nfs   41h   Error

kubectl -n rook-nfs describe nfsservers.nfs.rook.io rook-nfs
Status:
  State:  Error
Events:
  Type     Reason  Age   From          Message
  ----     ------  ----  ----          -------
  Warning  Failed  41h   nfs-operator  Invalid NFSServer spec: [spec.exports[0].server.accessMode: Invalid value: "": valid values are (ReadOnly, ReadWrite, none), spec.exports[0].server.squash: Invalid value: "": valid values are (none, rootId, root, all)]

```

```
kubectl apply -f sc.yaml
kubectl apply -f pvc.yaml

kubectl get pvc
NAME                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
rook-nfs-pv-claim   Pending                                      nfs            91s
```
