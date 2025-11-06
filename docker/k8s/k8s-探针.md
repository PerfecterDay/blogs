# k8s 探针
{docsify-updated}

## 存活探针-探活失败重启POD
存活探针决定何时重启容器。 例如，当应用在运行但死锁变得无响应时，存活探针可以捕获这类死锁。如果一个容器的存活探针失败多次，kubelet 将**重启该容器**。

**存活探针不会等待就绪探针成功**。 如果你想在执行存活探针前等待，你可以定义 `initialDelaySeconds` ，或者使用启动探针。

## 就绪探针-就绪探测失败就禁止流量转发
**就绪探针决定容器何时准备好接受流量**。 这种探针在等待应用执行耗时的初始任务时非常有用； 例如：建立网络连接、加载文件和预热缓存。

在容器的生命周期后期， 就绪探针也很有用，例如，从临时故障或过载中恢复时。

如果就绪探针返回的状态为失败，**Kubernetes 会将该 Pod 从所有对应服务的端点中移除**。

就绪探针在容器的整个生命期内**持续运行**。


## 启动探针-启动成功之前不进行就绪探测和探活检测
启动探针检查容器内的应用是否已启动。 **启动探针可以用于对慢启动容器进行存活性检测，避免它们在启动运行之前就被 kubelet 杀掉。如果配置了这类探针，它会禁用存活检测和就绪检测，直到启动探针成功为止。**

这类探针**仅在启动时执行**，不像存活探针和就绪探针那样周期性地运行。

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    ports:
    - name: liveness-port
      containerPort: 8080
    livenessProbe:
      httpGet:
        path: /healthz
        port: liveness-port
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      failureThreshold: 1
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /healthz
        port: liveness-port
      failureThreshold: 30
      periodSeconds: 10
```