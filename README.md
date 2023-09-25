## EKS Cluster manifest
Management Node Cluster
- CA
- Karpenter
- ALL

## ingress controller install
```
kubectl label namespace default elbv2.k8s.aws/pod-readiness-gate-inject=enabled
```
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=skills-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set image.repository=602401143452.dkr.ecr.ap-northeast-2.amazonaws.com/amazon/aws-load-balancer-controller \
    --set region=ap-northeast-2 \
    --set vpcId=vpc-018aa39c1ecd1bfac
```

## pod security
```
kubectl set env daemonset aws-node -n kube-system ENABLE_POD_ENI=true
kubectl set env daemonset aws-node -n kube-system POD_SECURITY_GROUP_ENFORCING_MODE=standard
```