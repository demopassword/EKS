## configmap volume mount
configmap pods에 데이터를 삽입하는방법

#### 컨피그맵을 생성합니다.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-config
data:
  log_level: info
```

#### pod /etc/config/ 경로로 마운트
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```

command
```sh
k apply -f ./log-config.yaml && k apply -f ./log-pod.yaml
```


result
```
[root@ip-10-0-1-194 configmap-pod]# k exec -it configmap-pod -- cat /etc/config/log_level
info
```

## 응용
#### configmap으로 index.html 작성하고 nginx pod에 띄우기
#### 컨피그 맵을 생성합니다.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: hello world
```

#### pod /usr/share/nginx/html/ 경로로 마운트
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - name: nginx-vol
          mountPath: /usr/share/nginx/html/
  volumes:
    - name: nginx-vol
      configMap:
        name: nginx-config
        items:
          - key: index.html
            path: index.html
```
command
```sh
k apply -f ./nginx-config.yaml && k apply -f ./nginx-pod.yaml
```

result
```
k exec -it nginx-pods -- curl localhost
hello world
```