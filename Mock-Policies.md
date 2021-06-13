# Mock策略 #

## 总览 ##

  1. 在类上使用 `@RunWith(PowerMockRunner.class)` 注解。
  1. 在类上使用 `@MockPolicy(MyMockPolicy.class)` 注解。

## 示例 ##

可以使用Mock策略使得与PowerMock隔离的某些框架的代码的单元测试变得更容易。Mock策略的实现可以是如抑制某些方法，抑制静态初始化程序或拦截方法调用，并更改某些框架或一组类或接口的返回值（例如，返回模拟对象）。例如，可以实施Mock策略来避免为测试编写重复的设置代码。假设您使用的是框架X，并且要对其进行测试，则要求某些方法应始终返回模拟实现。或许也必须抑制某些静态初始化程序。与其在测试之间复制该代码，不如编写一个可重用的Mock策略。

PowerMock 1.1提供了三种开箱即用的模拟策略，用于模拟slf4j，java common-logging和log4j。让我们以slf4j为例，假设您有一个看起来像这样的类：

```java
public class Slf4jUser {
	private static final Logger log = LoggerFactory.getLogger(Slf4jUser.class);

	public final String getMessage() {
		log.debug("getMessage!");
		return "log4j user";
	}
}
```

这里有一个问题，因为logger在Slf4jUser类的静态初始化器中实例化。有时，这会导致问题，具体取决于日志配置，因此在单元测试中要做的是‘存根’（或者说是‘打桩’）日志实例。这是完全可行的，无需使用Mock策略。一种方法是从测试中禁止Slf4jUser类的静态初始化程序开始。然后，我们可以创建Logger类的存根或漂亮的模拟并将其注入Slf4jUser实例。但这还不够，假设我们已经配置了slf4j以使用log4j作为后端日志，那么在运行测试时，控制台中将显示以下错误：

```java
log4j:ERROR A "org.apache.log4j.RollingFileAppender" object is not assignable to a org.apache.log4j.Appender" variable.
log4j:ERROR The class "org.apache.log4j.Appender" was loaded by
log4j:ERROR [org.powermock.core.classloader.MockClassLoader@aa9835] whereas object of  type
log4j:ERROR "org.apache.log4j.RollingFileAppender" was loaded by [sun.misc.Launcher$AppClassLoader@11b86e7].
log4j:ERROR Could not instantiate appender named "R".
```

为避免出现此错误消息，我们需要准备`org.apache.log4j.Appender`进行测试。完整的测试设置如下所示：

```java
@RunWith(PowerMockRunner.class)
@SuppressStaticInitializationFor("org.myapp.Slf4jUser")
@PrepareForTest( Appender.class)
public class MyTest {

  @Before
  public void setUp() {
      Logger loggerMock = createNiceMock(Logger.class);
      Whitebox.setInternalState(Slf4jUser.class, loggerMock);
      ...
  }
  ...
}
```
此设置行为必须复制到处理slf4j的所有测试类中。相反，您可以使用Slf4j Mock策略来为您完成此设置。您的测试将如下所示：

```java
@RunWith(PowerMockRunner.class)
@MockPolicy(Slf4jMockPolicy.class)
public class Slf4jUserTest {
     ...
}
```
请注意，我们根本不需要进行任何设置来模拟slf4j，`Slf4jMockPolicy`会注意这一点。

Mock策略也可以像这样链接或嵌套：

```java
@RunWith(PowerMockRunner.class)
@MockPolicy({MockPolicyX.class, MockPolicyY.class})
public class MyTest {
    ...
}
```

请注意，链中的后续Mock策略可以覆盖先前策略的行为。在此示例中，这意味着`MockPolicyY`可能会覆盖由`MockPolicyX`定义的行为。如果编写自定义模拟策略，请务必牢记这一点。

## 创建自定义Mock策略

