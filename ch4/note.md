# Ch4
## Install Helm

https://formulae.brew.sh/formula/helm
```
brew install helm
```

## Install Kubernetes Dashboard

https://github.com/kubernetes/dashboard
```
# Add kubernetes-dashboard repository
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
# Deploy a Helm Release named "kubernetes-dashboard" using the kubernetes-dashboard chart
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
```
![image](https://github.com/user-attachments/assets/8742a4e8-38cd-4e7d-b873-65d5d80d62f6)

檢查 Kubernetes Dashboard 命名空間中的服務：
```
kubectl -n kubernetes-dashboard get svc
```
![image](https://github.com/user-attachments/assets/60d06ede-5c7e-4c63-b9c3-0195f6b77600)

若要訪問 Dashboard，請執行以下指令
```
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```
並訪問網址
https://localhost:8443
