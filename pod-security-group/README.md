## kubernetes pod security group setting

References
- https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/security-groups-for-pods.html

## VPC CNI plugin for Kubernetes 
```bash
cluster_role=$(aws eks describe-cluster --name skills-cluster --query cluster.roleArn --output text | cut -d / -f 2)
aws iam attach-role-policy --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController --role-name $cluster_role
```

```
kubectl set env daemonset aws-node -n kube-system ENABLE_POD_ENI=true
```

### TCP 초기 Demux를 비활성화
```bash
kubectl patch daemonset aws-node -n kube-system \
  -p '{"spec": {"template": {"spec": {"initContainers": [{"env":[{"name":"DISABLE_TCP_EARLY_DEMUX","value":"true"}],"name":"aws-vpc-cni-init"}]}}}}'
```

---
### calico 네트워크 정책이나, 보안 그룹을 할당하려는 Pods에 대해 externalTrafficPolicy를 Local로 설정한 인스턴스 대상을 사용하는 NodePort 및 LoadBalancer 유형의 Kubernetes 서비스가 있는 경우 사용함
```
kubectl set env daemonset aws-node -n kube-system POD_SECURITY_GROUP_ENFORCING_MODE=standard
```

```
cat >my-security-group-policy.yaml <<EOF
apiVersion: vpcresources.k8s.aws/v1beta1
kind: SecurityGroupPolicy
metadata:
  name: my-security-group-policy
  namespace: my-namespace
spec:
  podSelector: 
    matchLabels:
      role: my-role
  securityGroups:
    groupIds:
      - my_pod_security_group_id
EOF
```