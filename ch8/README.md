## 參數說明
headless-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: foo-service
  labels:
    app: foo
spec:
  ports:
    - port: 80
      name: http
  clusterIP: None # 表示這是一個 Headless Service，不會有代表他的 cluster IP，而是直接回傳後端 Pod 的 IP
  selector: # 說明哪些 Pod 會被這個 Service 管理
    app: foo

```

statefulset.yaml
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: foo-statefulset
spec:
  updateStrategy:
    type: RollingUpdate # 滾動更新，用戶更新 .spec.template 時將自動刪除並重建 StatefulSet 中的每一個 Pod
  selector:
    matchLabels:
      app: foo
  serviceName: "foo-service" # 指定與 StatefulSet 相關聯的 Headless Service 的名稱
  replicas: 3
  template:
    metadata:
      labels:
        app: foo
    spec:
      terminationGracePeriodSeconds: 10 # 在 k8s 發送終止信號到 Pod 中的所有容器後，應給予多長的緩衝期，以供 Pod 內的容器進行清理工作
      containers:
        - name: foo
          image: nginx
          ports:
            - containerPort: 80
              name: http
          volumeMounts:
            - name: pvc
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: pvc
      spec: # 這是一個 PersistentVolumeClanim (PVC) 的模板列表，定義 StatefulSet 中每個 Pod 所使用的 PVC
        accessModes: ["ReadWriteOnce"] # 定義如何存取 PersistentVolume(PV)
        storageClassName: "local-path" # 定義了用於建立 PV 的 StorageClass 的名稱
        resources:
          requests:
            storage: 1Gi # 定義了 PV 的大小需求

```
> Note: 此處 storageClassName 修改為 local-path 是因為我的 default StorageClass 為 `local-path`，若不修改此處則無法達到作者的範例結果

執行設定
```
kubectl apply -f statefulset.yaml,headless-service.yaml
```

## 查看 default 的 StorageClass
```
kubectl get storageclass
```
<img width="920" alt="image" src="https://github.com/user-attachments/assets/dbe54ded-46a8-4405-b7d4-d3cc81e07043" />

> Note: 從此可知 `local-path` 為我的 default StorageClass

## 驗證 Headless Service 是否已經正確設定並運行
Headless Service 的核心功能：
- 不分配 ClusterIP：不分配虛擬 IP，而是直接返回後端 Pod 的 IP
- DNS 結構：每個 Pod 都有穩定的 DNS 名稱
  ```
  <Pod 名稱>.<Service 名稱>.<Namespace>.svc.cluster.local
  ```
- Pod 直接訪問：讓內部 Pod 直接通過 Pod IP 進行通信，而不是通過 ClusterIP 進行負載平衡

測試 Headless Service 的功能點：
- DNS 名稱解析：測試是否能通過 Service 名稱解析到 Pod 的 IP
- 網路通路：測試是否能通過 Pod 的 IP 訪問該 Pod 的服務

以下指令達到了
- 進入「foo-statefulset-0」Pod
- 使用 Headless Service 訪問另一個 Pod「foo-statefulset-1」
```
kubectl exec -it foo-statefulset-0 -- bash
```

```
curl foo-statefulset-1.foo-service
```
<img width="615" alt="image" src="https://github.com/user-attachments/assets/3b102e99-9638-449d-9339-755bf6f540cf" />

## 更新 StatefulSet
```
kubectl rollout restart statefulset foo-statefulset
```
可以觀察到 StatefulSet 照著預期，依序由最後重新啟動並且等到每個 Pod 的前一位依賴者狀態為 Running 後才開始更新。
