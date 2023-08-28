## kubectl 常用命令
{docsify-updated}

1. 查看资源: `kubectl get pod,deployment,service -n <your-namespace>`
2. 进入pod内部： `kubectl exec -ti <your-pod-name>  -n <your-namespace>  -- /bin/sh`
3. 导出资源到yml文件: `kubectl get deployment,service,pod <your-namespace> -o yaml --export`
4. 应用yml文件部署/更新资源: `kubectl apply -f nginx-deployment.yml -n apisix`
