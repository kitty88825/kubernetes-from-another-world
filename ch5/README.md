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
