#范型
{docsify-updated}

- [范型](#范型)
	- [范型类](#范型类)
	- [类型变量的限定与范型通配符](#类型变量的限定与范型通配符)
	- [范型方法](#范型方法)
	- [泛型擦除](#泛型擦除)
	- [范型的约束与局限性](#范型的约束与局限性)
	- [泛型与继承](#泛型与继承)


### 范型类
Java 集合有个缺点：当我们把一个对象放入集合中时，集合就会忘记这个对象的数据类型，当再次取出对象时，对象的编译类型就变成了Object类型，必须进行强制类型转换才能转换成需要的编译类型。 

所谓范型：**就是允许在定义类、接口时指定类型参数(在类名后边加上一对尖括号，类型形参放在尖括号内，多个形参用逗号分隔)，这些类型形参将在声明变量、创建对象时确定（即传入实际的类型参数），类型形参在在整个类内部可用，几乎所有可以使用普通类型的地方都可以使用这种类型形参。**JDK1.5之后改写了所有集合类支持范型。

使用范型类时，可以传入不同的类型实参创建对象，如 `List<Object>`、`List<String>`、`List<Integer>`...，效果上，相当于创建了集合类的一个子类，子类只能存放特定的类型，但是系统并没有为这些声明生成新的 class 文件，更不会把他们当成新类来处理，它们是同样的 List 类。List 类的静态变量和方法会在所有实例之间共享，所以在**静态方法、静态初始化块或者静态成员变量的声明和初始化块中不允许使用类型形参**。自然，这些效果上的子类之间更不会存在继承关系，也就是说 `List<String>`、`List<Integer>` 不是 `List<Object>` 的子类。所以下述代码会出现编译错误：
```
public void test(List<Object> list){...}//严格要求传入 List<Object> 范型
List<String> sList = new ArrayList();
test(sList);
```
当创建了带范型声明的接口、父类之后，可以创建接口的实现类，或者从父类来派生子类，如果**实现类或者派生子类不是范型类（判断是不是范型类，只要看类名后边是否有尖括号包裹的类型形参即可），那么当使用这些接口、父类时，不能再包含类型形参，必须指定具体的类型实参；如果实现类或者派生子类是范型类，则可以为指定与父类相同的类型参数**。如下面的代码是错误的：
```
class Creatrue<T>{}
class Man extends Creatrue<T>{} //编译错误
class Man extends Creatrue<String>{} //可以正确编译
class Man<T> extends Creatrue<T>{} //可以正确编译，此时形参名必须相同为T
class Man<T> extends Creatrue<String>{} //可以正确编译
class Man<U,R> extends Creatrue<String>{} //可以正确编译
```
### 类型变量的限定与范型通配符
声明类型参数时使用 extends 关键字可以限定类型参数的范围：
```
<T extends BindingType>
<T extends Person & Comparable & Serializable>
```
表示T应该是绑定类型的子类型（subtype）。T和绑定类型可以是类，也可以是接口。一个类型变量或通配符可以有多个限定。限定类型用“&”分隔，而逗号用来分隔类型量。
在Java的继承中，可以根据需要拥有多个接口超类型，但限定中至多有一个类。如果用一个类作为限定，它必须是限定列表中的第一个。
`List<? extends Shape>` 表示匹配所有Shape及其子类类型实参的范型。
`List<? super Shape>` 表示匹配所有Shape及其父类类型实参的范型。

将 ? 作为类型实参传递范型类，表示匹配任何类型实参类型。上面的代码改成下面这样可以编译通过：
```
public void test(List<?> list){...}//可以
List<String> sList = new ArrayList();
test(sList);
```

### 范型方法
即使在定义类、接口时没有使用范型，但是在定义方法时想定义类型形参，这也是可以的。
```
修饰符 <T,S> 返回值类型 方法名（形参列表）{
    //方法体
}
```
类型参数 T,S只能在该方法中使用。与类型范型不同的是，在调用范型方法时，无需显示的传入类型实参，系统可以直接推断出类型形参的类型。
```
public <T> void arrayToCollection(T[] arr, Collection c){
    for(T v:arr){
        c.add(v);
    }
}
Integer[] arr3 = new Integer[10];
List<Integer> list2 = new ArrayList<>();
arrayToCollection(arr3,list2);
```
范型方法允许类型参数被用来表示方法的多个参数之间的类型依赖关系，或者方法返回值和参数之间的类型依赖关系。如果没有这样的依赖关系，不应该使用范型方法。


### 泛型擦除
无论何时定义一个泛型类型，都自动提供了一个相应的原始类型（raw type）。**原始类型的名字就是删去类型参数后的泛型类型名。擦除（erased）类型变量，并替换为限定类型（无限定的变量用Object）。如果有多个类型限定，原始类型用第一个限定的类型变量来替换，如果没有给定限定就用Object替换。**

总之，需要记住有关Java泛型转换的事实：
+ 虚拟机中没有泛型，只有普通的类和方法。
+ 所有的类型参数都用它们的限定类型替换。
+ 桥方法被合成来保持多态。（合成的桥方法有可能调用了新定义的方法。）
+ 为保持类型安全性，必要时插入强制类型转换。

### 范型的约束与局限性
1. 不能用基本类型作为类型参数。因此，没有`Pair<double>`，只有`Pair<Double>`。当然， 其原因是类型擦除。擦除之后，Pair 类含有Object 类型的域，而Object不能存储double值。
2. 不能在静态域或方法中引用类型变量。禁止使用带有类型变量的静态域和方法。
   ```
   public class Singleton<T> {
	private static T singleInstance; // EROR 
	public static T getSingleInstance{ // EROR
		if (singleInstance = nul) construct new instance of T 
		return singleInstance;
	} 
   ```
3. 不能抛出或捕获泛型类的实例，既不能抛出也不能捕获泛型类对象。实际上，甚至泛型类扩展 `Throwable` 都是不合法的 。

### 泛型与继承
假设我们定义了一个泛型类`Pair`，`Manager`是`Employee`的子类，那么它们的继承关系如下:
<center><img src="pics/generics-1.png" alt=""></center>

**总结一下就是无论S与T有什么关系，通常，`Pair<S>`与`Pair<T>`都没有任何关系。**

还需要说明的是，泛型类可以扩展或实现其他的泛型类。就这一点而言，它们与普通的类没有什么区别。  
ArrayList类实现了List接口。这意味着，一个`ArrayList<Manager>`可以转换为一个`List<Manager>`。但是，如前面所见，`ArrayList<Manager>`不是一个`ArrayList<Employee>`或`List<Employee>`。

<center><img src="pics/generics-2.png" alt=""></center>

