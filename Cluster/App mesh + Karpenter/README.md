## appmesh-controller install
```
helm upgrade -i appmesh-controller eks/appmesh-controller \
    --namespace appmesh-system \
    --set region=$AWS_REGION \
    --set serviceAccount.create=false \
    --set serviceAccount.name=appmesh-controller
```

## jaeger
```
helm upgrade -i appmesh-controller eks/appmesh-controller     --namespace appmesh-system     --set tracing.enabled=true     --set tracing.provider=jaeger     --set tracing.address=appmesh-jaeger.appmesh-system     --set tracing.port=9411     --set serviceAccount.create=false
```

## edit appmesh-controller image
```
k edit -n appmesh-system deploy/appmesh-controller
:%s/us-west-2/ap-northeast-2/g
```

envoy proxy irsa
```
curl -o envoy-iam-policy.json https://raw.githubusercontent.com/aws/aws-app-mesh-controller-for-k8s/master/config/iam/envoy-iam-policy.json
aws iam create-policy \
    --policy-name AWSAppMeshEnvoyIAMPolicy \
    --policy-document file://envoy-iam-policy.json

eksctl create iamserviceaccount --cluster skills-cluster \
    --namespace skills \
    --name envoy-proxy \
    --attach-policy-arn arn:aws:iam::532003114460:policy/AWSAppMeshEnvoyIAMPolicy  \
    --override-existing-serviceaccounts \
    --approve
```
