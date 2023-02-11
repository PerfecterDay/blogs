## REDIS
内：r-3ns5xcjxon2hpod3zp.redis.rds.aliyuncs.com:6379
r-3ns5xcjxon2hpod3zp
iVWzhb80vUFyugxA

redis-cli -h r-3ns5xcjxon2hpod3zppd.redis.rds.aliyuncs.com
auth r-3ns5xcjxon2hpod3zp iVWzhb80vUFyugxA

---------

## Mysql


rm-3nszwm39bc4foon2blo.mysql.rds.aliyuncs.com
高权限账号：
gtja_uatadmin
p0BWm71pakWIZ3pH

mysql -hrm-3nszwm39bc4foon2blo.mysql.rds.aliyuncs.com -ugtja_uatadmin -pp0BWm71pakWIZ3pH

----------

## Kafka
alikafka-post-public-intl-sg-uq832vxmo01-1-vpc.alikafka.aliyuncs.com:9092,
alikafka-post-public-intl-sg-uq832vxmo01-2-vpc.alikafka.aliyuncs.com:9092,
alikafka-post-public-intl-sg-uq832vxmo01-3-vpc.alikafka.aliyuncs.com:9092



curl -X POST localhost:10000/user/app/configuration
curl -X POST localhost:8888/user/app/configuration
