## Open Agent Policy
References
- https://devocean.sk.com/experts/techBoardDetail.do?page=&boardType=undefined&query=&ID=165215&searchData=&subIndex=

command
```
k apply -f constrainte-template.yaml && k apply -f constrainte.yaml
```

constrainte.yaml
```
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-gk
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels: ["apps"]
```

deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-1
  namespace: default
  labels:
    app.kubernetes.io/name: app-v1
spec:
  minReadySeconds: 2
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/name: app-v1
  template:
    metadata:
      labels:
        app.kubernetes.io/name: app-v1
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.0
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-2
  namespace: default
  labels:
    apps: app-v2
spec:
  minReadySeconds: 2
  replicas: 3
  selector:
    matchLabels:
      apps: app-v2
  template:
    metadata:
      labels:
        apps: app-v2
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.0
        ports:
        - containerPort: 80
```

result
```
[root@ip-10-0-0-153 k8s-app]# k apply -f ./deployment.yaml
deployment.apps/test-2 unchanged
Error from server (Forbidden): error when creating "./deployment.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [ns-must-have-gk] you must provide labels: {"apps"}
```
