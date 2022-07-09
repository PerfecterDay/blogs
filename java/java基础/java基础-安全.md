# 安全相关
{docsify-updated}

### 安全管理器与访问权限
一旦某个类被加载到虚拟机中，并由检验器检查过之后， Java 平台的第二种安全机制就会启动，这个机制就是**安全管理器**。安全管理器是一个负责控制具体操作是否允许执行的类。安全管理器负责检查的操作包括以下内容：
+ 创建一个新的类加载器
+ 退出虚拟机
+ 使用反射访问另一个类的成员
+ 访问本地文件
+ 打开 socket 连接
+ 启动打印作业
+ 访问系统剪贴板
+ 访问 AWT 事件队列
+ 打开一个顶层窗口

整个 Java 类库中还有许多其他类似的检查。运行Java 程序时，默认是不安装安全管理器的，这样所有的操作都是被允许的。

Java的安全策略建立了**代码来源**和**访问权限集**之间的映射关系。
<center><img src="pics/code-permission.png" width="30%"></center>

代码来源(code source)是由一个代码位置和一个证书集指定的。代码位置指定了代码的来源。例如，远程applet代码的代码位置是下载 applet 的 HTTP URL，位于JAR文件中的代码的代码位置是该文件的URL。证书的目的是要由某一方来保障代码没有被篡改过。

权限（ permission ）是指由安全管理器负责检查的任何属性。Java 平台支持许多访问权限类，每个类都封装了特定权限的详细信息。如： `FilePermission permission = new FilePermission("/tmp/*","read,write");` 还有许多类似 `FilePermission` 的类。
更为重要的是 `Policy` 类默认实现了可以从**访问权限文件**中读取权限。在权限文件中，同样的权限表示为：`permission java.io.FilePermission "/tmp/*,"read,write"`; 这样的话，我们就可以直接在权限文件中配置一些权限控制。

每个类(Class对象)都有一个保护域，它是一个用于封装类的代码来源和权限集合的对象。当 `SecurityManager` 类需要检查某个权限时，它要查看当前位于调用堆栈上的所有方法的类，然后它要获得所有类的保护域，并且询问每个保护域，其权限集合是否允许执行当前正在被检查的操作。如果所有的域都同意，那么检查得以通过。否则，就会抛出一个 `SecurityException` 异常。

#### 安全策略文件
策略管理器要读取相应的策略文件，这些文件包含了将代码来源映射为权限的指令。可以将策略文件的配置安装在标准位置上。 默认情况下，有两个位置可以配置策略文件：
+ Java 平台主目录的 `java.policy` 文件。
+ 用户主目录的 `.java.policy` 文件（注意文件名前面的圆点）。

也可以在 `java.security` 配置文件中修改这些文件的位直，默认位直设定为：
    ```
    policy.url.1=file:${java.home}/conf/security/java.policy
    policy.url.2=file:${user.home}/.java.policy 
    ```
系统管理员可以修改 `java.security` 文件，并可以指定驻留在另外一台服务器上并且用户无法修改的策略 URL。但是通常情况下，我们不愿意经常去修改这些默认标准文件，而更愿意为自己的程序单独建立一个策略配置文件 myapp.policy，要应用这个策略文件有两种方法：
1. 在 main 方法启动时，设置系统属性：`System.setProperty("java.security.policy","myapp.policy");`
2. 启动程序时，使用 -D 设置：`java -Djava.security.policy=myapp.policy myapp`，这样自定义的策略文件和标准策略文件会一起生效，如果使用`java -Djava.security.policy==myapp.policy myapp`则会忽略标准策略文件

另外，正如前文所说，默认时不安装安全管理器的，因此在安装安全管理器之前，是看不到策略文件的作用的。可以在 main 方法中调用 `System.setSecurityManager(new SecurityManager());` 设置安全管理器。也可以在启动时设置：`java -Djava.security.policy==myapp.policy -Djava.security.manager myapp`

策略文件包含一系列 grant 项。每一项都具有以下的形式：
```
grant codebase <url>{
    permission <permissionClassName> <targetName>, <actionList>;
    permission <permissionClassName> <targetName>, <actionList>;
    permission <permissionClassName> <targetName>, <actionList>;
	...
}
```
如果 url 以 “/” 结束，则被视为一个目录，否则视为一个jar 的名字。


### JAAS(Java Authentication and Authorization)
Java认证和授权服务包含两部分：
+ 认证部分主要负责确定程序使用者的身份
+ 授权将各个用户映射到相应的权限

Java的安全基础架构主要有以下特点：
+ 实现是互相独立的，java中的安全服务是由 provider 提供的，这些 provider 通过标准接口插入到 java 平台，应用程序可以直接请求这些 provider 提供的安全服务。
+ provider 可跨应用程序进行互操作。具体而言，应用程序未绑定到特定的 provider ，并且 provider 也未绑定到特定的应用程序。
+ 可扩展的， java 中提供了一些基础的安全服务实现，应用程序可以对这些服务进行扩展。且 java 平台也支持安装自定义的 provider

