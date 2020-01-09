# 1. Create your monitoring certificates (optional)
>##### 1.  Generate your CA's private key by issuing the following command.  
>`openssl genrsa -out monitoring.key 2048`   
>##### 2. Create a certificate signing request.  
>`openssl req -verbose -new -key monitoring.key -out monitoring.csr -sha256`   
>##### 3. You need to create directory and files in where you are running openssl commands at. According to the directory in /etc/ssl/openssl.cnf   
>`mkdir -p ./demoCA/newcerts && touch ./demoCA/index.txt && echo 01 > ./demoCA/serial`        
>##### 4. generate Self-sign your certificate     
>`openssl ca -extensions v3_ca -out monitoring.crt -keyfile monitoring.key -verbose -selfsign -md sha256 -days 10000 -infiles monitoring.csr`    
>##### 5. Create a new namespace for monitoring    
>`kubectl create namespace monitoring`   
>##### 6. Import trusted certificate to Kubernetes cluster    
>`kubectl create -n monitoring secret tls monitoring-tls --key  monitoring.key --cert monitoring.crt`   


# 2. Activate helm in your kubernetes
>```
>kubectl create serviceaccount --namespace kube-system tiller
>kubectl create clusterrolebinding tiller --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
>sudo zypper install helm
>helm repo add suse https://kubernetes-charts.suse.com
>helm init --tiller-image registry.suse.com/caasp/v4/helm-tiller:2.16.0 --service-account tiller
>helm version
>helm list  #It should not return any error
>```
# 3. Prometheus
>##### 1. Configure Authentication
>We need to create a basic-auth secret so the NGINX Ingress Controller can perform authentication.
>```
>sudo zypper in apache2-utils
>htpasswd -c auth admin
>kubectl create secret generic -n monitoring prometheus-basic-auth --from-file=auth
>```
>##### 2. Edit the configuration file prometheus-config-values.yaml
>```
># Alertmanager configuration
>alertmanager:
>  enabled: true
> service:
>   type: NodePort
>#  ingress:
>#    enabled: true
>#    hosts:
>#    -  prometheus-alertmanager.example.com
>#    annotations:
>#      kubernetes.io/ingress.class: nginx
>#      nginx.ingress.kubernetes.io/auth-type: basic
>#      nginx.ingress.kubernetes.io/auth-secret: prometheus-basic-auth
>#      nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
>#    tls:
>#      - hosts:
>#        - prometheus-alertmanager.example.com
>#        secretName: monitoring-tls
>  persistentVolume:
>    enabled: true
>    ## Use a StorageClass
>    storageClass: nfs-sc
>    ## Create a PersistentVolumeClaim of 2Gi
>    size: 2Gi
>    ## Use an existing PersistentVolumeClaim (my-pvc)
>    #existingClaim: my-pvc
>
># Create a specific service account
>serviceAccounts:
>  nodeExporter:
>    name: prometheus-node-exporter
>
># Allow scheduling of node-exporter on master nodes
>nodeExporter:
>  hostNetwork: false
>  hostPID: false
>  podSecurityPolicy:
>    enabled: true
>    annotations:
>      apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
>      apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
>      seccomp.security.alpha.kubernetes.io/allowedProfileNames: runtime/default
>      seccomp.security.alpha.kubernetes.io/defaultProfileName: runtime/default
>  tolerations:
>    - key: node-role.kubernetes.io/master
>      operator: Exists
>      effect: NoSchedule
>
># Disable Pushgateway
>pushgateway:
>  enabled: false
>
># Prometheus configuration
>server:
> service:
>   type: NodePort
>#  ingress:
>#    enabled: true
>#    hosts:
>#    - prometheus.example.com
>#    annotations:
>#      kubernetes.io/ingress.class: nginx
>#      nginx.ingress.kubernetes.io/auth-type: basic
>#      nginx.ingress.kubernetes.io/auth-secret: prometheus-basic-auth
>#      nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
>#    tls:
>#      - hosts:
>#        - prometheus.example.com
>#        secretName: monitoring-tls
>  persistentVolume:
>    enabled: true
>    ## Use a StorageClass
>    storageClass: nfs-sc
>    ## Create a PersistentVolumeClaim of 8Gi
>    size: 8Gi
>    ## Use an existing PersistentVolumeClaim (my-pvc)
>    #existingClaim: my-pvc
>```
>##### 3. Add SUSE helm charts repository    
>`helm repo add suse https://kubernetes-charts.suse.com`    
>##### 4. Deploy SUSE prometheus helm chart and pass our configuration values file   
>`helm install --name prometheus suse/prometheus --namespace monitoring --values prometheus-config-values.yaml`     
> Or once you changed config file      
>`helm upgrade prometheus suse/prometheus --namespace monitoring --values prometheus-config-values.yaml`     

