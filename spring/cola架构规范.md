## COLA 架构规范
{docsify-updated}
> https://mp.weixin.qq.com/s/QRq_mnifw62LMoHey-tMiQ

- [COLA 架构规范](#cola-架构规范)
	- [分层架构](#分层架构)
	- [定义](#定义)


### 分层架构

<center><img src="pics/cola.png" width="60%"></center>

+ 用户接口层：调用应用层完成具体用户请求。包含：controller，远程调用服务等
+ 应用层App：尽量简单，不包含业务规则，而只为了下一层中的领域对象做协调任务，分配工作，重点对领域层做编排完成复杂业务场景。包含：AppService，消息处理等
+ 领域层Domain：负责表达业务概念和业务逻辑，领域层是系统的核心。包含：模型，值对象，域服务，事件
+ 基础层：对所有上层提供技术能力，包括：数据操作，发送消息，消费消息，缓存等

<center><img src="pics/cola-ac.png" width="60%"></center>

### 定义
1. 实体（ENTITY）

	实体有唯一的标识，有生命周期且具有延续性。例如一个交易订单，从创建订单我们会给他一个订单编号并且是唯一的这就是实体唯一标识。同时订单实体会从创建，支付，发货等过程最终走到终态这就是实体的生命周期。订单实体在这个过程中属性发生了变化，但订单还是那个订单，不会因为属性的变化而变化，这就是实体的延续性。

2.  值对象（VALUE-OBJECT）
	通过对象属性值来识别的对象，它将多个相关属性组合为一个概念整体。在 DDD 中用来描述领域的特定方面，并且是一个没有标识符的对象，叫作值对象。值对象没有唯一标识，没有生命周期，不可修改，当值对象发生改变时只能替换（例如String的实现）。
3.  
mvn archetype:generate -DgroupId='com.panda.baicy' -DartifactId='demo-web' -Dversion='1.0.0-SNAPSHOT' -Dpackage='com.panda.baicy' -DarchetypeArtifactId='cola-framework-archetype-web' -DarchetypeGroupId='com.alibaba.cola' -DarchetypeVersion='4.3.2'
