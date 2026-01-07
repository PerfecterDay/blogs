
## Token/session
1. User Session
2. Dynamic Agent Session -client-authenticated SSL，不会闲置超时和过期
Accessing REST API over HTTPS URL using mutual/client-authenticated SSL using agent/client certificate,
or setting simulated agent certificate property in amsystem.properties (this is a quick workaround only for convenience of testing in non-production environments; it should NOT be used in production environments)

"dynamicAgentSession": "true",

3. Access Token - You can generate an access token for the non-interactive user in the Access Token tab of that user's configurations page in the Admin Console.
"Authorization: Bearer " request header is set

Dynamic Agent Session 是如何设置的？我看测试环境用的是这种方式 ？ 到时候生产同样嘛？


## 绑定
1. `update/user/create`  
   1. 只需要 jmeter 中的参数嘛？ assignRealm 什么意思？

2. `/OATH/create-assign-and-encrypt`
   1. clientPublicKey/clientPublicKeyId 等参数如何生成 ？app SDK ？
   2. batchId 有什么用 ？ userStoreId 是什么，API 文档里边没有 ？

3. `/token/activateByOTP`
   1. serialNumber 是 SDK 生成的嘛 ？
   2. otp 是发送的 activationCode ?

三步中间某一步中断的话，如何继续绑定流程 ？

## pin 码验证
1. SDK 验证即可

## 设备查询
4. 查询客户绑定的设备 `device/listDevicesOfUser` ？

## 重置 pin
5. 生物认证解锁需要调用 UAS API 嘛？ 
6. 重置 pin 是用的哪（几）个 API ？ `password/reset` ?



那客户端是不是 调用 activationTokenEx 之后，就能使用 pin 认证了，即使 est/token/activateByOTP  没有成功 ？因为之前说 pin 认证是纯SDK 操作的

是可以，但是服务器令牌的状态不是Activated的话，服务器验证不了OTP(生成的OTP验证码)


/token/activateByOTP 这步是可选的，在create-assign-and-encrypt时，这个参数改成status: Activation，那么服务器令牌就是激活状态


## 问题
如何使客户端的iSprint 失效 ？ 如果 pin 认证不在后端验证

 @yik(i-Sprint)   @Tony  如果pin 验证直接在 前段时间SDK完成，不用跟后端交互。那是不是没有办法在后端控制一台设备禁止 pin 登录了啊 ？

