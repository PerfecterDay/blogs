## Skywalking
{docsify-updated}


<center><img src="pics/skywalking-framework.jpg" width="70%"></center>

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
java  -javaagent:/Users/coder_wang/Workspace/skywalking-agent/skywalking-agent.jar
-jar demo.jar

日志集成：

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
