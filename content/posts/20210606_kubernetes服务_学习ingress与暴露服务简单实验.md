---
title: kubernetes服务：学习ingress与暴露服务简单实验
date: 2021-06-06T16:30:14+08:00
tags:
  - 分类/实践记录
  - 计算机/云计算/kubernetes
categories:
  - 操作实践
---

参考资料：
* [kubernetes ingress实战](https://blog.frognew.com/2018/06/kubernetes-ingress-2.html)
* [Intro to Kube ingress: Set up nginx Ingress in Kubernetes Bare Metal](https://www.fairwinds.com/blog/intro-to-kubernetes-ingress-set-up-nginx-ingress-in-kubernetes-bare-metal)
* [ingress的nip.io相关资料](https://github.com/OpenLiberty/ci.docker/blob/master/docs/ingress-configurations.md)

实验室集群中已经有一个ingress-controller，ingress.class为nginx。
本次实验的目标是将服务通过ingress暴露到外部服务，最好能够直接通过外网IP访问。

## 实验所需镜像

原本用的是google的镜像，但是因为在国内，弄起来太麻烦了。所以还是从dockerhub上选了一个类似的镜像。
```
apiVersion: v1
kind: Namespace
metadata:
  name: echoserver
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echoserver
  namespace: echoserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app: echoserver
  template:
    metadata:
      labels:
        app: echoserver
    spec:
      containers:
      - image: ealen/echo-server:latest
        imagePullPolicy: IfNotPresent
        name: echoserver
        ports:
        - containerPort: 80
        env:
        - name: PORT
          value: "80"
---
apiVersion: v1
kind: Service
metadata:
  name: echoserver
  namespace: echoserver
spec:
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
  type: ClusterIP
  selector:
    app: echoserver
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - http:
      paths:
      - path: /echoserver
        pathType: Prefix
        backend:
          service:
            name: echoserver
            port:
              number: 80
```

## 进行测试

### 集群内功能测试
通过执行`kubectl get svc -n=echoserver`可以拿到对应服务的ClusterIP
再执行`kubectl run -i --rm --restart=Never dummy --image=dockerqa/curl:ubuntu-trusty -n=echoserver --command -- curl 10.108.58.132:80 `即可在集群内部访问到目标服务。结果可以经过jq整理，为
```json
{
  "host": {
    "hostname": "10.108.58.132",
    "ip": "::ffff:10.244.3.188",
    "ips": []
  },
  "http": {
    "method": "GET",
    "baseUrl": "",
    "originalUrl": "/",
    "protocol": "http"
  },
  "request": {
    "params": {
      "0": "/"
    },
    "query": {},
    "cookies": {},
    "body": {},
    "headers": {
      "user-agent": "curl/7.35.0",
      "host": "10.108.58.132",
      "accept": "*/*"
    }
  },
  "environment": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOSTNAME": "echoserver-6944fb9c86-mn6vz",
    "PORT": "80",
    "KUBERNETES_PORT_443_TCP": "tcp://10.96.0.1:443",
    "KUBERNETES_PORT_443_TCP_ADDR": "10.96.0.1",
    "ECHOSERVER_PORT_80_TCP": "tcp://10.108.58.132:80",
    "KUBERNETES_SERVICE_PORT_HTTPS": "443",
    "KUBERNETES_PORT": "tcp://10.96.0.1:443",
    "KUBERNETES_PORT_443_TCP_PROTO": "tcp",
    "KUBERNETES_PORT_443_TCP_PORT": "443",
    "ECHOSERVER_SERVICE_HOST": "10.108.58.132",
    "ECHOSERVER_PORT": "tcp://10.108.58.132:80",
    "KUBERNETES_SERVICE_HOST": "10.96.0.1",
    "KUBERNETES_SERVICE_PORT": "443",
    "ECHOSERVER_SERVICE_PORT": "80",
    "ECHOSERVER_PORT_80_TCP_PROTO": "tcp",
    "ECHOSERVER_PORT_80_TCP_PORT": "80",
    "ECHOSERVER_PORT_80_TCP_ADDR": "10.108.58.132",
    "NODE_VERSION": "12.19.0",
    "YARN_VERSION": "1.22.5",
    "HOME": "/root"
  }
}
```

### 集群外功能测试

通过`kubectl get ingress -n=echoserver`可以确定目标ingress的内网IP地址。
再使用`curl 192.168.1.10/echoserver`，ingress-controller在工作正常的情况下即可完成对应的转发
```json
{
  "host": {
    "hostname": "192.168.1.10",
    "ip": "::ffff:10.244.1.201",
    "ips": []
  },
  "http": {
    "method": "GET",
    "baseUrl": "",
    "originalUrl": "/echoserver",
    "protocol": "http"
  },
  "request": {
    "params": {
      "0": "/echoserver"
    },
    "query": {},
    "cookies": {},
    "body": {},
    "headers": {
      "host": "192.168.1.10",
      "x-request-id": "1e1d9bcdb01d497e7cb1745a1ac6ff0e",
      "x-real-ip": "10.244.0.0",
      "x-forwarded-for": "10.244.0.0",
      "x-forwarded-host": "192.168.1.10",
      "x-forwarded-port": "80",
      "x-forwarded-proto": "http",
      "x-scheme": "http",
      "user-agent": "curl/7.63.0",
      "accept": "*/*"
    }
  },
  "environment": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOSTNAME": "echoserver-6944fb9c86-mn6vz",
    "PORT": "80",
    "KUBERNETES_PORT_443_TCP": "tcp://10.96.0.1:443",
    "KUBERNETES_PORT_443_TCP_ADDR": "10.96.0.1",
    "ECHOSERVER_PORT_80_TCP": "tcp://10.108.58.132:80",
    "KUBERNETES_SERVICE_PORT_HTTPS": "443",
    "KUBERNETES_PORT": "tcp://10.96.0.1:443",
    "KUBERNETES_PORT_443_TCP_PROTO": "tcp",
    "KUBERNETES_PORT_443_TCP_PORT": "443",
    "ECHOSERVER_SERVICE_HOST": "10.108.58.132",
    "ECHOSERVER_PORT": "tcp://10.108.58.132:80",
    "KUBERNETES_SERVICE_HOST": "10.96.0.1",
    "KUBERNETES_SERVICE_PORT": "443",
    "ECHOSERVER_SERVICE_PORT": "80",
    "ECHOSERVER_PORT_80_TCP_PROTO": "tcp",
    "ECHOSERVER_PORT_80_TCP_PORT": "80",
    "ECHOSERVER_PORT_80_TCP_ADDR": "10.108.58.132",
    "NODE_VERSION": "12.19.0",
    "YARN_VERSION": "1.22.5",
    "HOME": "/root"
  }
}
```

如果将ingress改为
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: http-svc.frognew.com
    http:
      paths:
      - path: /echoserver
        pathType: Prefix
        backend:
          service:
            name: echoserver
            port:
              number: 80
```
则可以使用如下命令进行测试` curl 192.168.1.10/echoserver -H "Host: http-svc.frognew.com"`
但是在集群内部使用`curl http-svc.frognew.com/echoserver`则无法解析，会报404错误。就是说nginx并不会对其进行解析。我的一个猜想是这个设置应该要在IngressController中进行，而不能只在ingress中设置，不过暂时还未成功。

### 快速总结

从上述两个实验可以很快速的认识到yaml配置文件中的项目内容。
ingress中配置的实际上是转发的URL，其本质和服务器的router是类似的。对于给定的IP请求，一旦满足指定的router，nginx controller就会将对应的请求发送到指定的服务上，服务通过port收到内容，再通过targetPort将请求投送到pod的指定端口中，完成转发。

## host与nip.io

如果需要使用host解析的话，正常情况下需要ELB的设置。但是使用nip.io也可以达到类似的效果
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: echoserver.192.168.1.10.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: echoserver
            port:
              number: 80
```

通过上述的修改后，可以直接在集群外部使用`curl echoserver.192.168.1.10.nip.io`访问到服务，得到的结果如下（经过jq整理）
```json
{
  "host": {
    "hostname": "echoserver.192.168.1.10.nip.io",
    "ip": "::ffff:10.244.1.201",
    "ips": []
  },
  "http": {
    "method": "GET",
    "baseUrl": "",
    "originalUrl": "/",
    "protocol": "http"
  },
  "request": {
    "params": {
      "0": "/"
    },
    "query": {},
    "cookies": {},
    "body": {},
    "headers": {
      "host": "echoserver.192.168.1.10.nip.io",
      "x-request-id": "7b8b5cd857f5771388cd349d395fe979",
      "x-real-ip": "10.244.0.0",
      "x-forwarded-for": "10.244.0.0",
      "x-forwarded-host": "echoserver.192.168.1.10.nip.io",
      "x-forwarded-port": "80",
      "x-forwarded-proto": "http",
      "x-scheme": "http",
      "user-agent": "curl/7.63.0",
      "accept": "*/*"
    }
  },
  "environment": {
    "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "HOSTNAME": "echoserver-6944fb9c86-btmf8",
    "PORT": "80",
    "KUBERNETES_SERVICE_PORT_HTTPS": "443",
    "KUBERNETES_PORT_443_TCP_PROTO": "tcp",
    "ECHOSERVER_SERVICE_PORT": "80",
    "ECHOSERVER_PORT_80_TCP": "tcp://10.108.58.132:80",
    "ECHOSERVER_PORT_80_TCP_PORT": "80",
    "ECHOSERVER_PORT_80_TCP_PROTO": "tcp",
    "KUBERNETES_SERVICE_HOST": "10.96.0.1",
    "KUBERNETES_PORT_443_TCP_PORT": "443",
    "ECHOSERVER_PORT": "tcp://10.108.58.132:80",
    "KUBERNETES_PORT": "tcp://10.96.0.1:443",
    "KUBERNETES_PORT_443_TCP": "tcp://10.96.0.1:443",
    "KUBERNETES_PORT_443_TCP_ADDR": "10.96.0.1",
    "ECHOSERVER_SERVICE_HOST": "10.108.58.132",
    "ECHOSERVER_PORT_80_TCP_ADDR": "10.108.58.132",
    "KUBERNETES_SERVICE_PORT": "443",
    "NODE_VERSION": "12.19.0",
    "YARN_VERSION": "1.22.5",
    "HOME": "/root"
  }
}
```