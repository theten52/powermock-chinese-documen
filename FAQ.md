# 常见问题

1：PowerMockRunner 抛出了

`java.lang.NoClassDefFoundError: org/junit/internal/runners/BeforeAndAfterRunner`

或

`java.lang.SecurityException: class "org.junit.internal.runners.TestClass"'s signer information does not match signer information of other classes in the same package`

异常。哪里错了?

>你可能使用了一个错误的 PowerMockRunner. 有一个用于JUnit 4.4及更高版本的Runner程序，另一个是用于JUnit 4.0-4.3的Runner程序（尽管后者也适用于JUnit 4.4的某些较旧的次要版本）。 尝试从`org.powermock.modules.junit4.PowerMockRunner` 切换到 `org.powermock.modules.junit4.legacy.PowerMockRunner`，反之亦然。 查看 <a href='GettingStarted.md'>getting started</a>，了解如何在Maven中进行配置。

2:在Maven中运行PowerMock测试时，Cobertura会给我错误或产生奇怪的结果，我该如何解决呢？

>使用如下任意一个以解决问题:
>a:升级到Cobertura 2.4+或
>b:按照 <a href='http://www.jsfblog.info/2010/02/cobertura-code-coverage-with-maven-and-powermock/'>此博客</a>上的说明进行操作，或者，
>c:将以下内容添加到您的pom.xml文件中:
>
>```xml
>    <build>
>        <plugins>
>            <plugin>
>                <artifactId>maven-surefire-plugin</artifactId>
>                <configuration>
>                    <forkMode>pertest</forkMode>
>                </configuration>
>            </plugin>
>        </plugins>
>    </build>
>```

3:我从`DocumentBuilderFactory`，`SaxParserFactory`或其他XML相关类中得到了一个`ClassCastException`错误，请问该怎么办？