>##### 5. verify Prometheus deployment
>```
>kubectl -n monitoring get pod 
>NAME                                             READY   STATUS    RESTARTS   AGE
>prometheus-alertmanager-6cc78b9b84-9prmb         2/2     Running   0          58m
>prometheus-kube-state-metrics-69cd64d4d6-nxbd2   1/1     Running   3          58m
>prometheus-node-exporter-5wnc9                   1/1     Running   0          58m
>prometheus-node-exporter-9dz9q                   1/1     Running   0          58m
>prometheus-node-exporter-hr7f9                   1/1     Running   0          58m
>prometheus-node-exporter-m9m9h                   1/1     Running   0          58m
>prometheus-node-exporter-n9q5z                   1/1     Running   0          58m
>prometheus-node-exporter-rd8nf                   1/1     Running   0          58m
>prometheus-server-9f78cfffd-5tgxl                2/2     Running   0          58m
>
>kubectl get svc  -n monitoring
>NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
>prometheus-alertmanager         NodePort    10.110.157.231   <none>        80:31788/TCP   179m
>prometheus-kube-state-metrics   ClusterIP   None             <none>        80/TCP         179m
>prometheus-node-exporter        ClusterIP   None             <none>        9100/TCP       179m
>prometheus-server               NodePort    10.104.81.83     <none>        80:31502/TCP   179m
>
>kubectl get ingress  -n monitoring #When you activated ingress 
>NAMESPACE    NAME                      HOSTS                                 ADDRESS   PORTS     AGE
>monitoring   prometheus-alertmanager   prometheus-alertmanager.example.com             80, 443   77m
>monitoring   prometheus-server         prometheus.example.com                          80, 443   77m
>```
>##### 6. test
>```
>Alertmanager
>In your web browser, type worker-node-ip:NodePort. 10.17.3.0:31788 in the above example #5.
>Prometheus
>In your web browser, type worker-node-ip:NodePort. 10.17.3.0:31502 in the above example #5.
>```

>##### 7. Use this ip address and port for Grafana congiguration
>```
>export NODE_PORT=$(kubectl get --namespace monitoring -o jsonpath="{.spec.ports[0].nodePort}" services prometheus-server)
>export NODE_IP=$(kubectl get nodes --namespace monitoring -o jsonpath="{.items[0].status.addresses[0].address}")
>echo http://$NODE_IP:$NODE_PORT
>```    
# 4. Grafana
>##### 1. edit grafana-datasource.yaml (Configure Grafana provisioning)
>```
>kind: ConfigMap
>apiVersion: v1
>metadata:
>  name: grafana-datasources
>  namespace: monitoring
>  labels:
>     grafana_datasource: "1"
>data:
>  datasource.yaml: |-
>    apiVersion: 1
>    deleteDatasources:
>      - name: Prometheus
>        orgId: 1
>    datasources:
>    - name: Prometheus
>      type: prometheus
>      url: http://10.17.2.0:31502
>      access: proxy
>      orgId: 1
>      isDefault: true
>```
>##### 2. Create the ConfigMap in Kubernetes cluster
>`kubectl create -f grafana-datasource.yaml`
>##### 3. Edit grafana-config-values.yaml 
>```
># Configure admin password
>adminPassword: admin
>
>service:
>  type: NodePort
>
># Ingress configuration
>#ingress:
>#  enabled: true
>#  annotations:
>#    kubernetes.io/ingress.class: nginx
>#  hosts:
>#    - grafana.example.com
>#  tls:
>#    - hosts:
>#      - grafana.example.com
>#      secretName: monitoring-tls
>
># Configure persistent storage
>persistence:
>  enabled: true
>  accessModes:
>    - ReadWriteOnce
>  ## Use a StorageClass
>  storageClassName: nfs-sc
>  ## Create a PersistentVolumeClaim of 10Gi
>  size: 10Gi
>  ## Use an existing PersistentVolumeClaim (my-pvc)
>  #existingClaim: my-pvc
>
># Enable sidecar for provisioning
>sidecar:
>  datasources:
>    enabled: true
>    label: grafana_datasource
>  dashboards:
>    enabled: true
>    label: grafana_dashboard
>```
>##### 4. Add SUSE helm charts repository
>`helm repo add suse https://kubernetes-charts.suse.com`
>##### 5. deploy grafana
>`helm install --name grafana suse/grafana --namespace monitoring --values grafana-config-values.yaml`
> Or once you changed config file     
>`helm upgrade grafana suse/grafana --namespace monitoring --values grafana-config-values.yaml`    
>##### 6. test
>```
>#grafana will take up to 10 min. check if all the three pods are deployed
> kubectl -n monitoring get pod | grep grafana
>```
>##### 7. Login to grafana-server with username/password from grafana-config.yaml
>```
>export NODE_PORT=$(kubectl get -o jsonpath="{.spec.ports[0].nodePort}" services grafana -n monitoring)
>export NODE_IP=$(kubectl get nodes -o jsonpath="{.items[0].status.addresses[0].address}")
>echo http://$NODE_IP:$NODE_PORT/
>```
>##### 7-1 (optional). On Prometheus from url #11, you can query to Prometheus
>##### 8. Add a new dashboard in the garfana web - "Kubernetes All Nodes" will be shown "System load" panel only.
>```
>1. On the Grafana from url #11 with admin/admin (username/password), enter new password.
>2. hover your mousecursor over the + button on the left sidebar and click on the importmenuitem.
>3. Paste the URL (for example) https://grafana.com/dashboards/3131 into the first input field to import the "Kubernetes All Nodes" Grafana Dashboard. After pasting in the url, the view will change to another form.
>4. Now select the "Prometheus" datasource in the prometheus field and click on the import button.
>```
>##### 9. There are extra dashboards configured for CAASP. To appply this
>```
>1. git clone https://github.com/kubic-project/monitoring.git
>2. kubectl apply -f monitoring/grafana-dashboards-caasp-cluster.yaml
>3. kubectl apply -f monitoring/grafana-dashboards-caasp-namespaces.yaml
>4. kubectl apply -f monitoring/grafana-dashboards-caasp-nodes.yaml
>5. kubectl apply -f monitoring/grafana-dashboards-caasp-pods.yaml
>```
>##### 14. All the configured dashboards will be shown in grafana


References:
https://documentation.suse.com/suse-caasp/4.0/html/caasp-admin/_monitoring.html
https://docs.giantswarm.io/guides/advanced-ingress-configuration/

using general repo for prometheus    
`helm install stable/prometheus --namespace monitoring --name prometheus --values prometheus-config.yaml`

using general repo for grafana     
`helm install stable/grafana --namespace monitoring --name grafana --values grafana-config.yaml`

