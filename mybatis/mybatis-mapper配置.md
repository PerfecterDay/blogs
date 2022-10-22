


首先，如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置为目标属性就 OK 了。
```
 <insert id="save" parameterType="com.gtja.gjyw.repo.dao.entity.SmsEntity"
	useGeneratedKeys="true" keyProperty="id,create_time,update_time">
	INSERT INTO sms(mobile,sms_code,purpose) VALUES ('${mobile}','${smsCode}','${purpose}')
</insert>
```
For insert statement, the method return type must be void or int.
If you use int, the number of inserted rows (always 1 in your case) will be returned.
The generated key will be set to the id field of the parameter instance.
Also, you should specify keyColumn.



https://stackoverflow.com/questions/42387576/how-to-get-a-field-value-which-is-default-to-current-timestamp-in-an-insert-st

mybatis-plus:
  mapper-locations: classpath*:mappers/**/*Mapper.xml