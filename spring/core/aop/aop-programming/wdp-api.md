# WDP 接口文档
uat环境请求域名：
https://envoyhkuat.gtjaidemo.net:10001

## soft token limit相关接口
1. 设置upliftAmount并计算soft token limit值
请求地址

```
POST
https://envoyhkuat.gtjaidemo.net:10001/cap/v1.0/wdp/uplift-amount
```

请求参数：
```
{
    "clntCode":"002", #交易账号
    "upliftAmount":2 #upliftAmount取值
}
```
响应参数
成功示例
```
{
    "code": 0,
    "data": {
        "clntCode": "002", # 交易账号
        "finalLimit": 5,  # soft token limit计算结果值
        "upliftAmount": 2 # 设置值
    },
    "msg": "success"
}
```
失败示例
```
{
    "code": -1,
    "data": {},
    "msg": "交易账号不存在或无账户持有人信息"
}
```

2. 查询当前upliftAmount以及soft token limit值

请求地址
```
GET
https://envoyhkuat.gtjaidemo.net:10001/cap/v1.0/wdp/uplift-amount/{clntcode}
```

请求参数：
```
clntcode为实际的交易账号
```

响应参数
成功示例
```
{
    "code": 0,
    "data": {
        "clntCode": "001",
        "finalLimit": 8,
        "upliftAmount": 5
    },
    "msg": "success"
}
```
失败示例
```
{
    "code": -1,
    "data": {},
    "msg": "交易账号不存在或无账户持有人信息"
}
```

## 账号解锁相关
1. 查询用户是否锁定状态
请求
```
curl --location 'https://envoyhkuat.gtjaidemo.net:10001/cap/v1/wdp/user/query' \
--header 'x-api-key: 98shd00a00wer8-a8dagf0dsa8d0-0' \
--header 'x-app-id: wdp' \
--header 'Content-Type: application/json' \
--data '{
    "acct": "203736"
}'
```

响应：
```
{
    "code": 0,
    "data": {
        "acct":"203736",
        "status": "Locked",
        "failCount": 5
    },
    "msg": "success"
}
```
失败：
```
{
    "code": -1,
    "data": {},
    "msg": "交易账号不存在或无账户持有人信息"
}
```
解锁客户信息
请求
```
curl --location 'https://envoyhkuat.gtjaidemo.net:10001/cap/v1/wdp/user/unlock' \
--header 'x-app-id: wdp' \
--header 'x-api-key: 98shd00a00wer8-a8dagf0dsa8d0-0' \
--header 'Content-Type: application/json' \
--data '{
    "acct":"203736",
    "status":"Activated"
}'
```
响应
```
{
    "status": "0",
    "bgStatus": "0",
    "data": {
    },
    "msg": "success"
}
```
失败
```
{
    "code": -1,
    "data": {},
    "msg": "无法设置该状态"
}
```