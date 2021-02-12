# 抑制不需要的行为 #

## 总览 ##
  1. 在测试用例的类上使用 `@RunWith(PowerMockRunner.class)`注解.
  1. 在测试用例的类级别结合使用`@PrepareForTest(ClassWithEvilParentConstructor.class)`和`suppress(constructor(EvilParent.class))`注解，以禁止调用EvilParent类的所有构造函数。
  1. 使用`Whitebox.newInstance(ClassWithEvilConstructor.class)` 方法去实例化一个类而无须调用其构造函数。
  1. 使用`@SuppressStaticInitializationFor("org.mycompany.ClassWithEvilStaticInitializer")`注解移除类`org.mycompany.ClassWithEvilStaticInitializer`的静态初始化器。
  1. 在类级别使用 `@PrepareForTest(ClassWithEvilMethod.class)` 注解结合`suppress(method(ClassWithEvilMethod.class, "methodName"))` 去抑制类ClassWithEvilMethod中的 "methodName" 方法。
  1. 在类级别使用 `@PrepareForTest(ClassWithEvilField.class)` 注解结合 `suppress(field(ClassWithEvilField.class, "fieldName"))` 去抑制类 ClassWithEvilField 中的"fieldName" 字段.

您可以在此处找到成员修改和成员匹配器方法：
  * `org.powermock.api.support.membermodification.MemberModifier`
  * `org.powermock.api.support.membermodification.MemberMatcher`

## 示例 ##
有时，您甚至希望抑制某些构造函数，方法或静态初始化程序的行为，以便对您自己的代码进行单元测试。一个典型的例子是您的类需要从某种第三方框架中的另一个类扩展而来。当第3方类在其构造函数中执行某些操作而导致您无法对自己的代码进行单元测试时，就会出现问题。例如，由于某种原因，框架可能尝试加载dll或访问网络或文件系统。让我们看一些例子。

### 禁止超类构造函数 ###
作为示例，让我们看一下名为`ExampleWithEvilParent`的类，它非常简单：

```java
public class ExampleWithEvilParent extends EvilParent {

	private final String message;

	public ExampleWithEvilParent(String message) {
		this.message = message;
	}

	public String getMessage() {
		return message;
	}
}
```
这似乎是一个易于进行单元测试的类（实际上如此简单，您可能不应该对其进行测试，但是为了演示起见，让我们继续进行测试）。但是，等等，让我们看一下`EvilParent`类的样子：

```java
public class EvilParent {

	public EvilParent() {
		System.loadLibrary("evil.dll");
	}
}
```
该父类尝试加载一个dll文件，该文件在您对`ExampleWithEvilParent`类运行单元测试时将不存在。使用PowerMock，您可以仅抑制EvilParent的构造函数，以便可以对ExampleWithEvilParent类进行单元测试。这是通过使用PowerMock API中的`suppress`方法完成的。在这种情况下，我们将执行以下操作：

```java
suppress(constructor(EvilParent.class));
```

您还必须准备ExampleWithEvilParent以进行测试（因为从此类中调用了EvilParent的构造函数）。您可以通过将传递`ExampleWithEvilParent.class`给`@PrepareForTest`注解来实现：

```java
@PrepareForTest(ExampleWithEvilParent.class)
```

完整的测试如下所示：

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(ExampleWithEvilParent.class)
public class ExampleWithEvilParentTest {

	@Test
	public void testSuppressConstructorOfEvilParent() throws Exception {
		suppress(constructor(EvilParent.class));
		final String message = "myMessage";
		ExampleWithEvilParent tested = new ExampleWithEvilParent(message);
		assertEquals(message, tested.getMessage());
	}
}
```
如果超类具有多个构造函数，则可以告诉PowerMock仅抑制特定的构造函数。假设您有一个名为`ClassWithSeveralConstructors`的类，该类的一个构造函数采用`String`而另一个构造函数采用`int`作为参数，而您只想移除采用`String`构造函数。您可以使用

```java
suppress(constructor(ClassWithSeveralConstructors.class, String.class));
```
方法。

### 抑制自己的构造函数 ###
上面的示例在抑制超类构造函数和被测类时起作用。抑制被测构造函数的另一种方法是使用`Whitebox.newInstance`方法。如果您自己的代码在其构造函数中执行了某些操作，导致难以进行单元测试。这将实例化该类，而根本不调用构造函数。例如，假设您要对以下类进行单元测试：

```java
public class ExampleWithEvilConstructor {

	private final String message;

	public ExampleWithEvilConstructor(String message) {
		System.loadLibrary("evil.dll");
		this.message = message;
	}

	public String getMessage() {
		return message;
	}
}
```
要实例化此类，我们使用以下`Whitebox.newInstance`方法：

```java
ExampleWithEvilConstructor tested = Whitebox.newInstance(ExampleWithEvilConstructor.class);
```
完整的测试如下所示：

```java
public class ExampleWithEvilConstructorTest {

