# Why do we need customized Chart for CAP

1. When deploying CAP, We faced permission issues when we switched from V3 to V4.
   https://bugzilla.suse.com/show_bug.cgi?id=1138145
   
2. When we uses NFS deployment on pod (https://github.com/suseclee/manifests/tree/master/kubernetes/nfs),
   the size of node (automatic deployement from testrunner) has a 40 GB disk space.
   However, CAP requires   
   uaa; mysql 20 G
   scf: mysql: 20 G
        blob: 50 G
        grootfs: 50G
        postgres: 5G
        
    It seems you need at least ~ 150G.
    
So I changed by Jan's pathch adn reduced data sized to fit into testrunner node (40G disk storage)
   Applied:
   1. https://gist.github.com/jandubois/a7d307360d435f31fc4883621ac33c40
   2. reduced disk sapces 
      uaa; mysql 5 G
      scf: mysql: 5 G
           blob: 5 G
           grootfs: 5G
           postgres: 5G
           
           
           
 Step to deploy CAP on Skuba
 
 1. install nfs-client to all worker nodes in cluster-test folder since the relative path of id_shared
    ```
    ssh -i ../skuba/ci/infra/id_shared sles@10.86.1.192
    sudo zypper in -y  nfs-client
    ```
    
 2. deploy helm and tiller
    ```
    export KUBECONFIG=~/go/src/github.com/SUSE/test-cluster/admin.conf

    # Service account with cluster-admin role
    kubectl create -f - << EOF
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tiller
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: tiller
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
      - kind: ServiceAccount
        name: tiller
        namespace: kube-system
    EOF
    ```
   
    ```
    helm init --service-account tiller 
    ```
    
3. add suse helm repo
  ```
  helm repo add suse https://kubernetes-charts.suse.com/
  helm repo list
  helm search suse 
  ```
    
4. deploy NFS server for storage of CAP
   ```
   git clone git@github.com:suseclee/manifests.git

   kubectl create -f manifests/kubernetes/nfs/
   ```
 
 5. edit manifests/Charts/scf-config-values.yaml for node ip addresses
 
 6. install uaa and make sure that mysql-0 pod is deployed
    ```
    helm install manifests/Charts/uaa --name uaa --namespace uaa --values manifests/Charts/scf-config-values.yaml
    kubectl get pod -n uaa
    NAME                        READY   STATUS      RESTARTS   AGE
    mysql-0                     1/1     Running     0          4m
    secret-generation-1-7qkjp   0/1     Completed   0          11m
    uaa-0                       1/1     Running     0          11m
 
    ```
    
 7. prepare scf deployment
    ```
    SECRET=$(kubectl get pods --namespace uaa -o jsonpath='{.items[?(.metadata.name=="uaa-0")].spec.containers[?(.name=="uaa")].env[?(.name=="INTERNAL_CA_CERT")].valueFrom.secretKeyRef.name}')
    CA_CERT="$(kubectl get secret $SECRET --namespace uaa -o jsonpath="{.data['internal-ca-cert']}" | base64 --decode -)"
    ```
 8. install CAP and make sure all pod are deployed
    ```
    helm install manifests/Charts/cf --name scf --namespace scf --values=manifests/Charts/scf-config-values.yaml --set "secrets.UAA_CA_CERT=${CA_CERT}"
    
    kubectl get pod -n scf
    NAME                            READY   STATUS      RESTARTS   AGE
    adapter-0                       2/2     Running     0          10m
    api-group-0                     2/2     Running     0          3m4s
    bits-0                          1/1     Running     0          12m
    blobstore-0                     2/2     Running     0          12m
    cc-clock-0                      2/2     Running     0          17m
    cc-uploader-0                   2/2     Running     0          17m
    cc-worker-0                     2/2     Running     0          17m
    cf-usb-group-0                  1/1     Running     0          5m51s
    diego-api-0                     2/2     Running     0          17m
    diego-brain-0                   2/2     Running     0          17m
    diego-cell-0                    1/2     Running     6          12m
    diego-ssh-0                     2/2     Running     0          17m
    doppler-0                       2/2     Running     0          11m
    locket-0                        2/2     Running     0          17m
    log-api-0                       2/2     Running     0          10m
    log-cache-scheduler-0           2/2     Running     0          10m
    mysql-0                         1/1     Running     0          8m15s
    nats-0                          2/2     Running     0          12m
    nfs-broker-0                    1/1     Running     0          17m
    post-deployment-setup-1-9gwb7   0/1     Completed   0          17m
    router-0                        2/2     Running     0          12m
    routing-api-0                   2/2     Running     0          17m
    secret-generation-1-mzw6t       0/1     Completed   0          17m
    syslog-scheduler-0              2/2     Running     0          17m
    tcp-router-0                    2/2     Running     0          17m
  
    ```
    
  9. install stratos
    ```
    helm install kubernetes-charts-suse-com/stable/console/ --name susecf-console --namespace stratos --values=manifests/Charts/scf-config-values.yaml
    kubectl get pod -n stratos
    NAME                          READY   STATUS      RESTARTS   AGE
st  ratos-0                     3/3     Running     1          4m26s
st  ratos-db-6d74cd9fc5-zj7bm   1/1     Running     0          4m26s
vo  lume-migration-1-zxpf5      0/1     Completed   0          4m26s
    ```
    