## Mapstruct
{docsify-updated}

https://www.baeldung.com/mapstruct  
https://www.baeldung.com/java-mapstruct-mapping-collections


与 Lombok 配合出现：
No property named "sms" exists in source parameter/No property named "sms" exists in result parameter
需要先用Lombok处理后才能用mapstruct，加入下列配置：
```
<build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <source>${java.version}</source>
                        <target>${java.version}</target>
                        <encoding>${project.build.sourceEncoding}</encoding>
                        <annotationProcessorPaths>
                            <path>
                                <groupId>org.projectlombok</groupId> <!-- IMPORTANT - LOMBOK BEFORE MAPSTRUCT -->
                                <artifactId>lombok</artifactId>
                                <version>${lombok.version}</version>
                            </path>
                            <path>
                                <groupId>org.mapstruct</groupId>
                                <artifactId>mapstruct-processor</artifactId>
                                <version>${org.mapstruct.version}</version>
                            </path>
                        </annotationProcessorPaths>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
```

不同名字之间的map：
```
@Mapper
public interface EmployeeMapper {
    
    @Mapping(target="employeeId", source="entity.id")
    @Mapping(target="employeeName", source="entity.name")
    EmployeeDTO employeeToEmployeeDTO(Employee entity);

    @Mapping(target="id", source="dto.employeeId")
    @Mapping(target="name", source="dto.employeeName")
    Employee employeeDTOtoEmployee(EmployeeDTO dto);
}
```

当类型需要转换时，比如时间转字符串：
```

  @Mapping(target="employeeId", source = "entity.id")
  @Mapping(target="employeeName", source = "entity.name")
  @Mapping(target="employeeStartDt", source = "entity.startDt",
           dateFormat = "dd-MM-yyyy HH:mm:ss")
EmployeeDTO employeeToEmployeeDTO(Employee entity);

  @Mapping(target="id", source="dto.employeeId")
  @Mapping(target="name", source="dto.employeeName")
  @Mapping(target="startDt", source="dto.employeeStartDt",
           dateFormat="dd-MM-yyyy HH:mm:ss")
Employee employeeDTOtoEmployee(EmployeeDTO dto);
```


当某个字段需要特殊处理时：
```
// List<String> 转 String
@Mapper
public interface PublishVersionRequst2AppVersionEntityMapper {

    @Mapping(target = "memo",expression = "java(cn.hutool.core.util.StrUtil.toString(request.getMemo()))")
    AppVersionEntity map(PublishVersionRequest request);
}
```


转换 List 集合类型时，必须声明元素类型的转换方法：
```
@Mapper
public abstract class TradeNews2MessageMapper {

    public abstract List<UserMessageVo> toUserMessageVoList(List<TradeNews> tradeNews);

    @BeforeMapping
    protected void beforeMapping(TradeNews tradeNews, @MappingTarget UserMessageVo messageVo) {
        messageVo.setReadState(StrUtil.equals(tradeNews.getFstate(), "New") ? 0 : 1);
    }

    @Mapping(source = "ftitle", target = "title")
    @Mapping(source = "fdate", target = "time")
    @Mapping(source = "fstate", target = "readState")
    @Mapping(source = "fno", target = "msgId")
    protected abstract UserMessageVo toUserMessageVo(TradeNews tradeNews);

}
```

当有多个转换方法可调用时，需要使用 `@Named` 区别各个方法，并在转 List 的方法上使用 `@IterableMapping` 注解指明要使用的方法：
```
    @Mapping(target = "memo", expression = "java(cn.hutool.json.JSONUtil.toList(version.getMemo(),String.class))")
    @Named("toHistoryVersionResponse")
    HistoryVersionResponse toHistoryVersionResponse(AppVersionEntity version);

    @Mapping(target = "memo", expression = "java(cn.hutool.json.JSONUtil.toList(version.getMemoHk(),String.class))")
    @Named("toHistoryVersionResponseHk")
    HistoryVersionResponse toHistoryVersionResponseHk(AppVersionEntity version);

    @IterableMapping(qualifiedByName = "toHistoryVersionResponse")
    List<HistoryVersionResponse> toHistoryVersionResponseList(List<AppVersionEntity> versions);

    @IterableMapping(qualifiedByName = "toHistoryVersionResponseHk")
    List<HistoryVersionResponse> toHistoryVersionResponseListHk(List<AppVersionEntity> versions);
```