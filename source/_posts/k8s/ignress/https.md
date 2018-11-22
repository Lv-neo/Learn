---
title: ignress https证书配置
categories: k8s
tags:
  - k8s
  - ignress
---

# ignress https证书配置

我们使用阿里云的架构之中配置https证书有以下几种方式

* slb配置https泛域名证书
* ignress配置https证书
* 容器配置https证书

我们的k8s上面放了不同域名，所以我们选择更加轻便简洁的ignress上配置https证书

## k8s secret 配置tls证书

```
kubectl create secret tls secret-https --key tls.key --cert tls.crt -n php
```

## ingress上配置https

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ${JOB_NAME}
  namespace: php
spec:
  tls:
  - hosts:
    - foo.bar.com
    secretName: secret-https
  rules:
  - host: foo.bar.com
    http:
      paths:
      - backend:
         serviceName: ${JOB_NAME}
         servicePort: 80
```

## 附录

* k8s说明 https://kubernetes.github.io/ingress-nginx/user-guide/tls/
* 阿里云说明 https://help.aliyun.com/document_detail/93804.html
