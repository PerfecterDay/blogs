# Spring Mvc Controller Advice
{docsify-updated}

`@ExceptionHandler` 、 `@InitBinder` 和 `@ModelAttribute` 方法只会在声明它们的 `@Controller` 类或类层次结构中起作用。相反，如果在 `@ControllerAdvice` 或 `@RestControllerAdvice` 类中声明了这些方法，那么它们将适用于任何 Controller 或者指定 Controller 。此外，自 5.3 起，`@ControllerAdvice` 中的 `@ExceptionHandler` 方法可用于处理来自任何 `@Controller` 或任何其他处理程序的异常。

`@ControllerAdvice` 通过 `@Component` 进行元标注，因此可以通过组件扫描注册为 Spring Bean。 `@RestControllerAdvice` 通过 `@ControllerAdvice` 和 `@ResponseBody` 进行元标注，这意味着其中的 `@ExceptionHandler` 方法的返回值将通过响应体消息转换呈现，而不是通过 HTML 视图呈现。

启动时， `RequestMappingHandlerMapping` 和 `ExceptionHandlerExceptionResolver` 会检测 controller advice bean 并在程序运行时动地态应用它们。 `@ControllerAdvice` 中的 `@ExceptionHandler` 方法会在 `@Controller` 中处理请求的方法之后运行。相比之下，而 `@ModelAttribute` 和 `@InitBinder` 方法会在请求处理方法运行之前运行。

`@ControllerAdvice` 注解具有一些属性，可以缩小适用的控制器和处理程序的范围。例如
```
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```
选择器在运行时进行评估，如果大量使用，可能会对性能产生负面影响。

