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
9.  `-G` ：用来构造 URL 的get请求的查询字符串。
10. `-H` ：添加 HTTP 请求的标头。  `curl -H "X-First-Name: Joe" https://example.com`
11. `-i` ：打印出服务器回应的 HTTP 标头。
12. `-I` ：向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。
13. `-k` ：指定跳过 SSL 检测。
14. `-L` ：让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。
15. `--limit-rate` ：用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。
16. `-o` ：将服务器的回应保存成文件，等同于 `wget` 命令.
17. `-O` ：将服务器回应保存成文件，并将 URL 的最后部分当作文件名。
18. `-s` ：将不输出错误和进度信息。
19. `-S` ：指定只输出错误信息，通常与-s一起使用。
20. `-u` ：用来设置服务器认证的用户名和密码。
21. `-v` ：输出通信的整个过程，用于调试。
22. `-x` ：指定 HTTP 请求的代理。
23. `-X` ：指定 HTTP 请求的方法。
24. `--cert`： 



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

```