

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
`sudo zypper in nfs-client` Instead of installing manually, if you know that you will use storage in kubernetes, you can add nfs-client in terraform.tfvars.
```
packages = [
  "kernel-default",
  "-kernel-default-base",
  "ca-certificates-suse",
  "nfs-client"
]
```


### 1. Activate helm in your kubernetes (version cab be different in your system and time)
```
kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

helm init --tiller-image registry.suse.com/caasp/v4/helm-tiller:2.8.2 --service-account tiller

helm version

helm list
# It should not return any error
```
### 2. `git clone git@github.com:suseclee/manifests.git && cd manifests/storage/
helm/nfs`

### 3. Add psp for nfs
  	`kubectl create -f psp.yaml`

### 4. Installing the Chart after changing values.yaml
    `helm install stable/nfs-server-provisioner --name nfs-helm -f values.yaml`


### 4. Verify
```
helm list
NAME    	REVISION	UPDATED                 	STATUS  	CHART                       	APP VERSION  	NAMESPACE
nfs-helm	1       	Tue Jan  7 16:35:00 2020	DEPLOYED	nfs-server-provisioner-0.3.2	2.2.1-k8s1.12	default 

kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nfs-helm-nfs-server-provisioner-0   1/1     Running   0          3m56s


kubectl get StatefulSet
NAME                              READY   AGE
nfs-helm-nfs-server-provisioner   1/1     5m17s

kubectl get svc
NAME                              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                  AGE
kubernetes                        ClusterIP   10.96.0.1      <none>        443/TCP                                  3d23h
nfs-helm-nfs-server-provisioner   ClusterIP   10.102.1.170   <none>        2049/TCP,20048/TCP,51413/TCP,51413/UDP   40s

kubectl get sc
NAME            PROVISIONER                                     AGE
nfs (default)   cluster.local/nfs-helm-nfs-server-provisioner   67s

```

### 5. test
>##### 1. 
>`cd manifests/storage/helm/nfs/test`
>##### 2. deploy test-pvc
>`kubectl create test-pvc.yaml`
>##### 3. deploy write-pod.yaml
>`kubectl create -f write-pod.yaml`
>##### 4. deploy read-pod.yaml
>`kubectl create -f read-pod.yaml`
>##### 5. verify
```
kubectl get pod -o wide
NAME                                READY   STATUS      RESTARTS   AGE    IP             NODE       NOMINATED NODE   READINESS GATES
nfs-helm-nfs-server-provisioner-0   1/1     Running     0          7m3s   10.244.1.158   worker-0   <none>           <none>
read-pod                            0/1     Completed   0          5s     10.244.3.197   worker-2   <none>           <none>
write-pod                           0/1     Completed   0          24s    10.244.1.225   worker-0   <none>           <none>                          0/1     Completed   0          44m   10.244.1.184   worker-0   <none>           <none>

kubectl get pvc -o wide
NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE    VOLUMEMODE
nfs-test   Bound    pvc-f3985d1a-cc6c-4ef0-9483-5ce11b9b66e3   500Mi      RWX            nfs            104s   Filesystem

ssh sles@10.17.3.0 # which is worker-0

les@clee-worker-0:/> cd nfs/
sles@clee-worker-0:/nfs> cd pvc-0902a4ae-1a3d-4b37-98fa-4f43b2010479/
sles@clee-worker-0:/nfs/pvc-0902a4ae-1a3d-4b37-98fa-4f43b2010479> ls
SUCCESS

### I added 'nodeName: worker-2' in read-pod.yaml
kubectl delete -f read-pod.yaml
kubectl create -f read-pod.yaml
kubectl get pod -o wide
NAME                               READY   STATUS      RESTARTS   AGE    IP             NODE       NOMINATED NODE   READINESS GATES
nfs-provisioner-855d874b45-kb8pv   1/1     Running     0          108m   10.244.1.198   worker-0   <none>           <none>
read-pod                           0/1     Completed   0          38s    10.244.3.162   worker-2   <none>           <none>
write-pod                          0/1     Completed   0          56m    10.244.1.184   worker-0   <none>           <none>

NFS storage can be accessed from pods in other nodes. 

les@clee-worker-0:/> sudo find -name pvc-f96cebe9-b3cf-48ba-83e9-ac727662d48e
./var/lib/kubelet/pods/2781a8ea-5ef8-4304-b68e-64570b5bcfa6/volumes/kubernetes.io~empty-dir/data/pvc-f96cebe9-b3cf-48ba-83e9-ac727662d48e
sles@clee-worker-0:/> sudo find -name SUCCESS
./var/lib/kubelet/pods/2781a8ea-5ef8-4304-b68e-64570b5bcfa6/volumes/kubernetes.io~empty-dir/data/pvc-f3985d1a-cc6c-4ef0-9483-5ce11b9b66e3/SUCCESS
./var/lib/kubelet/pods/2781a8ea-5ef8-4304-b68e-64570b5bcfa6/volumes/kubernetes.io~empty-dir/data/pvc-f96cebe9-b3cf-48ba-83e9-ac727662d48e/SUCCESS
```
