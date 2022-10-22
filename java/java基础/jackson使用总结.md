## jackson 使用总结
{docsify-updated}

https://www.baeldung.com/jackson-serialize-dates
https://www.baeldung.com/jackson

### ObjectMapper
Jackson 最常用的 API 就是基于"对象绑定" 的 ObjectMapper：

ObjectMapper可以从字符串，流或文件中解析JSON，并创建表示已解析的JSON的Java对象。 将JSON解析为Java对象也称为从JSON反序列化Java对象。
ObjectMapper也可以从Java对象创建JSON。 从Java对象生成JSON也称为将Java对象序列化为JSON。
Object映射器可以将JSON解析为自定义的类的对象，也可以解析置JSON树模型的对象。

之所以称为ObjectMapper是因为它将JSON映射到Java对象（反序列化），或者将Java对象映射到JSON（序列化）。

### ObjectMapper如何匹配JSON对象的字段和Java对象的属性
默认情况下，Jackson通过将JSON字段的名称与Java对象中的getter和setter方法进行匹配，将JSON对象的字段映射到Java对象中的属性。 Jackson删除了getter和setter方法名称的“ get”和“ set”部分，并将其余名称的第一个字符转换为小写。



### 序列化相关的注解
1. @JsonAnyGetter  
    如果 POJO 中有个 Map 类型的字段，这个注解可以使 Map 中的 key-value 直接在序列化后的 JSON 字段中，假如 POJO 如下：
    ```
    public class ExtendableBean {
        public String name;
        private Map<String, String> properties;
    
        @JsonAnyGetter
        public Map<String, String> getProperties() {
            return properties;
        }
    }
    ```
    则序列化后的 JOSN ：
    ```
    {
        "name":"test",
        "attr2":"val2",
        "attr1":"val1"
    }
    ```
    如果不使用该注解，则
    ```
    {
        "name": "My bean",
        "properties": {
            "attr2": "val2",
            "attr1": "val1"
        }
    }
    ```
2. @JsonGetter  
    默认情况下，序列化时， jackson 使用 getter/setter 方法获取字段值，该注解用来标注一个方法，当序列化时，标注了该注解的方法会被 jackson 调用来获取 POJO 对象某个字段的值。假设上述改一下上边的 ExtendableBean:
    ```
    public class ExtendableBean {
        public String name;
        private Map<String, String> properties;
    
        @JsonAnyGetter
        public Map<String, String> getProperties() {
            return properties;
        }

        @JsonGetter("name")
        public String getMyName(){
            return name + "my";
        }
    }
    ```
    则序列化的结果为：
    ```
    {
        "name":"testmy",
        "attr2":"val2",
        "attr1":"val1"
    }
    ```
3. @JsonPropertyOrder  
    该注解用来指定序列化时字段的顺序。如：
    ```
    @JsonPropertyOrder({"id","name"})
    public class ExtendableBean {
        public String name;
        private String id;
    }
    ```
    则序列化后的 JSON 就会是这样：
    ```
    {
        "id":"123",
        "name":"test"
    }
    ```
4. @JsonSerialize  
    当序列化时（POJO->JSON）自定义一个序列化器来序列化指定的字段。
    ```
    public class EventWithSerializer {
        public String name;
    
        @JsonSerialize(using = CustomDateSerializer.class)
        public Date eventDate;
    }
    ```
    ```
    public class CustomDateSerializer extends StdSerializer<Date> {
    
        private static SimpleDateFormat formatter 
        = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    
        public CustomDateSerializer() { 
            this(null); 
        } 
    
        public CustomDateSerializer(Class<Date> t) {
            super(t); 
        }
    
        @Override
        public void serialize(
        Date value, JsonGenerator gen, SerializerProvider arg2) 
        throws IOException, JsonProcessingException {
            gen.writeString(formatter.format(value));
        }
    }
    ```
5. @JsonDeserialize  
    当反序列化时（JSON->POJO），自定义一个反序列化器来反序列化指定的字段值。
    ```
    public class EventWithSerializer {
        public String name;
    
        @JsonDeserialize(using = CustomDateDeserializer.class)
        public Date eventDate;
    }
    ```
    ```
    public class CustomDateDeserializer
    extends StdDeserializer<Date> {
    
        private static SimpleDateFormat formatter
        = new SimpleDateFormat("dd-MM-yyyy hh:mm:ss");
    
        public CustomDateDeserializer() { 
            this(null); 
        } 
    
        public CustomDateDeserializer(Class<?> vc) { 
            super(vc); 
        }
    
        @Override
        public Date deserialize(
        JsonParser jsonparser, DeserializationContext context) 
        throws IOException {
            
            String date = jsonparser.getText();
            try {
                return formatter.parse(date);
            } catch (ParseException e) {
                throw new RuntimeException(e);
            }
        }
    }
    ```
6. @JsonIgnoreProperties  
    该注解是一个类注解。当我们在序列化或反序列化时想要忽略某些字段时，可以使用该注解。与此类似的有一个 `@JsonIgnore` 注解，该注解用在字段上，与 `JsonIgnoreProperties` 作用相同。
    ```
    @JsonIgnoreProperties({ "id" })
    public class BeanWithIgnore {
        public int id;
        public String name;
    }
    ```
    **当我们反序列化时，只想映射 JSON 中的部分字段而忽略掉 POJO 中没有定义的字段时，可以使用 @JsonIgnoreProperties(ignoreUnknown = true)**

7. @JsonProperty  
    该注解用来申明字段在 JSON 中对应的名字。
8. @JsonFormat  
    当字段的类型是 Date/Time 类型时，用来指定字符串的格式。
    ```
    public class EventWithFormat {
    public String name;
 
    @JsonFormat(
      shape = JsonFormat.Shape.STRING,
      pattern = "dd-MM-yyyy hh:mm:ss")
    public Date eventDate;
    }
    ```

### 常见设置
1. 序列化的时候忽略 NULL ：
   1. 在类上标注 @JsonInclude(Include.NON_NULL)，忽略类中所有 NULL 字段：
        ```
        @JsonInclude(Include.NON_NULL)
        public class MyDto { ... }
        ```
    2. 在要忽略的某个字段上标注  @JsonInclude(Include.NON_NULL)
    3. 设置 ObjectMapper：
        mapper.setSerializationInclusion(Include.NON_NULL）

