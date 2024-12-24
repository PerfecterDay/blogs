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


## Cipher
**第一步**：`Cipher` 对象使用 `getInstance` 工厂方法创建：
```
public static Cipher getInstance(String transformation);
public static Cipher getInstance(String transformation,String provider);
```
`transformation` 是一个字符串，用于描述对给定输入执行的操作（或操作集），以产生某些输出。 转换总是包括加密算法的名称（如 AES/DES），后面还可以加上模式和填充方案，如下形式：
+ `algorithm/mode/padding`, 如 "DES/CBC/PKCS5Padding"
+ `algorithm` ， 如 "AES"

Algorithm:
+ `AES`: 支持 128, 192, 和 256 bits.
+ `Blowfish`: The block cipher designed by Bruce Schneier.
+ `DES`
+ `DESede`: Triple DES Encryption (DES-EDE).
+ `ElGamal`: Asymmetric key encryption based on Diffie–Hellman key exchange.
+ `MARS`: A shared-key (symmetric) block cipher, supporting 128-bit blocks and variable key size created by IBM.
+ `PBEWith<digest>And<encryption>`
+ `RC2` and `RC4`: Variable-key-size encryption algorithms developed by Ron Rivest for RSA Data Security, Inc.
+ `RSA`: The RSA encryption algorithm as defined in PKCS #1.
+ `Seal`: A software-efficient stream cipher by Phil Rogaway, IBM.

Mode:
+ `ECB`: Electronic Codebook Mode, as defined in: The National Institute of Standards and Technology (NIST) Federal Information Processing Standard (FIPS) PUB 81, DES Modes of Operation,U.S. + Department of Commerce, Dec 1980.
+ `CBC`: Cipher Block Chaining Mode, as defined in FIPS PUB 81.
+ `CFB`: Cipher Feedback Mode, as defined in FIPS PUB 81.
+ `CTR`: Counter Mode, as defined in NIST SP 800-38A.
+ `OFB`: Output Feedback Mode, as defined in FIPS PUB 81.
+ `PCBC`: Plaintext Cipher Block Chaining, as defined by Kerberos.
+ `CTS`: Cipher Text Stealing Mode, as defined in RFC 2040.

Padding:
+ `NoPadding`: No padding.
+ `PKCS5Padding`: The padding scheme described in: RSA Laboratories, PKCS #5: Password-Based Encryption Standard, version 1.5, November 1993.
+ `ISO10126Padding`: The padding scheme required by JSR 105 & JSR 106.

**第二步**：使用 `getInstance` 获取的密码对象必须针对四种模式之一进行初始化，这四种模式在密码类中定义为最终整数常量：
1. `Cipher.ENCRYPT_MODE`-加密模式
2. `Cipher.DECRYPT_MODE`-解密模式
3. `Cipher.WRAP_MODE`-将密钥封装成字节，以便安全地传输密钥。
4. `Cipher.UNWRAP_MODE`-将先前封装的密钥解包为 `java.security.Key` 对象。

每个密码初始化方法都需要一个模式参数（opmode），并根据该模式初始化密码对象。 其他参数还包括密钥（key）或包含密钥的证书（certificate）、算法参数（params）和随机性来源（random）：
```
public void init(int opmode, Key key);
public void init(int opmode, Certificate certificate)
public void init(int opmode, Key key, SecureRandom random);
public void init(int opmode, Certificate certificate, SecureRandom random)
public void init(int opmode, Key key,AlgorithmParameterSpec params);
public void init(int opmode, Key key,AlgorithmParameterSpec params,SecureRandom random);
public void init(int opmode, Key key,AlgorithmParameters params)
public void init(int opmode, Key key,AlgorithmParameters params,SecureRandom random)
```

**第三步**：加密或解密数据，数据加密或解密可以一步完成（单部分操作），也可以多步完成（多部分操作）。 如果事先不知道数据有多长，或者数据太长无法一次性存储在内存中，多部分操作就很有用。
单步操作：
```
public byte[] doFinal(byte[] input);
public byte[] doFinal(byte[] input, int inputOffset,int inputLen);
public int doFinal(byte[] input, int inputOffset, int inputLen, byte[] output);
public int doFinal(byte[] input, int inputOffset, int inputLen, byte[] output, int outputOffset)
```

