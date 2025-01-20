## Simple Kustomization
deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
  labels:
    type: demo
spec:
  replicas: 2 # 需要產生多少個 Pod (水平擴展的關鍵)
  selector:
    matchLabels: # 定義匹配標籤的 Pod 被 deployment 管理
      type: demo
  template:
    metadata:
      labels: # 設定 template.spec 的 Label
        type: demo
    spec:
      containers: # Pod 中運行的容器設置
        - name: api-service
          image: mikehsu0618/api-service:tag
          ports:
            - containerPort: 8080

```

service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: service # Service name (UQ)
spec:
  selector: # 用於選擇將流量導向哪些 Pod
    type: demo # 標籤 type=demo 的 Pod
  type: LoadBalancer # Service 的型別。可為 ClusterIP(default) / NodePort / LoadBalancer / ExternalName
  ports:
    - protocol: TCP # 可為 TCP、SCTP、UDP
      port: 8000 # Local
      targetPort: 8080 # Pod

```

kustomization.yaml
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 在對應資源名稱添加前後綴，用以區分服務名稱
namePrefix: foo-
nameSuffix: -v1

commonLabels: # 在所有導入資源皆附上對應 Label，如果已經存在就覆蓋
  by: kustomization

commonAnnotations: # 在所有導入資源皆附上對應 Annotations，如果已經存在就覆蓋
  note: Hello, I am foo!


# 使用 images.name 從導入設定檔中指定要覆寫的 newName、newTag
images:
  - name: mikehsu0618/api-service
    newName: mikehsu0618/foo
    newTag: v1.0.0

resources: # 需要被 kustomization 覆寫的設定擋路徑
  - deployment.yaml
  - service.yaml

```

查看 Kustomize 產生的結果
```
kubectl kustomize ./
```

## Overlay Kustomization
development/kustomization.yaml
```
resources: # 需要被 kustomization 覆寫的設定擋路徑
  - ../../base

# 在對應資源名稱添加前後綴，用以區分服務名稱
namePrefix: dev-

namespace: dev-namespace

commonLabels: # 在所有導入資源皆附上對應 Label，如果已經存在就覆蓋
  type: dev-demo
  app: dev-foo

commonAnnotations: # 在所有導入資源皆附上對應 Annotations，如果已經存在就覆蓋
  note: Hello, I am development!

# 使用 images.name 從導入設定檔中指定要覆寫的 newName、newTag
images:
  - name: mikehsu0618/api-service
    newTag: development

patches:
  - patch: |
      - op: replace
        path: /metadata/name
        value: the-dev-development
      - op: replace
        path: /spec/template/spec/containers/0/name
        value: the-dev-container
    target:
      group: apps
      kind: Deployment
      version: v1
      name: foo-deployment-v1

```

production/kustomization.yaml
```
resources: # 需要被 kustomization 覆寫的設定擋路徑
  - ../../base # 指向另一個 kustomization.yaml 並且疊加設定上去

# 在對應資源名稱添加前後綴，用以區分服務名稱
namePrefix: prod-

namespace: production-namespace

commonLabels: # 在所有導入資源皆附上對應 Label，如果已經存在就覆蓋
  type: prod-demo
  app: prod-foo

commonAnnotations: # 在所有導入資源皆附上對應 Annotations，如果已經存在就覆蓋
  note: Hello, I am Production!

# 使用 images.name 從導入設定檔中指定要覆寫的 newName、newTag
images:
  - name: mikehsu0618/api-service
    newTag: production

patches: # 指定符合 target 條件資源中的任何 key value，並使用各種 op 達到替換、添加、移除等操作
  - patch: |
      - op: replace
        path: /metadata/name
        value: the-prod-development
      - op: replace
        path: /spec/template/spec/containers/0/name
        value: the-prod-container
    target:
      group: apps
      kind: Deployment
      version: v1
      name: foo-deployment-v1

```
![image](https://github.com/user-attachments/assets/07e10cb5-751f-4d7b-8830-5967eb797263)
