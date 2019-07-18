git clone git@github.com:suseclee/manifests.git

# Deploy nfs server and storageClass
kubectl create -f manifests/kubernetes/nfs

# Test
kubectl create -f manifests/kubernetes/nfs/test/write-pod.yaml
kubectl create -f manifests/kubernetes/nfs/test/read-pod.yaml
kubectl get pod # You need to see completed

# Delete  
kubectl delete -f manifests/kubernetes/nfs