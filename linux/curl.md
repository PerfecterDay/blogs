# curl 命令简介
{docsify-updated}

1. 不带有任何参数时，curl 就是发出 GET 请求。
	`curl http://www.baidu.com`
2. `-A` ：指定客户端的用户代理标头，即 `User-Agent` 。curl 的默认用户代理字符串是`curl/[version]`
3. `-b` ：用来向服务器发送 Cookie。
	`curl -b 'foo=bar' https://google.com`
	会生成一个http头`Cookie: foo=bar`，向服务器发送一个名为foo、值为bar的 Cookie
4. `-c` ：将服务器设置的 Cookie 写入一个文件。
	`curl -c cookies.txt https://www.google.com`
5. `-d` ：用于发送 POST 请求的数据体。
	`curl -d'login=emma＆password=123'-X POST https://google.com/login`
	还可以读取本地文件的数据发送给服务器。
	`curl -d '@data.txt' https://google.com/login`
6. `--data-urlencode`：等同于`-d`，但是会自动进行URL编码。
7. `--data-raw`： 发送原始数据体
	```
	curl --location --request POST 'https://10.187.144.42:8000/trade/' \
	--header 'App-Name: gtjagjapp' \
	--header 'App-ID: 3E1EEECDB7A577CCA1A2E2C1885DBE7A' \
	--header 'Content-Type: application/json' \
	--data-raw '[
		{
			"code": "1231",
			"params": {
				"username":"700210",
				"token":"ABCDEFGHIJKLMNOPQRST0123456789",
				"account_type":"",
				"market_code":"",
				"currency_code":"HKD",
				"balance_type":"HKD",
				"language":"1"
			},
			"params_ext":{}
		}
	]'
	```
8. `-e` ：用来设置 HTTP 的标头Referer，表示请求的来源。
9.  `-F` ：用来向服务器上传二进制文件。
	`curl -F 'file=@photo.png' https://google.com/profile`
	会给 HTTP 请求加上标头Content-Type: multipart/form-data，然后将文件photo.png作为file字段上传。
10. `-G` ：用来构造 URL 的get请求的查询字符串。
11. `-H` ：添加 HTTP 请求的标头。  `curl -H "X-First-Name: Joe" https://example.com`
12. `-i` ：打印出服务器回应的 HTTP 标头。
13. `-I` ：向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。
14. `-k` ：指定跳过 SSL 检测。
15. `-L` ：让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。
16. `--limit-rate` ：用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。
17. `-o` ：将服务器的回应保存成文件，等同于 `wget` 命令.
18. `-O` ：将服务器回应保存成文件，并将 URL 的最后部分当作文件名。
19. `-s` ：将不输出错误和进度信息。
20. `-S` ：指定只输出错误信息，通常与-s一起使用。
21. `-u` ：用来设置服务器认证的用户名和密码。
22. `-v` ：输出通信的整个过程，用于调试。
23. `-x` ：指定 HTTP 请求的代理。
24. `-X` ：指定 HTTP 请求的方法。
25. `--cert`: 指定证书
26. `--form`: 指定 form 参数， `--from 'UserID=67buu'`
27. `--http1.1/--http2/--http3`: 指定使用的http 版本 


