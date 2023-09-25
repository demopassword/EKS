## aws-auth setting
```
k edit -n kube-system cm/aws-auth
```

#### Grant all privileges to the iam role (default)
```
apiVersion: v1
data:
  mapUsers: |
    - groups:
      - rbac.authorization.k8s.io/v1
      userarn: arn:aws:iam::532003114460:user/test
      username: test
```

#### setting
```
aws configure
```
![Alt text](image.png)

#### Permission Control Settings
```
k apply role.yaml && k apply -f ./rolebinding.yaml
```

## Test
![Alt text](image-1.png)
---

#### setting
```
aws configure
```
![Alt text](image.png)

#### Create per cluster
```
k apply -f ./cluster-role.yaml && k apply -f ./clusterrolebinding.yaml
```

## Test
![Alt text](image-2.png)