# Sa-token

## 获取token值

<center><img src="pics/sa-token.png" alt=""></center>

## 生成的token/session key 名
`StpLogic`  类的这些方法：
<center><img src="pics/sa-token-2.png" alt=""></center>

## Session 会话
在 Sa-Token 中，Session 分为三种，分别是：

+ User-Session: 指的是框架为每个 账号id 分配的 Session
+ Token-Session: 指的是框架为每个 token 分配的 Session
+ Custom-Session: 指的是以一个 特定的值 作为SessionId，来分配的 Session

StpLogic -> distUsableToken 顶下线功能实现


+ token session: "gtja-international-app-token:login:token-session:eb02cb66-d1db-43bc-ade3-4c81+ 5ce2673"
+ session: gtja-international-app-token:login:session:2"
+ 长token： "gtja-international-app-token:login:token:eb02cb66-d1db-43bc-ade3-4c8175ce2673"
+ 短 token: "gtja-international-app-token:short-token:S-b0f3e72d-d8c0-41bf-8c24-298ff06927f1"


