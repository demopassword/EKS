References
- https://github.com/kubernetes-sigs/metrics-server

install
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

edit commands
```
k edit -n kube-system deploy/metrics-server
```

```
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --kubelet-insecure-tls # add field
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

```
# k get hpa -A
NAMESPACE   NAME         REFERENCE                   TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
default     deploy-hpa   Deployment/default-deploy   0%/60%, 0%/60%   2         20        2          2m10s
```