# Spring 数据绑定
{docsify-updated}

> https://docs.spring.io/spring-framework/reference/core/validation/data-binding.html

数据绑定可将用户输入与目标对象关联，其中用户输入遵循JavaBeans规范，以属性路径为键的映射形式呈现。DataBinder作为核心支持类，提供两种绑定方式：

构造函数绑定——将用户输入绑定至公共数据构造函数，从用户输入中查找构造函数参数值。

属性绑定 - 将用户输入绑定至设置器，将用户输入的键与目标对象结构的属性进行匹配。

可同时应用构造函数绑定与属性绑定，也可仅采用其中一种方式。