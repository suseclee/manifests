https://confluence.suse.com/display/~clee/Ceph-Rook

### Prerequite
For every worker node
```
sudo zypper in lvm2
```

```
kubectl create -f common.yaml
kubectl create -f operator.yaml
# Add virtual disk to worker VMs
# Edit cluster.yaml for node names and device names
kubectl apply -f cluster.yaml
kubectl apply -f storageclass.yaml
kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
kubectl apply -f toolbox.yaml
kubectl -n rook-ceph exec -it rook-ceph-tools-<xxx> -- /bin/bash
> ceph status
> ceph osd status
> ceph df
> exit
kubectl apply -f rook-ceph-block-pvc.yaml

```
