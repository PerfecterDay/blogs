## REDIS
内：r-3nsu3uebubzx2j48lxpd.redis.rds.aliyuncs.com:6379
r-3ns5xcjxon2hpod3zp
iVWzhb80vUFyugxA

redis-cli -h r-3nsu3uebubzx2j48lxpd.redis.rds.aliyuncs.com
auth r-3ns5xcjxon2hpod3zp iVWzhb80vUFyugxA

redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com -a r-3ns5xcjxon2hpod3zp iVWzhb80vUFyugxA

redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com -a r-3ns5xcjxon2hpod3zp iVWzhb80vUFyugxA --ldb --eval "if (redis.call('hexists', KEYS[1], ARGV[3]) == 0) then return nil;end; local counter = redis.call('hincrby', KEYS[1], ARGV[3], -1); if (counter > 0) then redis.call('pexpire', KEYS[1], ARGV[2]); return 0; else redis.call('del', KEYS[1]); redis.call(ARGV[4], KEYS[2], ARGV[1]); return 1; end; return nil;" 2 30310700409S-8101ca80-9ecf-481c-8244-6217171d913d redisson_lock__channel:{30310700409S-8101ca80-9ecf-481c-8244-6217171d913d}  0 30000 f9548f24-fc64-4b28-901f-c20815e5fb42:276 PUBLISH



auth r-3ns5xcjxon2hpod3zp iVWzhb80vUFyugxA

redis-cli -h r-3nsu3uebubzx2j48lxpd.redis.rds.aliyuncs.com -p 6379 -a qAHFtvXgb4176hQa --raw keys 'info:*' | cat
redis-cli -h r-3nsu3uebubzx2j48lxpd.redis.rds.aliyuncs.com -p 6379 -a qAHFtvXgb4176hQa keys 'info:*' | cat

redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com -p 6379 -a iVWzhb80vUFyugxA keys 'info:*' | xargs redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com -p 6379 -a iVWzhb80vUFyugxA del

redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com -p 6379 -a iVWzhb80vUFyugxA del `redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com -p 6379 -a iVWzhb80vUFyugxA --raw keys 'info:*'`


redis-cli -h r-3nsu3uebubzx2j48lxpd.redis.rds.aliyuncs.com -p 6379 -a qAHFtvXgb4176hQa keys 'info:*'


session中获取长 token:
get gtja-international-app-token:login:session:11804
de4b240d-eac3-4063-b42d-f2bd82559a1b

token-session:
get gtja-international-app-token:login:token-session:1bfeba8f-9a7a-4e74-b27f-3a3986e0e8f5
gtja-international-app-token:login:token-session:55ed3436-5c03-40f9-b8f1-d45bdd04043e

长token key：
gtja-international-app-token:login:token:86dadc83-260d-420e-8ccd-398b7e28048b
gtja-international-app-token:login:token:55ed3436-5c03-40f9-b8f1-d45bdd04043e

短token key：
gtja-international-app-token:short-token:S-08ab2d20-7777-4015-b0e3-06175297dcbc
gtja-international-app-token:short-token:S-0d464f90-8be0-4d5c-9794-fc865f4806ec

JYB token:
get jybToken::token

---------

## Mysql


rm-3nszwm39bc4foon2blo.mysql.rds.aliyuncs.com
高权限账号：
gtja_uatadmin
p0BWm71pakWIZ3pH

mysql -hrm-3nszwm39bc4foon2blo.mysql.rds.aliyuncs.com -ugtja_uatadmin -pp0BWm71pakWIZ3pH

----------

## Kafka
alikafka-post-public-intl-sg-uq832vxmo01-1-vpc.alikafka.aliyuncs.com:9092,
alikafka-post-public-intl-sg-uq832vxmo01-2-vpc.alikafka.aliyuncs.com:9092,
alikafka-post-public-intl-sg-uq832vxmo01-3-vpc.alikafka.aliyuncs.com:9092



curl -X POST localhost:10000/user/app/configuration
curl -X POST localhost:8888/user/app/configuration

## 远程登录
1. VNC登录：重置密码
2. Workbench登录：Sunthy80@


## 日志收集配置
1. [通过日志服务采集Kubernetes容器日志](https://www.alibabacloud.com/help/zh/container-service-for-kubernetes/latest/collect-log-data-from-containers-by-using-log-service)
	基本上就是部署应用的时候，添加日志配置即可
2. [ARMS应用性能监控](https://www.alibabacloud.com/help/zh/container-service-for-kubernetes/latest/monitor-application-performance)
	```
		armsPilotAutoEnable: "on"
		armsPilotCreateAppName: "user-center-service"
		one-agent.jdk.version: "OpenJDK11"
		armsSecAutoEnable: "on" 
	```

应用设置->应用日志关联配置:  
开启后业务日志中会自动生成调用链的TraceId。此设置在重启应用后生效。支持Log4j/Log4j2/Logback日志组件。业务应用需要在日志的Layout中通过声明%X{EagleEye-TraceID}来输出TraceId信息。参考文档注意：2.7.3.5及以后版本探针中默认打开。

%X{EagleEye-TraceID}



## 修改密码
R123456(7)

1973-06-10

86-15100700219

1000862Email@test.com