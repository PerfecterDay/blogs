## K8S 集群创建与交互
{docsify-updated}
- [K8S 集群创建与交互](#k8s-集群创建与交互)
  - [使用 podman-desktop 创建 k8s 集群](#使用-podman-desktop-创建-k8s-集群)
  - [使用 kubectl 与集群交互](#使用-kubectl-与集群交互)

### 使用 podman-desktop 创建 k8s 集群
1. 安装 podman
2. 安装 podman-desktop
3. 安装 kind
4. 在podman-desktop 安装 kind 扩展
5. Rootful模式重启 podman:
   ```
   podman machine stop
   podman machine set --rootful
   podman machine start
   ```
6. 打开 podman-desktop, settings->resources->kind -> create new

### 使用 kubectl 与集群交互
1. 查看资源: `kubectl get pod,deployment,service -n <your-namespace>`
2. 进入pod内部： `kubectl exec -ti <your-pod-name>  -n <your-namespace>  -- /bin/sh`
3. 导出资源到yml文件: `kubectl get deployment,service,pod <your-namespace> -o yaml --export`
4. 应用yml文件部署/更新资源: `kubectl apply -f nginx-deployment.yml -n apisix`