## OpenSSL
{docsify-updated}

- [OpenSSL](#openssl)
	- [命令](#命令)
	- [证书制作](#证书制作)
		- [生成公私钥对](#生成公私钥对)
		- [Creating Certiﬁcate Signing Requests](#creating-certiﬁcate-signing-requests)
		- [生成自签名证书](#生成自签名证书)
			- [制作多域名的自签名证书](#制作多域名的自签名证书)
			- [查看自签名证书](#查看自签名证书)
	- [生成自己的CA-Certification Authority](#生成自己的ca-certification-authority)
		- [创建配置文件 root-ca.conf](#创建配置文件-root-caconf)
		- [创建文件夹](#创建文件夹)
		- [创建CA](#创建ca)
		- [使用CA签发证书](#使用ca签发证书)
	- [测试TLS](#测试tls)
		- [证书验证](#证书验证)
		- [域名验证](#域名验证)
	- [常见的证书格式](#常见的证书格式)
	- [实战](#实战)

+ [Mozilla的证书](https://hg.mozilla.org/releases/mozilla-beta/file/tip/security/nss/lib/ckfw/builtins/certdata.txt)
+ [Curl的证书](https://curl.se/docs/caextract.html)

### 命令
+ `openssl version -a` : 查看版本信息，`OPENSSLDIR` 中存放了openssl配置的路径
+ `openssl help` : 帮助信息

### 证书制作
大多数用户转向OpenSSL，因为他们希望配置和运行支持SSL的Web服务器。该过程包括三个步骤：
1. 生成私钥
2. 创建证书签名请求（CSR）并将其发送到CA
3. 在Web服务器中安装CA提供的证书

#### 生成公私钥对
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

#### Creating Certiﬁcate Signing Requests
1. 以交互的方式生成CSR  
	```
	openssl req -new -key fd.key -out fd.csr
	```

	这种方式会让你填写一系列选项，如果想跳过输入`.`而不是直接回车。

2. 查看生成的CSR 
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

#### 生成自签名证书
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

##### 制作多域名的自签名证书
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

##### 查看自签名证书
```
openssl x509 -text -in fd.crt -noout
```

### 生成自己的CA-Certification Authority

#### 创建配置文件 root-ca.conf
(windows 注意文件编码问题： configuration file routines:def_load_bio:missing equal sign)

```
   [default]
   name                    = root-ca
   domain_suffix           = example.com
   aia_url                 = http://$name.$domain_suffix/$name.crt
   crl_url                 = http://$name.$domain_suffix/$name.crl
   ocsp_url                = http://ocsp.$name.$domain_suffix:9080
   default_ca              = ca_default
   name_opt                = utf8,esc_ctrl,multiline,lname,align
   
   [ca_dn]
   countryName             = "GB"
   organizationName        = "Example"
   commonName              = "Root CA"
   
   [ca_default]
   home                    = .
   database                = $home/db/index
   serial                  = $home/db/serial
   crlnumber               = $home/db/crlnumber
   certificate             = $home/$name.crt
   private_key             = $home/private/$name.key
   RANDFILE                = $home/private/random
   new_certs_dir           = $home/certs
   unique_subject          = no
   copy_extensions         = none
   default_days            = 3650
   default_crl_days        = 365
   default_md              = sha256
   policy                  = policy_c_o_match
   
   [policy_c_o_match]
   countryName             = match
   stateOrProvinceName     = optional
   organizationName        = match
   organizationalUnitName  = optional
   commonName              = supplied
   emailAddress            = optional
   
   [req]
   default_bits            = 4096
   encrypt_key             = yes
   default_md              = sha256
   utf8                    = yes
   string_mask             = utf8only
   prompt                  = no
   distinguished_name      = ca_dn
   req_extensions          = ca_ext
   
   [ca_ext]
   basicConstraints        = critical,CA:true
   keyUsage                = critical,keyCertSign,cRLSign
   subjectKeyIdentifier    = hash
   
   [sub_ca_ext]
   authorityInfoAccess     = @issuer_info
   authorityKeyIdentifier  = keyid:always
   basicConstraints        = critical,CA:true,pathlen:0
   crlDistributionPoints   = @crl_info
   extendedKeyUsage        = clientAuth,serverAuth
   keyUsage                = critical,keyCertSign,cRLSign
   nameConstraints         = @name_constraints
   subjectKeyIdentifier    = hash
   
   [crl_info]
   URI.0                   = $crl_url
   
   [issuer_info]
   caIssuers;URI.0         = $aia_url
   OCSP;URI.0              = $ocsp_url
   
   [name_constraints]
   permitted;DNS.0=example.com
   permitted;DNS.1=example.org
   excluded;IP.0=0.0.0.0/0.0.0.0
   excluded;IP.1=0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0
   
   [ocsp_ext]
   authorityKeyIdentifier  = keyid:always
   basicConstraints        = critical,CA:false
   extendedKeyUsage        = OCSPSigning
   noCheck                 = yes
   keyUsage                = critical,digitalSignature
   subjectKeyIdentifier    = hash
   ```
#### 创建文件夹
```
   $ mkdir root-ca
   $ cd root-ca
   $ mkdir certs db private
   $ chmod 700 private
   $ touch db/index
   $ openssl rand -hex 16  > db/serial (windos: openssl rand -hex 16 | Out-File -encoding ascii -NoNewline "db\serial"，否则会报 error while loading serial number openssl)
   $ echo 1001 > db/crlnumber
   ```
 + certs/: 证书存储；新的证书将在发布时放置在此处。
 + db/:该目录用于证书数据库（索引）和保存下一个证书和CRL序列号。OpenSSL将创建一些附加文件需要。
 + private/: 此目录将存储私钥，一个用于CA，另一个用于OCSP响应者。储存这些信息的机器应该只有最少数量的用户帐户。

#### 创建CA
1. 创建密钥和CSR：
	```
	openssl req -new \
	-config root-ca.conf \
	-out root-ca.csr \
	-keyout private/root-ca.key
	```
2. 创建自签名root证书
	```
	openssl ca -selfsign \
	-config root-ca.conf \
	-in root-ca.csr \
	-out root-ca.crt \
	-extensions ca_ext
	```

#### 使用CA签发证书
   

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


CRL： Certiﬁcate Revocation List (CRL) 
OSCP: Online Certiﬁcate Status Protocol 
AIA: Authority Information Access

### 常见的证书格式
1. 二进制（DER）证书

	包含原始形式的X.509证书，使用DER ASN.1编码。

2. ASCII（PEM）证书

	包含一个base64编码的DER证书，使用`-----BEGIN CERTIFICATE`开始，`-----ENDCERTIFICATE-----`结束。通常每个文件只有一个证书，尽管有些程序允许多个证书。例如，较旧的Apache web服务器版本需要服务器的证书单独在一个文件中，将所有中间证书一起放在另一个文件中。

3. 旧版OpenSSL密钥格式
	
	包含原始形式的私钥，使用DER ASN.1编码。从历史上看，OpenSSL使用了一种基于`PKCS#1`的格式。现在，如果你使用正确的命令（即genpkey），OpenSSL默认为 `PKCS#8`。

4. ASCII（PEM）密钥

	包含一个base64编码的DER密钥，有时还包含其他元数据（例如用于密码保护的算法）。页眉和页脚中的文本根据底层密钥格式不同会显示不同。  
	PKCS #1 标准主要用于 RSA密钥，其RSA公钥和RSA私钥PEM格式：
	```
	// PKCS#1公钥格式
	-----BEGIN RSA PUBLIC KEY-----
	BASE64 DATA...
	-----END RSA PUBLIC KEY-----
	// PKCS#1私钥格式
	-----BEGIN RSA PRIVATE KEY-----
	BASE64 DATA...
	-----END RSA PRIVATE KEY-----
	```
	PKCS#8 标准定义了一个密钥格式的通用方案，其公钥和私钥PEM格式：
	```
	// PKCS#8公钥格式
	-----BEGIN PUBLIC KEY-----
	BASE64 DATA...
	-----END PUBLIC KEY-----
	// PKCS#8私钥格式
	-----BEGIN PRIVATE KEY-----
	BASE64 DATA...
	-----END PRIVATE KEY-----
	```

5. PKCS#7证书

	一种为传输签名或加密数据而设计的复杂格式，定义为RFC 2315。它通常以 `.p7b` 和 `.p7c` 扩展名一起出现，并且根据需要可以包含整个证书链。Java的keytool实用程序支持这种格式。

6. PKCS#8密钥

	新的私钥默认存储格式。`PKCS#8`在RFC 5208定义。无论出于何种原因，如果您需要从`PKCS#8`转换为传统格式，使用`pkcs8`命令。

7. PKCS#12（PFX）密钥和证书

	一种复杂的格式，可以存储和保护服务器密钥以及整个证书链，它通常与`.p12`和`.pfx`扩展名一起使用。这种格式通常用于Microsoft产品，但也用于客户端证书。如今，PFX 被用作`PKCS#12`的同义词。

### 实战
Here we’ll implement all the steps of that protocol, using openssl terminal commands. In
practice you’re more likely to use openssl in the form of an API in another language- but
learning the terminal commands is still valuable as a transferable skill. Each command is
displayed with some explanations of its flags below.

If you want to follow along, you can make 3 folders, 1 for Alice, Bob and the CA respectively.
You need to repeat steps 1 and 2 for Bob and CA so they can have their own pair of keys.
And you need to generate a self-signed certificate for the CA (shown below).

1. 为 Alice 生成一个私钥：
	```
	openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -pkeyopt rsa_keygen_pubexp:3 -out privkey-A.pem
	```
	+ genpkey ➝ generate a private key
	+ -algorithm RSA ➝ use the RSA algorithm (can also take “EC” for elliptic-curve)
	+ -pkeyopt opt:value ➝ set opt to value (see items below)
	+ rsa_keygen_bits:2048 ➝ sets the size of the key to 2048 bits (the default is 1024)
	+ rsa_keygen_pubexp:3 ➝ sets the public exponent e to 3 (default is 65, 537)
	+ -out privkey-A.pem ➝ outputs to the file privkey-A.pem

2. 基于生成的私钥，为 Alice 生成一个公钥：
	```
	openssl pkey -in privkey-A.pem -pubout -out pubkey-A.pem
	```
	+ pkey ➝ processes public or private keys
	+ -in privkey-A.pem ➝ read the key from filename privkey-A.pem
	+ -pubout ➝ output a public key (by default, a private key is output)

3. 查看生成的密钥信息
	```
	openssl pkey -in privkey-A.pem -text -noout
	openssl pkey -pubin -in pubkey-A.pem -text -noout
	```
	+ -noout ➝ suppresses the command from printing out the base64 encoding as well.

4. Alice生成一个证书签名请求（CSR-certificate signing request）
	```
	openssl req -new -key privkey-A.pem -out A-req.csr
	```
	+ req ➝ creates and processes signing requests
	+ -new ➝ generates a new certificate request, will prompt Alice for some information
	+ -key privkey-A.pem ➝ signs the request with Alice’s private key
	The command will prompt Alice with these questions:
	+ Country code [C]: {Alice fills in her country code}
	+ Province/STate name [ST]: {Alice fills in her province name fully}
	+ City/Location [L]: {The city Alice’s business is registered in, for example}
	+ Organization Name [O]: {Alice’s business name, for example}
	+ Organizational Unit Name [OU]: (Optional) {What part of the company is she?}
	+ Common Name [CN]: the hostname+domain, i.e. “www.alice.com”
	+ A challenge password []: {this can be used as a secret nonce between Alice and CA}

5. 为CA机构生成一个自签名的CA证书
	```
	openssl req -x509 -new -nodes -key rootkey.pem -sha256 -days 1024 -out root.crt
	```

6. 使用CA证书为 Alice签名 CSR
	```
	openssl x509 -req -in A-req.csr -CA root.crt -CAkey rootkey.pem -CAcreateserial -out A.crt -days 500 -sha256
	```
	+ x509 ➝ an x509 certificate utility (displays, converts, edits and signs x509 certificates)
	+ -req ➝ a certificate request is taken as input (default is a certificate)
	+ -CA root.crt ➝ specifies the CA certificate to be used as the issuer of Alice’s certificate
	+ -CAkey rootkey.pem ➝ specifies the private key used in signing (rootkey.pem)
	+ -CAcreateserial ➝ creates a serial number file which contains a counter for how many certificates were signed by this CA
	+ -days 500 ➝ sets Alice’s certificate to expire in 500 days
	+ -sha256 ➝ specifies the hashing algorithm to be used for the certificate’s signature

7. 查看Alice签名后的证书
	```
	openssl x509 -in Alice.crt -text -noout
	```

8. Alice验证bob 的公钥
	```
	openssl verify -CAfile root.crt Bob.crt
	```
	+ verify ➝ a utility that verifies certificate chains
	+ -CAfile root.crt ➝ specified the trusted certificate (root.crt)
	+ Bob.crt ➝ the certificate to verify
	+ If you get an OK, you know the certificate can be trusted

9. Alice 提取出 Bob 的公钥
	```
	openssl x509 -pubkey -in Bob.crt -noout > pubkey-B.pem
	```

10. Alice 使用 Bob 的公钥加密文件
	```
	openssl pkeyutl -encrypt -in largefile.txt -pubin -inkey pubkey-B.pem -out ciphertext.bin
	```
	+ pkeyutl ➝ utility to perform public key operations
	+ -encrypt ➝ encrypt the input data
	+ error! (recall: RSA is not meant for encrypting arbitrary large files- Alice needs to use symmetric key encryption for that)

11. ALice 生成一个对称密钥
	```
	openssl rand -base64 32 -out symkey.pem
	```
	+ rand ➝ generates pseudo-random bytes (seeded by default by $HOME/.rnd)
	+ -base64 32 ➝ outputs 32 random bytes and encodes it in base64

12. Alice 使用Bob的公钥加密生成的对称密钥
	```
	openssl pkeyutl -encrypt -in symkey.pem -pubin -inkey pubkey-B.pem -out symkey.enc
	```

13. Alice 使用私钥签名加密后的对称密钥,并且使用sha1 哈希签名后的文件
	```
	openssl dgst -sha1 -sign privkey-A.pem -out signature.bin symkey.pem
	```
	+ dgst -sha1 ➝ hash the input file using the sha1 algorithm
	+ -sign privkey-A.pem ➝ sign the hash with the specified private key
	+ symkey.pem ➝ the input file to be hashed

14. Bob 使用他的私钥解密加密后的文件
	```
	openssl pkeyutl -decrypt -in symkey.enc -inkey privkey-B.pem -out symkey.pem
	```
	+ -decrypt ➝ decrypt the input file

15. Bob重复上面的步骤获取Alice 的公钥

16. Bob 验证这条消息是来自 Alice
	```
	openssl dgst -sha1 -verify pubkey-A.pem -signature signature.bin symkey.pem
	```
	+ -verify pubkey-A.pem ➝ verify the signature using the specified filename
	+ -signature signature.bin ➝ specifies the signature to be verified
	+ symkey.pem ➝ the file to be hashed

	```
	openssl enc -aes-256-cbc -pass file:symkey.pem -p -md sha256 -in largefile.txt -out ciphertext.bin
	```
	+ enc -aes-256-cbc ➝ encrypt a file using the aes-256-cbc symmetric key algorithm
	+ -pass file:symkey.pem ➝ specified the file to get the symmetric key from
	+ -p ➝ prints the key, salt, initialization vector to the screen
	+ -md sha256 ➝ uses sha256 as part of the key derivation function (a function that derives one or more secondary secret keys from a primary secret key)

17. Bob使用对称密钥解密加密的文件
	```
	openssl enc -aes-256-cbc -d -pass file:symkey.pem -p -md sha256 -in ciphertext.bin -out largefile.txt
	```
	+ -d ➝ decryption flag


其它openssl 命令：
```
导出一个网站的公钥：
openssl s_client -connect 10.18.172.216:44300 -showcerts < /dev/null | openssl x509 -outform pem > cms_cert.pem

查看公钥的信息：
openssl x509 -in cms_cert.pem -noout -text

用CA证书验证某个证书的合法性：
sudo openssl verify -CAfile ~/Desktop/ISRGRootX1.pem ~/Desktop/R3.cer
```
