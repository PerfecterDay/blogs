## Ingress
{docsify-updated}

- [Ingress](#ingress)
	- [Ingress 和 Service 的区别](#ingress-和-service-的区别)


Ingress 公开从集群外部到集群内服务的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。

### Ingress 和 Service 的区别
由于您只有一个提供 http 服务的服务，因此您目前使用 LoadBalancer 服务类型的解决方案效果很好。试想一下，你有多个基于 http 的服务，你想让它们在不同的路由上对外提供服务。您必须为每个服务创建一个 LoadBalancer 服务，默认情况下，您将为每个服务获得不同的 IP 地址。相反，您可以使用 Ingress，它位于这些服务之前并负责路由选择。
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /cart
        backend:
          serviceName: cart
          servicePort: 80
     - path: /payment
        backend:
          serviceName: payment
          servicePort: 80
```

这样的话，对外部调用方来说，只要调用暴露的这个ingress端点即可。由 ingress 去做路由分发到不通服务。