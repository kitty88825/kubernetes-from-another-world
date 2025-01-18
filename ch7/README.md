## 參數說明
deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: foo-deployment
  labels:
    type: demo
spec:
  replicas: 1 # 需要產生多少個 Pod (水平擴展的關鍵)
  selector:
    matchLabels: # 定義匹配標籤的 Pod 被 deployment 管理
      type: demo
  template:
    metadata:
      labels: # 設定 template.spec 的 Label
        type: demo
    spec:
      containers: # Pod 中運行的容器設置
        - name: foo
          image: mikehsu0618/foo
          ports:
            - containerPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bar-deployment
  labels:
    type: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      type: demo
  template:
    metadata:
      labels:
        type: demo
    spec:
      containers:
        - name: bar
          image: mikehsu0618/bar
          ports:
            - containerPort: 8080

```
執行指令
```
kubectl apply -f deployment.yaml
```
<img width="750" alt="image" src="https://github.com/user-attachments/assets/8fa70861-44fc-475c-a0fa-178e59c93750" />

## 調整 Replicas 達到水平擴展
v2 deployment.yaml
```
...
metadata:
  name: foo-deployment
  labels:
    type: demo
spec:
  replicas: 2 # 需要產生多少個 Pod (水平擴展的關鍵)
  selector:
    matchLabels: # 定義匹配標籤的 Pod 被 deployment 管理
      type: demo
...
```

> Note：作者使用 `--record` 查看變更歷史，但已被 Kubernetes 宣告為棄用，將在未來的版本中移除。Kubernetes 社群認為，--record 的功能並不必要，因為使用 kubectl annotate 等命令可以手動更新 kubernetes.io/change-cause annotation。

<img width="753" alt="image" src="https://github.com/user-attachments/assets/c1a00224-db49-4eb7-b502-e0a9a161d13f" />

## 查看更新狀態
查看 foo-deployment 的管理狀態
```
kubectl rollout status deployment foo-deployment
```

## 調整 Deployment 的方式
1. 直接修改 yaml file
2. 指令更新
   ```
   kubectl scale deployment bar-deployment --replicas 3
   ```
3. 直接編輯在 Kubernetes 運行中的 Deployment 設定
   ```
   kubectl edit deploy bar-deployment
   ```
## 查看歷史版本並回滾
v3 deployment.yaml
```
...
  template:
    metadata:
      labels:
        type: demo
    spec:
      containers:
        - name: bar
          image: mikehsu0618/bar:v1 # 製造錯誤
          ports:
            - containerPort: 8080

...
```

更新並產生紀錄
```
kubectl apply -f deployment.yaml --record
```

查看 revision
```
kubectl rollout history deployment bar-deployment
```
<img width="549" alt="image" src="https://github.com/user-attachments/assets/164c2862-cf54-41b6-9dea-0e154b93cfad" />

查看指定 revision 詳細資訊
```
kubctl rollout history deployment bar-deployment --revision=2
```

<img width="794" alt="image" src="https://github.com/user-attachments/assets/5c494ef7-4ebc-45c9-8d4d-f75c5b370d60" />

<img width="814" alt="image" src="https://github.com/user-attachments/assets/0655504d-4035-4a79-b73a-942ec194c86b" />

使用 rollout 進行回滾
```
kubectl rollout undo deployment bar-deployment
```
or 指定版本的回滾
```
kubectl rollout undo deployment bar-deployment --to-revision=1
```

再次查看狀態可以看到回滾到沒問題的 revision 1 版本了

<img width="728" alt="image" src="https://github.com/user-attachments/assets/65ab1cef-6676-4462-9978-8d3dc772d09c" />
