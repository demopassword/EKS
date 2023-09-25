#### create user
```
adduser -m testuser
su testuser
cd ~
```


#### connect eks cluster
```
aws eks update-kubeconfig --name skills-cluster --region ap-northeast-2
exit
```

#### sa & sa token create
```
k apply -f sa.yaml && k apply -f ./secret.yaml
kubectl get secret demo-secret -o jsonpath='{.data.*}' | base64 -d
```
// copy token
![Alt text](image-2.png)


```
k apply -f role.yaml && k apply -f rolebinding.yaml
```

Apply Linux Users
```
su testuser
vim ~/.kube/config
```
![Alt text](image-3.png)

## TEST
![Alt text](image.png)

---

cluster role apply
```
k apply -f ./clusterrole.yaml && k apply -f ./clusterrolebinding.yaml
```
Apply Linux Users
```
su testuser
vim ~/.kube/config
```

## TEST
![Alt text](image-1.png)