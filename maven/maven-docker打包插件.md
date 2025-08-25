# maven docker 打包插件
{docsify-updated}


使用 `io.fabric8:docker-maven-plugin` 插件
```
<plugin>
    <groupId>io.fabric8</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>0.46.0</version>
    <!--全局配置-->
    <configuration>
        <images>
            <!-- 单个镜像配置 -->
            <image>
                <!--镜像名(含版本号)-->
                <name>gtja-registry-registry.cn-hongkong.cr.aliyuncs.com/gtja/channel-gmt:${project.version}</name>
                <!--镜像build相关配置-->
                <build>
                    <!--使用dockerFile文件-->
                    <dockerFile>${project.basedir}/Dockerfile</dockerFile>
                    <contextDir>${project.basedir}</contextDir>
                </build>
            </image>
        </images>
    </configuration>
</plugin>
```