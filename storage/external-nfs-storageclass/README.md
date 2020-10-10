https://confluence.suse.com/display/~clee/Dynamic+Provisioning+from+an+external+NFS+storage+from+Y+Bare-Metal

### Prerequite
For every work node
```
sudo zypper in nfs-client
```

```
kubectl apply -f nfs-rbac.yaml
kubectl apply -f nfs-sc.yaml
kubectl patch storageclass nfs -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
kubectl apply -f nfs-client-provisioner
kubectl apply -f nfs-pvc-test.yaml
```
