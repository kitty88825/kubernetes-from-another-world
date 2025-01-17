## 參數說明
pod.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: foo # 表示 Pod 資源的名稱
  labels: # 附加在 k8s 物件上的 tag, 可以利用 selector 來挑選具有特定 label 的物件
    app: foo
    type: demo
spec: # 此處可以用來設置一個或多個 container
  containers:
    - name: foo
      image: mikehsu0618/foo
      ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Pod
metadata:
  name: bar
  labels:
    app: bar
    type: demo
spec:
  containers:
    - name: bar
      image: mikehsu0618/bar
      ports:
        - containerPort: 8080

```

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: my-service # Service name (UQ)
spec:
  selector: # 用於選擇將流量導向哪些 Pod
    type: demo # 標籤 type=demo 的 Pod
  type: LoadBalancer # Service 的型別。可為 ClusterIP(default) / NodePort / LoadBalancer / ExternalName
  ports:
    - protocol: TCP # 可為 TCP、SCTP、UDP
      port: 8000 # Local
      targetPort: 8080 # Pod

```

## 執行 Service 和 Pod
```
kubectl apply -f pod.yaml,service.yaml
```

## 查看所有 Service
```
kubectl get services
```
![image](https://github.com/user-attachments/assets/c1827b7d-d1f9-4334-a9bb-f716d4636e48)


## 查看全部元件狀態
```
kubectl get all
```
![image](https://github.com/user-attachments/assets/7b764496-9c69-4175-82f0-9fd3be59cbac)
