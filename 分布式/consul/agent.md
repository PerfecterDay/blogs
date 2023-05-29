## Agent API
{docsify-updated}

`/agent`端点用于与本地Consul代理进行交互。通常，服务和检查是在代理处注册的，然后由代理承担起保持该数据与集群同步的责任。例如，代理向目录注册服务和检查，并执行反熵以从中断中恢复。

除了这些端点之外，其他的端点也被分组在检查和服务的导航中


ConsulServiceRegistry 注册服务时使用的 endpoint：
http://consul:8500/v1/agent/service/register?token=
body:
```
{
	"ID": "trade-center-service-8100",
	"Name": "trade-center-service",
	"Tags": [],
	"Address": "10.176.81.23",
	"Meta": {
		"secure": "false",
		"gRPC_port": "8101"
	},
	"Port": 8101,
	"Check": {
		"Interval": "10s",
		"HTTP": "http://10.176.81.23:8100/actuator/health",
		"Header": {},
		"DeregisterCriticalServiceAfter": "3m"
	}
}
```