	@Test
	public void testSuppressOwnConstructor() throws Exception {
		ExampleWithEvilConstructor tested = Whitebox.newInstance(ExampleWithEvilConstructor.class);
		assertNull(tested.getMessage());
	}
}
```
请注意，您不需要使用`@RunWith(..)`注解或将类传递给`@PrepareForTest`注解。这样做并没有什么害处，但这不是必需的。要理解的另一件重要事情是，由于从不执行构造函数，因此该`tested.getMessage()`方法将返回`null`。您当然可以通过[绕过封装](BypassEncapsulation)来更改测试实例中message字段的状态。

### 抑制方法 ###
在某些情况下，您只想抑制一个方法并使其返回一些默认值，在其他情况下，您可能*需要*抑制或模拟一个方法，因为该方法会阻止您对自己的类进行单元测试。看下面的虚构的示例：

```java
public class ExampleWithEvilMethod {

	private final String message;

	public ExampleWithEvilMethod(String message) {
		this.message = message;
	}

	public String getMessage() {
		return message + getEvilMessage();
	}

	private String getEvilMessage() {
		System.loadLibrary("evil.dll");
		return "evil!";
	}
}
```
如果在测试`getMessage()`方法时执行了`System.loadLibrary("evil.dll")`语句，则测试将失败。避免这种情况的一种简单方法是简单地抑制`getEvilMessage`方法。您可以使用PowerMock API中的`suppress(method(..))`的方法来执行此操作，在这种情况下，您可以执行以下操作：

```java
suppress(method(ExampleWithEvilMethod.class, "getEvilMessage"));
```
完整的测试如下所示：

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(ExampleWithEvilMethod.class)
public class ExampleWithEvilMethodTest {

	@Test
	public void testSuppressMethod() throws Exception {
		suppress(method(ExampleWithEvilMethod.class, "getEvilMessage"));
		final String message = "myMessage";
    //TODO 如何返回一个getEvilMessage方法的非默认值？
		ExampleWithEvilMethod tested = new ExampleWithEvilMethod(message);
		assertEquals(message, tested.getMessage());
	}
}
```

### 抑制静态初始化器 ###
有时，第三方类在其静态初始化程序（也称为静态构造函数）中执行某些操作，从而阻止您对自己的类进行单元测试。您自己的类也有可能在静态初始化器中执行某些操作，而您在单元测试类时不希望发生这种情况。因此，PowerMock可以简单地抑制该类的静态初始化。为此，您可以在测试的类级别或方法级别指定`@SuppressStaticInitializationFor`注解。例如，假设您要对以下类进行单元测试：

```java
public class ExampleWithEvilStaticInitializer {

	static {
		System.loadLibrary("evil.dll");
	}

	private final String message;

	public ExampleWithEvilStaticInitializer(String message) {
		this.message = message;
	}

	public String getMessage() {
		return message;
	}
}
```
这里的问题是，当加载ExampleWithEvilStaticInitializer类时，将执行静态代码块，并且`System.loadLibrary("evil.dll")`代码块将被执行，从而导致单元测试失败（无法加载evil.dll）。为了抑制此静态初始化器，我们这样做：

```java
@SuppressStaticInitializationFor("org.mycompany.ExampleWithEvilStaticInitializer")
```

如您所见，我们没有将`ExampleWithEvilStaticInitializer.class`传递给`@SuppressStaticInitializationFor`而是给了它全称的类名。原因是，如果要传递`ExampleWithEvilStaticInitializer.class`给注解，则静态初始化程序将在测试开始之前运行，因此测试将失败。因此，在删除静态初始化器时，必须将全限定名传递给该类。整个测试需要：

```java
@RunWith(PowerMockRunner.class)
@SuppressStaticInitializationFor("org.mycompany.ExampleWithEvilStaticInitializer")
public class ExampleWithEvilStaticInitializerTest {

	@Test
	public void testSuppressStaticInitializer() throws Exception {
		final String message = "myMessage";
		ExampleWithEvilStaticInitializer tested = new ExampleWithEvilStaticInitializer(message);
		assertEquals(message, tested.getMessage());
	}
}
```

### 抑制字段 ###
You can also suppress fields using `suppress(..)`. For example let's say you have to following class:

您也可以使用`suppress(..)`抑制字段。例如，假设您有以下类：

```java
public class MyClass {
	private MyObject myObject = new MyObject();


	public MyObject getMyObject() {
		return myObject;
	}
}
```

To suppress the `myObject` field above you can do:

要抑制上面的`myObject`字段，您可以执行以下操作：

```java
suppress(field(MyClass.class, "myObject"));
```
Invoking the `getMyObject()` will then return `null` for every instance of `MyClass` when it has been prepared for test.

调用`getMyObject()`之后，将为准备进行测试的每个`MyClass`实例返回`null`。

## References ##
  * [ExampleWithEvilParentTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/suppress/constructor/ExampleWithEvilParentTest.java)
  * [ExampleWithEvilConstructorTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/suppress/constructor/ExampleWithEvilConstructorTest.java)
  * [ExampleWithEvilMethodTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/suppress/method/ExampleWithEvilMethodTest.java)
  * [ExampleWithEvilStaticInitializerTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/suppress/staticinitializer/ExampleWithEvilStaticInitializerTest.java)
  * [ExampleWithEvilParent](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/suppress/constructor/ExampleWithEvilParent.java)
  * [EvilParent](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/suppress/constructor/EvilParent.java)
  * [ExampleWithEvilConstructor](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/suppress/constructor/ExampleWithEvilConstructor.java)
  * [ExampleWithEvilMethod](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/suppress/method/ExampleWithEvilMethod.java)
  * [ExampleWithEvilStaticInitializer](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/suppress/staticinitializer/ExampleWithEvilStaticInitializer.java)