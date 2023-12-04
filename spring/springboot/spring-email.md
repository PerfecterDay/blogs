## 使用Spring email 发送邮件
{docsify-updated}

- [使用Spring email 发送邮件](#使用spring-email-发送邮件)
	- [配置与使用](#配置与使用)
	- [邮件支持HTML渲染](#邮件支持html渲染)
	- [常见问题](#常见问题)


### 配置与使用
```
spring.mail.host=smtp.gmail.com
spring.mail.protocol=smtps
spring.mail.port=465
spring.mail.username=perfect_a_day
spring.mail.password=dsaioi0njn9
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true

注入 JavaMailSender
@Resource
JavaMailSender javaMailSender;
```

### 邮件支持HTML渲染
```
MimeMessage message = this.mailSender.createMimeMessage();
// Set From: header field of the header.
message.setFrom(new InternetAddress(from));
// Set To: header field of the header.
message.addRecipient(Message.RecipientType.TO, new InternetAddress(to));
// Set Subject: header field
message.setSubject("This is the Subject Line!");
// Now set the actual message
message.setText(body, "UTF-8", "html");
```

### 常见问题
1. org.springframework.mail.MailSendException: Failed messages: com.sun.mail.smtp.SMTPSendFailedException: 550 Invalid User
message.setFrom("perfect_a_day@163.com"); 填写正确的用户名

2. response: [EOF]. Failed messages: javax.mail.MessagingException: Got bad greeting from SMTP host: smtp.163.com, port: 465, response: [EOF]
升级协议为 smtps
```
spring.mail.host=smtp.gmail.com
spring.mail.protocol=smtps
spring.mail.port=465
spring.mail.username=perfect_a_day
spring.mail.password=dsaioi0njn9
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```