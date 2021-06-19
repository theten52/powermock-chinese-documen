# 使用 JUnit Rule 引导程序 #

```text
从 PowerMock 1.4 时功能可用。
```

从 1.4 版开始，可以使用 [JUnit Rule](http://www.infoq.com/news/2009/07/junit-4.7-rules) 而不是使用 PowerMockRunner 和 RunWith 注解来引导 PowerMock 。这允许您在使用其他 JUnit 运行程序的同时仍然受益于 PowerMock 的功能。您可以通过指定：

```java
@PrepareForTest(X.class);
public class MyTest {
    @Rule
    PowerMockRule rule = new PowerMockRule();

    // Tests goes here
    ...
}
```

## 和 Maven 一起使用 PowerMockRule ##
你需要依赖这些项目：

```xml
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-module-junit4-rule</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-classloading-xstream</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
```

您还可以替换`powermock-classloading-xstream`为 Objenesis 版本：

```xml
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-classloading-objenesis</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
```

然而，这个版本不如 xstream 版本稳定，但它不需要任何额外的依赖项。

**警告**：使用版本 PowerMock 1.4（或更高版本），因为在此版本之前，该Rule实际上并未执行任何测试方法（但看起来确实如此）。

## 不和 Maven 一起使用 PowerMockRule

您需要下载[powermock-module-junit4-rule](http://repo1.maven.org/maven2/org/powermock/powermock-module-junit4-rule/2.0.2/powermock-module-junit4-rule-2.0.2.jar)、[powermock-classloading-base](http://repo1.maven.org/maven2/org/powermock/powermock-classloading-base/2.0.2/powermock-classloading-base-2.0.2.jar)和
[powermock-classloading-xstream](http://repo1.maven.org/maven2/org/powermock/powermock-classloading-xstream/2.0.2/powermock-classloading-xstream-2.0.2.jar)或[powermock-classloading-objenesis 之一](http://repo1.maven.org/maven2/org/powermock/powermock-classloading-objenesis/2.0.2/powermock-classloading-objenesis-2.0.2.jar)，并将其放入您的类路径中。

## 参考 ##

* [Mockito 示例](https://github.com/powermock/powermock/tree/master/tests/mockito/junit4-rule-xstream/src/test/java/samples/powermockito/junit4/rule/xstream)
* 使用 PowerMock 和 Mockito[示例进行](https://github.com/jayway/powermock/tree/master/examples/spring-mockito)Spring 集成测试。
