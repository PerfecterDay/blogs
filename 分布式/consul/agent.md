## Agent API




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