# envoy 常见配置问题

+ uatfile upload， host 不匹配返回403   ---->  route: { cluster: uatfile-cluster, timeout: { seconds: 10 } ,auto_host_rewrite: true}
+ uatfile/开户转发需要配置 https -----> https://www.envoyproxy.io/docs/envoy/latest/start/quick-start/securing
+ uatfile 返回413,Payload Too Large -- > https://www.envoyproxy.io/docs/envoy/latest/faq/configuration/flow_control#faq-flow-control
+ uatfile 返回408 ---- >  route: { cluster: uatfile-cluster, timeout: { seconds: 60 } ,auto_host_rewrite: true}
+ 正则表达式超长报错---> 需要修改正则表达式长度
regex '^/user/(488/sms|h5/getJwt|captcha|version/getAIDrawTemplate|version/isMainLand|version/promotionTemplate|version/activityTemplate2).*' RE2 program size of 106 > max program size of 100 set for the error level threshold. Increase configured max program size if necessary.

+ x-forwarded-for 失效
    Envoy will only append to XFF if the use_remote_address HTTP connection manager option is set to true and the skip_xff_append is set false. This means that if use_remote_address is false (which is the default) or skip_xff_append is true, the connection manager operates in a transparent mode where it does not modify XFF.

+ x-envoy-external-address
	It is a common case where a service wants to perform analytics based on the origin client’s IP address. Per the lengthy discussion on XFF, this can get quite complicated, so Envoy simplifies this by setting x-envoy-external-address to the trusted client address if the request is from an external client. x-envoy-external-address is not set or overwritten for internal requests. This header can be safely forwarded between internal services for analytics purposes without having to deal with the complexities of XFF.