
## Jackson注解

> https://www.baeldung.com/jackson-annotations

- [Jackson注解](#jackson注解)
	- [序列化注解](#序列化注解)
		- [序列化时忽略某些属性](#序列化时忽略某些属性)
		- [@JsonAnyGetter](#jsonanygetter)
		- [`@JsonGetter`](#jsongetter)
		- [`@JsonPropertyOrder({ "name", "id" })`:指定序列化的字段顺序](#jsonpropertyorder-name-id-指定序列化的字段顺序)
		- [`@JsonInclude(Include.NON_NULL)`: 去除掉null 的属性](#jsonincludeincludenon_null-去除掉null-的属性)


### 序列化注解

#### 序列化时忽略某些属性
如果我们在序列化时需要过滤掉某些属性，可以使用下面注解：
1. 忽略某个属性，类级别注解：`@JsonIgnoreProperties`
	```
	@JsonIgnoreProperties(value = { "intValue" })
	public class MyDto {

		private String stringValue;
		private int intValue;
		private boolean booleanValue;

		public MyDto() {
			super();
		}

		// standard setters and getters are not shown
	}
	```
	这样不管是序列化还是反序列化，intValue 的值都是不会额外设置的。由构造器设置值。
2. **忽略所有未知属性，类级别注解**：`@JsonIgnoreProperties(ignoreUnknown = true)`
3. 属性级别的注解：`JsonIgnore`
	```
	public class MyDto {

		private String stringValue;
		@JsonIgnore
		private int intValue;
		private boolean booleanValue;

		public MyDto() {
			super();
		}

		// standard setters and getters are not shown
	}
	```
3. 忽略特定类型的所有属性，类级别注解：`@JsonIgnoreType`。当一个被 @JsonIgnoreType 注解的类作为其他类的属性时，在序列化和反序列化期间将忽略该属性。**注意只有在该类作为其他类的属性时才会被忽略，直接序列化、反序列化该类本身是正常的。**
4. 使用过滤器过滤属性
	```
	@JsonFilter("myFilter")
	public class MyDtoWithFilter { ... }

    ObjectMapper mapper = new ObjectMapper();
    SimpleBeanPropertyFilter theFilter = SimpleBeanPropertyFilter
      .serializeAllExcept("intValue");
    FilterProvider filters = new SimpleFilterProvider()
      .addFilter("myFilter", theFilter);

    MyDtoWithFilter dtoObject = new MyDtoWithFilter();
    String dtoAsString = mapper.writer(filters).writeValueAsString(dtoObject);
	```

#### @JsonAnyGetter
`@JsonAnyGetter` 可以允许将Map类型的属性中的key-value作为类型本身的属性输出。比如：
```
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnyGetter
    public Map<String, String> getProperties() {
        return properties;
    }
}

序列化输出时将会得到：
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```

#### `@JsonGetter`
Jackson序列化时默认调用标准的 getter 方法获取对象值，如果我们想要使用特定方法作为Jackson获取对象值的方法，就可以用 `@JsonGetter` 标注该方法：
```
public class MyBean {
    public int id;
    private String name;

    @JsonGetter("name")
    public String getTheName() {
        return name;
    }
}
```

#### `@JsonPropertyOrder({ "name", "id" })`:指定序列化的字段顺序
#### `@JsonInclude(Include.NON_NULL)`: 去除掉null 的属性