# Mockito 简介（基于3.11.1）

## Mockito 主要功能：“创建mock对象”、“验证交互”和“存根（stub）方法调用”。

名词解释：

- stub：存根。即配置mock对象的某个方法被调用时该返回什么样的结果的过程。
- 交互：发生了某个方法的调用。
- spy：监视，间谍。

### 验证交互：

```java
import static org.mockito.Mockito.*;

// 创建mock对象
List mockedList = mock(List.class);

// 使用mock对象 - 它不会抛出任何“非期望的交互”异常
mockedList.add("one");
mockedList.clear();

// 可选的、明确的，可读性强的验证
verify(mockedList).add("one");
verify(mockedList).clear();
```

### 存根方法调用：

```java
// 你可以mock具体的类，不仅仅是mock接口
LinkedList mockedList = mock(LinkedList.class);

// 在实际调用之前存根
when(mockedList.get(0)).thenReturn("first");

// 此处会打印“first”
System.out.println(mockedList.get(0));

// 此处会打印"null"因为get(999)方法没有被存根
System.out.println(mockedList.get(999));
```

### 更多：

- [`mock()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#mock-java.lang.Class-)/[`@Mock`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mock.html): 创建mock对象
  - 具体的行为可以通过 [`Answer`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/stubbing/Answer.html)/[`MockSettings`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/MockSettings.html)指定
  - [`when()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#when-T-)/[`given()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/BDDMockito.html#given-T-) 指定mock对象应该具有的行为
  - 如果mock对象提供的结果不是你需要的，你可以通过继承 [`Answer`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/stubbing/Answer.html)接口自己进行扩展。
- [`spy()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#spy-T-)/[`@Spy`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Spy.html): 部分mock, 真正的方法会被调用但是仍然可以被验证和存根。
- [`@InjectMocks`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/InjectMocks.html): 自动使用主机注入 mocks/spies 字段通过 `@Spy` 或 `@Mock`
- [`verify()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#verify-T-): 检查方法使用被给定的参数调用过
  - 可以使用灵活的参数匹配,比如通过 [`any()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/ArgumentMatchers.html#any--)表示任何表达式
  - 或者捕获使用 [`@Captor`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Captor.html) 调用的参数
- 用 [BDDMockito](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/BDDMockito.html)尝试行为驱动开发的语法
- 在Android上使用 Mockito, 感谢 [dexmaker](https://github.com/crittercism/dexmaker)团队的工作

### 记住：

- 不要mock不属于你的类型
- 不要mock值对象
- 不要mock所有的东西
- 爱你的测试！

点击 [这里](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html) 查看更对的的文档和示例. 所有的文档都是使用JavaDoc生成的所以你不需要经常查出此文档。这儿也有一个 [RefCard](https://dzone.com/refcardz/mockito).

如果您有建议，文档查找不清楚，或发现bug，请写信到我们的[邮件列表](http://groups.google.com/group/mockito)。 您可以在 [GitHub](https://github.com/mockito/mockito/issues)中报告功能请求和错误。



## Mockito 文档

Mockito 库允许创建mock对象，验证方法代用和存根方法调用。

### 1.验证mock对象的行为（方法是否被调用以及调用返回值）

```java
 //让我们静态导入Mockito是我们的代码看起来更干净一点
 import static org.mockito.Mockito.*;

 //创建mock对象
 List mockedList = mock(List.class);

 //使用mock对象
 mockedList.add("one");
 mockedList.clear();

 //验证
 verify(mockedList).add("one");
 verify(mockedList).clear();
```

### 2.添加一些存根（stub）：指定mock对象方法调用的返回值

```java
 //你可以mock具体的类，而不仅仅是接口
 LinkedList mockedList = mock(LinkedList.class);

 //添加存根
 when(mockedList.get(0)).thenReturn("first");
 when(mockedList.get(1)).thenThrow(new RuntimeException());

 //输出"first"
 System.out.println(mockedList.get(0));

 //抛出运行时异常
 System.out.println(mockedList.get(1));

 //输出"null"因为get(999)没有做存根
 System.out.println(mockedList.get(999));

 //虽然可以验证存根方法的调用，但通常它只是多余的。
 //如果你的代码关心get(0)的返回值, 那么有些东西就被打破了 (通常在调用verify()之前)。
 //如果你的代码不关心get(0)的返回值, 那么它就不应该被存根。
 verify(mockedList).get(0);

```

- 默认情况下，对于所有方法的返回值，mock将返回 null、原始/原始包装值或空集合，视情况而定。例如 int/Integer 返回0，布尔值/布尔值返回false。
- 存根可以被覆盖：例如，普通存根可以进入夹具设置（fixture setup），但测试方法可以覆盖它。请注意，覆盖存根是一种潜在的代码异味，表示存根过多。
- 一旦被存根，该方法将始终返回一个存根值，无论它被调用多少次。
- 最后的存根更重要 - 当您多次用相同的参数存根相同的方法时。换句话说：**存根的顺序**很**重要，**但它只是很少有意义，例如当存根完全相同的方法调用或有时使用参数匹配器时等。

#### 参数匹配

Mockito验证参数值使用自然java风格。即通过使用`equals()`方法。有时，当需要额外的灵活性时，您可以使用参数匹配器：

```java

 //使用内置的anyInt()参数匹配器创建存根
 when(mockedList.get(anyInt())).thenReturn("element");

 //使用自定义的参数匹配器isValid()
 when(mockedList.contains(argThat(isValid()))).thenReturn(true);

 //输出"element"
 System.out.println(mockedList.get(999));

 //你也使用使用参数匹配器进行验证
 verify(mockedList).get(anyInt());

 //参数匹配器也可以写成java 8 Lambdas的方式
 verify(mockedList).add(argThat(someString -> someString.length() > 5));
```

参数配器允许灵活的验证或存根。 查看更多内置匹配器和**自定义参数匹配器/ hamcrest 匹配器的**示例。 [`点击这里`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html) [`或这里。`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/hamcrest/MockitoHamcrest.html)

有关仅**自定义参数匹配器的信息，**请查看[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)类的javadoc 。

合理使用复杂的参数匹配。偶尔使用`equals()`结合`anyX()`匹配器的自然匹配风格往往会提供干净和简单的测试。有时最好重构代码以允许`equals()`匹配甚至实现`equals()`方法来帮助测试。

另外，请阅读[第 15 节](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#15)或 javadoc 以了解[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)类。 [`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)是参数匹配器的一种特殊实现，它捕获参数值以进行进一步的断言。

**参数匹配器警告：**

如果您使用参数匹配器，则**所有参数**都必须由匹配器提供。

以下示例展示了验证，但同样适用于存根：

```java
   verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
   //上面的写法是正确的 - eq() 也是参数匹配器

   verify(mock).someMethod(anyInt(), anyString(), "third argument");
   //上面的写法是错误的 - 异常会被抛出因为第三个参数并不是参数匹配器的形式
```

匹配器方法如`anyObject()`，`eq()` **不会**返回匹配器。在内部，它们在堆栈上记录一个匹配器并返回一个虚拟值（通常为空）。此实现是由于 java 编译器强加的静态类型安全。结果是您不能在验证/存根之外的方法使用`anyObject()`,`eq()`方法。

### 4.验证确切的调用次数/至少调用x次/从未调用

```java
 //使用mock对象
 mockedList.add("once");

 mockedList.add("twice");
 mockedList.add("twice");

 mockedList.add("three times");
 mockedList.add("three times");
 mockedList.add("three times");

 //下面两个验证的效果相同 - times(1) 是默认的使用的
 verify(mockedList).add("once");
 verify(mockedList, times(1)).add("once");

 //确切的调用次数验证
 verify(mockedList, times(2)).add("twice");
 verify(mockedList, times(3)).add("three times");

 //使用never()验证。 never() 是 times(0) 的别名
 verify(mockedList, never()).add("never happened");

 //使用atLeast()/atMost()验证
 verify(mockedList, atMostOnce()).add("once");
 verify(mockedList, atLeastOnce()).add("three times");
 verify(mockedList, atLeast(2)).add("three times");
 verify(mockedList, atMost(5)).add("three times");
```

**times(1) 是默认值。**因此可以省略显式使用 times(1) 。

#### 5.存根有异常的void方法

```java

   doThrow(new RuntimeException()).when(mockedList).clear();

   //下面的代码会抛出异常
   mockedList.clear();
```

在 [第 12 节中](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#12)阅读更多关于`doThrow()`|`doAnswer()`方法族的信息。

#### 6.调用顺序验证

```java
 // A.必须以特定顺序调用其方法的单个mock对象
 List singleMock = mock(List.class);

 //使用单个mock对象
 singleMock.add("was added first");
 singleMock.add("was added second");

 //为单个mock对象创建一个 inOrder 验证
 InOrder inOrder = inOrder(singleMock);

 //以下代码会确认首先add方法被"was added first"调用，其次被"was added second"调用
 inOrder.verify(singleMock).add("was added first");
 inOrder.verify(singleMock).add("was added second");

 // B. 必须以特定顺序调用其方法的多个mock对象
 List firstMock = mock(List.class);
 List secondMock = mock(List.class);

 //使用mock对象
 firstMock.add("was called first");
 secondMock.add("was called second");

 //创建 inOrder 对象通过多个需要被按顺序校验的mock对象
 InOrder inOrder = inOrder(firstMock, secondMock);

 //以下代码会确认 firstMock 是在 secondMock 之前调用的
 inOrder.verify(firstMock).add("was called first");
 inOrder.verify(secondMock).add("was called second");

 //A + B 可以被混合在一起使用 
```

按顺序验证是灵活的 - **您不必**一一**验证所有交互**，而只需按顺序验证您感兴趣的那些**交互**。

此外，您可以创建一个 InOrder 对象，仅传递与有序验证相关的moc k对象。

#### 7.确保在mock对象从未发生交互

```java

 //使用mock对象 - 只有 mockOne 是可交互的
 mockOne.add("one");

 //普通验证
 verify(mockOne).add("one");

 //验证某个方法从未被调用
 verify(mockOne, never()).add("two");

 //验证其他方法从未发送过交互
 verifyZeroInteractions(mockTwo, mockThree);
```

#### 寻找多余的调用

```java

 //使用mock对象
 mockedList.add("one");
 mockedList.add("two");

 verify(mockedList).add("one");

 //下面的验证会失败
 verifyNoMoreInteractions(mockedList);
```

一句**警告**：一些做过很多经典的、expect-run-verify mock的用户往往会经常使用`verifyNoMoreInteractions()`，即使在每个测试方法中也是如此。 `verifyNoMoreInteractions()`不建议在每个测试方法中使用。 `verifyNoMoreInteractions()`是来自交互测试工具包的一个方便的断言。仅在相关时使用它。滥用它会导致**过度指定**、**不易维护的**测试。

另请参阅[`never()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#never--)- 它更明确并且能很好地传达意图。

#### 9.mocks的简单创建方式--[`@Mock`注解](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mock_annotation)

- 最大限度地减少重复的mock创建代码。
- 使测试类更具可读性。
- 使验证错误更易于阅读，因为**字段名称**会被用于标识mock。

```java
public class ArticleManagerTest {

       @Mock private ArticleCalculator calculator;
       @Mock private ArticleDatabase database;
       @Mock private UserProvider userProvider;

       private ArticleManager manager;

       @org.junit.jupiter.api.Test
       void testSomethingInJunit5(@Mock ArticleDatabase database) {
             //do something
       }
}
```

**重要的：以下代码需要在基类或测试运行器的某个地方：

```java
 MockitoAnnotations.openMocks(testClass);
```

您可以使用内置 runner:[`MockitoJUnitRunner`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnitRunner.html)或规则: [`MockitoRule`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoRule.html)。对于 JUnit5 测试，请参阅[第 45 节中](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#45)描述的 JUnit5 扩展。

在此处阅读更多信息： [`MockitoAnnotations`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoAnnotations.html)

#### 10. [存根连续调用](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#stubbing_consecutive_calls)（iterator-style stubbing）

有时我们需要为同一个方法调用使用不同的返回值/异常进行存根。典型的用例可能是mock迭代器。Mockito 的原始版本没有这个功能来促进简单的mock。例如，可以使用[`Iterable`](https://docs.oracle.com/javase/6/docs/api/java/lang/Iterable.html?is-external=true)或简单地集合来代替迭代器。这些提供了自然的存根方式（例如使用真实的集合）。不过，在极少数情况下，存根连续调用可能很有用：

```java

 when(mock.someMethod("some arg"))
   .thenThrow(new RuntimeException())
   .thenReturn("foo");

 //第一次调用: 抛出运行时异常:
 mock.someMethod("some arg");

 //第二次调用: 输出 "foo"
 System.out.println(mock.someMethod("some arg"));

 //任何接下来的调用: 会输出 "foo" (最后的存根会起作用).
 System.out.println(mock.someMethod("some arg"));

```

也可以是另一种方式，较短版本的连续存根：

```java
 when(mock.someMethod("some arg")).thenReturn("one", "two", "three");
```

**警告**：如果不是连续的`.thenReturn()`调用，而是使用具有相同匹配器或参数的多个存根，那么后面的存根将覆盖前一个：

```java
 //所有的 mock.someMethod("some arg") 调用会返回 "two"
 when(mock.someMethod("some arg"))
   .thenReturn("one")
 when(mock.someMethod("some arg"))
   .thenReturn("two")
 
```

#### 11.[使用回调进行存根](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#answer_stubs)

允许使用[`Answer`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/stubbing/Answer.html)接口进行存根。

另一个有争议的功能最初没有包含在 Mockito 中。我们建议简单地使用`thenReturn()`或`thenThrow()`进行存根，这应该足以 测试/测试驱动 任何干净和简单的代码。但是，如果您确实需要使用通用的 Answer 接口存根，这里是一个示例：

```java
 when(mock.someMethod(anyString())).thenAnswer(
     new Answer() {
         public Object answer(InvocationOnMock invocation) {
             Object[] args = invocation.getArguments();
             Object mock = invocation.getMock();
             return "called with arguments: " + Arrays.toString(args);
         }
 });

 //以下将会输出 "called with arguments: [foo]"
 System.out.println(mock.someMethod("foo"));
 
```

#### 12. [`doReturn()`| `doThrow()`| `doAnswer()`| `doNothing()`| `doCallRealMethod()`方法族](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#do_family_methods_stubs)

存根 void 方法需要一种和[`when(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#when-T-)不同的方法，因为编译器不喜欢`when()`括号内使用返回 void 的方法......

当你想要存根void方法抛出异常时使用`doThrow()`方法：

```java
   doThrow(new RuntimeException()).when(mockedList).clear();

   //下面的代码会抛出一个 RuntimeException:
   mockedList.clear();
 
```

您可以使用`doThrow()`，`doAnswer()`，`doNothing()`，`doReturn()` 和`doCallRealMethod()`在与`when()`相应的地方，对于任何方法。当你有必要

- 存根void方法
- 存根spy对象的方法（见下文）
- 多次存根相同的方法，以在测试过程中更改mock对象的行为。

但是对于`when()`所有存根调用，您可能更喜欢使用以下这些代替方法。

阅读有关这些方法的更多信息：

[`doReturn(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doReturn-java.lang.Object-)

[`doThrow(Throwable...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doThrow-java.lang.Throwable...-)

[`doThrow(Class)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doThrow-java.lang.Class-)

[`doAnswer(Answer)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doAnswer-org.mockito.stubbing.Answer-)

[`doNothing()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doNothing--)

[`doCallRealMethod()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doCallRealMethod--)

### 13.[监视真实对象：使用spy](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy)

您可以创建真实对象的间谍。当您使用 spy 时，将调用**真正的**方法（除非方法被存根）。

真正的间谍应该**谨慎**使用**，偶尔使用**，例如在处理遗留代码时。

对真实对象的监视可以与“部分mock”概念相关联。 **在 1.8 版本之前**，Mockito 间谍并不是真正的部分mock。原因是我们认为部分mock是一种代码异味。在某些时候，我们发现了部分mock的合法用例（第 3 方接口，遗留代码的临时重构）。



```java
   List list = new LinkedList();
   List spy = spy(list);

   //你可以存根一些方法:
   when(spy.size()).thenReturn(100);

   //使用spy调用真实的方法
   spy.add("one");
   spy.add("two");

   //输出 "one" - list的第一个元素
   System.out.println(spy.get(0));

   //size() 方法已被存根 - 输出 100
   System.out.println(spy.size());

   //你也可以进行验证
   verify(spy).add("one");
   verify(spy).add("two");
 
```

#### 监视真实对象的重要问题！

1. 有时将`when(Object)`用于已经存根的spy对象是不可能或不切实际的。因此在使用间谍时请考虑`doReturn`|`Answer`|`Throw()`存根方法族。例子：

   ```java
      List list = new LinkedList();
      List spy = spy(list);
   
      //以下代码是不可能: 真正的函数会被调用，spy.get(0) 会抛出 IndexOutOfBoundsException (list仍然是空的)
      when(spy.get(0)).thenReturn("foo");
   
      //你需要用 doReturn() 去存根
      doReturn("foo").when(spy).get(0);
    
   ```

2. Mockito **不会**将调用委托传递给的真实实例，而是实际上创建了它的副本。因此，如果您保留真实实例并与之交互，则不要指望被监视的人会知道这些交互及其对真实实例状态的影响。相应的，当**unstubbed**（没有进行存根）的方法在**spy对象上**调用但**不在真实实例上时**，您将看不到对真实实例的任何影响。

3. 注意最后的方法。Mockito 不mock final 方法，所以底线是：当您监视真实对象时 + 尝试存根 final 方法 = 麻烦。您也将无法验证这些方法。

### 14. 更改未[存根调用的默认返回值](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#defaultreturn)（自 1.7 起）

您可以为mock对象创建具有指定策略的返回值。这是一项非常高级的功能，通常您不需要用它来编写像样的测试。但是，它对于处理**遗留系统**很有帮助。

```java
//它是默认返回，因此**仅当您**在非存根方法**调用时**才会使用它。
Foo mock = mock(Foo.class, Mockito.RETURNS_SMART_NULLS);
//自定义的返回值策略
Foo mockTwo = mock(Foo.class, new YourOwnAnswer());
```

阅读有关这个有趣的*Answer*实现的更多信息：[`RETURNS_SMART_NULLS`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#RETURNS_SMART_NULLS)

### 15.为进一步的断言[捕获参数](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#captors)（自 1.8.0 起）

Mockito 通过使用`equals()`方法实现了自然的 Java 风格验证参数值。这也是推荐的匹配参数的方式，因为它使测试变得干净和简单。但在某些情况下，捕获参数对某些进行断言是有帮助的。例如：

```java
   ArgumentCaptor<Person> argument = ArgumentCaptor.forClass(Person.class);
   verify(mock).doSomething(argument.capture());
   assertEquals("John", argument.getValue().getName());
```

**警告：**建议将 ArgumentCaptor 与验证一起使用，**但不要**与存根一起使用。使用带有存根的 ArgumentCaptor 可能会降低测试的可读性，因为 captor 是在断言（又名验证或“then”）块之外创建的。它还会降低缺陷定位，因为如果未调用存根方法，则不会捕获任何参数。

在某种程度上 ArgumentCaptor 与自定义参数匹配器有关（请参阅[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)类的javadoc ）。这两种技术都可用于确保将某些参数传递给mock对象。但是，在以下情况下，ArgumentCaptor 可能更适合：

- 自定义参数匹配器不太可能被重用
- 您只需要它对参数值进行断言即可完成验证

自定义参数匹配器[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)通常更适合存根。

### 16.[真正的部分mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#partial_mocks)（自 1.8.0 起）

最后，经过邮件列表上的多次内部辩论和讨论，Mockito 添加了对部分mock支持。以前我们将部分mock视为代码异味。但是，我们发现了部分mock的合法用例。

**在 1.8 版之前** `spy()`并没有产生真正的部分mock，这让一些用户感到困惑。阅读有关spy的更多信息：[此处](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#13)或在 javadoc 中获取[`spy(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy-T-)方法。



```java
    //你可以使用spy()方法创建部分mock
    List list = spy(new LinkedList());

    //您可以在mock中选择性地启用部分mock功能：
    Foo mock = mock(Foo.class);
    //首先要确保真正的实现是安全的。
    //如果真正的实现抛出异常或者依赖特殊的对象那么你会遇到麻烦。
    when(mock.someMethod()).thenCallRealMethod();
  
```

像往常一样，您将阅读**部分mock警告**：面向对象编程通过将复杂性划分为单独的、特定的 SRPy 对象来减少复杂性。部分部分mock如何适应这种范式？嗯，它只是没有......部分mock通常意味着复杂性已转移到同一对象上的不同方法。在大多数情况下，这不是您想要设计应用程序的方式。

但是，在极少数情况下，部分mock会派上用场：处理您无法轻松更改的代码（第 3 方接口、遗留代码的临时重构等）但是，我不会将部分mock用于新的、测试驱动的 & 良好设计的代码。

### 17.[重置mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#resetting_mocks)（自 1.8.0 起）

聪明的 Mockito 用户几乎不使用此功能，因为他们知道这可能是测试不佳的迹象。通常，您不需要重置mock，只需为每个测试方法创建新mock。

请考虑在冗长、过度指定的测试中编写简单、小而集中的测试方法而不是`reset()`。 **第一个潜在的代码异味是在测试方法中间的`reset()`。**这可能意味着您测试过多。遵循您的测试方法的耳语：“请让我们保持小规模并专注于单一行为”。在 mockito 邮件列表上有几个关于它的主题。

我们添加`reset()`方法的唯一原因是可以使用容器注入的mock。有关更多信息，请参阅常见问题解答（[此处](https://github.com/mockito/mockito/wiki/FAQ)）。

**不要伤害自己。** `reset()`测试方法的中间是代码异味（您可能测试过多）。

```java
   List mock = mock(List.class);
   when(mock.size()).thenReturn(10);
   mock.add(1);

   reset(mock);
   //在此时mock忘掉了所有的交互和存根
```

### 18.[故障排除和验证框架的使用](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#framework_validation)（自 1.8.0 起）

首先，如果有任何问题，我鼓励您阅读 Mockito FAQ：[https](https://github.com/mockito/mockito/wiki/FAQ) : [//github.com/mockito/mockito/wiki/FAQ](https://github.com/mockito/mockito/wiki/FAQ)

如果有问题，您也可以发布到 mockito 邮件列表：[https](https://groups.google.com/group/mockito) : [//groups.google.com/group/mockito](https://groups.google.com/group/mockito)

接下来，你应该在了解**所有的**Mockito验证，如果你想要正确地使用它。但是，这儿有一个问题，请阅读 [`validateMockitoUsage()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#validateMockitoUsage--) javadoc。

### 19.[行为驱动开发的别名](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#bdd_mockito)（自 1.8.0 起）

使用**//given //when //then**注释作为测试方法的基本部分可以编写行为的驱动开发风格测试。这正是我们编写测试的方式，我们热烈鼓励您这样做！

从这里开始学习 BDD：[https](https://en.wikipedia.org/wiki/Behavior-driven_development) : [//en.wikipedia.org/wiki/Behavior-driven_development](https://en.wikipedia.org/wiki/Behavior-driven_development)

问题是当前的存根 api 与 **when** 关键字不能很好地与**//given //when //then**注释集成。这是因为存根属于测试的**given**组件，而不是测试的**when**组件。因此`BDDMockito`类引入了一个别名，以便您使用[`BDDMockito.given(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/BDDMockito.html#given-T-)方法存根方法调用。现在它真的很好地与BDD 风格测试的**给定**组件集成了！

以下是测试的样子：

```java
 import static org.mockito.BDDMockito.*;

 Seller seller = mock(Seller.class);
 Shop shop = new Shop(seller);

 public void shouldBuyBread() throws Exception {
   //given: when方法的别名
   given(seller.askForBread()).willReturn(new Bread());

   //when
   Goods goods = shop.buyBread();

   //then
   assertThat(goods, containBread());
 }
 
```

### 20. [可序列化的mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#serializable_mocks)（自 1.8.1 起）

mock可以被序列化。使用此功能，您可以在需要可序列化依赖项的地方使用mock。

警告：这应该很少用于单元测试。

该行为是针对具有不可靠外部依赖性的 BDD 规范的特定用例实现的。这是在 Web 环境中，来自外部依赖项的对象会被序列化以在层之间传递。

创建可序列化的mock使用[`MockSettings.serializable()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockSettings.html#serializable--)：

```java
   List serializableMock = mock(List.class, withSettings().serializable());
```

假设类满足所有正常的[ 序列化要求](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)，mock则可以被序列化。

由于 spy(...) 方法没有接受 MockSettings 的重载版本，因此使真正的对象 spy 可序列化需要更多的努力。不用担心，您几乎永远不会使用它。

```java
 List<Object> list = new ArrayList<Object>();
 List<Object> spy = mock(ArrayList.class, withSettings()
                 .spiedInstance(list)
                 .defaultAnswer(CALLS_REAL_METHODS)
                 .serializable());
 
```

### 21. 新注释：[`@Captor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#captor_annotation), [`@Spy`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy_annotation), [`@InjectMocks`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#injectmocks_annotation)（自 1.8.3 起）

1.8.3 版带来了有时可能有用的新注释：

- @[`Captor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Captor.html)简化了[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html) - 当要捕获的参数是一个讨厌的泛型类并且您想避免编译器警告时很有用
- @ [`Spy`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Spy.html)-您可以用它替代[`spy(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy-T-)。
- @ [`InjectMocks`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InjectMocks.html)- 自动将mock或spy字段注入测试对象。

注意@[`InjectMocks`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InjectMocks.html)也可以与@[`Spy`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Spy.html)注解结合使用，这意味着Mockito会将mock注入到被测的部分mock中。这种复杂性是您应该只使用部分mock作为最后手段的另一个很好的原因。请参阅有关部分mock的第 16 点。

所有新注解**仅**在 [`MockitoAnnotations.openMocks(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoAnnotations.html#openMocks-java.lang.Object-) 上处理。就像 @[`Mock`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mock.html)注释一样，您可以使用内置的 runner:[`MockitoJUnitRunner`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnitRunner.html)或 rule: [`MockitoRule`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoRule.html)。



### 22.[超时验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verification_timeout)（自 1.8.5 起）

允许超时验证。它会等待指定的时间以验证进行所需的交互，而不是在尚未发生的情况下立即失败。可能对在并发条件下进行测试很有用。

这个特性应该很少使用——找出一种更好的方法来测试你的多线程系统。

尚未实现与 InOrder 验证一起使用。

例子：

```java
   //处理当 someMethod() 被调用少于100ms的情况
   //满足验证时立即退出（例如，可能不等待100毫秒）
   verify(mock, timeout(100)).someMethod();
   //上面的代码也可以写作:
   verify(mock, timeout(100).times(1)).someMethod();

   //当someMethod()在少于100ms内被调用2此时通过
   verify(mock, timeout(100).times(2)).someMethod();

   //等价于上面的代码:
   verify(mock, timeout(100).atLeast(2)).someMethod();
 
```

### 23.[使用`@Spies`， `@InjectMocks`自动实例化对象](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#automatic_instantiation)并具有[良好的构造函数注入](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#constructor_injection)（1.9.0以后）

Mockito 现在将尝试使用 @[`Spy`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Spy.html)实例化对象并将[`InjectMocks`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InjectMocks.html)实例化的字段使用**构造函数**注入、**setter**注入或**字段**注入。

要利用此功能，您需要使用[`MockitoAnnotations.openMocks(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoAnnotations.html#openMocks-java.lang.Object-),[`MockitoJUnitRunner`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnitRunner.html) 或[`MockitoRule`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoRule.html)。

在 [`InjectMocks`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InjectMocks.html)的javadoc 中阅读有关可用技巧和注入规则的更多信息。

```java
 @Spy BeerDrinker drinker = new BeerDrinker();
 //也可以这样写：
 @Spy BeerDrinker drinker;

 //@InjectMocks 注解也可以这样用:
 @InjectMocks LocalPub;
 
```

### 24.[单行存根](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#one_liner_stub)（自 1.9.0 起）

Mockito 现在允许您在存根时创建mock。基本上，它允许在一行代码中创建一个存根。这有助于保持测试代码干净。例如，一些无聊的存根可以在测试中的字段初始化时创建和存根：

```java
 public class CarTest {
   Car boringStubbedCar = when(mock(Car.class).shiftGear()).thenThrow(EngineNotStarted.class).getMock();

   @Test public void should... {}
 
```

### 25.[忽略存根的验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#ignore_stubs_verification)（自 1.9.0 起）

Mockito 现在允许为了验证而忽略存根。有时与`verifyNoMoreInteractions()`或`inOrder()` 验证结合使用时很有用。有助于避免对存根调用的冗余验证 - 通常我们对验证存根不感兴趣。

**警告**，`ignoreStubs()`可能会导致 verifyNoMoreInteractions(ignoreStubs(...)) 过度使用; 请记住，出于`verifyNoMoreInteractions()` javadoc 中概述的原因，Mockito 不建议对每个测试都使用[`verifyNoMoreInteractions(Object...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verifyNoMoreInteractions-java.lang.Object...-)。

一些例子：

```java
 verify(mock).foo();
 verify(mockTwo).bar();

 //忽略所有的存根方法:
 verifyNoMoreInteractions(ignoreStubs(mock, mockTwo));

 //创建 InOrder，它会忽略存根
 InOrder inOrder = inOrder(ignoreStubs(mock, mockTwo));
 inOrder.verify(mock).foo();
 inOrder.verify(mockTwo).bar();
 inOrder.verifyNoMoreInteractions();
 
```

可以在  [`ignoreStubs(Object...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#ignoreStubs-java.lang.Object...-)的 javadoc 中找到高级示例和更多详细信息。

### 26. [Mocking 细节](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mocking_details)（2.2.x 改进）

Mockito 提供 API 来检查mock对象的详细信息。此 API 对高级用户和mock框架集成者很有用。

```java
   //确定特定对象是mock还是spy:
   Mockito.mockingDetails(someObject).isMock();
   Mockito.mockingDetails(someObject).isSpy();

   //获取详细信息比如mock的类型或者默认的Answer:
   MockingDetails details = mockingDetails(mock);
   details.getMockCreationSettings().getTypeToMock();
   details.getMockCreationSettings().getDefaultAnswer();

   //获取mock对象的交互或者存根:
   MockingDetails details = mockingDetails(mock);
   details.getInvocations();
   details.getStubbings();

   //打印所有的交互（包括使用的存根，未使用的存根）
   System.out.println(mockingDetails(mock).printInvocations());
 
```

有关更多信息，请参阅[`MockingDetails`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockingDetails.html).

### 27.[将调用委托给真实对象](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#delegating_call_to_real_instance)（自 1.9.5 起）

对于使用一般的spy API**难以mock或监视**的对象，可以使用其spy或部分**mock**。从 Mockito 1.10.11 开始，委托可能与mock的类型相同，也可能不同。如果类型不同，则需要在委托类型上找到匹配的方法，否则抛出异常。此功能的可能用例：

- Final类但带有接口
- 已经自定义代理对象
- 具有finalize方法的特殊对象，即避免执行 2 次

与普通spy的区别：

- 常规 spy ( [`spy(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy-T-)) 包含来自被监视实例的**所有**状态和被调用方法。被监视的实例仅用于mock创建以从中复制状态。如果您调用常规 spy 上的方法，并且它在内部调用此 spy 上的其他方法，则这些调用会被记住以进行验证，并且可以有效地将它们存根。
- 委托的mock只是将所有方法委托给委托类。委托类一直在使用，因为方法被委托给它。如果您在委托类的mock上调用方法，并且它在内部调用此mock上的其他方法，则**不会**记住这些调用以进行验证，存根也不会对它们产生影响。Mock 委托的功能不如常规 spy 强大，但在无法创建常规 spy 时很有用。

在[`AdditionalAnswers.delegatesTo(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/AdditionalAnswers.html#delegatesTo-java.lang.Object-)文档中查看更多信息。

### 28. [`MockMaker`API](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mock_maker_plugin)（从 1.9.5 开始）

在谷歌 Android 人员的要求和补丁的驱动下，Mockito 现在提供了一个扩展点，允许替换`代理生成引擎`。默认情况下，Mockito 使用[Byte Buddy](https://github.com/raphw/byte-buddy) 创建动态代理。

扩展点适用于想要扩展 Mockito 的高级用户。例如，现在可以使用针对的Mockito Android版的[dexmaker](https://github.com/crittercism/dexmaker)帮助测试。

有关更多详细信息、动机和示例，请参阅[`MockMaker`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/MockMaker.html).

### 29. [BDD 风格验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#BDD_behavior_verification)（自 1.10.0 起）

通过使用 BDD **then**关键字开始验证，启用行为驱动开发 (BDD) 样式验证。

```
 given(dog.bark()).willReturn(2);

 // when
 ...

 then(person).should(times(2)).ride(bike);
 
```

有关更多信息和示例，请参见 [`BDDMockito.then(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/BDDMockito.html#then-T-)

### 30.[监视或mock抽象类（自 1.10.12 起，在 2.7.13 和 2.7.14 中进一步增强）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spying_abstract_classes)

现在可以方便地监视抽象类。请注意，过度使用 spies 暗示了代码设计的异味（请参阅 参考资料[`spy(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy-T-)）。

以前，只能对对象实例进行监视。新的 API 使得在创建mock实例时使用构造函数成为可能。这对于mock抽象类特别有用，因为用户不再需要提供抽象类的实例。目前只支持无参数构造函数，如果还不够，请告诉我们。

```java
 //便利的 API, 新重载的 spy() 方法:
 SomeAbstract spy = spy(SomeAbstract.class);

 //Mocking 抽象, spy 接口的默认方法(从2.7.13可用)
 Function function = spy(Function.class);

 //健壮的 API, 来自 settings builder:
 OtherAbstract spy = mock(OtherAbstract.class, withSettings()
    .useConstructor().defaultAnswer(CALLS_REAL_METHODS));

 //Mock一个抽象类，使用构造函数(从 2.7.14 可用)
 SomeAbstract spy = mock(SomeAbstract.class, withSettings()
   .useConstructor("arg1", 123).defaultAnswer(CALLS_REAL_METHODS));

 //mock一个非静态内部抽象类
 InnerAbstract spy = mock(InnerAbstract.class, withSettings()
    .useConstructor().outerInstance(outerInstance).defaultAnswer(CALLS_REAL_METHODS));
 
```

有关更多信息，请参阅[`MockSettings.useConstructor(Object...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockSettings.html#useConstructor-java.lang.Object...-)。

### 31. [Mockito 模拟可以跨类加载器*序列化*/*反序列化*（自 1.10.0 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#serilization_across_classloader)

Mockito 引入了跨类加载器的序列化。与任何其他形式的序列化一样，mock层次结构中的所有类型都必须可序列化，包括Answer。由于此序列化模式需要更多的工作，因此这是一个可选设置。

```java
 // 使用常规序列化
 mock(Book.class, withSettings().serializable());

 // 使用类加载器序列化
 mock(Book.class, withSettings().serializable(ACROSS_CLASSLOADERS));
 
```

有关更多详细信息，请参阅[`MockSettings.serializable(SerializableMode)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockSettings.html#serializable-org.mockito.mock.SerializableMode-)。

### 32.[对深存根（deep stubs）更好的通用支持（自 1.10.0 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#better_generic_support_with_deep_stubs)

已改进深存根以查找类中可用的通用信息。这意味着可以使用这样的类而不必mock行为。

```java
 class Lines extends List<Line> {
     // ...
 }

 lines = mock(Lines.class, RETURNS_DEEP_STUBS);

 // 现在 Mockito 这不是一个对象而是一个 Line
 Line line = lines.iterator().next();
 
```

请注意，在大多数情况下，返回mock的mock是错误的。

### 33. [Mockito JUnit rule（自 1.10.17 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_junit_rule)

Mockito 现在提供了一个 JUnit rule。直到现在JUnit中有两种方法来初始化注解的字段，通过Mockito的注解 `@Mock` `@Spy` `@InjectMocks` 等。

- 用`@RunWith(MockitoJUnitRunner.class)`注解一个 JUnit 测试类 
- 在`@Before`方法中调用`MockitoAnnotations.openMocks(Object)`

现在您可以选择使用rule：

```java
 @RunWith(YetAnotherRunner.class)
 public class TheTest {
     @Rule public MockitoRule mockito = MockitoJUnit.rule();
     // ...
 }
 
```

有关更多信息，请参阅[`MockitoJUnit.rule()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnit.html#rule--)。

### 34.[切换插件的*启用*或*禁用*（15年10月1日以来）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#plugin_switch)

孵化功能，使其成为 mockito 中的一种方式，可以允许切换 mockito-plugin。更多信息在这里[`PluginSwitch`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/PluginSwitch.html)。

### 35.[自定义验证失败信息](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Custom_verification_failure_message)（自 2.1.0 起）

允许指定在验证失败时打印的自定义消息。

例子：

```java
 // 会打印一个自定义的失败信息
 verify(mock, description("This will print on failure")).someMethod();

 // 在任何的验证模式都可用
 verify(mock, times(2).description("someMethod should be called twice")).someMethod();
 
```

### 36. [Java 8 Lambda 匹配器支持](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Java_8_Lambda_Matching)（自 2.1.0 起）

您可以使用 Java 8 lambda 表达式和[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)来减少对[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)的依赖. 如果您需要验证mock对象上函数调用的输入是否正确，那么您通常会使用[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)来查找使用的参数，然后对它们进行后续断言。虽然对于复杂的示例，这可能很有用，但也是冗长的。

编写一个 lambda 表达式来表达匹配是很容易的。当与 argThat 结合使用时，函数的参数将作为强类型对象传递给 ArgumentMatcher，因此可以对它执行任何操作。

例子：

```java
 // 验证list只添加了一定长度的字符串
 // 注意 - 这只会在Java8下编译
 verify(list, times(2)).add(argThat(string -> string.length() < 5));

 // Java7下等效的写法，不是很简洁
 verify(list, times(2)).add(argThat(new ArgumentMatcher(){
     public boolean matches(String arg) {
         return arg.length() < 5;
     }
 }));

 // 更复杂的Java 8示例 - 您可以在函数上指定复杂的验证行为
 verify(target, times(1)).receiveComplexObject(argThat(obj -> obj.getSubObject().get(0).equals("expected")));

 // 也可以使用这一点定义不同输入下mock的行为，
 // 在这种情况下，如果输入列表少于3项，则模拟返回null
 when(mock.someMethod(argThat(list -> list.size()<3))).thenReturn(null);
 
```

### 37. [Java 8 自定义Answer支持](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Java_8_Custom_Answers)（自 2.1.0 起）

由于[`Answer`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/stubbing/Answer.html)接口只有一种方法，因此可以在 Java 8 中使用 lambda 表达式非常简单的实现它。越需要使用方法调用的参数，就越需要使用[`InvocationOnMock`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/InvocationOnMock.html).

例子：

```java
 // 每次都返回12的Answer
 doAnswer(invocation -> 12).when(mock).doSomething();

 // 使用参数的Answer - 转换成你期望的正确的类型 - 在这个例子中，返回第二个参数的长度
 // 这样快速解决了冗长的参数转换
 doAnswer(invocation -> ((String)invocation.getArgument(1)).length())
     .when(mock).doSomething(anyString(), anyString(), anyString());
 
```

为方便起见，可以编写自定义Answer/Action，它们使用 Java 8 lambdas作为方法调用的参数。即使在 Java 7 中，降低这些基于类型化接口的自定义Answer也可以减少样板代码。特别是，这种方法将使使用回调的测试函数变得更加容易。方法[`answer`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/AdditionalAnswers.html#answer-org.mockito.stubbing.Answer1-)和[`answerVoid`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/AdditionalAnswers.html#answerVoid-org.mockito.stubbing.VoidAnswer1-) 可用于创建Answer。他们依赖`org.mockito.stubbing`相关的Answer接口，支持最多 5 个参数的Answer。

例子：

```java
 // 被mock的示例接口具有以下函数:
 void execute(String operand, Callback callback);

 // 示例回调有个函数而且被测试的类会依赖这个回调当它被调用时
 void receive(String item);

 // Java 8 - 方式 1
 doAnswer(AdditionalAnswers.answerVoid((operand, callback) -> callback.receive("dummy"))
     .when(mock).execute(anyString(), any(Callback.class));

 // Java 8 - 方式 2 - 假设已经静态导入了类：AdditionalAnswers
 doAnswer(answerVoid((String operand, Callback callback) -> callback.receive("dummy"))
     .when(mock).execute(anyString(), any(Callback.class));

 // Java 8 - style 3 - 当被mock函数是测试类的静态成员
 private static void dummyCallbackImpl(String operation, Callback callback) {
     callback.receive("dummy");
 }

 doAnswer(answerVoid(TestClass::dummyCallbackImpl)
     .when(mock).execute(anyString(), any(Callback.class));

 // Java 7
 doAnswer(answerVoid(new VoidAnswer2() {
     public void answer(String operation, Callback callback) {
         callback.receive("dummy");
     }})).when(mock).execute(anyString(), any(Callback.class));

 // 使用answer()返回一个可能值以及函数接口的非void版本，
 // 也就是说这个mock接口像下面这样
 boolean isSameString(String input1, String input2);

 // 可以被这样mock
 // Java 8
 doAnswer(AdditionalAnswers.answer((input1, input2) -> input1.equals(input2))))
     .when(mock).execute(anyString(), anyString());

 // Java 7
 doAnswer(answer(new Answer2() {
     public String answer(String input1, String input2) {
         return input1 + input2;
     }})).when(mock).execute(anyString(), anyString());
 
```

### 38.[元数据和泛型类型的保留](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Meta_Data_And_Generics)（自 2.1.0 起）

Mockito 现在保留对mock方法的类型以及通用元数据的注解。以前，mock类型不会保留类型的注解，除非它们被显式继承并且从不保留方法的注解。因此，以下条件现在成立：

```java
  @MyAnnotation
  class Foo {
    List<String> bar() { ... }
  }

  Class<?> mockType = mock(Foo.class).getClass();
  assert mockType.isAnnotationPresent(MyAnnotation.class);
  assert mockType.getDeclaredMethod("bar").getGenericReturnType() instanceof ParameterizedType;
 
```

使用 Java 8 时，Mockito 现在还保留了类型注解。这是默认行为，[如果使用替代方法](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#28)[`MockMaker`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/MockMaker.html)可能不会成立。

### 39.[mock final类、枚举和 final方法](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Mocking_Final)（自 2.1.0 起）

Mockito 现在为mock final 类和方法提供了一个可选的支持，[`Incubating`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Incubating.html)。这是一项了不起的改进，展示了 Mockito 对改进测试体验的永恒追求。我们的目标是 Mockito “只适用于”final 类和方法。以前它们被认为是*unmockable*的，防止用户mock。我们已经开始讨论如何默认启用此功能。目前，该功能仍然是可选的，因为我们正在等待社区的更多反馈。

这个替代的mock生成器使用 Java Instrumentation API 和子类的组合，而不是创建一个新类来表示mock。这样，就可以mock final类和方法了。

这个mock生成器**默认**是**关闭的，**因为它基于完全不同的mock机制，需要更多来自社区的反馈。它可以通过 mockito 扩展机制显式激活，只需在类路径中创建一个包含 value为`mock-maker-inline`的`/mockito-extensions/org.mockito.plugins.MockMaker` 文件。

为方便起见，Mockito 团队提供了一个工件，其中预配置了此mock生成器。不要使用*mockito-core*工件，而是 在您的项目中包含*mockito-inline*工件。请注意，一旦对final类和方法的mock集成到默认mock生成器中，此工件可能会停止使用。

关于这个mock marker的一些值得注意的说明：

- mock final类和枚举与mock设置不兼容，例如：
  - 显式序列化支持 `withSettings().serializable()`
  - 额外接口 `withSettings().extraInterfaces()`
- 有些方法不能被mock
  - `java.*`中包可见的方法 
  - `native` 方法
- 这个 mock maker 是围绕 Java Agent 运行时附件设计的；这需要一个兼容的 JVM，它是 JDK（或 Java 9 VM）的一部分。但是，在 Java 9 之前的非 JDK VM 上运行时，可以 在启动 JVM 时手动添加[Byte Buddy Java 代理 jar](https://bytebuddy.net/)`-javaagent`参数使用。

如果您对此功能的更多详细信息感兴趣，请阅读`org.mockito.internal.creation.bytebuddy.InlineByteBuddyMockMaker` 的 javadoc。

### 40.[ 使用“更严格”的 Mockito 提高生产力和编写更简明的测试](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#strict_mockito)（自 2.+ 起）

要快速了解“更严格”的 Mockito 如何提高您的工作效率并使您的测试更清晰，请参阅：

- 使用 JUnit4 规则进行严格存根 -使用[`Strictness.STRICT_STUBS`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/quality/Strictness.html#STRICT_STUBS) 的 [`MockitoRule.strictness(Strictness)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoRule.html#strictness-org.mockito.quality.Strictness-)
- 使用 JUnit4 Runner 进行严格存根 - `MockitoJUnitRunner.Strict`
- 使用 JUnit5 扩展进行严格存根 - `org.mockito.junit.jupiter.MockitoExtension`
- 使用 TestNG Listener [MockitoTestNGListener ](https://github.com/mockito/mockito-testng)进行严格存根
- 如果您不能使用runner/rule，则使用 [`MockitoSession`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoSession.html)进行严格存根
- 不必使用 [`MockitoJUnitRunner`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnitRunner.html)进行存根检测
- 存根参数不匹配警告，记录在 [`MockitoHint`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/quality/MockitoHint.html)

默认情况下，Mockito 是一个“松散”的mock框架。可以在不预先设定任何期望的情况下与mock交互。这是有意的，它通过强制用户明确他们想要存根/验证的内容来提高测试质量。它也非常直观、易于使用，并且与干净的测试代码的“given”、“when”、“then”模板很好地融合在一起。这也不同于过去的经典mock框架，它们默认是“严格的”。

默认情况下“松散”使得 Mockito 测试有时更难调试。在某些情况下，错误配置的存根（例如使用错误的参数）会强制用户使用调试器运行测试。理想情况下，测试失败是显而易见的，不需要调试器来确定根本原因。从 2.1 版开始，Mockito 已经获得了将框架推向“严格”的新功能。我们希望 Mockito 提供出色的可调试性，同时不失去其核心mock风格，针对直观性、明确性和干净的测试代码进行了优化。

帮助Mockito！尝试新功能，给我们反馈，在 GitHub[问题 769](https://github.com/mockito/mockito/issues/769)上加入关于 Mockito 严格性的讨论 。

### 41.[ 用于框架集成的高级公共 API（自 2.10.+ 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#framework_integrations_api)

2017 年夏季，我们决定 Mockito [应该](https://www.linkedin.com/pulse/mockito-vs-powermock-opinionated-dogmatic-static-mocking-faber) 为高级框架集成[提供更好的 API ](https://www.linkedin.com/pulse/mockito-vs-powermock-opinionated-dogmatic-static-mocking-faber)。新 API 不适用于想要编写单元测试的用户。它适用于需要使用一些自定义逻辑扩展或包装 Mockito 的其他测试工具和mock框架。在设计和实施过程中（[issue 1110](https://github.com/mockito/mockito/issues/1110)），我们开发并更改了以下公共 API 元素：

- 新[`MockitoPlugins`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/MockitoPlugins.html)- 使框架集成商能够访问默认的 Mockito 插件。当需要实现自定义插件时很有用，例如[`MockMaker`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/MockMaker.html) 并将某些行为委托给默认的 Mockito 实现。
- 新[`MockSettings.build(Class)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockSettings.html#build-java.lang.Class-)- 创建 Mockito 稍后使用的mock设置的不可变视图。用于使用[`InvocationFactory`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/InvocationFactory.html)或在实现自定义 [`MockHandler`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/MockHandler.html) 时创建调用。
- 新[`MockingDetails.getMockHandler()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockingDetails.html#getMockHandler--)- 其他框架可以使用mock处理程序以编程方式mock`被mock对象`上的调用。
- 新[`MockHandler.getMockSettings()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/MockHandler.html#getMockSettings--)- 用于获取创建mock对象的设置。
- 新建[`InvocationFactory`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/InvocationFactory.html)- 提供创建[`Invocation`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/Invocation.html)对象实例的方法。对于需要以编程方式mock`被mock对象`上的方法调用的框架集成很有用。
- 新[`MockHandler.getInvocationContainer()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/MockHandler.html#getInvocationContainer--)- 提供对没有方法（标记接口）的调用容器对象的访问。需要容器来隐藏内部实现并避免将其泄漏到公共 API。
- 改变`Stubbing`- 它现在扩展[`Answer`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/stubbing/Answer.html)接口。它向后兼容，因为 Stubbing 接口不可扩展（请参阅 参考资料[`NotExtensible`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/NotExtensible.html)）。这种变化对我们的用户来说应该是无缝的。
- 弃用`InternalMockHandler`- 为了适应 API 更改，我们需要弃用此接口。该接口始终被记录为内部，我们没有证据表明它被社区使用。对于我们的用户来说，弃用应该是完全无缝的。
- [`NotExtensible`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/NotExtensible.html)- 向用户表明她不应提供给定类型的自定义实现的公共注解。帮助框架集成商和我们的用户了解如何安全地使用 Mockito API。

你有反馈吗？请在[issue 1110 中](https://github.com/mockito/mockito/issues/1110)发表评论。

### 42.[ 用于集成的新 API：监听验证开始事件（自 2.11.+ 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verifiation_started_listener)

[Spring Boot](https://projects.spring.io/spring-boot) 等框架集成需要公共 API 来处理双代理用例（[问题 1191](https://github.com/mockito/mockito/issues/1191)）。我们补充说：

- 启用新增的[`VerificationStartedListener`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/listeners/VerificationStartedListener.html)和[`VerificationStartedEvent`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/listeners/VerificationStartedEvent.html) 对象，框架集成商使用其可以替换用于验证的mock对象。主要的驱动用例是[Spring Boot](https://projects.spring.io/spring-boot/)集成。有关详细信息，请参阅 [`VerificationStartedListener`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/listeners/VerificationStartedListener.html) 的 Javadoc。
- 新的公共方法[`MockSettings.verificationStartedListeners(VerificationStartedListener...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockSettings.html#verificationStartedListeners-org.mockito.listeners.VerificationStartedListener...-) 允许在mock创建时提供验证启动的侦听器。
- [`MockingDetails.getMock()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockingDetails.html#getMock--)添加了新的便捷方法以使`MockingDetails`API 更加完整。我们发现这种方法在实施过程中很有用。

### 43.[ 用于集成的新 API：可用于测试框架的`MockitoSession`（自 2.15.+ 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_session_testing_frameworks)

[`MockitoSessionBuilder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/session/MockitoSessionBuilder.html)和[`MockitoSession`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoSession.html)通过测试框架集成（例如JUnit的[`MockitoRule`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoRule.html)对于 ）来增强以实现重用：

- [`MockitoSessionBuilder.initMocks(Object...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/session/MockitoSessionBuilder.html#initMocks-java.lang.Object...-)允许传入多个测试类实例，以初始化使用 Mockito 注解（如[`Mock`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mock.html). 当测试使用多个（例如嵌套的）测试类实例时，此方法对于高级框架集成（例如 JUnit Jupiter）很有用。
- [`MockitoSessionBuilder.name(String)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/session/MockitoSessionBuilder.html#name-java.lang.String-)允许将测试框架中的名称传递给 [`MockitoSession`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoSession.html)，它将在使用时用于打印警告[`Strictness.WARN`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/quality/Strictness.html#WARN)的名称。
- [`MockitoSessionBuilder.logger(MockitoSessionLogger)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/session/MockitoSessionBuilder.html#logger-org.mockito.session.MockitoSessionLogger-) 可以自定义的用于在完成mock时生成的提示/警告的记录器（对于测试和连接由 JUnit Jupiter 等测试框架提供的报告功能很有用）。
- [`MockitoSession.setStrictness(Strictness)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoSession.html#setStrictness-org.mockito.quality.Strictness-)允许更改[`MockitoSession`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoSession.html) 一次性场景的严格性，例如，它可以为类中的所有测试配置默认严格性，但可以更改单个或几个测试的严格性。
- [`MockitoSession.finishMocking(Throwable)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoSession.html#finishMocking-java.lang.Throwable-)为了避免可能出现的混淆而添加，因为存在多个竞争失败。当提供的*失败* 不是`null`时，它将禁用某些检查。

### 44.[ 已弃用，`org.mockito.plugins.InstantiatorProvider`因为它会泄漏内部 API。它被替换为`org.mockito.plugins.InstantiatorProvider2 (Since 2.15.4)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_instantiator_provider_deprecation)

[`InstantiatorProvider`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/InstantiatorProvider.html)返回一个内部 API。因此它被弃用并由[`InstantiatorProvider2`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/InstantiatorProvider2.html)替换. 旧的[`instantiator providers`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/plugins/InstantiatorProvider.html)将继续工作，但建议切换到新的 API。

### 45.[新的 JUnit Jupiter (JUnit5+) 扩展](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#junit5_mockito)

要与 JUnit Jupiter (JUnit5+) 集成，请使用 `org.mockito:mockito-junit-jupiter` 工件。有关整合的用法的详细信息，请参阅[`MockitoExtension`](https://javadoc.io/doc/org.mockito/mockito-junit-jupiter/latest/org/mockito/junit/jupiter/MockitoExtension.html) 的JavaDoc。

### 46.[ 新的`Mockito.lenient()`和`MockSettings.lenient()`的方法（自2.20.0）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_lenient)

从早期的 Mockito 2 开始就提供了严格的存根功能。它非常有用，因为它可以推动更清晰的测试并提高生产力。严格存根报告不必要的存根，检测存根参数不匹配并使测试更加 DRY ( [`Strictness.STRICT_STUBS`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/quality/Strictness.html#STRICT_STUBS))。这需要权衡：在某些情况下，您可能会从严格的存根中得到假阴性。为了解决这些情况，您现在可以将特定存根配置为宽松，而所有其他存根和mock使用严格存根：

```java
   lenient().when(mock.foo()).thenReturn("ok");
 
```

如果您希望给定mock上的所有存根都是宽松的，您可以相应地配置mock：

```mock
   Foo mock = Mockito.mock(Foo.class, withSettings().lenient());
 
```

有关更多信息，请参阅[`lenient()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#lenient--)。通过打开 GitHub 问题进行讨论，让我们知道您如何找到新功能！

### 47.[用于清除内联mock中mock状态的新 API（自 2.25.0 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#clear_inline_mocks)

在某些特定的、罕见的情况下（问题[#1619](https://github.com/mockito/mockito/pull/1619)）内联mock会导致内存泄漏。没有干净的方法可以完全缓解这个问题。因此，我们引入了一个新的 API 来明确清除mock状态（仅在内联mock中有意义！）。请参阅 [`MockitoFramework.clearInlineMocks()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/MockitoFramework.html#clearInlineMocks--) 中的示例用法。如果您有反馈或更好的如何解决问题的想法，请联系。

### 48.[mock静态方法](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#static_mocks)（自 3.4.0 起）

使用[内联 mock maker 时](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#0.2)，可以在当前线程和用户定义的范围内mock静态方法调用。这样，Mockito 可确保同时和顺序运行的测试不会相互干扰。为了确保静态mock保持临时，建议在 try-with-resources 构造中定义范围。在以下示例中，除非mock，否则`Foo`类的静态方法将返回`foo`：

```java
 assertEquals("foo", Foo.method());
 try (MockedStatic mocked = mockStatic(Foo.class)) {
     mocked.when(Foo::method).thenReturn("bar");
     assertEquals("bar", Foo.method());
     mocked.verify(Foo::method);
 }
 assertEquals("foo", Foo.method());
 
```

由于静态mock的定义范围，一旦范围被释放，它就会返回其原始行为。要定义mock行为并验证静态方法调用，请使用`MockedStatic`返回的对象 。

### 49.[mock对象构造](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mocked_construction)（自 3.5.0 起）

使用[内联mock maker 时](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#0.2)，可以在当前线程和用户定义的范围内对构造函数调用生成[mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#0.2)。这样，Mockito 可确保同时和顺序运行的测试不会相互干扰。为了确保构造函数mock保持临时，建议在 try-with-resources 构造中定义范围。在以下示例中，构造的`Foo`类将生成一个mock：

```java
 assertEquals("foo", new Foo().method());
 try (MockedConstruction mocked = mockConstruction(Foo.class)) {
 		Foo foo = new Foo();
 		when(foo.method()).thenReturn("bar");
 		assertEquals("bar", foo.method());
 		verify(foo).method();
 }
 assertEquals("foo", new Foo().method());
 
```

由于mock对象构造的定义范围，一旦释放范围，对象构造将返回其原始行为。要定义模拟行为并验证方法调用，请使用`MockedConstruction`返回的对象 。