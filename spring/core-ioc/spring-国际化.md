# Spring 国际化
{docsify-updated}

```
public class PersonValidator implements Validator {

    public boolean supports(Class clazz) {
        return Person.class.equals(clazz);
    }

    public void validate(Object obj, Errors e) {
        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
        Person p = (Person) obj;
        if (p.getAge() < 0) {
            e.rejectValue("age", "negativevalue");
        } else if (p.getAge() > 110) {
            e.rejectValue("age", "too.darn.old");
        }
    }

    public static void main(String[] args) {
        PersonValidator pe = new PersonValidator();
        Person p = new Person();
        p.setAge(120);
        MapBindingResult xcc = new MapBindingResult(new HashMap<>(), "xcc");
        pe.validate(p,xcc);
        System.out.println("axs");
    }
}

@Data
class Person{
    private String name;
    private int age;
}
```
<center><img src="pics/validate-errcode.png" width="50%"></center>

我们介绍了数据库绑定和验证。本节将介绍与验证错误相对应的信息输出。在上一节的示例中，我们剔除了姓名和年龄字段。如果我们想使用 `MessageSource` 输出错误信息，可以使用拒绝字段时提供的错误代码（本例中为 "姓名 "和 "年龄"）。在调用（直接调用或间接调用 `ValidationUtils` 类等） `rejectionValue` 或 `Errors` `接口中的其他剔除方法时，底层实现不仅会注册您传入的代码，还会注册一些额外的错误代码。MessageCodesResolver` 决定 `Errors` 接口注册哪些错误代码。默认情况下，使用的是 `DefaultMessageCodesResolver` ，它（例如）不仅会注册包含您提供的代码的消息，还会注册包含您传递给剔除方法的字段名的消息。因此，如果使用 `rejectValue("age", "too.darn.old")` 剔除一个字段，除了 `too.darn.old` 代码外，Spring 还会注册 `too.darn.old.age` 和 `too.darn.old.age.int` 消息（前者包括字段名称，后者包括字段类型）。这样做是为了方便开发人员定位错误信息。

## MessageSource
`ApplicationContext` 接口扩展了一个名为 `MessageSource` 的接口，该接口提供了国际化（"i18n"）功能。Spring 还提供了 `HierarchicalMessageSource` 接口，该接口可以分层解析消息。这些接口共同构成了 Spring 实现消息解析的基础。这些接口定义的方法包括：
+ `String getMessage(String code, Object[] args, String default, Locale loc)`: 用于从 `MessageSource` 获取消息的基本方法。如果在指定的本地没有找到消息，则使用默认消息 default 。通过标准库提供的 `MessageFormat` 功能，传入的任何参数 args 都会成为替换值。
+ `String getMessage(String code, Object[] args, Locale loc)`: 与前一种方法基本相同，但有一点不同：不能指定默认信息。如果找不到信息，就会抛出 `NoSuchMessageException` 异常。
+ `String getMessage(MessageSourceResolvable resolvable, Locale locale)`: 前面方法中使用的所有属性也都封装在一个名为 `MessageSourceResolvable` 的类中，你可以使用该方法。

加载 `ApplicationContext` 时，它会自动搜索上下文中定义的 `MessageSource` Bean。该 Bean 的名称必须是 `messageSource` 。如果找到了这样一个 Bean，对前面方法的所有调用都会委托给该 bean。如果没有找到消息源， `ApplicationContext` 会尝试从父容器查找包含同名的 Bean 。如果找到了，它就会使用该 bean 作为消息源。如果 `ApplicationContext` 无法找到任何消息源，则会实例化一个空的 `DelegatingMessageSource` ，以便能够接受对上述方法的调用。

Spring 提供了三种 MessageSource 实现：
+ `ResourceBundleMessageSource`
+ `ReloadableResourceBundleMessageSource`
+ `StaticMessageSource`

它们都实现了 `HierarchicalMessageSource` ，以便进行嵌套消息传递。 `StaticMessageSource` 很少使用，但它提供了向消息源添加消息的编程方法 。

由于 Spring 的 `MessageSource` 基于 Java 的 `ResourceBundle` ，因此它不会合并具有相同基名的捆绑包，而只会使用找到的第一个捆绑包。具有相同基名的后续消息捆绑包将被忽略。

## ResourceBundles
https://www.baeldung.com/java-resourcebundle