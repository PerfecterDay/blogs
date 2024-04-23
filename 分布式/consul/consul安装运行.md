# consul安装与运行
{docsify-updated}

> https://developer.hashicorp.com/consul/docs/agent  
> https://developer.hashicorp.com/consul/docs/agent/config


## homebrew 安装运行
`consul agent -data-dir=tmp/consul -dev`

`/opt/homebrew/Cellar/consul/1.16.2/homebrew.mxcl.consul.plist`
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>KeepAlive</key>
	<true/>
	<key>Label</key>
	<string>homebrew.mxcl.consul</string>
	<key>LimitLoadToSessionType</key>
	<array>
		<string>Aqua</string>
		<string>Background</string>
		<string>LoginWindow</string>
		<string>StandardIO</string>
		<string>System</string>
	</array>
	<key>ProgramArguments</key>
	<array>
		<string>/opt/homebrew/opt/consul/bin/consul</string>
		<string>agent</string>
		<!-- <string>-server</string> -->
		<string>-dev</string>
		<!-- <string>-bootstrap</string> -->
		<!-- <string>-ui</string> -->
		<string>-data-dir=~/consul</string>
		<string>-bind</string>
		<string>127.0.0.1</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
	<key>StandardErrorPath</key>
	<string>/opt/homebrew/var/log/consul.log</string>
	<key>StandardOutPath</key>
	<string>/opt/homebrew/var/log/consul.log</string>
	<key>WorkingDirectory</key>
	<string>/opt/homebrew/var</string>
</dict>
</plist>
```
默认以dev模式运行，不支持任何数据的持久化，除非加上  -data-dir=~/consul 启动项，如果是个相对地址，则是相对 WorkingDirectory 的路径

## Helm安装 consul 集群
1. 前提（安装Helm）:
    1. curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    2. chmod 700 get_helm.sh
    3. ./get_helm.sh

2. 添加 consul repo:
    1. `helm repo add hashicorp https://helm.releases.hashicorp.com`
    2. `helm install consul hashicorp/consul --set global.name=consul-cluster --set server.storage=2Gi --create-namespace --namespace consul`

https://developer.hashicorp.com/consul/docs/k8s/installation/install

+ `helm install consul hashicorp/consul --set global.name=consul --set server.storage=2Gi --create-namespace --namespace consul`  
+ `helm install consul hashicorp/consul --set global.name=consul-cluster --set server.storage=500M --set server.storageClass=alicloud-disk-available --namespace consul`
+ `helm install consul hashicorp/consul --set global.name=consul-cluster --set server.storage=500M --namespace consul`

存储声明 PVC ：
```
name: data-consul-consul-cluster-server-0
  namespace: consul
  resourceVersion: '34177039'
  uid: 91fe6fc4-85c7-48b5-b3b1-8b1085294d43
```
要与存储卷 PV ：
```
claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: data-consul-consul-cluster-server-0
    namespace: consul
    resourceVersion: '6283149'
    uid: e64352c1-47e3-462e-a5b7-724ac08d7862
```
uid保持一致