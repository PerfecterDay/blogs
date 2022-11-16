## Skywalking
{docsify-updated}

- [Skywalking](#skywalking)
	- [基础概念](#基础概念)
	- [基础框架](#基础框架)
	- [skywalking-oap](#skywalking-oap)
	- [skywalking-ui](#skywalking-ui)
	- [Skywalking agent](#skywalking-agent)

### 基础概念
<center><img src="pics/base-concept.png" width="70%"></center>

日志示例：
2022-10-10 13:01:29.052 [TID:bb95da5ea5fa49acb6dcd9e1ec045e8e.50.16653780884380001] [grpc-default-executor-1] INFO  com.gtja.gjyw.HelloService -Received request name:xx



### 基础框架
SkyWalking 逻辑上分为四部分: 探针, 平台后端, 存储和用户界面.
<center><img src="pics/skywalking-framework.png" width="70%"></center>

+ 探针 基于不同的来源可能是不一样的, 但作用都是收集数据, 将数据格式化为 SkyWalking 适用的格式.
+ 平台后端, 支持数据聚合, 数据分析以及驱动数据流从探针到用户界面的流程。分析包括 Skywalking 原生追踪和性能指标以及第三方来源，包括 Istio 及 Envoy telemetry , Zipkin 追踪格式化等。 你甚至可以使用 Observability Analysis Language 对原生度量指标 和 用于扩展度量的计量系统 自定义聚合分析。
+ 存储 通过开放的插件化的接口存放 SkyWalking 数据. 你可以选择一个既有的存储系统, 如 ElasticSearch, H2 或 MySQL 集群(Sharding-Sphere 管理),也可以选择自己实现一个存储系统. 当然, 我们非常欢迎你贡献新的存储系统实现。
+ UI 一个基于接口高度定制化的Web系统，用户可以可视化查看和管理 SkyWalking 数据

### skywalking-oap
失败了：
docker run --net elastic --name oap --restart always -d -e SW_STORAGE=elasticsearch7 -e SW_STORAGE_ES_CLUSTER_NODES=es01:9200 apache/skywalking-oap-server:latest

成功：
docker run --name oap --restart always -d apache/skywalking-oap-server:latest  
docker run --net elastic -p 11800:11800 --name oap -d apache/skywalking-oap-server:latest


### skywalking-ui

docker run --name oap-ui --restart always -d -e SW_OAP_ADDRESS=http://oap:12800 apache/skywalking-ui  
docker run --net elastic -p 8080:8080 --name oap-ui -d -e SW_OAP_ADDRESS=http://oap:12800 apache/skywalking-ui


### Skywalking agent

1. 7.x版本中代理支持 JDK 8-14， 6.x版本支持JDK 1.6- JDK12 
2. 在SkyWalking发行包中找到agent文件夹
3. 配置config/agent.config中的agent.service_name。可以是任意的英文字符串。
4. 配置config/agent.config中的collector.backend_service。默认指向127.0.0.1:11800，表示仅作用于本地后端。可以修改配置为部署的skywalking OAP 集群地址
5. JVM参数中添加-javaagent:/path/to/skywalking-package/agent/skywalking-agent.jar，并且确保这个参数在-jar参数之前。
	`java  -javaagent:/Users/coder_wang/Workspace/skywalking-agent/skywalking-agent.jar -jar demo.jar`
6. 启动应用时设置 `SW_AGENT_NAME=trade-biz` 环境变量，这样在 skywalking 中就能对应显示你设置的应用名字 

Java Agent 的默认配置在config/agent.config 中，常见的配置可参考[此处](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/8.0.0/setup/service-agent/java-agent/#agent%E7%9A%84%E5%8F%AF%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E5%88%97%E8%A1%A8)

[日志集成](https://skywalking.apache.org/docs/skywalking-java/v8.12.0/en/setup/service-agent/java-agent/application-toolkit-logback-1.x/)：
```
<dependency>
	<groupId>org.apache.skywalking</groupId>
	<artifactId>apm-toolkit-logback-1.x</artifactId>
	<version>8.12.0</version>
</dependency>

<appender name="grpc-log" class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.log.GRPCLogClientAppender">
	<encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
		<layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.mdc.TraceIdMDCPatternLogbackLayout">
			<Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{tid}] [%thread] %-5level %logger{36} -%msg%n</Pattern>
		</layout>
	</encoder>
</appender>

<logger name="com.gtja" level="INFO">
	<appender-ref ref="grpc-log" />
</logger>
```