可以创建自定义的Mock策略来处理类似的设置和模拟行为。为此，您需要创建一个实现[org.powermock.core.spi.PowerMockPolicy](https://github.com/jayway/powermock/blob/master/core/src/main/java/org/powermock/core/spi/PowerMockPolicy.java)接口的类。它包含两种方法：

```java
void applyClassLoadingPolicy(MockPolicyClassLoadingSettings settings);
```
和

```java
void applyInterceptionPolicy(MockPolicyInterceptionSettings settings);
```
一个Mock策略的实现可以禁止某些方法，禁止静态初始化程序或拦截方法调用并更改其返回值（例如返回模拟对象）。模拟策略的实现必须是无状态的。之所以有两种设置方法的原因是，PowerMock需要知道在加载这些类*之前*，应由Mock类加载器修改哪些类。第一种方法告诉PowerMock应该加载哪些类，然后从Mock类加载器本身调用第二种方法。这意味着您可以在`applyInterceptionPolicy`方法中为例如final和static方法创建模拟，否则将无法实现。

由于模拟策略可以链接在一起，因此后续策略可以覆盖先前策略的行为。为避免意外覆盖，建议*添加*行为而不是*设置*行为，因为后者会覆盖所有先前的配置。

让我们用一个例子来解释一下。想象一下，我们有一个看起来像这样的类：

```java
public class Dependency {

	private final DataObject dataObject;

	public Dependency(String data) {
		dataObject = new DataObject(data);
	}

	public DataObject getData() {
		return dataObject;
	}
}
```
假设我们想在每次调用`getData`该方法时都返回一个自定义`DataObject`，即我们要拦截`getData`对它的调用并使它返回我们的自定义对象。为了创建一个可重用的Mock策略来做到这一点，我们首先创建一个实现`org.powermock.core.spi.PowerMockPolicy`接口的新类`MyCustomMockPolicy`。完整的代码如下所示：

```java
public class MyCustomMockPolicy implements PowerMockPolicy {

	/**
	 * Add the {@link Dependency} to the list of classes that should be loaded
	 * by the mock class-loader.
	 */
	public void applyClassLoadingPolicy(MockPolicyClassLoadingSettings settings) {
		settings.addFullyQualifiedNamesOfClassesToLoadByMockClassloader(Dependency.class.getName());
	}

	/**
	 * Every time the {@link Dependency#getData()} method is invoked we return a
	 * custom instance of a {@link DataObject}.
	 */
	public void applyInterceptionPolicy(MockPolicyInterceptionSettings settings) {
		final Method getDataMethod = Whitebox.getMethod(Dependency.class, "getData");
		final DataObject dataObject = new DataObject("Policy generated data object");
		settings.addSubtituteReturnValue(getDataMethod, dataObject);
	}
}
```

让我们更详细地解释它。由于我们要拦截`Dependency`类的`getData()`方法调用，因此必须通过让Mock类加载器加载这个类来为测试做准备。因此，我们通过传递`Dependency`类的完全限定名称来从`applyClassLoadingPolicy`方法中调用`settings.addFullyQualifiedNamesOfClassesToLoadByMockClassloader(..)`方法，从而告诉PowerMock这样做。如果我们愿意，也可以在此处传递包名称，当我们希望特定类包（和子包）中的所有类都由Mock类加载器加载时。在下一步中，我们告诉PowerMock进行实际的拦截。首先，使用`Whitebox.getMethod(..)`来获取要拦截的方法（在本例中为getData方法）。然后，创建我们希望返回的自定义DataObject，然后通过调用`settings.addSubtituteReturnValue(..)`方法指示PowerMock返回此对象，而已！

假设我们有一个实际上使用我们的依赖关系的类，如下所示：

```java
public class DependencyUser {

	public DataObject getDependencyData() {
		return new Dependency("some data").getData();
	}
}
```

如果将Mock策略应用于`DependencyUser`实例的测试，则`Dependency`实例返回的值`DataObject`将是我们在`MyCustomMockPolicy#applyInterceptionPolicy(..)`方法中创建的。这是一个简单的测试，可以证明这种情况确实发生了：

```java
@RunWith(PowerMockRunner.class)
@MockPolicy(MyCustomMockPolicy.class)
public class DependencyUserTest {

	@Test
	public void assertThatMyFirstMockPolicyWork() throws Exception {
		DataObject dependencyData = new DependencyUser().getDependencyData();
		assertEquals("Policy generated data object", dependencyData.getData());
	}
}
```

如果未应用Mock策略，`dependencyData.getData()`则将返回"some data"。

## 参考 ##
  * [MyCustomMockPolicy](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/mockpolicy/policy/MyCustomMockPolicy.java)
  * [DependencyUserTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/mockpolicy/DependencyUserTest.java)
  * [DependencyUser](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/mockpolicy/DependencyUser.java)
  * [Dependency](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/mockpolicy/nontest/Dependency.java)
  * [DataObject](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/mockpolicy/nontest/domain/DataObject.java)