1. git clone https://github.com/suseclee/manifests.git && cd manifests/kubernetes/dashboard
2. kubectl apply -f kubernetes-dashboard.yaml
3. kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
4. go to website with https://NodeIP:30001
5. Add Token from #3 for login
