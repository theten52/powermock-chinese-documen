# 入门

## API's ##
PowerMock由两个扩展API组成。

一个用于[EasyMock](EasyMock)，一个用于[Mockito](Mockito)。要使用PowerMock，您需要依赖这些API之一以及测试框架。

当前PowerMock支持JUnit和[TestNG](TestNG)。共有三种不同的JUnit测试执行器，一种用于JUnit 4.4-4.12，一种用于JUnit 4.0-4.3。从PowerMock 2.0开始，不再提供JUnit 3的测试执行器。

有一个TestNG的测试执行器，根据您使用的PowerMock版本，它需要5.11+版本。

```
   注意: 从 PowerMock 2.0 之后对 jUnit 3 的支持已被移除。
```

## 编写测试 ##

编写这样的测试：

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({YourClassWithEgStaticMethod.class})
public class YourTestCase {
  ...
}
```

## Maven 设置 ##

### JUnit ###
  * [EasyMock JUnit Maven 设置](EasyMock-Maven)
  * [Mockito JUnit Maven 设置](Mockito-Maven)
  * [Mockito2 JUnit Maven 设置](Mockito-2-Maven)

### TestNG ###
  * [EasyMock TestNG Maven 设置](EasyMock-Maven)
  * [Mockito TestNG Maven 设置](Mockito-Maven#testng)
  * [Mockito2 TestNG Maven 设置](Mockito-2-Maven#testng)

## 非maven用户 ##

### EasyMock

JUnit：下载带有PowerMock及其所有依赖项的[EasyMock](https://dl.bintray.com/powermock/generic/distributions/powermock-easymock-junit-2.0.2.zip) zip文件，并将其添加到您的项目中。

TestNG：下载带有PowerMock及其所有依赖项的[EasyMock](https://dl.bintray.com/powermock/generic/distributions/powermock-easymock-testng-2.0.2.zip) zip文件，并将其添加到您的项目中。

### Mockito

JUnit：下载带有PowerMock及其所有依赖项的[Mockito](https://dl.bintray.com/powermock/generic/distributions/powermock-mockito2-junit-2.0.2.zip) zip文件，并将其添加到您的项目中。

TestNG：下载带有PowerMock及其所有依赖项的[Mockito](https://dl.bintray.com/powermock/generic/distributions/powermock-mockito2-testng-2.0.2.zip) zip文件，并将其添加到您的项目中。



## 是否需要将PowerMock与另一个JUnit运行程序结合使用？

  * 首先尝试[JUnit委托运行器](JUnit_Delegating_Runner)，如果不起作用，则尝试使用[PowerMockRule](PowerMockRule)或[PowerMock Java代理](PowerMockAgent)。

## 是否需要使用JUnit rule进行引导？

  * [PowerMockRule](PowerMockRule)可以实现这种情况。

## 基于Java agent的引导

如果你使用PowerMock且具有类加载的问题，使用[PowerMock的Java Agent](PowerMockAgent)。

## 需要教程吗？

1.使用以下命令从git克隆项目：

 使用 SSH
  ```bash
  git clone git@github.com:powermock/powermock-examples-maven.git
  ```

 使用 HTTPS
  ```bash
  git clone https://github.com/powermock/powermock-examples-maven.git
  ```

2.签出版本1.7.1：

  ```bash
  git checkout powermock-1.7.1
  ```

3.进入`examples/tutorial/`文件夹，并按照[readme.txt]()文件中的说明进行操作。