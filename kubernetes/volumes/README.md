

# NFS Storage - PersistentVolume

kubectl get pv
   prometheus-nfs 50Gi RWX Retain Bound default/prometheus-prometheus-k8s-db-prometheus-prometheus-k8s-0 19s
kubectl get pvc
   prometheus-prometheus-k8s-db-prometheus-prometheus-k8s-0 Bound prometheus-nfs 50Gi RWX ssd 31s
kubectl get sc
   prometheus-nfs kubernetes.io/nfs 36s
   

kubectl apply -f  test-pod.yaml # after this command, you need to see a file which is created in /mnt.

Verify using "sudo mount -t nfs 10.86.1.244:/var/nfs /mnt" command.



USAGE in your box:
This storage has 190 GB. No use as your own risk.
cleanup process can happen when space is limited. 

In your client machine:
sudo mount -t nfs 10.86.1.244:/var/nfs /mnt

WARNING:
It only has 190 GB storage space.
   
   
  
# NFS Storage - StorageClass 
   
 Verify:

sudo mount -t nfs 10.86.1.244:/var/nfs /mnt

ls /mnt/
default-test-claim-pvc-8a33885f-f76a-11e8-b0a2-0aacb4ada2c5 README

kubectl get pv
kubectl get pvc
kubectl get deployment
kubectl get role
kubectl get rolebinding
kubectl get clusterrole | grep provisioner
kubectl get clusterrolebinding | grep provisioner
