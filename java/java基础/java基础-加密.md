# Java 加密
{docsify-updated}

> https://www.ibm.com/docs/en/sdk-java-technology/8?topic=class-creating-cipher-object

## 加密相关的概念

### 加密与解密
加密是获取数据（称为明文）和一个短字符串（密钥），并生成对不知道密钥的第三方毫无意义的数据（密文）的过程。 解密则是一个相反的过程：即获取密文和一串简短的密钥，然后生成明文。

### 基于密码的加密 (Password-Based Encryption)
基于密码的加密（PBE）从密码中提取加密密钥。 为了让攻击者在从密码到密钥的过程中耗费大量时间，大多数 PBE 实现都会在创建密钥时混入一个随机数（称为 "盐"）。

### Cipher
加密和解密是通过 `Cipher` 完成的。  `Cipher` 是能够根据加密方案（算法）进行加密和解密的对象。

### Key Agreement
密钥协议是一种协议，通过它，两方或多方可以建立相同的加密密钥，而无需交换任何秘密信息。

### Message Authentication Code
信息验证码 (Message Authentication Code-MAC) 提供了一种基于密钥检查通过不可靠介质传输或存储的信息完整性的方法。 通常，信息验证码用于共享密钥的双方之间，以验证双方之间传输的信息。  
基于加密散列函数的 MAC 机制称为 HMAC。 HMAC 可与任何加密散列函数（如 MD5 或 SHA-1）和共享密钥结合使用。 HMAC 由 RFC 2104 规定。

### 消息摘要-防篡改

消息摘要是数据块的指纹。可以基于任意长的数据块计算出一个定长的不可逆的数字序列，通常称为指纹。人们希望，任何两个不同的输入数据，计算得到的指纹也不一样。当然这是不可能的，因为输入的数据有无数种可能，而定长的指纹序列是可数的，与长度有关。但是，通过设计合适的算法，人们可以使得伪造一个具有相同指纹得数据块在计算复杂度上是不可能。

Java实现了
+ MD5
+ SHA-1
+ SHA-256
+ SHA-384
+ SHA-512

等摘要算法。 总结来说，消息摘要就是能提取出一个数据块的指纹，任何对数据块的篡改都能导致指纹的不匹配，所以消息摘要具有**防篡改**功能。

#### MessageDigest
`MessageDigest` 类是创建封装了指纹算法得对象的工厂，它的静态方法 `getInstance(String algorithm)` 返回继承自 `MessageDigest` 的一个算法对象。 `MessageDigest`既是一个工厂，也是消息摘要算法的超类。

#### 消息签名-防伪造

假如要传送的消息被截获，并且攻击者篡改了内容后又重新生成了指纹（重放攻击），接收者是无法识别出这条消息被篡改了的。基于这点，发送者可以用自己的私钥对消息和指纹进行签名，然后将公钥告知接收方，接收方收到消息后，用公钥进行验签。如果验签成功，就能说明这条消息确实是由发送者发送的，因为没有人可以伪造发送方的私钥。但是，在这里有一个很重要的环节是你必须确保你得到的公钥确实是发送者自己的公钥。因为任何人都可以自己生成一对公私钥对。一个可用的方案是我们找一个权威机构来对公钥进行认证，对真实可信的公钥就用权威机构的私钥进行签名，并且权威机构将自己的公钥公布在网上。这样接收方在确认收到的公钥是否可信时，就能直接用权威机构的公钥去验签，验签成功的公钥就是可信的公钥。
常用的数字签名算法包括:
+ DSA
+ RSA
+ HMAC-SHA

SHA-256 和 HMAC-SHA256 都是加密散列函数，但它们的用途不同。 SHA-256 是一个单向函数，它接收输入信息并产生一个固定大小的输出（称为散列），该输出对输入信息是唯一的。 另一方面，HMAC-SHA256 是一种密钥散列函数，它在应用 SHA-256 散列函数之前将密钥与输入信息结合在一起。 这就提供了一个额外的安全层，确保只有获得秘钥的人才能验证信息的完整性和真实性。 HMAC-SHA256 常用于各种协议和应用中的信息验证和完整性检查，包括数字签名、VPN 和安全信息传送。

#### 对称加密

上述摘要、签名算法只能确认消息来源的可靠性，具体的说，他们能保证消息是特定的发送者发出的而且没有被篡改。但是，消息的内容本身没有被加密（当然如果使用公钥加密传送内容也是可以的）。通常使用对称加密来加密传送的消息。