>原因是 XML 框架尝试使用反射实例化类，并从线程上下文类加载器（PowerMock 的类加载器）执行此操作，然后尝试将创建的对象分配给未由同一类加载器加载的字段。发生这种情况时，您需要使用 @PowerMockIgnore 注解来告诉 PowerMock 将某个包的加载推迟到系统类加载器。您需要忽略的是特定于案例的，但通常是 XML 框架或与之交互的一些包。例如`@PowerMockIgnore({"org.xml.\*", "javax.xml.\*"})`。另一种选择是尝试使用我们的[Java Agent](https://github.com/powermock/powermock/wiki/PowerMockAgent)进行引导。

4:我不能对`java.lang`，`java.net`，`java.io`或其他系统类进行mock，为什么呢？

>这是因为它们是由 Java 的引导类加载器加载的，并且不能由 PowerMock 的类加载器操作字节码。从 PowerMock 1.2.5 开始，有一个变通方法，请看一下[这个](https://github.com/jayway/powermock/blob/master/modules/module-test/mockito/junit4/src/test/java/samples/powermockito/junit4/system/SystemClassUserTest.java)简单的例子，看看它是如何完成的。


5:模拟 Hibernate 时，我会收到类似于以下内容的错误：

```java
java.lang.ClassCastException: org.hibernate.ejb.HibernatePersistence cannot be cast to javax.persistence.spi.PersistenceProvider
    at javax.persistence.Persistence.findAllProviders(Persistence.java:80)
    at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:49)
    at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:34)
    ...
```
> 解决方案：升级到 PowerMock 1.3+ 或在测试的类级别使用`@PowerMockIgnore("javax.persistence.*")`。

6：运行 PowerMock 测试时，log4j 给了我以下（或类似的）错误，
```java
log4j:ERROR A "org.apache.log4j.xml.DOMConfigurator" object is not assignable to a "org.apache.log4j.spi.Configurator" variable.
log4j:ERROR The class "org.apache.log4j.spi.Configurator" was loaded by
log4j:ERROR [org.powermock.core.classloader.MockClassLoader@14a55f2] whereas object of type
log4j:ERROR "org.apache.log4j.xml.DOMConfigurator" was loaded by [sun.misc.Launcher$AppClassLoader@92e78c].
log4j:ERROR Could not instantiate configurator [org.apache.log4j.xml.DOMConfigurator].
```
或者
```java
Caused by: org.apache.commons.logging.LogConfigurationException:
Invalid class loader hierarchy.  You have more than one version of
'org.apache.commons.logging.Log' visible, which is not allowed.
```
现在我该怎么办？

> 对此有几种不同的解决方案：
>
> > 1：升级到 PowerMock 1.3+。
> >
> > 2：在测试的类级别使用 @PowerMockIgnore 注解。例如，如果使用 log4j，则使用注解：@PowerMockIgnore("org.apache.log4j.\*")；如果使用Commons logging，则使用注解@PowerMockIgnore("org.apache.commons.logging.\*")。
> >
> > 3：添加`-Dlog4j.ignoreTCL=true`作为虚拟机参数到您的测试运行配置中。
> >
> > 4：如果您使用的是 PowerMock 1.1 或更高版本，则应使用@MockPolicy注解并指定mock策略。例如，如果您将 slf4j 与 log4j 结合使用则应该再使用注解@MockPolicy(Slf4jMockPolicy.class)，或者您正在独立使用 Log4j 则应该再使用注解 @MockPolicy(Log4jMockPolicy.class)。这是推荐的方式。例如：
>
> ```java
> @RunWith(PowerMockRunner.class)
> @MockPolicy(Log4jMockPolicy.class)
> public class MyTest {
> 	...
> }
> ```
> > 
> >
> > 5：创建一个很好的 Logger mock类并将 Logger 字段设置为此实例。如果该字段是静态的，则抑制该类的静态初始化程序（使用@SuppressStaticInitializerFor注解），然后将 logger 字段设置为您刚刚创建的mock类。接下来为测试准备org.apache.log4j.Appender并使用@PrepareForTest 注解进行测试。例如：
>
> ```java
> @RunWith(PowerMockRunner.class)
> @SuppressStaticInitializationFor("org.myapp.MyClassUsingLog4J")
> @PrepareForTest({Appender.class})
> public class MyTest {
> 
> 	@Before
> 	public void setUp() {
>  	  Logger loggerMock = createNiceMock(Logger.class);
>  	  Whitebox.setInternalState(MyClassUsingLog4J.class, loggerMock);
>  	  ...
> 	}
> 	...
> }
> ```
> > 
> >
> > 6：遵循与上一步相同的过程，但不是将`org.apache.log4j.Appender`类添加到@PrepareForTest注解中，而是将 `org.apache.log4j.LogManager`类添加到@SuppressStaticInitializerFor注解中。例如：
>
> ```java
> @RunWith(PowerMockRunner.class)
> @SuppressStaticInitializationFor({
> 		"org.myapp.MyClassUsingLog4J",
> 		"org.apache.log4j.LogManager"})
> public class MyTest {
> 
> 	@Before
> 	public void setUp() {
>    	Logger loggerMock = createNiceMock(Logger.class);
>    	Whitebox.setInternalState(MyClassUsingLog4J.class, loggerMock);
>    	...
> 	}
> 	...
> }
> ```
> > 
> >
> > 7：您可以尝试使用@PrepareEverythingForTest注解（不推荐）。

7：PowerMock 是否与 TestNG 一起工作？
>可以，因为PowerMock 1.3.5版本确实有基本的 TestNG 支持。

8：PowerMock 是 EasyMock 的一个分支吗？

>不是。PowerMock 扩展了其他mock框架，例如 EasyMock，具有强大的功能，例如静态模拟。

9：您可以将 PowerMock 与其他使用 JUnit Runner 的框架一起使用吗？

>是的，您可以使用PowerMockRule。

10：我正在使用 Java Agent，而 Java 7 遇到诸如“Unable to load Java agent”之类的错误，该怎么办？
>您可以尝试以下解决方法：
>```xml
>    <plugin>
>        <groupId>org.apache.maven.plugins</groupId>
>        <artifactId>maven-surefire-plugin</artifactId>
>        <version>2.14</version>
>        <configuration>
>            <argLine>
>                -javaagent:${settings.localRepository}/org/powermock/powermock-module-javaagent/${powermock.version}/powermock-module-javaagent-${powermock.version}.jar -XX:-UseSplitVerifier
>            </argLine>
>        </configuration>
>    </plugin>
>```

11：我升级到 1.6.5 版但是 PowerMock 开始抛出异常：`Extension API internal error: org.powermock.api.extension.reporter.MockingFrameworkReporterFactoryImpl could not be located in classpath.`。

>如果您使用 Maven，请将以下内容添加到您的 pom.xml
>```xml
       <dependency>
              <groupId>org.powermock</groupId>
              <artifactId>powermock-api-mockito-common</artifactId>
              <version>1.6.5</version>
       </dependency>
>```
>如果您下载完整的 jar，那么请从 Maven Central Repository再下载一个jar并将其添加到您的类路径中。