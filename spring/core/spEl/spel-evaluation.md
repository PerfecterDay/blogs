# Evaluation
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/expressions/evaluation.html

`Hello World` 示例:
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'");
String message = (String) exp.getValue();
```

最常使用的SpEL类和接口位于 `org.springframework.expression` 包及其子包中，例如 `org.springframework.expression.spel.support` 。

`ExpressionParser` 接口负责解析表达式字符串。在上例中，表达式字符串是由单引号包围的字符串字面量。 `Expression` 接口负责评估计算定义的表达式字符串。调用  `parser.parseExpression(…​)` 和 `exp.getValue(…​)` 时可能分别抛出的两种异常: `ParseException` 和 `EvaluationException` 。

SpEL支持多种功能，包括调用方法、访问属性以及调用构造函数。
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello World'.concat('!')");
String message = (String) exp.getValue();

// invokes 'getBytes()'
Expression exp = parser.parseExpression("'Hello World'.bytes");
byte[] bytes = (byte[]) exp.getValue();
```


SpEL还支持使用标准点表示法（如 `prop1.prop2.prop3` ）访问嵌套属性，也可以设置嵌套属性值。公共字段同样可被访问。
```
ExpressionParser parser = new SpelExpressionParser();

// invokes 'getBytes().length'
Expression exp = parser.parseExpression("'Hello World'.bytes.length");
int length = (Integer) exp.getValue();
```

字符串的构造函数可替代字符串字面量使用，如下例所示：
```
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("new String('hello world').toUpperCase()");
String message = exp.getValue(String.class);
```
注意，如果想构造一个自定义的类型，必须将类型定义为 `public class`，并且指定包含包路径的全限定名：
```
@Data
public class Person{
    int age;
    String name;
}

Expression expression = parser.parseExpression("new spring.spel.Person()");
```

请注意泛型方法的使用： `public <T> T getValue(Class<T> desiredResultType)` 。使用此方法可免去将表达式值强制转换为预期结果类型的步骤。若值无法转换为类型 `T` 或通过注册的类型转换器进行转换，则会抛出 `EvaluationException` 。

SpEL更常见的用法是提供一个表达式字符串，该字符串将针对特定对象实例（称为根对象）进行评估。以下示例演示了如何从 `Inventor` 类的实例中获取 `name` 属性，以及如何在布尔表达式中引用该属性。
```
// Create and set a calendar
GregorianCalendar c = new GregorianCalendar();
c.set(1856, 7, 9);

// The constructor arguments are name, birthday, and nationality.
Inventor tesla = new Inventor("Nikola Tesla", c.getTime(), "Serbian");

ExpressionParser parser = new SpelExpressionParser();

Expression exp = parser.parseExpression("name"); // Parse name as an expression
String name = (String) exp.getValue(tesla);
// name == "Nikola Tesla"

exp = parser.parseExpression("name == 'Nikola Tesla'");
boolean result = exp.getValue(tesla, Boolean.class);
// result == true
```


## EvaluationContext
在评估表达式时， `EvaluationContext` API 用于解析属性、方法或字段，并协助执行类型转换。Spring 提供了两种实现方案。

+ `SimpleEvaluationContext`: 暴露了SpEL语言中一组核心功能和配置选项，适用于无需完整语法支持且应合理限制的表达式类别。例如数据绑定表达式和基于属性的过滤器等（但不限于此）。
+ `StandardEvaluationContext`: 全面包含 SpEL 的语言特性和配置选项。可以使用它来指定默认根对象，并配置所有可用的评估相关策略。

值得一提的是， `SimpleEvaluationContext.forReadOnlyDataBinding()` 可通过 `DataBindingPropertyAccessor` 实现对属性的只读访问。同样地，`SimpleEvaluationContext.forReadWriteDataBinding()` 则支持对属性的读写操作。此外，还可通过 `SimpleEvaluationContext.forPropertyAccessors(…)` 配置自定义访问器，可选择禁用赋值操作，并可通过构建器选项激活方法解析和/或类型转换器。

### 类型转换


## Parser 配置

## 编译 SpEL


