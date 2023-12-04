## maven 常用命令参数和配置
{docsify-updated}

- [maven 常用命令参数和配置](#maven-常用命令参数和配置)
    - [maven 常用命令](#maven-常用命令)
    - [maven 常用配置](#maven-常用配置)
    - [maven 问题](#maven-问题)


#### maven 常用命令
0. `mvn -B archetype:generate -DgroupId=com.my -DartifactId=simple-weather -DpackageName=com.baicy -Dversion=1.0 `: 以batch模式创建新项目，会选择 Archetype ，可能需要删除 settings配置文件中 archetypeRepository 和 archetypeCatalog 配置。
1. `mvn archetype:generate -DgroupId=com.coder.wang -DartifactId=maven-count-plugin -DarchetypeArtifactId=maven-archetype-plugin -DarchetypeVersion=3.0 `:使用指定的 Archetype 生成项目。
2. ` mvn dependency:list` ：输出依赖所有依赖的包
3. ` mvn dependency:tree -Dverbose` ：输出依赖树，解决依赖冲突时有用
4. ` mvn dependency:analyze` ：分析依赖树
5. `-U` :强制更新本地库
6. `mvn clean cobertura:cobertura -Dcobertura.report.format=html`:生成html格式的测试覆盖率报告
7. `mvn help:effective-pom`：生成完整的 POM 文件，包含 super pom
8. `mvn help:effective-settings`: 查看当前 maven 生效的设置
9. 安装本地JAR包到本地仓库：`mvn install:install-file -Dfile="D:\workspace\trade\trade-center\trade-center-service\lib\ZHConverter.jar" -DgroupId="com.spreada.utils.chinese" -DartifactId="ZHConverter" -Dversion="1.0" -Dpackaging="jar"`
10. `mvn package -DskipTests`: 跳过测试运行
11. `mvn package -Dmvn.test.skip=true`：跳过测试代码编译同时跳过测试运行
12. `mvn test -Dtest=testClass1,testClass2|Random*Test`：运行指定的测试类（|代表或）
13. `mvn -pl '!submodule1,!submodule2' install 或者 mvn -pl -submodule1,-submodule2 install` ：打包时跳过某些模块

#### maven 常用配置
最佳实践是将 maven 安装目录下的 `conf/settings.xml` 文件复制到用户目录下的 `.m2` 文件夹中，并修改其中的配置。
1. 修改本地仓库路径: `<localRepository>D:/repository</localRepository>`
2. 修改远程仓库镜像：
  ```
    <mirror>
    <id>aliyun</id>
    <name>aliyun Maven</name>
    <mirrorOf>*</mirrorOf>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    <!-- <url>http://maven.oschina.net/content/groups/public</url> -->
    </mirror>
  ```
3. 配置Java的版本
```
<properties>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.source>1.8</maven.compiler.source>
</properties>
或者
<plugins>
    <plugin>    
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

#### maven 问题
有时候使用 https 时会遇到如下错误：`sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target`

解决方法：加上 `-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true`