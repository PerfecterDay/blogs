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