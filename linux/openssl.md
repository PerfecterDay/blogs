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
  - [测试TLS](#测试tls)
    - [证书验证](#证书验证)
    - [域名验证](#域名验证)

Mozilla的证书：
> https://hg.mozilla.org/releases/mozilla-beta/file/tip/security/nss/lib/ckfw/builtins/certdata.txt

Curl的证书：
> https://curl.se/docs/caextract.html

### 命令
`openssl version -a` : 查看版本信息，`OPENSSLDIR` 中存放了openssl配置的路径
`openssl help` : 


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
1. 创建配置文件 root-ca.conf(windows 注意文件编码问题： configuration file routines:def_load_bio:missing equal sign)

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
2. 创建文件夹
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

3. 创建CA
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

4. 使用签发证书
   

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
