## Env From Secret
#### setting env
```
k apply -f ./configmap.yaml && k apply -f ./secret.yaml
```

#### deploy application
```
k apply -f ./deployment.yaml
```

result
```
[root@ip-10-0-1-194 123]# k exec -it deploy/test -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=test-fb9f7f8ff-mmbxp
NGINX_VERSION=1.25.2
NJS_VERSION=0.8.0
PKG_RELEASE=1~bookworm
ENV1=hello
ENV2=hello
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://172.20.0.1:443
KUBERNETES_PORT_443_TCP=tcp://172.20.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=172.20.0.1
KUBERNETES_SERVICE_HOST=172.20.0.1
TERM=xterm
HOME=/root
```