```
curl -k --proxy 10.184.161.160:443 --cert /home/5.11/server.pem https://10.18.172.216:44300/APIServer/SendRequest -F
'xml="
<sp_Profile_CheckDuplicateTel_ForGTJAApp ID="" ProfileID="@NULL" ContactCountry="" ContactArea=""
	ContactDetail="1122334455" />"' -F 'appID=YYgfdg4654HGF76547gfg65x2'

<STPAPI_RESULT>
	<ANSWER ID="" ReturnCode="0">
		<RECORDS Total_Records="40">
			<ROW AccountCode="009991" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD.-  ERROR ACCOUNT." AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="009992" AccountName="GUOTAI JUNAN SECURITIES (HK) - WAREHOUSE 0.25%" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="009993" AccountName="GUOTAI JUNAN SECURITIES (HK) - SPECIAL ERROR A/C" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Closed" Is2FAMobile="true" />
			<ROW AccountCode="018324" AccountName="GTJA - TESTING A/C" AECode="" AccSettingType="Profile-MobTel (1)"
				ContactType="MobTel" DuplicateInfo="86-1122334455" ProfileID="133996"
				ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="018324S" AccountName="GTJA - TESTING A/C" AECode="" AccSettingType="Profile-MobTel (1)"
				ContactType="MobTel" DuplicateInfo="86-1122334455" ProfileID="133996"
				ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="018334" AccountName="GUOTAI JUNAN SECURITIES (H.K.) LTD. - TESTING A/C," AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="018334S" AccountName="GUOTAI JUNAN SECURITIES (H.K.) LTD. - TESTING A/C," AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="018344" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - TESTING A/C." AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="031798" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD - IBD DERIVATIVE" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="088706" AccountName="GUOTAI JUNAN SECURITIES (H.K.) LTD. - TESTING A/C." AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="095308" AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LIMITED" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="096508" AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LIMITED - MARKET MAKING A/C"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Closed" Is2FAMobile="true" />
			<ROW AccountCode="096509" AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LIMITED - MARKET MAKING SBL A/C"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Closed" Is2FAMobile="true" />
			<ROW AccountCode="132977" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD - DERIVATIVES" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="156009" AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LIMITED - ED DEPT DERIVATIVES"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="169553" AccountName="GUOTAI JUNAN SEC. (HK) LTD - IS DEPT MARKET MAKING" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="169778" AccountName="GUOTAI JUNAN SEC. (HK) LIMITED - GM DESK" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="199615" AccountName="GUOTAI JUNAN SECURITIES (H.K.) LTD.- TESTING A/C 1" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="199616" AccountName="GUOTAI JUNAN SECURITIES (H.K.) LTD.- TESTING A/C 2" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="199617" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - TESTING A/C 3" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="200280" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - WEALTH MANAGE" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="200738"
				AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - WEALTH MANAGE - SUB A/C - GUOTAI JUNAN SEC. (HK) LTD. - WM PRODUCT"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="201165" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - IPO STABILIZATION" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="201263"
				AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - WEALTH MANAGEMENT - SUB A/C - GUOTAI JUNAN SECURITIES (HK) LTD. - WM PRODUCT COLLATERAL"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="201265"
				AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - WEALTH MANAGEMENT - SUB A/C - GUOTAI JUNAN SECURITIES (HK) LTD. - WM PRODUCT STAMP DUTY"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="201266"
				AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - WEALTH MANAGEMENT - SUB A/C - GUOTAI JUNAN SECURITIES (HK) LTD. - WM PRODUCT REPO"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="202381" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD. - PCS BONDS" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="202997"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WEALTH MANAGEMENT - SUB A/C - GUOTAI JUNAN SECURITIES (HK) LTD. - WM PRODUCT SBL"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="206269" AccountName="GUOTAI JUNAN SECURITIES(HK)LTD-IPO STABILIZATION A" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="206272" AccountName="GUOTAI JUNAN SECURITIES (HK) LTD- IBD BONDS" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910002"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM" AECode=""
				AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910009"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - GUOTAI JUNAN SEC (HK) LTD. – WM STOCKS MM"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910010"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - GUOTAI JUNAN SEC (HK) LTD. – WM STOCKS NON-MM"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910011"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - GUOTAI JUNAN SEC (HK) LTD. – WM GTJA WTS &amp; CBBC"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910012"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - GUOTAI JUNAN SEC (HK) LTD. –WM NON-GTJA WTS &amp; CBBC"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910013"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - SEC - WM MM"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910015"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - SEC - WM SBL"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910016"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - GUOTAI JUNAN SEC (HK) LTD. –WM SP (NON- NOTES)"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910017"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - GUOTAI JUNAN SEC (HK) LTD. –WM SP (NOTES)"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
			<ROW AccountCode="910018"
				AccountName="GUOTAI JUNAN SECURITIES (HONG KONG) LTD. - WM FINANCIAL PRODUCTS TEAM - SUB A/C - SEC - WMFP TRADING"
				AECode="" AccSettingType="Profile-MobTel (1)" ContactType="MobTel" DuplicateInfo="86-1122334455"
				ProfileID="133996" ProfileName="GUOTAI JUNAN SECURITIES (HK) LTD." AccountCategory="House Institution"
				MainAccountStatus="Active" Is2FAMobile="true" />
		</RECORDS>
	</ANSWER>
</STPAPI_RESULT>


curl -k --proxy 10.184.161.160:443 --cert /home/5.11/server.pem https://10.18.172.216:44300/APIServer/SendRequest -F
'xml="
<sp_AccountIDInfoDateAndEamil_ListSearch ID="" AccountCode="018324" />"' -F 'appID=YYgfdg4654HGF76547gfg65x2'


<STPAPI_RESULT>
	<ANSWER ID="" ReturnCode="0">
		<RECORDS Total_Records="1">
			<ROW AccountCode="018324" ProfileType="LicensedCorporate" DateOfBirth="" DateOfIncorporate="19930708"
				IDNo="433562" Email="xxx@qq.com" />
		</RECORDS>
	</ANSWER>
</STPAPI_RESULT>


-----------Xgate--------------
测试：
curl -X POST https://smsc.xgate.com.hk/smshub/sendsms \
--form 'MessageLanguage=UTF8' \
--form 'userId=guotaijas_api8' \
--form 'userPassword=npxYE0l6m3p1' \
--form 'MessageReceiver=86-17674090666' \
--form 'MessageBody=730245'

生产：
curl -X POST https://smsc.xgate.com.hk/smshub/sendsms \
--form 'MessageLanguage=UTF8' \
--form 'userId=guotaijas_api9' \
--form 'userPassword=8UfQBMHS9A' \
--form 'MessageReceiver=86-15121127343' \
--form 'MessageBody=【国泰君安国际】尊敬的客户，您的君弘全球通一次性验证码为:123456(3分钟内有效)。如非本人操作，请忽略本短信。'

<?xml version='1.0' encoding='UTF-8'?><!DOCTYPE XGATE_Response><ShortMessageResponse><Success>true</Success><ResponseCode>A000</ResponseCode><ResponseMessage>OK</ResponseMessage><MessageBatchID>309463164</MessageBatchID><SessionID>null</SessionID><NumberOfMessage>1</NumberOfMessage><NumberOfSuccess>1</NumberOfSuccess><NumberOfFailure> 0</NumberOfFailure><ReceiveTime>2023-05-17 22:36:42</ReceiveTime><ReceiverList><Receiver><MessageID>6</MessageID><MessageType>TEXT</MessageType><MessageLanguage>UTF8</MessageLanguage><MessageScheduleTime> 000000000000</MessageScheduleTime><TimeToLive>-1</TimeToLive><AreaCode>86</AreaCode><DestinationCountry>China</DestinationCountry><MobileNumber>15121127343</MobileNumber><OperatorID>CHINA-MOBILE</OperatorID><MessageBody>Not available</MessageBody><ACK>1</ACK><PartNo>1</PartNo><Status>Sent to SMS Centre</Status></Receiver></ReceiverList></ShortMessageResponse>

------捷利---------
UAT:
curl -X POST -d '{
    "appkey": "D133F72A44D7082C56B1DB3FE0332E33",
    "username": "gtj.gtjuser2022",
    "password": "gtj221p128wd"
   }'  https://gtjdata-uat.iqdii.com/jybapp/login/serverlogin


   curl -X POST -d '{
    "token": "A56C697D8808861029BCC1AE",
    "username": "700239",
    "ipaddress":"223.165.23.63"
   }'  https://gtjdata-uat.iqdii.com/jybapp/login/getusersession

    curl -X POST -d '{
    "token": "3153B693F85E11232D5F9B82",
    "username": "000328"
   }'  https://gtjdata-uat.iqdii.com/jybapp/service/GetUserPkg


PRD:
curl -X POST -d '{
    "appkey": "D133F72A44D7082C56B1DB3FE0332E33",
    "username": "gtj.gtjuser2023",
    "password": "gtj231p0w1d"
   }'  https://gtjdata.iqdii.com/jybapp/login/serverlogin

   curl -X POST -d '{
    "token": "9B1E73E0A8F8B16C0AE16CF5",
    "username": "700239",
    "ipaddress":"223.165.23.63"
   }'  https://gtjdata.iqdii.com/jybapp/login/getusersession


   curl -X POST -d '{
    "token": "9B1E73E0A8F8B16C0AE16CF5",
    "username": "621002",
    "ipaddress":"223.165.23.63"
   }'  https://gtjdata.iqdii.com/jybapp/service/GetUserPkg


curl -X GET https://gtjdata.iqdii.com/jybapp/price/GetStockMarketInfo?session=
   
```