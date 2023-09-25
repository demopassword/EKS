References
- https://docs.aws.amazon.com/ko_kr/secretsmanager/latest/userguide/integrating_csi_driver.html


```
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install -n kube-system csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver

helm repo add aws-secrets-manager https://aws.github.io/secrets-store-csi-driver-provider-aws
helm install -n kube-system secrets-provider-aws aws-secrets-manager/secrets-store-csi-driver-provider-aws
```
```
aws --region ap-northeast-2 secretsmanager \
  create-secret --name secret_test \
  --secret-string '{"username":"foo", "password":"super-sekret"}'

SECRET_ARN=$(aws --region ap-northeast-2 secretsmanager \
    describe-secret --secret-id  secret_test \
    --query 'ARN' | sed -e 's/"//g' )

echo $SECRET_ARN
```

```
aws --region ap-northeast-2 iam \
	create-policy --query Policy.Arn \
    --output text --policy-name secret_policy \
    --policy-document '{
    "Version": "2012-10-17",
    "Statement": [ {
        "Effect": "Allow",
        "Action": ["secretsmanager:GetSecretValue", "secretsmanager:DescribeSecret"],
        "Resource": ["'"$SECRET_ARN"'" ]
    } ]
}'
```

```
eksctl utils associate-iam-oidc-provider \
    --region=ap-northeast-2 --cluster=skills-cluster \
    --approve

eksctl create iamserviceaccount \
    --region=ap-northeast-2 --name "secret-deployment-sa"  \
    --cluster skills-cluster \
    --attach-policy-arn arn:aws:iam::532003114460:policy/secret_policy --approve \
    --override-existing-serviceaccounts
```

```
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: test-deployment-spc
spec:
  provider: aws
  parameters:
    objects: |
        - objectName: "arn:aws:secretsmanager:ap-northeast-2:532003114460:secret:secret_test-BAT9AM"
          objectType: "secretsmanager"
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: secret-deployment-sa
      containers:
      - name: nginx-deployment
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets"
          readOnly: true
      volumes:
      - name: secrets-store-inline
        csi:
          driver: secrets-store.csi.k8s.io
          readOnly: true
          volumeAttributes:
            secretProviderClass: test-deployment-spc
```

```
export POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath='{.items[].metadata.name}')
kubectl exec -it ${POD_NAME} -- cat /mnt/secrets/arn\:aws\:secretsmanager\:ap-northeast-2\:532003114460\:secret\:secret_test-BAT9AM; echo
{"username":"foo", "password":"super-sekret"}
```