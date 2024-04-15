#  SpEL
{docsify-updated}



### 文本字符解析
文本表达式支持字符串、日期、数字（正数、实数及八进制、十六进制等）、布尔类型、null。其中字符串必须使用单引号或者`\"`(反斜杠+双引号)括起来。
```
ExpressionParser expressionParser = new SpelExpressionParser();
String helloWorld = expressionParser.parseExpression("\"Hello World\"").getExpressionString();
String helloWorld2 = expressionParser.parseExpression("'Hello World'").getExpressionString();
Double doubleValue = expressionParser.parseExpression("6.098").getValue(Double.class);
Integer intVal = expressionParser.parseExpression("34").getValue(Integer.class);
Boolean boolVal = expressionParser.parseExpression("true").getValue(Boolean.class);
```

### 对象属性解析
可以使用 `xx.yy.zz` 的形式访问访问对象的属性值。