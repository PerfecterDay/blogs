# JNI
{docsify-updated}


chainMaker 的demo中 `ChainmakerX509CryptoSuite` 中会加载
```
Class clazz = Class.forName("io.netty.handler.ssl.OpenSsl");
```

OpenSsl 会依赖原生的叫 tcnative 的东西，实例化 OpenSsl 时会尝试加载这些本地库，这些库的名字和运行时（JDK版本）是相关的，比如17是下边这些名字：
```
netty_tcnative_osx_aarch_64
netty_tcnative_aarch_64
netty_tcnative
```

1.8 是：
```
netty_tcnative_osx_x86_64
netty_tcnative_x86_64
netty_tcnative
```

如果强行通过参数修改运行时的配置：
```
-Dos.name=osx
-Dos.arch=x86_64
```

可以达到让类加载器加载到路径下的 `libnetty_tcnative_osx_x86_64.jnilib`，但是这个jnilib 文件是和运行时相关的，所以 17 会报不兼容错误：
```
'/private/var/folders/jn/2_tytdt13111lvxr205tjg840000gn/T/libnetty_tcnative_osx_x86_64223919015082559059.dylib' (mach-o file, but is an incompatible architecture (have 'x86_64', need 'arm64e' or 'arm64')), '/System/Volumes/Preboot/Cryptexes/OS/private/var/folders/jn/2_tytdt13111lvxr205tjg840000gn/T/libnetty_tcnative_osx_x86_64223919015082559059.dylib' (no such file)
```
 
如果不加VM参数， JDK17正常加载的是 ：`libnetty_tcnative_osx_aarch_64.jnilib` 名字的文件。

解决方案，找到与平台（mac M1）/JDK版本/ netty 版本对应的依赖：
```<dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-tcnative-boringssl-static</artifactId>
        <version>2.0.53.Final</version>
        <classifier>osx-aarch_64</classifier>
    </dependency>
```

如果版本不对，会报 `java.lang.LinkageError: Possible multiple incompatible native libraries on the classpath` 错误，提示方法缺失或不匹配
