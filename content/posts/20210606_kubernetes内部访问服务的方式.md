---
title: kubernetes内部访问服务的方式
date: 2021-06-06T16:23:14+08:00
tags:
  - 内容/实践记录
  - 计算机/kubernetes
categories:
  - codebase
---

最近因为实验室集群整体爆炸，需要修改旧服务器的网络配置。为此，需要进行一系列的网络测试。同时，这也是一个新的label系列。总是有一些代码，非常常用，但是用的时候就是找不到，想也想不起来，就非得去查。很难受。

之前做网络测试的手段太原始了（指新建ubuntu容器后登陆进去），不够灵活方便，而且也找不到代码和镜像了。为此，我总结了几个比较好的快速访问方式

## 方式1

最直接的方式肯定是登陆进服务内部，比如istio中提到的

在执行命令后：`kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml`，如果此时所有的svc和pods都跑起来了，可以通过运行`kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"`来检测到结果。

其中的`kubectl exec -it`可以登陆容器并打开控制台，而服务的名称由`$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')`，`-c`参数保证了在存在注入的情况下能正常运行。

`--`连接两条不同的命令，后面就不需要过多的解释了。

甚至不一定是同一个镜像，比如使用sleep镜像进行（足够小）
```bash
export SLEEP_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})#这里要自行加上-n=test
kubectl exec -it $SLEEP_POD -c sleep curl http://ratings.default.svc.cluster.local:9080/ratings/1
{"id":1,"ratings":{"Reviewer1":5,"Reviewer2":4}}
```

## 方式2

临时开一个curl镜像进行网络检查

官方在httpbin里也展示了如何通过`curl`镜像来进行内网测试(同样，注意命名空间)
```bash
kubectl run -i --rm --restart=Never dummy --image=dockerqa/curl:ubuntu-trusty --command -- curl --silent httpbin:8000/html
kubectl run -i --rm --restart=Never dummy --image=dockerqa/curl:ubuntu-trusty --command -- curl --silent --head httpbin:8000/status/500
time kubectl run -i --rm --restart=Never dummy --image=dockerqa/curl:ubuntu-trusty --command -- curl --silent httpbin:8000/delay/5
```

## 网络测试的对象
我一开始想用hello来做，但是其实挺不好的，没什么代表性。
echo-server这个就很不错，像是一个回音壁一样，将所有的请求全部打回。
`docker pull ealen/echo-server`

```yaml
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
      - image: ealen/echo-server:0.5.1
        imagePullPolicy: Always
        name: echoserver
        ports:
        - containerPort: 80
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
  selector:
    app: echoserver
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: echoserver
  namespace: echoserver
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: echo.minikube.local
    http:
      paths:
      - path: /
        backend:
          serviceName: echoserver
          servicePort: 80
```