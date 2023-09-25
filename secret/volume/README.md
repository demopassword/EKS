## Secret Volume Mount

configmap & secret setting
```bash
k apply -f ./configmap.yaml && k apply -f ./secret.yaml
```

deployment apply
```bash
k apply -f ./deployment.yaml
```

result
```
k exec -it nginx-788cdb494c-6fncs -- cat /tmp/config/ENV1
hello
[root@ip-10-0-1-194 test1]# k exec -it nginx-788cdb494c-6fncs -- cat /tmp/secret/ENV2
hello
```