## 包和访问权限
{docsify-updated}

- [包和访问权限](#包和访问权限)
	- [package、import和import static](#packageimport和import-static)
	- [访问权限控制](#访问权限控制)


### package、import和import static
为了避免众多开发者使用相同的类名而导致类的冲突，Java引入了包机制，允许在类前面加上前缀，提供多层命名空间，用于解决类的命名冲突、类文件管理等问题。

Java允许把一组功能相关的类放在一个包下，从而组成逻辑上的类库单元。如果希望把类放到指定的包下，应该在java 源程序文件的第一行写下：
`package packagename`
引入包之后，源文件中定义的类都属于这个包。完整类名是包名+类名。如果其它类中要使用这个类，也需要使用包名+类名，除非在同一个包内。没有 package 语句的的源文件中的类位于默认包中。 package 语句必须作为第一条非注释性语句出现在源文件中，且一个源文件只能有一条 package 语句。

使用下列命令编译带包命令的源文件：
`javac -d . Hello.java`
这样会在当前目录下生成 `packagename\Hello.class` 的结构。假如不使用 -d 选项，则会在当前目录下生成 Hello.class 文件。此时，若用 `java Hello` 运行程序会报错。

因为 JVM 在装载 packagename.Hello.class时，会依次在 CLASSPATH 路径下查找 packagename 指定的路径层次下查找 Hello.class。同一包内的两个类可以放在不同文件夹下，如 lee.Person 和 lee.PersonTest 两个类，它们完全可以一个放在 C 盘，一个放在 D 盘，只要所在目录位于 CLASSPATH 下即可（IDEA中 test 和 src不在同一个目录，却在同一个包中）。

引入包之后，如果想使用不在同一个包中的其他类，必须使用包名+类名的形式：
`lee.Person p = new lee.Person();`
为了简化编程，Java引入了 import 语句， import 语句可以导入指定包下的某个类或所有类。 import 语句需要出现在 package 语句之后、类定义之前。 import 中的 \* 只能代表类，不能代表包。就是说如果引用指定包下的 \*，不能导入该包中子包下的类。 

`import static` 可以导入指定类中的静态成员或方法。使用 import 导入类后，可以省略包名，使用 `import static` 导入后，连类名都可以省略了。

### 访问权限控制
<center><img src="pics/access-control.jpg" alt="" width=60%></center>

类中的**实例方法**可以访问该类的**所有对象的所有域**，即使是 private 的域。也就是说，类A有两个对象a1,a2，以及一个方法 test。在 test 方法中，可以访问任意A对象a1,a2的私有域，而不管test是在哪个对象上调用的，但是不能访问其它类型对象的私有域。也就是说，方法的访问权限范围是与类绑定的而不是对象。