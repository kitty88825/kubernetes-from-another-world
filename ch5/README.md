## 說明參數

pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: foo # 表示 Pod 資源的名稱
  labels: # 附加在 k8s 物件上的 tag, 可以利用 selector 來挑選具有特定 label 的物件
    app: foo
spec: # 此處可以用來設置一個或多個 container
  containers:
    - name: foo
      image: mikehsu0618/foo
      ports:
        - containerPort: 8080

```

## 建立資源
- Create or Update Resource
```
kubectl apply -f pod.yaml
```
- Create (Only if the resource does not exist)
```
kubectl create -f pod.yaml
```
## 查看 Pod list
```
kubectl get pods
```

## 查看 Pod 詳細資訊
```
kubectl describe pod foo
```

## Local 進行訪問
> Note：由於 Kubernetes 的特性，大多數資源和服務都是在集群內部運行，通常無法直接從本地機器訪問。

將集群內 foo Pod 的 8080 port 映射到本地主機的 8080 port
```
kubectl port-forward pod/foo 8080:8080
```
並訪問 http://localhost:8080
