## SpringMvc 的处理方法绑定


Spring 会根据请求方法签名的不同，将请求消息中的信息 以一定的方式转换并绑定到请求方法的入参中。当请求消息到达真正需要调用的方法时(如指定的业务方法)，Spring MVC 还有很多工作要做，包括数据转换、数据格式化及数据校验等。


### 数据绑定流程剖析
Spring MvC通过反射机制对目标处理方法的签名进行分析，将请求消息绑定到处理方法的入参中。数据鄉定的核心部件是DataBinder，其运行机制描述如下所示：
<center><img src="pics/databind.jpg" alt=""></center>


https://www.baeldung.com/spring-data-redis-pub-sub

https://github.com/Homebrew/discussions/discussions/2530
https://www.baeldung.com/spring-retry