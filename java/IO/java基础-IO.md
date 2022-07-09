# java基础-IO
{docsify-updated}

## 老的File类
`File` 类看上去是指代文件，其实它既能代表一个特定文件，也能代表一个目录。

### 常见方法
+ `String[] list()`: 返回所有文件名的字符串数组
+ `String[] list(FileNameFilter filter)`: 返回 `FileNameFilter` 过滤后的字符串数组
+ `File[] listFiles()`: 返回所有的文件数组
+ `File[] listFiles(FilenameFilter filter)`: 返回 `FilenameFilter` 过滤后的文件数组
+ `String getAbsolutePath()` :获取绝对路径
+ `String getName()` :获取名字
+ `File getParent()` :获取父目录
+ `long length()` :获取目录/文件大小
+ `long lastModified()` :获取最后修改时间
+ `boolean canExecute()` :是否可执行
+ `boolean canRead()` :是否可读
+ `boolean canWrite()` :是否可写
+ `boolean createNewFile()` :创建新文件当 `File` 代表一个文件时
+ `boolean mkdir()` :创建新目录当 `File` 代表一个目录时

假如 `File` 指代的是一个目录，那么就可以使用 `list()` 方法获取目录下的文件列表，如果想获取所有文件列表，使用不带参数的 `list()` 的方法即可；如果想获得一个受限列表，那么就要使用“目录过滤器”了。 `FilenameFilter` 接口的 `boolean accept(File dir,String name)` 返回 `true` 的才会返回到数组中。 `listFiles` 同理。

```
public class FileList {
    public static void main(String[] args) {****
        File f = new File("/usr/local/etc");
        File[] allFiles = f.listFiles();
        File[] filterFiles = f.listFiles((dir,name)->{
            return name.contains("a"); 
        });
        for (File allFile : allFiles) {
            System.out.print(allFile.getName()+", ");
        }
        System.out.println();
        for (File filterFile : filterFiles) {
            System.out.print(filterFile.getName()+", ");
        }
        System.out.println();
        String[] allNames = f.list();
        String[] filterNames = f.list((dir,name)-> {
                return name.contains(".");
        });
        for (String file : allNames) {
            System.out.print(file+", ");
        }
        System.out.println();
        for (String file : filterNames) {
            System.out.print(file+", ");
        }
    }
}
```

## 新的 Path 和 Files 类

## IO 流
流按照不同的方式可以分为如下几类：
+ 输入流：从外设/网络等流入程序内存的流，只能从其中读数据，不能写
+ 输出流：从程序内存输出到外设/网络等的流，只能向其中写数据，不能读
+ 节点流：可以直接从/向一个特定的IO设备（磁盘、网络等）读写数据的流，称为节点流，也称为低级流
+ 处理流：对于一个已经存在的流进行连接或封装，通过封装后流来实现数据读写的功能，也称为高级流；关闭最上层的处理流，系统会自动关闭该处理流包装的节点流

<center><img src="pics/stream-2.png" width=20% /></center> 

字节流/字符流：两种流操作的数据单元不同，字节流按字节处理数据，而字符流按16位字符处理数据
<center><img src="pics/stream-1.png" width=60%/></center> 

### 输入输出流体系
<center><img src="pics/stream-framework.png" width=60%></center> 
粗体标出的类代表节点流，必须直接与指定的物理节点关联；斜体标出的类代表抽象基类，无法直接创建实例。  
访问数组和字符串的流看似没什么用，因为数组和字符串本来就可以在内存中直接访问，为啥还要使用流来读写呢？当你的API接口只接受IO流参数，但是你又想用字符串或数组去调用这个API时，就可以使用这种流来调用。

