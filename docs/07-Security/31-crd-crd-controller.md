# Kubernetes 自定义资源定义（CRD）实践

## 自定义资源定义（CRD）示例

### CRD 定义文件 (crd.yaml)
```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com
spec:
  group: datasets.kodekloud.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
```

### 创建 CRD
```bash
controlplane ~ ➜  kubectl apply -f crd.yaml 
customresourcedefinition.apiextensions.k8s.io/internals.datasets.kodekloud.com created
```

### 查看 CRD
```bash
controlplane ~ ➜  kubectl get crd
NAME                               CREATED AT
collectors.monitoring.controller   2026-01-01T13:47:08Z
globals.traffic.controller         2026-01-01T13:47:08Z
internals.datasets.kodekloud.com   2026-01-01T13:58:18Z
```

## 自定义资源（CR）示例

### 自定义资源文件 (custom.yaml)
```yaml
---
kind: Internal
apiVersion: datasets.kodekloud.com/v1
metadata:
  name: internal-space
  namespace: default
spec:
  internalLoad: "high"
  range: 80
  percentage: "50"
```

### 创建自定义资源
```bash
controlplane ~ ✖ kubectl apply -f custom.yaml
internal.datasets.kodekloud.com/internal-space created
```

### 查看自定义资源
```bash
controlplane ~ ➜  kubectl get internal
NAME             AGE
internal-space   6s
```

## 另一个自定义资源示例

### Global 类型自定义资源
```yaml
kind: Global
apiVersion: traffic.controller/v1
metadata:
  name: datacenter
spec:
  dataField: 2
  access: true
```
