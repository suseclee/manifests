
## git clone git@github.com:suseclee/manifests.git

# Deploy nfs server and storageClass in Kubernetes
### 1. git clone git@github.com:suseclee/manifests.git
### 2. cd manifests/storage/kubernetes/nfs
### 3. Add SecurityPolicy
   `kubectl create -f psp.yaml`
### 4. Add Role, ClusterRole RoleBinding and ClusterRoleBinding
	`kubectl create -f rbac.yaml`
### 5. Change location of space and deploy pod
   	`kubectl create -f deployment.yaml`
### 6. Deploy storage class
	`kubectl create -f sc.yaml`
### 7. Change the size of storage and deploy persistentVolumeClaim
   	`kubectl create -f pvc.yaml`

### 8. Verify deployments
```
kubectl get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
nfs-provisioner-855d874b45-kb8pv   1/1     Running   0          14m   10.244.1.198   worker-0   <none>           <none>

kubectl logs nfs-provisioner-855d874b45-kb8pv
I0107 22:21:11.733907       1 main.go:63] Provisioner example.com/nfs specified
I0107 22:21:11.734219       1 main.go:87] Setting up NFS server!
I0107 22:21:11.894416       1 server.go:144] starting RLIMIT_NOFILE rlimit.Cur 1048576, rlimit.Max 1048576
I0107 22:21:11.894493       1 server.go:155] ending RLIMIT_NOFILE rlimit.Cur 1048576, rlimit.Max 1048576
I0107 22:21:11.895179       1 server.go:129] Running NFS server!
I0107 22:21:43.480075       1 provision.go:439] using service SERVICE_NAME=nfs-provisioner cluster IP 10.104.69.112 as NFS server IP


kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                              AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP                              3d20h
nfs-provisioner   ClusterIP   10.104.69.112   <none>        2049/TCP,20048/TCP,111/TCP,111/UDP   3m25s

 kubectl get sc
NAME     PROVISIONER       AGE
nfs-sc   example.com/nfs   3m33s

kubectl get pvc
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc   Bound    pvc-cab00ae7-ade3-457c-8fbc-82ba64a5b41a   2Gi        RWX            nfs-sc         3m58s

kubectl describe pvc nfs-pvc
kubectl describe pods
```

# Test

### 1. 
	`cd manifests/storage/kubernetes/nfs/test`


### 2. deploy test-pvc
	`kubectl create test-pvc.yaml`


### 3. deploy write-pod.yaml
    `kubectl create -f write-pod.yaml`


### 4. deploy read-pod.yaml
	`kubectl create -f read-pod.yaml`


### 5. verify
```
kubectl get pod -o wide
NAME                               READY   STATUS      RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
nfs-provisioner-855d874b45-kb8pv   1/1     Running     0          96m   10.244.1.198   worker-0   <none>           <none>
read-pod                           0/1     Completed   0          44m   10.244.1.181   worker-0   <none>           <none>
write-pod                          0/1     Completed   0          44m   10.244.1.184   worker-0   <none>           <none>

kubectl get pvc -o wide
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE   VOLUMEMODE
nfs-pvc    Bound    pvc-cd8349a0-7c0c-45b1-8b16-3a36c68247c6   20Gi       RWX            nfs-sc         87m   Filesystem
nfs-test   Bound    pvc-0902a4ae-1a3d-4b37-98fa-4f43b2010479   500Mi      RWX            nfs-sc         72m   Filesystem

ssh sles@10.17.3.0 # which is worker-0

les@clee-worker-0:/> cd nfs/
sles@clee-worker-0:/nfs> cd pvc-0902a4ae-1a3d-4b37-98fa-4f43b2010479/
sles@clee-worker-0:/nfs/pvc-0902a4ae-1a3d-4b37-98fa-4f43b2010479> ls
SUCCESS

# I added 'nodeName: worker-2' in read-pod.yaml
kubectl delete -f read-pod.yaml
kubectl create -f read-pod.yaml
kubectl get pod -o wide
NAME                               READY   STATUS      RESTARTS   AGE    IP             NODE       NOMINATED NODE   READINESS GATES
nfs-provisioner-855d874b45-kb8pv   1/1     Running     0          108m   10.244.1.198   worker-0   <none>           <none>
read-pod                           0/1     Completed   0          38s    10.244.3.162   worker-2   <none>           <none>
write-pod                          0/1     Completed   0          56m    10.244.1.184   worker-0   <none>           <none>

NFS storage can be accessed from pods in other nodes. 

```


# Delete  
```
kubectl delete -f manifests/kubernetes/nfs
```

# warning
### 1. Libvirt
disk_size in variable.tf in default has 24G disk. So your storage class will be bounded by your node's disk space.
Default nodes from terraform in CaaSP, unless specific disk size was mentioned, has 24 G bytes. So max NFS storage NFS you can have is 20G.
if you want to increase 50G to your node, add `disk_size = 59055800320` (55G) to your node.


### 2. Openstack
worker_size in variable.tf in default has 2 vCPUs, 4G , 40G disk
Choose a size with large storage option:
```
caasp.xlarge has 4 vCPUs, 16 G, 300 G disk 
c8.large has 8 vCPUs, 8G, 160G disk
```

### 3. VMware
worker_disk_size in variable.tf in default 40 G disk.
Add `worker_disk_size = 100` with size you want.

### 4. Install nfs-client in each node
Kubernetes will ask each node to mount the NFS into a specific directory to be available to the pod. As mount.nfs was not found, the mounting process failed.
So  you need to install nfs-client in all Kubernetes worker nodes (because you do not know which node will need nfs-client).
`sudo zypper in nfs-client`. Instead of installing manually, if you know that you will use storage in kubernetes, you can add nfs-client in terraform.tfvars.
```
packages = [
  "kernel-default",
  "-kernel-default-base",
  "ca-certificates-suse",
  "nfs-client"
]
```
