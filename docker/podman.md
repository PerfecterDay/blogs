## Podman
{docsify-updated}

podman 安装：
```
	brew install podman
	podman machine init /  podman machine init --memory=8192 --cpus=2
	podman machine start
	podman info
```

`podman machine set --rootful` 设置 root 权限

`podman machine ssh`;
`sudo sysctl -w vm.max_map_count=262144`