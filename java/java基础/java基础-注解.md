# 注解
{docsify-updated}

## 元注解
1. @Documented —— 指明拥有这个注解的元素可以被javadoc此类的工具文档化。
2. @Target —— 指明该类型的注解可以注解的程序元素的范围。该元注解的取值可以为TYPE,METHOD,CONSTRUCTOR,FIELD等。如果Target元注解没有出现，那么定义的注解可以应用于程序的任何元素。
3. @Inherited —— 指明该注解类型被自动继承。如果用户在当前类中查询这个元注解类型并且当前类的声明中不包含这个元注解类型，那么也将自动查询当前类的父类是否存在Inherited元注解，这个动作将被重复执行知道这个标注类型被找到，或者是查询到顶层的父类。
4. @Retention——指明了该Annotation被保留的时间长短。RetentionPolicy取值为SOURCE,CLASS,RUNTIME。

## 自定义注解语法
创建自定义注解和创建一个接口相似，但是注解的interface关键字需要以@符号开头。我们可以为注解声明方法。

    @Documented
    @Target(ElementType.METHOD)
    @Inherited
    @Retention(RetentionPolicy.RUNTIME)
        public @interface MethodInfo{
        String author() default 'Pankaj';
        String date();
        int revision() default 1;
        String comments();
    }
+ 注解方法不能带有参数；
+ 注解方法返回值类型限定为： 基本类型 、String 、Enums 、Class 、Annotation 或者是这些类型的数组；
+ 注解方法可以有默认值；
+ 注解本身能够包含元注解，元注解被用来注解其它注解。

## Java注解解析
使用反射技术来解析java类的注解。那么注解的RetentionPolicy应该设置为RUNTIME，否则java类的注解信息在执行过程中将不可用，那么我们也不能从中得到任何和注解有关的数据。

    public class AnnotationParsing {
     
    public static void main(String[] args) {
        try {
        for (Method method : AnnotationParsing.class
            .getClassLoader()
            .loadClass(('com.journaldev.annotations.AnnotationExample'))
            .getMethods()) {
            // checks if MethodInfo annotation is present for the method
            if (method.isAnnotationPresent(com.journaldev.annotations.MethodInfo.class)) {
                try {
            // iterates all the annotations available in the method
                    for (Annotation anno : method.getDeclaredAnnotations()) {
                        System.out.println('Annotation in Method ''+ method + '' : ' + anno);
                        }
                    MethodInfo methodAnno = method.getAnnotation(MethodInfo.class);
                    if (methodAnno.revision() == 1) {
                        System.out.println('Method with revision no 1 = '+ method);
                        }
     
                } catch (Throwable ex) {
                        ex.printStackTrace();
                        }
            }
        }
        } catch (SecurityException | ClassNotFoundException e) {
                e.printStackTrace();
             }
        }
     
    }

## 编译时处理注解
APT(Annotation Processing Tool)是一种处理注解的工具，它对源代码文件进行检测，并找出源文件所包含的注解信息，然后针对注解信息进行额外的处理。

Java 提供的 javac.exe 编译工具有一个 `-processor` 选项，该选项可以指定一个注解处理器，如果编译时，指定了注解处理器，那么这个注解处理器将会起作用。

每个注解处理器都需要实现 `javax.annotation.processing` 包下的 `Processor` 接口。不过实现该接口必须实现它里边的所有方法，因此通常会采用继承 `AbstractProcessor` 的方式来实现注解处理器。一个注解处理器可以处理一个或多个注解。