#### InputStream 和 Reader
InputStream/Reader 里包含以下三个读方法：  
+ `int read()`: 从输入流中读取一个字节，返回读取到的字节数据
+ `int read(byte[]/char[] b)`: 从输入流中读取数据到字节/字符数组b中，最多读取b.length个字节，返回实际读取的字节/字符数
+ `int read(byte[] b, int off, int len)`: 从输入流中最多读取len个字节/字符存入字节/字符数组b中，从off位置开始存放，返回实际读取的字节/字符数
除此之外，它们还支持如下几个方法来移动记录指针：
+ `boolean markSupported()`: 判断流是否支持记录标记
+ `synchronized void mark(int readlimit)`: 在记录指针（读取位置的记录，读取时会从这个位置开始读）当前位置记录一个标记
+ `synchronized void reset()`: 将记录指针重新定位到上一个记录标记的位置
+ `long skip(long n)`: 将记录指针向前移动 n 个字节/字符

#### OutputStream 和 Writer
OutputStream/Writer 也提供如下三个写方法：
+ `void write(int b)`: 将字节/字符 b写入到流中
+ `void write(byte[]/char[] buf)`: 将字节/字符数组buf中的数据写入输出流中
+ `void write(byte[]/char[] buf, int off, int len)`: 将字节/字符数组buf中从off位置开始长度为len的数据写入输出流中
因为字符流直接以字符为操作单位，因此 Writer 也支持直接写字符串：
+ `void write(String str)`: 将字符串 str 中的字符写入输出流中
+ `void write(String str,int off, int len)`: 将字符串 str 中从 off 位置开始，长度为len的字符写入输出流中

#### 标准输入输出
Java的标准输入/输出分别通过 `System.in/System.out` 来代表，默认情况下分别代表键盘和显示器，当程序从 System.in 读数据时，实际上是从键盘读取输入；当程序通过 System.out 来输出数据时，数据会显示在屏幕上。
System类还提供了三个重定向标准输入/输出的方法：
+ `static void setIn(InputStream in)`:  重定向标准输入流
+ `static void setOut(PrintStream out)`: 重定向标准输出流
+ `static void setErr(PrintStream err)`: 重定向标准错误输出流

获取用户输入的几种方法：
1. 使用 BufferedReader 包装标准输入,一行一行的读取：`String line = (new BufferedReader(new InputStreamReader(System.in))).readLine();`
2. 使用 Scanner(System.in)：`Integer input = new Scanner(System.in).nextInt();`

#### JVM读取其它进程的数据
我们知道使用 `Runtime` 类的 `Process exec(String command)` 方法可以启动一个进程，并返沪一个 Process 对象， 该 Process 对象代表由 Java 启动的子进程，Process 提供了三个方法用于程序和其子进程进行通信：
+ `OutputStream getOutputStream()`: 获取子进程的输出流，实际对应子进程的输入流，对本进程来说是输出流，用于向子进程写数据
+ `InputStream getInputStream()`: 获取子进程的输入流，实际上对应子进程的输出流，对本进程来说是输入流，用于读取子进程的输出数据
+ `InputStream getErrorStream()`: 获取子进程的错误输入流，实际上对应子进程的错误输出流，对本进程来说是输入流，用于读取子进程的错误输出数据

#### RandomAccessFile
`RandomAccessFile` 功能非常强大，能够支持随机的读写数据，就是能够随机的从指定位置开始读/写数据，也可以跳过某些数据的读写。`RandomAccessFile`提供两种构造方法来生成对象：
1. RandomAccessFile(File file, String mode)
2. RandomAccessFile(String name, String mode)

mode 参数指定 RandomAccessFile 的访问模式，该参数有如下四个值：
   + "r": 以只读方式打开指定文件
   + "rw": 以读写方式打开指定文件，如果文件不存在，则尝试创建文件
   + "rws": 以读写方式打开指定文件，相对于 "rw"模式，还要求对文件内容或元数据的每个更新都同步写入到底层存储设备。
   + "rwd": 以读写方式打开指定文件，相对于 "rw"模式，还要求对文件内容的每个更新都同步写入到底层存储设备。

打开一个 `RandomAccessFile` 对象后，可以使用下述两个方法来操纵文件记录指针：
   + `long getFilePointer()`: 获取文件记录指针的位置
   + `void seek(long pos)`: 将文件记录指针定位到指定的 pos 位置。

RandomAccessFile 还包含类似于了 InputStream/OutputStream 中的三种 read()/write() 方法，用法是完全一样的，另外还包含了系列方便的 readXXX 和 writeXXX 方法。