# 绕过封装（Bypass encapsulation） #

## 总览 ##

[Whitebox](http://www.javadoc.io/doc/org.powermock/powermock-reflect/2.0.0) 类提供了一组方法，如果需要，您可以帮助绕过封装。通常，获取/修改非公共字段不是一个好主意，但有时这是通过测试来覆盖代码以供将来重构的唯一方法。

 

  1. 使用 `Whitebox.setInternalState(..)` 去设置实例或类的非公共成员。
  1. 使用 `Whitebox.getInternalState(..)` 去获取实例或类的非公共成员。
  1. 使用 `Whitebox.invokeMethod(..)` 去调用实例或类的非公共方法。
  1. 使用 `Whitebox.invokeConstructor(..)` 从私有构造函数创建类的实例。



## 示例 ##

### 访问内部状态 ###
对于可变对象，在调用方法后内部状态可能会更改。在对此类对象进行单元测试时，最好有一种简单的方法来保持此状态并查看其是否已相应更新。PowerMock提供了几种有用的反工具，这些反射工具专门设计用于单元测试。所有这些反射工具都位于`org.powermock.reflect.Whitebox`类中。

出于演示目的，我们假设有一个如下所示的类：

```java
public class ServiceHolder {

	private final Set<Object> services = new HashSet<Object>();

	public void addService(Object service) {
		services.add(service);
	}

	public void removeService(Object service) {
		services.remove(service);
	}
}
```

假设我们要测试该`addService`方法（当然可以认为它“太简单了”，但是出于演示目的对其进行测试也足够了）。在这里，我们要确保在调用`addService`方法后，`ServiceHolder`的状态已正确更新。也就是说，新对象已添加到`services`集合中。一种方式是添加一个名为`getServices()`的package-private或protected的方法，该方法返回该`services`集合。但是通过这样做，我们为`ServiceHolder`该类添加了一个方法，该方法除了使该类可测试外没有其他目的。该方法也可能在代码的其他地方被误用。另一种选择是使用PowerMock中的`Whitebox.getInternalState(..)`方法来完成相同的事情而无需更改生产代码。在这种简单情况下，整个`addService`方法的测试如下所示：

```java
@Test
public void testAddService() throws Exception {
	ServiceHolder tested = new ServiceHolder();
	final Object service = new Object();

	tested.addService(service);

        // This is how you get the private services set using PowerMock
	Set<String> services = Whitebox.getInternalState(tested,
			"services");

	assertEquals("Size of the \"services\" Set should be 1", 1, services
			.size());
	assertSame("The services Set should didn't contain the expect service",
			service, services.iterator().next());
}
```

使用PowerMock 1.0或更高版本，您还可以通过指定要获取的字段的类型来获取内部状态。在上述情况下，我们可以这样写

```java
Set<String> services = Whitebox.getInternalState(tested, Set.class);
```
去获取 `services` 字段. 这是一种获取内部状态的类型安全性更高的方法，也是首选方法。在类`Set`中有多个`Set`类型的字段的情况下，您仍然必须恢复为使用字段名称方法。

设置对象的内部状态同样容易。举例来说，我们有以下类：

```java
public class ReportGenerator {

	@Injectable
	private ReportTemplateService reportTemplateService;

	public Report generateReport(String reportId) {
		String templateId = reportTemplateService.getTemplateId(reportId);
		/*
		 * Imagine some other code here that generates the report based on the
		 * template id.
		 */
		return new Report("name");
	}
}
```

在这里，我们想象我们正在使用一个依赖项注入框架，该框架会在运行时自动为我们提供`ReportTemplateService`实例。我们有很多选择可以测试此类。例如，我们可以重构类以使用构造函数或setter注入，使用java反射，或者我们可以简单地为`reportTemplateService`创建一个setter，该setter*仅*用于测试目的（即，设置模拟实例）。另一种方法是让PowerMock使用`Whitebox.setInternalState(..)`方法为您进行反射。在这种情况下，这真的很容易：

```java
Whitebox.setInternalState(tested, "reportTemplateService", reportTemplateServiceMock);
```

使用PowerMock 1.0或更高版本使它变得更加容易，因为您甚至可以忽略字段名称：

```java
Whitebox.setInternalState(tested, reportTemplateServiceMock);
```
这是首选方法，因为它对重构更友好。如果您的类中有多个`ReportTemplateService`类型的字段，则可能仍必须恢复为字段名方法。

我们刚刚看到的set和get内部状态方法也有一些更高级的用例。很多时候，修改或读取对象的类层次结构中的内部状态很有用。通常，setInternalState和getInternalState遍历超类层次结构，以在子类中查找字段，然后返回或设置与提供的字段名称匹配的第一个字段。但是在某些情况下，您确实希望获取或设置层次结构中某个位置的特定字段。例如，假设我们有一个扩展了B的类A的实例，并且A和B都有一个名为`myPrivateString`的`java.lang.String`类型的字段，但是在这种情况下，我们想读取B的字段的内容。通过使用`Whitebox.getInternalState(..)`我们可以通过指定应从类层次结构中读取字段的位置来轻松实现此目的。在这种情况下，我们将编写如下内容：

```java
String myPrivateString = Whitebox.<String> getInternalState(instanceOfA, "myPrivateString", B.class);
```
或者，如果您使用的是PowerMock 1.0或更高版本：

```java
String myPrivateString = Whitebox.getInternalState(instanceOfA, String.class, B.class);
```
在大多数情况下，这也消除了强制转换。

使用第一种方法，可以通过指定返回类型来避免转换：

```java
String myPrivateString = Whitebox.getInternalState(instanceOfA, "myPrivateString", B.class, String.class);
```

如果我们改为设置`myPrivateString`的状态，我们会这样做：

```java
Whitebox.setInternalState(instanceOfA, "myPrivateString", "this is my private string", B.class);
```
或者，如果您使用的是PowerMock 1.0或更高版本：

```java
Whitebox.setInternalState(instanceOfA, String.class, "this is my private string", B.class);
```

### 调用私有方法 ###
要调用私有方法，可以使用PowerMock中的`Whitebox.invokeMethod(..)`方法。例如，假设您在实例`myInstance`中有一个名为sum的私有方法，想要单独测试：

```java
private int sum(int a, int b) {
	return a+b;
}
```

为此，您只需执行以下操作：

```java
int sum = Whitebox.<Integer> invokeMethod(myInstance, "sum", 1, 2);
```
然后打印总和将显示3。

这在大多数情况下都可以正常工作。但是，在某些情况下，PowerMock无法使用此简单语法找出要调用的方法。例如，如果重载方法使用原始类型，而另一种方法使用包装器类型，例如：

```java
...
private int myMethod(int id) {		
	return 2*id;
}

private int myMethod(Integer id) {		
		return 3*id;
}
...
```

这当然是一个幼稚的示例，但是您仍然可以使用`Whitebox.invokeMethod(..)`调用这两种方法。对于这种`myMethod(int id)`情况，您会这样做：

```java
int result = Whitebox.<Integer> invokeMethod(myInstance, new Class<?>[]{int.class}, "myMethod", 1);
```

在这里，我们明确告诉PowerMock期望对`myMethod`带有`int`as参数的调用。但是在大多数情况下，您不需要指定参数类型，因为PowerMock会自动找到正确的方法。

您还可以调用类级（静态）方法。假设该`sum`方法也是静态的，在这种情况下，我们可以通过以下方式调用它：

```java
int sum = Whitebox.<Integer> invokeMethod(MyClass.class, "sum", 1, 2);
```

如果方法是在名为的`MyClass`类中定义的。

从PowerMock 1.2.5开始，您还可以在不指定方法名称的情况下调用方法，这对于重构目的非常有用。例如

```java
Whitebox.invokeMethod(myInstance, param1, param2);
```
请注意，这仅在PowerMock可以基于参数类型定位唯一方法的情况下起作用。


### 用私有构造函数实例化一个类 ###

用私有构造函数实例化一个类，例如：

```java
public class PrivateConstructorInstantiationDemo {

	private final int state;

	private PrivateConstructorInstantiationDemo(int state) {
		this.state = state;
	}

	public int getState() {
		return state;
	}
}
```

您只需：

```java
PrivateConstructorInstantiationDemo instance =  WhiteBox.invokeConstructor(
				PrivateConstructorInstantiationDemo.class, 43);
```
其中43是state的参数。

这在大多数情况下都可以正常工作。但是，在某些情况下，PowerMock无法使用此简单语法找出要调用的构造函数。例如，如果重载的构造函数使用原始类型，而另一个构造函数使用包装类型，例如：

```java
public class PrivateConstructorInstantiationDemo {

	private final int state;

	private PrivateConstructorInstantiationDemo(int state) {
		this.state = state;
	}

	private PrivateConstructorInstantiationDemo(Integer state) {
              this.state = state;
              // do something else
	}

	public int getState() {
		return state;
	}
}
```

这当然是一个幼稚的示例，但是您仍然可以使用`Whitebox.invokeConstructor(..)`来调用这两个构造函数。对于`Integer`参数构造函数，您可以这样做：

```java
PrivateConstructorInstantiationDemo instance = Whitebox.invokeConstructor(PrivateConstructorInstantiationDemo.class, new Class<?>[]{Integer.class}, 43);
```

在这里，我们明确告诉PowerMock期望对采用`Integer`作为参数的构造函数的进行调用。但是在大多数情况下，您不需要指定参数类型，因为PowerMock会自动找到正确的方法。

### 注意 ###
所有这些事情都可以在不使用PowerMock的情况下实现，这只是正常的Java反映。但是，反射需要大量样板代码，并且容易出错，因此PowerMock可以为您提供这些实用程序方法。PowerMock使您可以选择是否重构代码并添加用于检查/更改内部状态的getter / setter方法，或者是否使用其工具方法来完成相同的事情而不更改生产代码。由你决定！

###  

### 参考 ###
  * [ServiceHolderTest](https://github.com/powermock/powermock-examples-maven/blob/master/DocumentationExamples/src/test/java/powermock/examples/bypassencapsulation/ServiceHolderTest.java)
  * [ServiceHolder](https://github.com/powermock/powermock-examples-maven/blob/master/DocumentationExamples/src/main/java/powermock/examples/bypassencapsulation/ServiceHolder.java)
  * [ReportGeneratorTest](https://github.com/powermock/powermock-examples-maven/blob/master/DocumentationExamples/src/test/java/powermock/examples/bypassencapsulation/ReportGeneratorTest.java)
  * [ReportGenerator](https://github.com/powermock/powermock-examples-maven/blob/master/DocumentationExamples/src/main/java/powermock/examples/bypassencapsulation/ReportGenerator.java)
  * [ReportDaoTest](https://github.com/powermock/powermock-examples-maven/blob/master/DocumentationExamples/src/test/java/powermock/examples/bypassencapsulation/ReportDaoTest.java)
  * [ReportDao](https://github.com/powermock/powermock-examples-maven/blob/master/DocumentationExamples/src/main/java/powermock/examples/bypassencapsulation/ReportDao.java)
