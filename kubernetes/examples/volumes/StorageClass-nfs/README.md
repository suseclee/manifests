# When you have your nsf server ready.

easy solution:
```
helm install stable/nfs-client-provisioner --set nfs.server=10.86.1.244 --set nfs.path=/var/nfs

```