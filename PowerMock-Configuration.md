# PowerMock 配置 #

```
自 PowerMock 1.7.0 起可用的功能
```

从1.7.0版本开始，可以全局配置PowerMock。此配置适用于类路径中的所有测试类。您可以通过将以下文件添加到类路径来创建来启用配置：

```
org/powermock/extensions/configuration.properties
```

## 全局的 @PowerMockIgnore ##

默认情况下，PowerMock使用其[MockClassLoader](http://www.javadoc.io/doc/org.powermock/powermock-core/2.0.0)加载所有类。该类加载器加载并修改了所有类，除了：

* 系统类。它们被推迟到系统类加载器
* 位于指定为忽略的包中的类。

在1.7.0之前，忽略包的唯一方法是使用 [@PowerMockIgnore](http://www.javadoc.io/doc/org.powermock/powermock-core/2.0.0) 注解:

```java
@PowerMockIgnore("org.myproject.*")
@PrepareForTest(MyClass.class)
@RunWith(PowerMockRunner.class)
public class MyTest {
    ...
}
```

在 [某些情况下](FAQ) ，避免某些类加载相关问题的唯一方法是使用`@PowerMockIgnore`，这通常意味着`@PowerMockIgnore`已在许多测试类中进行了复制和粘贴。

但是从PowerMock 1.7.0开始，您可以使用配置文件指定要忽略的软件包：

```properties
powermock.global-ignore="org.myproject.*"
```

可以使用逗号指定多个包/类：

```properties
powermock.global-ignore="org.myproject.*","org.3rdpatproject.SomeClass"
```

这是使用PowerMock对`@PowerMockIgnore`进行全局配置的[示例](https://github.com/powermock/powermock-examples-maven/tree/master/global-ignore)。

## Mockito mock-maker-inline ##

PowerMock 可以将调用委托给另一个 `MockMaker`, 这些测试无需PowerMock即可运行。可以通过在配置文件中添加以下属性来配置它：

```properties
mockito.mock-maker-class=mock-maker-inline
```

这是一个关于在PowerMock中使用 Mockito `mock-maker-inline` 的[示例](https://github.com/powermock/powermock-examples-maven/tree/master/mockito2) 。