### java SPI(Servicec provider Interface)机制
是JDK内置的一种服务提供发现机制。SPI是一种动态替换发现的机制， 比如有个接口，想运行时动态的给它添加实现，你只需要添加一个实现。我们经常遇到的就是java.sql.Driver接口，其他不同厂商可以针对同一接口做出不同的实现，mysql和postgresql都有不同的实现提供给用户，而Java的SPI机制可以为某个接口寻找服务实现。

当服务的提供者提供了一种接口的实现之后，需要在classpath下的META-INF/services/目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）的META-INF/services/中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务实现的工具类是：java.util.ServiceLoader。  

    ServiceLoader<SPITest>  spiTests = ServiceLoader.load(com.company.SPITest.class);
            for (SPITest spiTest : spiTests) {
                System.out.println(spiTest.test());
            }

### 安全加密算法
1. 消息摘要
   
   消息摘要是数据块的指纹。可以基于任意长的数据块计算出一个定长的不可逆的数字序列，通常称为指纹。人们希望，任何两个不同的输入数据，计算得到的指纹也不一样。当然这是不可能的，因为输入的数据有无数种可能，而定长的指纹序列是可数的，与长度有关。但是，通过设计合适的算法，人们可以使得伪造一个具有相同指纹得数据块在计算复杂度上是不可能。

   Java实现了MD5、SHA-1、SHA-256、SHA-384、SHA-512等摘要算法。 `MessageDigest` 类是创建封装了指纹算法得对象的工厂，它的静态方法 `getInstance(String algorithm)` 返回继承自 `MessageDigest` 的一个算法对象。 `MessageDigest`既是一个工厂，也是消息摘要算法的超类。

   总结来说，消息摘要就是能提取出一个数据块的指纹，任何对数据块的篡改都能导致指纹的不匹配，所以消息摘要具有防篡改功能。

2. 消息签名

	假如要传送的消息被截获，并且攻击者篡改了内容后又重新生成了指纹（重放攻击），接收者是无法识别出这条消息被篡改了的。基于这点，发送者可以用自己的私钥对消息和指纹进行签名，然后将公钥告知接收方，接收方收到消息后，用公钥进行验签。如果验签成功，就能说明这条消息确实是由发送者发送的，因为没有人可以伪造发送方的私钥。但是，在这里有一个很重要的环节是你必须确保你得到的公钥确实是发送者自己的公钥。因为任何人都可以自己生成一对公私钥对。一个可用的方案是我们找一个权威机构来对公钥进行认证，对真实可信的公钥就用权威机构的私钥进行签名，并且权威机构将自己的公钥公布在网上。这样接收方在确认收到的公钥是否可信时，就能直接用权威机构的公钥去验签，验签成功的公钥就是可信的公钥。

3. 对称加密
   
   上述摘要、签名算法只能确认消息来源的可靠性，具体的说，他们能保证消息是特定的发送者发出的而且没有被篡改。但是，消息的内容本身没有被加密（当然如果使用公钥加密传送内容也是可以的）。通常使用对称加密来加密传送的消息。
   Java种的 `Cipher` 类是所有加密算法的超类，通过调用 `getInstance(String algorithm)` 或者 `getInstance(String algorithm,String provider)` 来获得一个加密算法对象。一个`Cipher`可以有多种使用方式，比如加密、解密等，Java中定义了4种模式：
   1. Cipher.ENCRYPT_MODE
   2. Cipher.DECRYPT_MODE
   3. Cipher.WRAP_MODE
   4. Cipher.UNWRAP_MODE
   在使用 Cipher 之前必须调用它的 `init(...)` 方法初始化设置好密钥（Key类来表示）和模式或者其它参数，不同的加密算法要求初始化的参数不同。调用`update()`方法将要加密的数据传递给 Cipher，最后调用 `doFinal()`方法完成加密并返回加密后的字节数据。

   生成密钥：
   1. 为加密算法获取 `KeyGenerator`，通过`KeyGenerator.getInstance("AES")`来获得。
   2. 用随机源来初始化 `KeyGenerator`。
   3. 调用 `KeyGenerator` 的 `generateKey()` 生成密钥
   ```
    Cipher cipher = Cipher.getInstance("AES");
	KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
	SecureRandom random = new SecureRandom();
	keyGenerator.init(random);
	Key key = keyGenerator.generateKey();
	cipher.init(Cipher.ENCRYPT_MODE,key);
   ```

4. 非对称加密
   
   非对称加密在需要一对公私钥对，所以在生成密钥时与对称加密算法有一点差别。
   ```
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