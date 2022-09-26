## protobuf


Maven 插件配置：
```
<build>
	<extensions>
		<extension>
			<groupId>kr.motd.maven</groupId>
			<artifactId>os-maven-plugin</artifactId>
		</extension>
	</extensions>
	<plugins>
		<plugin>
			<groupId>org.xolstice.maven.plugins</groupId>
			<artifactId>protobuf-maven-plugin</artifactId>
			<configuration>
				<includes>
					<include>**/*.proto</include>
				</includes>
<!--                    <protoSourceRoot>${basedir}/src/main/proto</protoSourceRoot>-->
<!--                    <outputDirectory>${basedir}/src/main/proto/gen-java</outputDirectory>-->
				<protocArtifact>com.google.protobuf:protoc:${protobuf.version}:exe:${os.detected.classifier}</protocArtifact>
				<pluginId>grpc-java</pluginId>
				<pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}</pluginArtifact>
			</configuration>
			<executions>
				<execution>
					<id>compile</id>
					<goals>
						<goal>compile</goal>
						<goal>compile-custom</goal>
					</goals>
				</execution>
			</executions>
		</plugin>

		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-clean-plugin</artifactId>
			<configuration>
				<verbose>true</verbose>
				<filesets>
					<fileset>
						<directory>${basedir}/src/main/proto/gen-java</directory>-->
					</fileset>
				</filesets>
			</configuration>
		</plugin>
	</plugins>
</build>
```