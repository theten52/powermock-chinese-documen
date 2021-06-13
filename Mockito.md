# 在Mockito中使用PowerMock



## 内容
1. [介绍](#introduction)
1. [支持版本](#supported-versions)
1. [Maven配置](#maven-configuration)
1. [用法](#usage)
    1. [Mock静态方法](#mocking-static-method)
        1. [验证行为](#how-to-verify-behavior)
        1. [参数匹配器](#how-to-use-argument-matchers)
        1. [验证准确的调用次数](#how-to-verify-exact-number-of-calls)
        1. [存根（打桩）静态方法来引发异常](#how-to-stub-void-static-method-to-throw-exception)
        1. [Mock，存根和验证静态方法的完整示例](#a-full-example-for-mocking-stubbing--verifying-static-method)
    1. [部分Mock](#partial-mocking)
        1. [验证行为（方法是否被调用）](#how-to-verify-behavior-1)
        1. [验证私有（方法）行为](#how-to-verify-private-behavior)
        1. [Mock新对象的构造](#how-to-mock-construction-of-new-objects)
        1. [验证新对象的构造](#how-to-verify-construction-of-new-objects)
        1. [使用参数匹配器](#use-argument-matchers-1)
        1. [Spying](#a-full-example-of-spying)
        1. [部分mock私有方法](#a-full-example-of-partial-mocking-of-a-private-method)
1. [Mockito Inline Mock Maker](#mockito-mock-maker-inline)
1. [更多信息](#further-information)
1. [Mockito 1.7](Mockito-Usage-Legacy)

## 介绍 ##
事实上，PowerMock提供了一个名为“PowerMockito”的类，用于创建mock/对象/类并开始验证和期望（一个行为/调用或者返回值），您仍然可以使用Mockito设置和验证期望的所有其他内容（例如times()，anyInt()）。

所有的用法需要`@RunWith(PowerMockRunner.class)`，并在类上标识`@PrepareForTest`注解。

## 支持版本 ##

PowerMock 2.0.0及更高版本具有Mockito 2的支持。

PowerMock 1.7.0及更高版本具有Mockito 2的实验性支持。



许多问题仍未解决。PowerMock使用Mockito 内部API，但至少可以同时使用两个模拟框架。

<table>
<tr><th align='left'> <b><u>Mockito</u></b></th><th align='left'><b><u>PowerMock</u></b></th></tr>
<tr><td>2.8.9+</td><td>2.x</td></tr>
<tr><td>2.8.0-2.8.9</td><td>1.7.x</td></tr>
<tr><td>2.7.5</td><td>1.7.0RC4</td></tr>
<tr><td>2.4.0</td><td>1.7.0RC2</td></tr>
<tr><td>2.0.0-beta - 2.0.42-beta</td><td>1.6.5-1.7.0RC</td></tr>
<tr><td>1.10.8 - 1.10.x</td><td>1.6.2 - 2.0</td></tr>
<tr><td>1.9.5-rc1 - 1.9.5</td><td>1.5.0 - 1.5.6</td></tr>
<tr><td>1.9.0-rc1 & 1.9.0</td><td>1.4.10 - 1.4.12</td></tr>
<tr><td>1.8.5</td><td>1.3.9 - 1.4.9</td></tr>
<tr><td>1.8.4</td><td>1.3.7 & 1.3.8</td></tr>
<tr><td>1.8.3</td><td>1.3.6</td></tr>
<tr><td>1.8.1 & 1.8.2</td><td>1.3.5</td></tr>
<tr><td>1.8 </td><td>1.3</td></tr>
<tr><td>1.7 </td><td>1.2.5</td></tr>
</table>

## Maven配置 ##

### JUnit ###
  * [Mockito JUnit Maven 配置](Mockito-Maven)
  * [Mockito2 JUnit Maven 配置](Mockito-2-Maven)

### TestNG ###
  * [Mockito TestNG Maven 配置](Mockito-2-Maven#TestNG)
  * [Mockito2 TestNG Maven 配置](Mockito-2-Maven#TestNG)


## 用法 ##
在下面的示例中，为了更好地了解方法的位置，我们在Mockito或PowerMockito API中未对方法使用静态导入。但是，我们强烈建议您在实际的测试用例中静态导入方法，以提高可读性。

注意：Mockito团队在Mockito 2.1.0中增加了模拟[final类/方法](https://github.com/mockito/mockito/wiki/What's-new-in-Mockito-2#mock-the-unmockable-opt-in-mocking-of-final-classesmethods)的能力。自PowerMock 1.7.0（经过Mockito 2.8.9测试）以来，PowerMock支持此功能。可以使用[PowerMock Configuration](#mockito-mock-maker-inline)启用该功能。如果使用Mockito 2，建议使用Mockito模拟final方法/类。

### Mock静态方法 ###
如何模拟（mock）和存根（stub）：

1. 在类上添加 `@PrepareForTest`。

  ```java
  @PrepareForTest(Static.class) // Static.class contains static methods
  ```
1. 调用`PowerMockito.mockStatic()`以Mock静态类（用`PowerMockito.spy(class)`mock特定方法）：

  ```java
  PowerMockito.mockStatic(Static.class);
  ```
1. 只需使用Mockito.when()来设置您的期望：

  ```java
  Mockito.when(Static.firstStaticMethod(param)).thenReturn(value);
  ```

注意：如果需要模拟java系统类加载器/bootstrap类加载器（在java.lang或java.net等中定义的类）加载的类，则需要使用[此](Mock-System)方法。

#### 如何验证行为 ####
静态方法的验证分两个步骤进行。
  1. 首先调用`PowerMockito.verifyStatic(Static.class) `以开始验证行为，然后
  1. 调用的静态方法`Static.class`进行验证。例如：

```java
PowerMockito.verifyStatic(Static.class); // 1
Static.firstStaticMethod(param); // 2
```

重要提示：您需要按方法逐个调用`verifyStatic(Static.class)`验证。

#### 如何使用参数匹配器

Mockito匹配器可能仍适用于PowerMock mock。例如，对每个mock的静态方法使用自定义参数匹配器：

```java
PowerMockito.verifyStatic(Static.class);
Static.thirdStaticMethod(Mockito.anyInt());
```

#### 如何验证确切的调用次数

您仍然可以将Mockito.VerificationMode（例如Mockito.times(x)）与`PowerMockito.verifyStatic(Static.class, Mockito.times(2))`结合使用：

```java
PowerMockito.verifyStatic(Static.class, Mockito.times(1));
```

#### 如何将静态void方法存根以引发异常 ####
如果不是私有方法:

```java
PowerMockito.doThrow(new ArrayStoreException("Mock error")).when(StaticService.class);
StaticService.executeMethod();
```
请注意，您可以对final类/final方法执行相同的操作：

```java
PowerMockito.doThrow(new ArrayStoreException("Mock error")).when(myFinalMock).myFinalMethod();
```
对于私有方法，请使用PowerMockito.when，例如：

```java
when(tested, "methodToExpect", argument).thenReturn(myReturnValue);
```

#### Mock，存根和验证静态方法的完整示例

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(Static.class)
public class YourTestCase {
    @Test
    public void testMethodThatCallsStaticMethod() {
        // mock all the static methods in a class called "Static"
        PowerMockito.mockStatic(Static.class);
        // use Mockito to set up your expectation
        Mockito.when(Static.firstStaticMethod(param)).thenReturn(value);
        Mockito.when(Static.secondStaticMethod()).thenReturn(123);

        // execute your test
        classCallStaticMethodObj.execute();

        // Different from Mockito, always use PowerMockito.verifyStatic(Class) first
        // to start verifying behavior
        PowerMockito.verifyStatic(Static.class, Mockito.times(2));
        // IMPORTANT:  Call the static method you want to verify
        Static.firstStaticMethod(param);


        // IMPORTANT: You need to call verifyStatic(Class) per method verification,
        // so call verifyStatic(Class) again
        PowerMockito.verifyStatic(Static.class); // default times is once
        // Again call the static method which is being verified
        Static.secondStaticMethod();

        // Again, remember to call verifyStatic(Class)
        PowerMockito.verifyStatic(Static.class, Mockito.never());
        // And again call the static method.
        Static.thirdStaticMethod();
    }
}
```

### 部分Mock ###
您可以通过使用PowerMockito的PowerMockito.spy部分mock方法。请小心（以下内容取自Mockito文档，同样适用于PowerMockito）：

有时，无法使用标准`when(..)`方法对spy()进行打桩（stubbing，存根）。例如：

```java
List list = new LinkedList();
List spy = spy(list);
//Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
when(spy.get(0)).thenReturn("foo");

//You have to use doReturn() for stubbing
doReturn("foo").when(spy).get(0);
```

### 如何验证行为 ###

只需使用Mockito.vertify()进行标准验证：

```java
Mockito.verify(mockObj, times(2)).methodToMock();
```

### 如何验证private行为 ###
使用PowerMockito.verifyPrivate()，例如

```java
verifyPrivate(tested).invoke("privateMethodName", argument1);
```

这也适用于私有静态方法。

### 如何mock新对象的构造

使用PowerMockito.whenNew，例如

```java
whenNew(MyClass.class).withNoArguments().thenThrow(new IOException("error message"));
```
请注意，您必须准备在测试过程中的用于*创建*`MyClass`新实例中的类，而不是其`MyClass`本身。例如，如果正在执行的类称为X，则必须先进行操作`@PrepareForTest(X.class)`才能使`whenNew`工作：

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(X.class)
public class XTest {
  	@Test
  	public void test() {
      	whenNew(MyClass.class).withNoArguments().thenThrow(new IOException("error message"));

      	X x = new X();
      	x.y(); // y is the method doing "new MyClass()"

      	..
    }
}
```

### 如何验证新对象的构造

使用PowerMockito.verifyNew，例如

```java
verifyNew(MyClass.class).withNoArguments();
```

### 如何使用参数匹配器

如何使用参数匹配器mock:

```java
Mockito.verify(mockObj).methodToMock(Mockito.anyInt());  
```


### Spy的完整示例 ###
```java
@RunWith(PowerMockRunner.class)
// We prepare PartialMockClass for test because it's final or we need to mock private or static methods
@PrepareForTest(PartialMockClass.class)
public class YourTestCase {
    @Test
    public void spyingWithPowerMock() {        
        PartialMockClass classUnderTest = PowerMockito.spy(new PartialMockClass());

        // use Mockito to set up your expectation
        Mockito.when(classUnderTest.methodToMock()).thenReturn(value);

        // execute your test
        classUnderTest.execute();

        // Use Mockito.verify() to verify result
        Mockito.verify(mockObj, times(2)).methodToMock();
    }
}
```

### 私有方法部分mock的完整示例 ###
（在PowerMock版本1.3.6+中可用）

```java
@RunWith(PowerMockRunner.class)
// We prepare PartialMockClass for test because it's final or we need to mock private or static methods
@PrepareForTest(PartialMockClass.class)
public class YourTestCase {
    @Test
    public void privatePartialMockingWithPowerMock() {        
        PartialMockClass classUnderTest = PowerMockito.spy(new PartialMockClass());

        // use PowerMockito to set up your expectation
        PowerMockito.doReturn(value).when(classUnderTest, "methodToMock", "parameter1");

        // execute your test
        classUnderTest.execute();

        // Use PowerMockito.verify() to verify result
        PowerMockito.verifyPrivate(classUnderTest, times(2)).invoke("methodToMock", "parameter1");
    }
}
```

## Mockito mock-maker-inline ##

```
在PowerMock版本1.7.0中可用
```

Mockito团队在Mockito 2.1.0中增加了对[final类/方法](https://github.com/mockito/mockito/wiki/What's-new-in-Mockito-2#mock-the-unmockable-opt-in-mocking-of-final-classesmethods)进行mock的支持。通过在`src/test/resources/mockito-extensions/org.mockito.plugins.MockMaker`中添加包含以下一行的文件，可以在PowerMock中启用此功能 ：

```
mock-maker-inline
```

PowerMock实现了自己的`MockMaker`，这导致与Mockito的`mock-maker-inline`不兼容，即使PowerMock只是作为依赖添加而未使用。如果在类路径中存在两个`org.mockito.plugins.MockMaker`，则只能使用其中一个，但不确定是哪一个。

但是PowerMock可以将调用委托给另一个`MockMaker`，因此无需PowerMock即可运行测试。从PowerMock 1.7.0开始，可以使用[PowerMock Configuration进行配置](PowerMock-Configuration)。

该`MockMaker`可以通过创建`org/powermock/extensions/configuration.properties`文件来配置和设置：

```properties
mockito.mock-maker-class=mock-maker-inline
```

将Mockito`mock-maker-inline`与PowerMock结合使用的示例：
https://github.com/powermock/powermock-examples-maven/tree/master/mockito2

## 更多信息 ##
请查看GitHub中的示例[源码](https://github.com/powermock/powermock/tree/master/tests/mockito/junit4/src/test/java/samples/powermockito/junit4)。也可以参考在[Jayway团队博客](http://blog.jayway.com/)中PowerMockito的相关[博客](http://blog.jayway.com/2009/10/28/untestable-code-with-mockito-and-powermock/)。