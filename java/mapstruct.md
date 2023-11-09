## Mapstruct
{docsify-updated}

- [Mapstruct](#mapstruct)
  - [依赖与插件](#依赖与插件)
  - [不同名字映射](#不同名字映射)
  - [类型转换](#类型转换)
  - [执行java语句](#执行java语句)
  - [增加自定义转换逻辑](#增加自定义转换逻辑)
  - [集合转换](#集合转换)
  - [命名方法](#命名方法)
  - [插件冲突](#插件冲突)
  - [](#)

https://www.baeldung.com/mapstruct  
https://www.baeldung.com/java-mapstruct-mapping-collections


### 依赖与插件
```
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version> 
</dependency>

<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.11.0</version>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.5.5.Final</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

### 不同名字映射
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

### 类型转换
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

### 执行java语句
当某个字段需要特殊处理时：
```
// List<String> 转 String
@Mapper
public interface PublishVersionRequst2AppVersionEntityMapper {

    @Mapping(target = "memo",expression = "java(cn.hutool.core.util.StrUtil.toString(request.getMemo()))")
    AppVersionEntity map(PublishVersionRequest request);
}
```

### 增加自定义转换逻辑
```
@Mapper
public abstract class OrderSyncRequestMapper {
    @BeforeMapping
    protected void getVouncherNo(FundOrder fundOrder, @MappingTarget OrderSyncRequest orderSyncRequest) {
        if (StringUtils.equalsIgnoreCase("s", fundOrder.getOrderType())) {
            orderSyncRequest.setVoucherNo(fundOrder.getHoldFundVoucherNo());
        } else {
            orderSyncRequest.setVoucherNo(fundOrder.getInstrumentVoucherNo());
        }
    }
    @Mapping(target = "instrument", source = "fund")
    public abstract OrderSyncRequest fromFundOrder(FundOrder fundOrder);
}
```

### 集合转换
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

### 命名方法
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


### 插件冲突
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


### 
I have Car:
+ id
+ brand
+ model
+ owner
And CarDTO:
+ id
+ brand
+ model
In my service class I'm passing additional parameter "owner" and I need to convert the list.

Is it possible to add "owner" to Mapper?

If yes then I suppose it should be something similar to this (not working).
```
@Mapper
public interface CarMapper {

@Mapping(target = "owner", source = "owner")
List<Car> mapCars(List<CarDTO> cars, String owner);
}
```

Firstly, add a single object mapping method:

@Maping(target = "owner", source = "owner")
Car mapCar(CarDTO car, String owner);
Then define a method for mapping a list of objects with @Context:

List<Car> mapCars(List<CarDTO> cars, @Context String owner);
Since @Context parameters are not meant to be used as source parameters, a proxy method should be added to point MapStruct to the right single object mapping method to make it work.

In the end, add the proxy method:

default Car mapContext(CarDTO car, @Context String owner) {
    return mapCar(car, owner);
}


```
@Mapper
public interface CustomerScrResultMapper {

    @Mapping(target = "tradeAccount", source = "tradeAccount")
    @Mapping(target = "scrCheckSeq", source = "scrSeq")
    @Mapping(target = "targetItemId", source = "result.targetItemName")
    CustomerScrResult transFromMatchResult(MatchItemResult result, String tradeAccount, String scrSeq);

    //    @Mapping(source = "targetItemName",target = "target_item_name")
//    @Mapping(source = "fundValue",target = "fund_value")
//    @Mapping(source = "customerValue",target = "customer_value")
//    @Mapping(source = "matchResult", target = "target_item_name")
    default CustomerScrResult mapContext(MatchItemResult matchItemResult, @Context ScrContext context) {
        return transFromMatchResult(matchItemResult, context.getCustomer().getTradeAccount(), context.getScrSeq());
    }

    List<CustomerScrResult> transFromMatchResults(List<MatchItemResult> resultList,
                                                  @Context ScrContext context);

}
```