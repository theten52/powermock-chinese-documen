# 测试监听器 #

## 总览 ##

  1. 在类上使用 `@RunWith(PowerMockRunner.class)` 注解。
  1. 在类上使用 `@PowerMockListener({Listener1.class, Listener2.class})` 注解。

## 示例 ##

PowerMock 1.1及更高版本具有测试侦听器的概念。测试侦听器可用于从测试框架获取事件，例如测试方法的开始和结束时间以及测试执行的结果。这些测试侦听器的目的是提供一种独立于测试框架的方法，以通过实现`org.powermock.core.spi.PowerMockTestListener`并将它们传递给`PowerMockListener`注解来获取和响应这些通知。PowerMock具有一些内置的测试侦听器供您使用。

### AnnotationEnabler ###
**注意:** 从PowerMock 1.3开始，您不再需要手动指定此侦听器。现在，它已集成到运行器中，并且自动注入了Mock。


该测试侦听器实现使您可以使用注解创建模拟对象。例如，考虑以下代码：

```java
@RunWith(PowerMockRunner.class)
@PowerMockListener(AnnotationEnabler.class)
public class PersonServiceTest {

    @Mock
    private PersonDao personDaoMock;

    private PersonService classUnderTest;

    @Before
    public void setUp() {
        classUnderTest = new PersonService(personDaoMock);
    }
    ...
}
```

使用@Mock注解消除了手动设置和拆卸模拟的需要，从而最大程度地减少了重复的测试代码并使测试更具可读性。AnnotationEnabler适用于EasyMock和Mockito API。在EasyMock版本中，如果要创建部分模拟，还可以提供希望模拟的方法的名称，例如：

```java
@RunWith(PowerMockRunner.class)
@PowerMockListener(AnnotationEnabler.class)
public class PersonServiceTest {

    @Mock("getPerson")
    private PersonDao personDaoMock;

    private PersonService classUnderTest;

    @Before
    public void setUp() {
        classUnderTest = new PersonService(personDaoMock);
    }
    ...
}
```
这段代码将指示PowerMock创建`PersonDao`且仅模拟“getPerson”方法的部分模拟。由于EasyMock支持良好且严格的Mock，因此您可以使用`@MockNice`和`@MockStrict`注解来获得此好处。

在Mockito中，您只需使用`spy(..)`部分模拟类或实例即可。

### FieldDefaulter ###
此测试侦听器实现可用于在每次测试后为junit测试中的每个成员字段设置默认值。对于许多开发人员来说，使用JUnit时，创建一个tearDown方法并清空所有引用几乎是一个标准过程（您可以[在此处](http://blogs.atlassian.com/developer/2005/12/reducing_junit_memory_usage.html)阅读有关此问题的更多信息）。但是，也可以使用FieldDefaulter自动完成此操作。举例来说，假设您有5个要在测试中模拟的协作者，并且要确保在每次测试后将每个协作者都设置为null，以允许对其进行垃圾回收。因此，与其做：

```java
@RunWith(PowerMockRunner.class)
public class MyTest {
	
  	private Collaborator1 collaborator1Mock;
  	private Collaborator2 collaborator2Mock;
  	private Collaborator3 collaborator3Mock;
  	private Collaborator4 collaborator4Mock;
  	private Collaborator5 collaborator5Mock;

        ...
  	@After
  	public void tearDown() {
            collaborator1Mock = null;
            collaborator2Mock = null;
            collaborator3Mock = null;
            collaborator4Mock = null;
            collaborator5Mock = null;
  	}
        ...

}
```
您可以使用`FieldDefaulter`测试侦听器完全摆脱tear-down方法：

```java
@RunWith(PowerMockRunner.class)
@PowerMockListener(FieldDefaulter.class)
public class MyTest {
	
  	private Collaborator1 collaborator1Mock;
  	private Collaborator2 collaborator2Mock;
  	private Collaborator3 collaborator3Mock;
  	private Collaborator4 collaborator4Mock;
  	private Collaborator5 collaborator5Mock;

        ...
}
```