多步操作：
```
public byte[] update(byte[] input);
public byte[] update(byte[] input, int inputOffset, int inputLen);
public int update(byte[] input, int inputOffset, int inputLen,byte[] output);
public int update(byte[] input, int inputOffset, int inputLen,byte[] output, int outputOffset)
```
多部分操作必须由前面的 `doFinal` 方法之一（如果最后一步仍有剩余输入数据）或下面的 `doFinal` 方法之一（如果最后一步没有剩余输入数据）终止：
```
public byte[] doFinal();
public int doFinal(byte[] output, int outputOffset);
```
`Cipher` 的某些 `update` 和 `doFinal` 方法允许调用者指定用于加密或解密数据的输出缓冲区。 在这些情况下，必须传递一个足够大的缓冲区，以容纳加密或解密操作的结果。
Cipher 中的以下方法可用于确定输出缓冲区的大小： `public int outOutputSize(int inputLen)` 。

调用 `doFinal` 会将 `Cipher` 对象重置为通过调用 `init` 初始化时的状态。 也就是说， `Cipher` 对象被重置后，可以继续加密或解密（取决于调用 `init` 时指定的操作模式）更多数据。

除了加密与解密之外， `Cipher` 还可以对密钥进行 `wrap/unwrap` 以实现密钥从一个地方到另一个地方的安全传输。由于 `wrap/unwrap` 应用程序接口可直接处理密钥对象，因此编写代码更加方便。 这些方法还可以安全地传输基于硬件的密钥。

要对密钥进行 `wrap` ，首先要初始化 `Cipher` 对象为 `WRAP_MODE` 模式，然后调用下面方法：
```
public final byte[] wrap(Key key);
```
如果您要将已封装的密钥字节（调用 `wrap` 的结果）提供给其他人，由其进行 `unwrap` ，请务必同时发送接收者进行解包所需的附加信息：
+ 密钥使用的算法 ， `Key` 接口的 `public String getAlgorithm();` 方法可以获取 `Key` 使用的算法
+ 密钥的类型(SECRET_KEY, PRIVATE_KEY,  PUBLIC_KEY)

有了以上信息，接受者首先要为 `UNWRAP_MODE` 初始化一个 `Cipher` 对象，然后调用下面的方法：
```
public final Key unwrap(byte[] wrappedKey,String wrappedKeyAlgorithm,int wrappedKeyType);
```
`wrappedKeyType` 是 `SECRET_KEY/PRIVATE_KEY/PUBLIC_KEY`中的一种。

## KeyGenerator
`KeyGenerator` 用于生成对称算法的密钥。使用下述方法获取 `KeyGenerator` 对象：
```
public static KeyGenerator getInstance(String algorithm);
public static KeyGenerator getInstance(String algorithm,String provider);
```

`algorithm` 可以使用以下列表中的值：
+ `AES`
+ `Blowfish`
+ `DES`
+ `DESede`
+ `HmacMD2`
+ `HmacMD5`
+ `HmacSHA1`
+ `HmacSHA224`
+ `HmacSHA256`
+ `HmacSHA384`
+ `HmacSHA512`
+ `MARS`
+ `RC2`
+ `RC4`
+ `Seal`

生成密钥有两种方式：一种是独立于算法的方式，另一种是特定于算法的方式。 两者唯一的区别在于 `KeyGenerator` 的初始化。
1. Algorithm-Independent Initialization
```
public void init(SecureRandom random);
public void init(int keysize);
public void init(int keysize, SecureRandom random);
```

2. Algorithm-Specific Initialization
```
public void init(AlgorithmParameterSpec params);
public void init(AlgorithmParameterSpec params,SecureRandom random);
```

最后一步，生成 `Key` :
```
public SecretKey generateKey();
```

## SecretKeyFactory



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