算法名称是一个字符串，比如“ AES ”或者“ DES/CBC/PKCS5Padding ” 。DES ，即数据加密标准，是一个密钥长度为 56 位的古老的分组密码 。 DES 加密算法在现在看来已经是过时了，因为可以用穷举法将它破译（[参见该网页中的例子](http://w2.eff.org/Privacy/Crypto/Crypto_misc/DESCracker) ） 。 更好的选择是采用它的后续版本，即高级加密标准AES。

AES目前支持三种模式：AES-128 (128 bits), AES-192 (192 bits), and AES-256 (256 bits)。

Java种的 `Cipher` 类是所有加密算法的超类，通过调用 `getInstance(String algorithm)` 或者 `getInstance(String algorithm,String provider)` 来获得一个加密算法对象。一个`Cipher`可以有多种使用方式，比如加密、解密等，Java中定义了4种模式：

1. Cipher.ENCRYPT_MODE
2. Cipher.DECRYPT_MODE
3. Cipher.WRAP_MODE
4. Cipher.UNWRAP_MODE

在使用 Cipher 之前必须调用它的

1. `init(...)` 方法初始化设置好密钥（ `Key` 类来表示）和模式或者其它参数，不同的加密算法要求初始化的参数不同
2. 调用`update()`方法将要加密的数据传递给 Cipher
3. 最后调用 `doFinal()`方法完成加密并返回加密后的字节数据。

生成密钥：

1. 为加密算法获取 `KeyGenerator`，通过`KeyGenerator.getInstance("AES")`来获得。
2. 用随机源或指定长度来初始化 `KeyGenerator`。
3. 调用 `KeyGenerator` 的 `generateKey()` 生成密钥

```java
KeyGenerator keyGenerator = KeyGenerator.getInstance(cipher);
keyGenerator.init(keySize);
Key key = keyGenerator.generateKey();

Cipher cipher = Cipher.getInstance("AES");
cipher.init(Cipher.ENCRYPT_MODE, key);
return new String(Base64.getEncoder().encode(cipher.doFinal(val.getBytes())));
```

#### 非对称加密

非对称加密在需要一对公私钥对，所以在生成密钥时与对称加密算法有一点差别。

```java
Cipher rsaCipher = Cipher.getInstance("RSA");
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
SecureRandom secureRandom = new SecureRandom();
keyPairGenerator.initialize(2048,secureRandom);
KeyPair keyPair = keyPairGenerator.generateKeyPair();
Key publicKey = keyPair.getPublic();
Key privateKey = keyPair.getPrivate();
rsaCipher.init(Cipher.UNWRAP_MODE,publicKey);
byte[] wrapKey = rsaCipher.wrap(aeskey);
```

### SSL证书主流的格式

仅在X.509证书中就有几种编码格式和扩展名。你可能见过.pem、.crt、.p7b、.der、pfx等等的证书。如果我们问是否所有的扩展名都是一样的，答案是既是又不是。所有这些证书都是一样的，但有不同的编码标准。所有这些都取决于你的应用程序，或操作系统接受的编码类型。

<center><img src="pics/x509.png" width="70%"></center>

SSL证书格式主要有公钥证书格式标准X.509中定义的PEM和DER、公钥密码标准PKCS中定义的PKCS#7和PKCS#12、Java环境专用的JKS。

PEM是基于Base64编码的证书格式，扩展名包括pem、crt和cer。Linux系统使用crt，Windows系统使用cer。PEM证书通常将根证书、中间证书和用户证书分开存放，主要用于Apache和Nginx。

DER是基于二进制编码的证书格式，扩展名包括der、crt和cer。DER证书相当于PEM证书的二进制版本，主要用于Java平台。

PKCS#7是基于Base64编码的证书格式，扩展名包括p7b和p7c。PKCS#7证书通常将根证书和中间证书合并存放，将用户证书单独存放。PKCS#7证书不包含私钥，主要用于Tomcat和Windows Server。

PKCS#12是基于二进制编码的证书格式，扩展名包括pfx和p12。PKCS#12证书通常将根证书、中间证书、用户证书和私钥合并存放并设置密码，主要用于Windows Server。

JKS是是基于二进制编码的证书格式，扩展名是jks。JKS证书通常将根证书、中间证书、用户证书和私钥合并存放并设置密码，主要用于Java Web Server。

证书颁发机构CA签发的证书通常是PEM格式或PKCS#7格式，而PKCS#12格式和JKS格式的证书需要进行证书格式转换才能得到。我们可以通过OpenSSL、Keytool或在线证书转换工具等方式将PEM格式或PKCS#7格式的证书转换为我们需要的其他格式。

#### SSL证书格式转换方法

常见SSL证书主要的文件类型和协议有: PEM、DER、PFX、JKS、KDB、CER、KEY、CSR、CRT、CRL、OCSP、SCEP等。而最常见的SSL证书格式之间的转换有：

1. 将JKS转换成PFX  
	可以使用Keytool工具，将JKS格式转换为PFX格式。
	```
		keytool -importkeystore -srckeystore D:\server.jks -destkeystore D:\server.pfx -srcstoretype JKS -deststoretype PKCS12
	```
2. 将PFX转换为JKS  
	可以使用Keytool工具，将PFX格式转换为JKS格式。
	```
	keytool -importkeystore -srckeystore D:\server.pfx -destkeystore D:\server.jks -srcstoretype PKCS12 -deststoretype JKS
	```
3. 将PEM/KEY/CRT转换为PFX  
	使用OpenSSL工具，可以将密钥文件KEY和公钥文件CRT转化为PFX文件。
	```
	openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt
	```
4. 将PFX转换为PEM/KEY/CRT  
	```
	openssl pkcs12 -in server.pfx -nodes -out server.pem
	openssl rsa -in server.pem -out server.key
	```

### keytool 工具

1. 生成密钥对： `keytool -genkeypair -keystore alice.certs -alias alice`
2. 将生成的密钥导出为一个证书： `keytool -exportcert -keystore alice.certs -alias alice -file alice.cer`
3. 打印证书内容： `keytool -printcert -file alice.cer`
4. `keytool -list -keystore jre/lib/security/cacerts`
5. 导入证书到密钥库： `keytool -importcert -keystore bob.certs -alias alice -file alice.cer` 绝对不妥将你并不完全信任的证书导入到密钥库中 。 一旦证书添加到密钥库中，使用密钥库的任何程序都会认为这些证书可以用来对签名进行校验。

### Https 双向认证

> https://help.aliyun.com/document_detail/160093.html



JCE API 要求并使用一套算法、算法模式和填充方案的标准名称。 本规范将以下名称确定为标准名称。 它是对《Java Cryptography Architecture API Specification & Reference》附录 A 中定义的标准名称列表的补充。 请注意，算法名称不区分大小写。

### Cipher

Algorithm
The following names can be specified as the algorithm component in a transformation when requesting an instance of Cipher:
+ AES: 支持 128, 192, 和 256 bits.
+ Blowfish: The block cipher designed by Bruce Schneier.
+ DES
+ DESede: Triple DES Encryption (DES-EDE).
+ ElGamal: Asymmetric key encryption based on Diffie–Hellman key exchange.
+ MARS: A shared-key (symmetric) block cipher, supporting 128-bit blocks and variable key size created by IBM.
+ PBEWith<digest>And<encryption>
+ RC2 and RC4: Variable-key-size encryption algorithms developed by Ron Rivest for RSA Data Security, Inc.
+ RSA: The RSA encryption algorithm as defined in PKCS #1.
+ Seal: A software-efficient stream cipher by Phil Rogaway, IBM.

Mode
+ The following names can be specified as the mode component in a transformation when requesting an instance of Cipher:
+ ECB: Electronic Codebook Mode, as defined in: The National Institute of Standards and Technology (NIST) Federal Information Processing Standard (FIPS) PUB 81, DES Modes of Operation,U.S. + Department of Commerce, Dec 1980.
+ CBC: Cipher Block Chaining Mode, as defined in FIPS PUB 81.
+ CFB: Cipher Feedback Mode, as defined in FIPS PUB 81.
+ CTR: Counter Mode, as defined in NIST SP 800-38A.
+ OFB: Output Feedback Mode, as defined in FIPS PUB 81.
+ PCBC: Plaintext Cipher Block Chaining, as defined by Kerberos.
+ CTS: Cipher Text Stealing Mode, as defined in RFC 2040.


Padding
The following names can be specified as the padding component in a transformation when requesting an instance of Cipher:
+ NoPadding: No padding.
+ PKCS5Padding: The padding scheme described in: RSA Laboratories, PKCS #5: Password-Based Encryption Standard, version 1.5, November 1993.
+ ISO10126Padding: The padding scheme required by JSR 105 & JSR 106.