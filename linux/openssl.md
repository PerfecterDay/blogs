## OpenSSL
{docsify-updated}

- [OpenSSL](#openssl)
	- [生成公私钥对](#生成公私钥对)
	- [Creating Certiﬁcate Signing Requests](#creating-certiﬁcate-signing-requests)
	- [生成自签名证书](#生成自签名证书)
		- [制作多域名的自签名证书](#制作多域名的自签名证书)
		- [查看自签名证书](#查看自签名证书)
	- [生成自己的CA](#生成自己的ca)
	- [测试TLS](#测试tls)
		- [证书验证](#证书验证)
		- [域名验证](#域名验证)


### 生成公私钥对
生成 RSA 公私密钥对，默认是 PKCS#8 格式：
```
openssl genpkey -out fd.key \
-algorithm RSA \
-pkeyopt rsa_keygen_bits:2048 \
-aes-128-cbc
```
`-aes-128-cbc` 表示必须使用密码加密，如不想加密可以省略该参数

查看生成的密钥：
```
openssl pkey -in fd.key -text -noout
```

查看公钥：
```
openssl pkey -in fd.key -pubout -out fd-public.key
```

生成 ECDSA 公私密钥对，默认是 PKCS#8 格式：
```
openssl genpkey -out fd.key \
-algorithm EC  \
-pkeyopt ec_paramgen_curve:P-256 \
-aes-128-cbc
```

### Creating Certiﬁcate Signing Requests
1. 以交互的方式生成CSR
	```
	openssl req -new -key fd.key -out fd.csr
	```

	这种方式会让你填写一系列选项，如果想跳过输入`.`而不是直接回车。

2. 查看生成的CSR：
	```
	openssl req -text -in fd.csr -noout
	```
3. 非交互（自动）生成CSR
   创建一个配置文件 fd.cnf ：
   ```
   	[req]
	prompt = no
	distinguished_name = dn
	req_extensions = ext
	input_password = PASSPHRASE

	[dn]
	CN = www.feistyduck.com
	emailAddress = webmaster@feistyduck.com
	O = Feisty Duck Ltd
	L = London
	C = GB

	[ext]
	subjectAltName = DNS:www.feistyduck.com,DNS:feistyduck.com
   ```
	然后执行 `openssl req -new -config fd.cnf -key fd.key -out fd.csr`

### 生成自签名证书
如果已经生成了CSR：
```
openssl x509 -req -days 365 -in fd.csr -signkey fd.key -out fd.crt
```

跳过生成CSR，直接使用密钥生成自签名证书：
```
openssl req -new -x509 -days 365 -key fd.key -out fd.crt
```
这样也会让你填写一系列选项,我们也可以直接在命令行通过 `-subj` 参数填写：

```
openssl req -new -x509 -days 365 -key fd.key -out fd.crt \
 -subj "/C=GB/L=London/O=Feisty Duck Ltd/CN=www.feistyduck.com"
```

#### 制作多域名的自签名证书
将域名信息填写到一个 txt 文件 fd.ext 中，内容如下:
```
subjectAltName = DNS:*.feistyduck.com, DNS:feistyduck.com
```

然后使用下面命令生成证书：
```
openssl x509 -req -days 365 \
-in fd.csr -signkey fd.key -out fd.crt \
-extfile fd.ext
```

#### 查看自签名证书
```
openssl x509 -text -in fd.crt -noout
```

### 生成自己的CA

### 测试TLS
```
openssl s_client -crlf \
-connect www.feistyduck.com:443 \
-servername www.feistyduck.com
```
+ `-connect` 选项用于建立TCP连接
+ `-servername` 用于指定在TLS级别发送的主机名。

1.1.1 之后的版本会自动发送 servername ，所以我们可以省略这个参数。但是以下情况仍让需要手动指定这个参数：
1. 使用的是老版本（1.1.1之前）
2. connect 的是一个 IP 地址
3. TLS 主机与 servername 不同

可以使用 `-noservername` 选项来禁止在 TLS 握手的时候发送主机域名信息。

#### 证书验证
如果证书验证失败，通常会显示：
`Verify return code: 20 (unable to get local issuer certificate)`

#### 域名验证
openssl 默认不会验证域名，需要手动加上`-verify_hostname www.feistyduck.com` 参数：
```
openssl s_client -connect www.feistyduck.com:443 -verify_hostname www.feistyduck.com
```
如果域名验证失败会返回:
`Verify return code: 62 (Hostname mismatch)`

如果测试的是IP，那么需要使用`-verify_ip` 选项指定验证的IP