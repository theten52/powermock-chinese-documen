# Mockito 详解：第二部分：创建存根

## 零：前提条件

先明确一个词：**存根**(或者说是**打桩**），指的是对某个方法指定返回策略的操作（具体表现为两种：1指定返回值，2使用doCallRealMethod()或者thenCallRealMethod()指定当方法被调用时执行实际代码逻辑），功能就是当测试执行到此方法时直接返回我们指定的返回值（此时不会执行此方法的实际代码逻辑）或者执行此方法的实际代码逻辑并返回。比如:

我们先定义一个对象：

```java
public class Mock {

    public String m1() {
        throw new RuntimeException();
    }
}
```

然后我们进行下面的操作:

```java
Mock m = Mockito.mock(Mock.class);

//对m1()方法进行存根
Mockito.doReturn("1").when(m).m1();
```

基于上面的代码我们可以说：我们对m对象的m1()方法进行了**存根**。

此时我们调用m对象的m1()方法时，可以直接得到返回值"1"而不会执行m1方法的实际代码逻辑：

```java
Assert.assertEquals("1",m.m1());
```

此时断言为真。

## 一.创建方法存根的方式:

- Mockito.when(foo.sum()).thenXXX(...);

  - 即对foo.sum()方法存根。
  - **存在问题**：
    - foo对象应该是一个mock对象。spy对象不建议使用此方式进行存根。因为当代码执行到when(foo.sum())时。foo.sum()方法会首先执行。导致sum()方法的实际代码逻辑被执行。（sum()的实际代码逻辑是否会被执行要看被spy对象的类型，当被spy对象是一个mock对象或者接口时不会执行-这些类型也没有实际代码逻辑可以执行。当被spy一个具体的对象时则实际代码逻辑会被执行）

- Mockito.doXXX(...).when(foo).sum();

  - 即对foo.sum()方法存根。
  - 可以存根void方法。
  - foo对象可以是一个mock对象，也可以是一个spy对象。
  - **推荐使用此方式做方法存根。它可以避免上文中thenXXX()方式的问题**。

- ~~Mockito.doXXX(....).when(foo.sum());~~

  - 你会得到一个异常，即不应该使用这种方式！

  - ```java
    org.mockito.exceptions.misusing.UnfinishedStubbingException: 
    Unfinished stubbing detected here:
    -> at c.FooTest.verifyTest(FooTest.java:23)
    
    E.g. thenReturn() may be missing.
    Examples of correct stubbing:
        when(mock.isOk()).thenReturn(true);
        when(mock.isOk()).thenThrow(exception);
        doThrow(exception).when(mock).someVoidMethod();
    Hints:
     1. missing thenReturn()
     2. you are trying to stub a final method, which is not supported
     3. you are stubbing the behaviour of another mock inside before 'thenReturn' instruction is completed
    ```

## 二.定义返回值的方式：

- | then_xxx方法                       | do_XXX方法                                                   | 功能                                                         |
  | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | then(Answer\<?> answer)            | doAnswer(Answer answer)                                      | 返回值使用自定义的Answer策略。                               |
  | thenAnswer(Answer\<?> answer)      | 同上                                                         | 同上。                                                       |
  | thenReturn(T value)                | doReturn(Object toBeReturned)                                | 直接指定返回值。                                             |
  | thenReturn(T value, T... values)   | doReturn(Object toBeReturned, Object... toBeReturnedNext)    | 直接指定返回值，可以定义多个返回值。第一次调用到存根方法返回第一个返回值。以此类推。超过返回值数量的调用返回参数的最后一个返回值。 |
  | thenCallRealMethod()               | doCallRealMethod()                                           | 调用实际的代码逻辑。不指定返回值。                           |
  | thenThrow(Throwable... throwables) | doThrow(Throwable... toBeThrown)                             | 调用到存根方法时抛出异常。                                   |
  | 同上                               | doThrow(Class\<? extends Throwable> toBeThrown)              | 调用到存根方法时抛出异常。可以定义多个异常。第一次调用到存根方法返回第一个异常。以此类推。超过异常数量的调用返回参数的最后一个异常。 |
  | 同上                               | doThrow(Class\<? extends Throwable> toBeThrown, Class\<? extends Throwable>... toBeThrownNext) | 同上。                                                       |
  | 无对应方法                         | doNothing()                                                  | void方法使用的存根方式。                                     |

两种方式的示例：

```java
public class Bar {

    public int add(int a, int b) {
        return a + b;
    }

    public void badCode() {
        throw new RuntimeException("bad bar code");
    }
}


public class Foo {

    private Bar bar;

    public int sum(int a, int b) {
        return bar.add(a, b);
    }

    public int count() {
        bar.badCode();
        return 5;
    }
}
```

```java
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.junit.MockitoJUnitRunner;
import org.mockito.stubbing.Answer;

@RunWith(MockitoJUnitRunner.class)
public class MockitoTest {

    //foo 对象内部的成员变量会自动被 @Mock 注解的生成的对象注入。
    @InjectMocks
    private Foo foo;

    //bar 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
    @Mock
    private Bar bar;

    @Test
    public void thenTest() {
        Mockito.when(bar.add(1, 2)).then(new Answer<Integer>() {
            @Override
            public Integer answer(InvocationOnMock invocation) throws Throwable {
                return 7;
            }
        });

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);
    }

    @Test
    public void thenAnswerTest() {
        Mockito.when(bar.add(1, 2)).thenAnswer(new Answer<Integer>() {
            @Override
            public Integer answer(InvocationOnMock invocation) throws Throwable {
                return 7;
            }
        });

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);
    }

    @Test
    public void thenReturnTest() {
        Mockito.when(bar.add(1, 2)).thenReturn(7);

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(7, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(7, result);
    }

    @Test
    public void thenReturn2Test() {
        Mockito.when(bar.add(1, 2)).thenReturn(7, 8, 9);

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(8, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(9, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(9, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(9, result);
    }

    @Test
    public void thenCallRealMethodTest() {
        Mockito.when(bar.add(1, 2)).thenCallRealMethod();

        int result = foo.sum(1, 2);

        Assert.assertEquals(3, result);
    }

    @Test
    public void thenThrowTest() {
        Mockito.when(bar.add(1, 2)).thenThrow(new IllegalArgumentException("xxx"));

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            if (e instanceof IllegalArgumentException) {
                RuntimeException re = (RuntimeException)e;
                Assert.assertEquals("xxx", re.getMessage());
                return;
            }
        }

        Assert.fail();
    }

    @Test
    public void thenThrow2Test() {
        Mockito.when(bar.add(1, 2))
            .thenThrow(new IllegalArgumentException("xxx"), new IllegalArgumentException("yyy"));

        Exception e1 = null;

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (e1 instanceof IllegalArgumentException) {
            Assert.assertEquals("xxx", e1.getMessage());
            e1 = null;
        } else {
            Assert.fail();
        }

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (e1 instanceof IllegalArgumentException) {
            Assert.assertEquals("yyy", e1.getMessage());
            e1 = null;
        } else {
            Assert.fail();
        }

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (e1 instanceof IllegalArgumentException) {
            Assert.assertEquals("yyy", e1.getMessage());
            return;
        } else {
            Assert.fail();
        }

        Assert.fail();
    }

}
```

```java
package cn.theten52.demo.maven;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Mockito;
import org.mockito.invocation.InvocationOnMock;
import org.mockito.junit.MockitoJUnitRunner;
import org.mockito.stubbing.Answer;

@RunWith(MockitoJUnitRunner.class)
public class MockitoTest {

    //foo 对象内部的成员变量会自动被 @Mock 注解的生成的对象注入。
    @InjectMocks
    private Foo foo;

    //bar 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
    @Mock
    private Bar bar;

    @Test
    public void doAnswerTest() {
        Answer<Integer> answer = new Answer<Integer>() {
            @Override
            public Integer answer(InvocationOnMock invocation) throws Throwable {
                return 7;
            }
        };

        Mockito.doAnswer(answer).when(bar).add(1, 2);

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);
    }

    @Test
    public void doReturnTest() {
        Mockito.doReturn(7).when(bar).add(1, 2);

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(7, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(7, result);
    }

    @Test
    public void doReturn2Test() {
        Mockito.doReturn(7, 8, 9).when(bar).add(1, 2);

        int result = foo.sum(1, 2);

        Assert.assertEquals(7, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(8, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(9, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(9, result);

        result = foo.sum(1, 2);
        Assert.assertEquals(9, result);
    }

    @Test
    public void doCallRealMethodTest() {
        Mockito.doCallRealMethod().when(bar).add(1, 2);

        int result = foo.sum(1, 2);

        Assert.assertEquals(3, result);
    }

    @Test
    public void doThrowTest() {
        Mockito.doThrow(NullPointerException.class, IllegalArgumentException.class).when(bar).add(1, 2);

        Exception e1 = null;

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (!(e1 instanceof NullPointerException)) {
            Assert.fail();
        }

        e1 = null;

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (!(e1 instanceof IllegalArgumentException)) {
            Assert.fail();
        }

        e1 = null;

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (!(e1 instanceof IllegalArgumentException)) {
            Assert.fail();
        } else {
            return;
        }

        Assert.fail();
    }

    @Test
    public void doThrow2Test() {
        Mockito
            .doThrow(new IllegalArgumentException("xxx"), new IllegalArgumentException("yyy"))
            .when(bar).add(1, 2);

        Exception e1 = null;

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (e1 instanceof IllegalArgumentException) {
            Assert.assertEquals("xxx", e1.getMessage());
            e1 = null;
        } else {
            Assert.fail();
        }

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (e1 instanceof IllegalArgumentException) {
            Assert.assertEquals("yyy", e1.getMessage());
            e1 = null;
        } else {
            Assert.fail();
        }

        try {
            foo.sum(1, 2);
        } catch (Exception e) {
            e1 = e;
        }
        if (e1 instanceof IllegalArgumentException) {
            Assert.assertEquals("yyy", e1.getMessage());
            return;
        } else {
            Assert.fail();
        }

        Assert.fail();
    }

    @Test
    public void doNotingTest() {
        Mockito.doNothing().when(bar).badCode();

        int count = foo.count();
        Assert.assertEquals(5, count);
    }

}
```

**注意事项**：

1. 默认情况下，对于所有方法的返回值，mock将返回 null、原始/原始包装值或空集合，具体视情况而定。例如 int/Integer 返回0，布尔值/布尔值返回false。

2. 存根可以被覆盖：例如，普通存根可以在进入测试夹具（before方法）时设置，但在正式的测试方法中可以被覆盖。请注意，覆盖存根是一种潜在的代码异味，表示存根过多。

   ```java
   import org.junit.Assert;
   import org.junit.Before;
   import org.junit.Test;
   import org.junit.runner.RunWith;
   import org.mockito.InjectMocks;
   import org.mockito.Mock;
   import org.mockito.Mockito;
   import org.mockito.junit.MockitoJUnitRunner;
   
   @RunWith(MockitoJUnitRunner.class)
   public class MockitoTest {
   
       //foo 对象内部的成员变量会自动被 @Mock 注解的生成的对象注入。
       @InjectMocks
       private Foo foo;
   
       //bar 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
       @Mock(lenient = true)
       private Bar bar;
   
       @Before
       public void before() {
       		//此行存根被覆盖了
           Mockito.doReturn(6).when(bar).add(1, 2);
       }
   
       @Test
       public void doReturnTest() {
           Mockito.doReturn(7).when(bar).add(1, 2);
   
           int result = foo.sum(1, 2);
   
           Assert.assertEquals(7, result);
       }
   
   }
   ```

   或者：

   ```java
    //所有的 mock.someMethod("some arg") 被调用时会返回 "two"
    Mockito.when(mock.someMethod("some arg")).thenReturn("one")
    Mockito.when(mock.someMethod("some arg")).thenReturn("two")
   ```

   当mock检查到有不必须的存根时（只定义而没有使用），会抛出异常：

   ```java
   org.mockito.exceptions.misusing.UnnecessaryStubbingException: 
   Unnecessary stubbings detected in test class: MockitoTest
   Clean & maintainable test code requires zero unnecessary code.
   Following stubbings are unnecessary (click to navigate to relevant line of code):
     1. -> at cn.xxx.demo.MockitoTest.before(MockitoTest.java:25)
   Please remove unnecessary stubbings or use 'lenient' strictness. More info: javadoc for UnnecessaryStubbingException class.
   ```

   解决此问题可以删除不必须的存根代码，或者使用@Mock(lenient = true)标识存在不必须存根的mock对象：

   ```java
       @Mock(lenient = true)
       private Bar bar;
   ```

3. 一旦被存根，该方法将始终返回一个存根值，无论它被调用多少次。

   1. 参考上文`thenReturnTest()`、`doReturnTest()`系列方法。

4. 最后的存根更重要 - 当您多次用相同的参数存根相同的方法时。超过存根次数的调用会最后存根的返回策略。换句话说：**存根的顺序**很**重要，**但它只是很少有意义，例如当存根完全相同的方法调用或有时使用参数匹配器时等。

   1. 参考上文`thenReturn2Test()`、`thenThrowTest()`、`thenThrow2Test()`、`doThrowTest()`、`doThrow2Test()`、`doReturn2Test()`系列方法。

5. 我们不仅能存根mock对象（一般指被@Mock注解的对象），还能存根spy对象（一般指被@Spy注解的对象）。

## 三：自定义返回策略

关于返回策略。请参考本系列文章【Java测试框架系列：Mockito 详解：第一部分：对象创建】。

## 四：更改为存根方法的默认返回值。

我们知道，默认情况下，对于所有方法的返回值，mock将返回 null、原始/原始包装值或空集合，具体视情况而定。

那么我们可以在对具体方法进行存根情况下更改其返回值吗？当然可以，具体细节请参考本系列文章【Java测试框架系列：Mockito 详解：第一部分：对象创建】。

## 五：参数匹配器

### 1:简介

参数匹配器可以使用在方法的存根时，当我们不确定待存根的方法在被调用时的具体的值是多少时，我们就可以使用它来解决这个问题。（它还可以使用在测试结束时的验证/断言处，我们会在下一篇文章中介绍）

Mockito参数验证值时使用自然java风格：即通过使用`equals()`方法进行验证。有时，当需要额外的灵活性时，您可以使用参数匹配器（[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)或[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)）：

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

[`Mockito`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html) 扩展 ArgumentMatchers 以便访问所有匹配器只需静态导入 Mockito 类。

```java
 //使用anyInt()参数匹配器进行存根
 when(mockedList.get(anyInt())).thenReturn("element");

 //以下代码会打印"element"
 System.out.println(mockedList.get(999));

 //你也可以在验证时使用参数匹配器
 verify(mockedList).get(anyInt());
 
```

由于 Mockito对`any(Class)`和`anyInt`家族的匹配器执行类型检查，因此它们不会匹配`null`参数。而是使用`isNull`匹配器。

```java
 // 使用anyBoolean()参数匹配器进行存根
 when(mock.dryRun(anyBoolean())).thenReturn("state");

 // 以下存根不会匹配,而且不会返回"state"
 mock.dryRun(null);

 // 可以将存根改成此方式：
 when(mock.dryRun(isNull())).thenReturn("state");
 mock.dryRun(null); // ok

 // 或者修改代码：
 when(mock.dryRun(anyBoolean())).thenReturn("state");
 mock.dryRun(true); // ok

 
```

以上同样适用于使用了参数匹配器的验证。

**提示**：

**当我们遇到可能为null的参数时，可以偷懒的使用any()方法进行匹配。它可以匹配null和非null值**。

**警告：**
如果您使用参数匹配器，则**所有参数**都必须由匹配器提供。例如：（示例显示的是验证操作，但同样适用于方法的存根）：

```java
 verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
 //以上代码是合适的 - eq()方法时参数匹配器

 verify(mock).someMethod(anyInt(), anyString(), "third argument");
 //以上代码不正确 - 将会抛出异常，因为第三个参数没有使用参数匹配器给出。
 
```

匹配器方法，如`anyObject()`，`eq()` **不**返回匹配器。在内部，它们在堆栈上记录一个匹配器并返回一个虚拟值（通常为空）。此实现是由于 java 编译器强加的静态类型安全。结果是您不能使用`anyObject()`,`eq()`方法在验证/存根之外的方法。

参数配器允许灵活的验证或存根。 查看更多内置匹配器和**[`自定义参数匹配器`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html) /  [`Hamcrest匹配器`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/hamcrest/MockitoHamcrest.html) **的示例。（下文也有介绍）

合理使用复杂的参数匹配。偶尔使用`equals()` 配合 `anyX()`匹配器 的自然匹配风格往往会提供干净和简单的测试。有时最好重构代码以允许`equals()`匹配甚至实现`equals()`方法来帮助测试。

另外，请阅读[第 15 节](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#15)或 javadoc 以了解[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)类。 [`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)是参数匹配器的一种特殊实现，它捕获参数值以进行进一步的断言。

**参数匹配器的相关警告：**

如果您使用参数匹配器，则**所有参数**都必须由匹配器提供。

以下示例展示了在验证时使用参数匹配器，但是在存根方法调用时它同样适用：

```java
   verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
   //上面的写法是正确的 - eq() 也是参数匹配器

   verify(mock).someMethod(anyInt(), anyString(), "third argument");
   //上面的写法是错误的 - 异常会被抛出因为第三个参数并不是参数匹配器的形式
```

匹配器方法如`anyObject()`，`eq()` **不会**返回匹配器。在内部，它们在堆栈上记录一个匹配器并返回一个虚拟值（通常为空）。此实现是由于 java 编译器强加的静态类型安全。结果是您不能在验证/存根之外的方法使用`anyObject()`,`eq()`方法。

### 2:@Captor

允许[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)在字段上创建注解。

例子：

```java
 public class Test{

    @Captor 
    ArgumentCaptor<AsyncCallback<Foo>> captor;

    private AutoCloseable closeable;

    @Before
    public void open() {
       closeable = MockitoAnnotations.openMocks(this);
    }

    @After
    public void release() throws Exception {
       closeable.close();
    }

    @Test public void shouldDoSomethingUseful() {
       //...
       verify(mock).doStuff(captor.capture());
       assertEquals("foo", captor.getValue());
    }
 }
 
```

使用 @Captor 注释的优点之一是您可以避免与捕获复杂泛型类型相关的警告。

### 2:自定义参数匹配器ArgumentMatcher

**在**实现自定义参数匹配器**之前** ，了解处理非平凡参数的用例和可用选项非常重要 。这样，您可以为给定场景选择最佳方法并生成最高质量的测试（干净且可维护）。请继续阅读 [`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html) 的javadoc以了解方法并查看示例。也可以看看：[`AdditionalMatchers`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/AdditionalMatchers.html)。

ArgumentMatcher 允许创建自定义参数匹配器。此 API 在 Mockito 2.1.0 中进行了更改，以将 Mockito 与 Hamcrest 解耦并降低版本不兼容的风险。

对于存根或验证中使用的非平凡方法参数，您有以下选项（无特定顺序）：

- 重构代码，以便与协作者的交互更容易被用于测试。可以将不同的参数传递给该方法，使得模拟更容易吗？如果有些很难测试，通常表明设计可以更好，所以应该为了可测试性而重构！
- 不要严格匹配参数，只需使用任一宽松的参数匹配器之一，例如 [`ArgumentMatchers.notNull()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html#notNull--). 有时，有一个简单的测试比一个看似有效的复杂测试更好。
- 在用作模拟参数的对象中实现 equals() 方法。Mockito 自然地使用 equals() 进行参数匹配。很多时候，这是一个干净而简单的选项。
- 用于[`ArgumentCaptor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentCaptor.html)捕获参数并对其状态执行断言。当您需要验证参数时很有用。如果您需要参数匹配来进行存根，则 Captor 没有用。很多时候，此选项会通过对参数的细粒度验证来实现干净且可读的测试。
- 通过实现[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)接口并将实现传递给[`ArgumentMatchers.argThat(org.mockito.ArgumentMatcher)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html#argThat-org.mockito.ArgumentMatcher-)方法来使用自定义参数匹配器。如果存根需要自定义匹配器并且可以多次重用，则此选项很有用。请注意，[`ArgumentMatchers.argThat(org.mockito.ArgumentMatcher)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html#argThat-org.mockito.ArgumentMatcher-)存在自动拆箱过程中的**NullPointerException**。
- 如果您已经有一个 hamcrest 匹配器，使用 hamcrest 匹配器的实例并将其传递给 [`MockitoHamcrest.argThat(org.hamcrest.Matcher)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/hamcrest/MockitoHamcrest.html#argThat-org.hamcrest.Matcher-)是有用的。重复使用会得到益处！请注意，[`MockitoHamcrest.argThat(org.hamcrest.Matcher)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/hamcrest/MockitoHamcrest.html#argThat-org.hamcrest.Matcher-)存在自动拆箱过程中的**NullPointerException**。
- 仅限 Java 8 - 使用 lambda 代替[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)。因为[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html) 实际上是一个功能接口。lambda 可以与[`ArgumentMatchers.argThat(org.mockito.ArgumentMatcher)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html#argThat-org.mockito.ArgumentMatcher-)方法一起使用。

ArgumentMatcher接口的实现可以与[`ArgumentMatchers.argThat(org.mockito.ArgumentMatcher)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html#argThat-org.mockito.ArgumentMatcher-)方法一起使用。使用`toString()`方法描述匹配器 - 它打印在验证错误中。

```java
 class ListOfTwoElements implements ArgumentMatcher<List> {
     public boolean matches(List list) {
         return list.size() == 2;
     }
     public String toString() {
         //打印验证错误
         return "[list of 2 elements]";
     }
 }

 List mock = mock(List.class);

 when(mock.addAll(argThat(new ListOfTwoElements()))).thenReturn(true);

 mock.addAll(Arrays.asList("one", "two"));

 verify(mock).addAll(argThat(new ListOfTwoElements()));
 
```

为了保持可读性，您可以提取方法，例如：

```java
   verify(mock).addAll(argThat(new ListOfTwoElements()));
   //替换为：
   verify(mock).addAll(listOfTwoElements());
 
```

在 Java 8 中，您可以将 ArgumentMatcher 视为函数式接口并使用 lambda，例如：

```java
   verify(mock).addAll(argThat(list -> list.size() == 2));
```

在 [`Matchers`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Matchers.html) 的 javadoc 中阅读有关其他匹配器的更多信息。

**2.1.0 迁移指南**

所有现有的自定义`ArgumentMatcher`实现将不再长期编译。传递 hamcrest 匹配器的所有位置的`argThat()`将不再长期编译。有两种方法可以解决问题：

- a) 将 hamcrest 匹配器重构为 Mockito 匹配器：使用“implements ArgumentMatcher”而不是“extends ArgumentMatcher”。然后将`describeTo()`方法重构为`toString()`方法。
- b) 使用`org.mockito.hamcrest.MockitoHamcrest.argThat()`代替`Mockito.argThat()`。确保[hamcrest](http://hamcrest.org/JavaHamcrest/)依赖存在于类路径（Mockito 不再依赖于 hamcrest）。

什么选择适合您？如果您不介意编译对 hamcrest 的依赖，那么选项 b) 可能适合您。您的选择应该不会产生太大影响并且是完全可逆的 - 您可以在将来选择不同的选项（并重构代码）。

### 3:ArgumentCaptor

Mockito 通过使用`equals()`方法实现了自然的 Java 风格验证参数值。这也是推荐的匹配参数的方式，因为它使测试变得干净和简单。在某些情况下，在实际验证后对某些参数断言是有帮助的。例如：

```java
   ArgumentCaptor<Person> argument = ArgumentCaptor.forClass(Person.class);
   verify(mock).doSomething(argument.capture());
   assertEquals("John", argument.getValue().getName());
```

捕获可变参数的示例：

```java
   //捕获变量：
   ArgumentCaptor<Person> varArgs = ArgumentCaptor.forClass(Person.class);
   verify(mock).varArgMethod(varArgs.capture());
   List expected = asList(new Person("John"), new Person("Jane"));
   assertEquals(expected, varArgs.getAllValues());
```

**警告**：建议将 ArgumentCaptor 与验证一起使用，**但不要**与存根一起使用。使用带有存根的 ArgumentCaptor 可能会降低测试的可读性，因为 captor 是在断言（又名验证或“then”）块之外创建的。它还会降低缺陷定位，因为如果未调用存根方法，则不会捕获任何参数。

在某种程度上 ArgumentCaptor 与自定义参数匹配器有关（请参阅[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)类的javadoc ）。这两种技术都可用于确保将某些参数传递给mock对象。但是，在以下情况下，ArgumentCaptor 可能更适合：

- 自定义参数匹配器不太可能被重用
- 您只需要它对参数值进行断言即可完成验证

自定义参数匹配器[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html)通常更适合存根。

### 4:附录：

#### [`AdditionalMatchers`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/AdditionalMatchers.html)方法简介：

[`AdditionalMatchers`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/AdditionalMatchers.html)类提供了很少的匹配器，尽管它们在组合多个匹配器或否定必要的匹配器时可能很有用。

| 修饰符和类型               | 方法和说明                                                   |
| -------------------------- | ------------------------------------------------------------ |
| `static <T> T`             | `any()`匹配**任何内容**，包括空值和可变参数。                |
| `static <T> T`             | `any(Class<T> type)`匹配给定类型的任何对象，不包括空值。     |
| `static boolean`           | `anyBoolean()`任何`boolean`或**非空** `Boolean`              |
| `static byte`              | `anyByte()`任何`byte`或者**非空** `Byte`。                   |
| `static char`              | `anyChar()`任何`char`或者**非空** `Character`。              |
| `static <T> Collection<T>` | `anyCollection()`任何**非 null** `Collection`。              |
| `static <T> Collection<T>` | `anyCollectionOf(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static double`            | `anyDouble()`任何`double`或者**非空** `Double`。             |
| `static float`             | `anyFloat()`任何`float`或者**非空** `Float`。                |
| `static int`               | `anyInt()`任何 int**或非 null** `Integer`。                  |
| `static <T> Iterable<T>`   | `anyIterable()`任何**非 null** `Iterable`。                  |
| `static <T> Iterable<T>`   | `anyIterableOf(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static <T> List<T>`       | `anyList()`任何**非 null** `List`。                          |
| `static <T> List<T>`       | `anyListOf(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static long`              | `anyLong()`任何`long`或者**非空** `Long`。                   |
| `static <K,V> Map<K,V>`    | `anyMap()`任何**非 null** `Map`。                            |
| `static <K,V> Map<K,V>`    | `anyMapOf(Class<K> keyClazz, Class<V> valueClazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static <T> T`             | `anyObject()`已弃用。 这将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static <T> Set<T>`        | `anySet()`任何**非 null** `Set`。                            |
| `static <T> Set<T>`        | `anySetOf(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static short`             | `anyShort()`任何`short`或者**非空** `Short`。                |
| `static String`            | `anyString()`任何**非空** `String`                           |
| `static <T> T`             | `anyVararg()`已弃用。 从 2.1.0 开始使用 [`any()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatchers.html#any--) |
| `static <T> T`             | `argThat(ArgumentMatcher<T> matcher)`允许创建自定义参数匹配器。 |
| `static boolean`           | `booleanThat(ArgumentMatcher<Boolean> matcher)`允许创建自定义`boolean`参数匹配器。 |
| `static byte`              | `byteThat(ArgumentMatcher<Byte> matcher)`允许创建自定义`byte`参数匹配器。 |
| `static char`              | `charThat(ArgumentMatcher<Character> matcher)`允许创建自定义`char`参数匹配器。 |
| `static String`            | `contains(String substring)` 包含给定`String` 子字符串的参数。 |
| `static double`            | `doubleThat(ArgumentMatcher<Double> matcher)`允许创建自定义`double`参数匹配器。 |
| `static String`            | `endsWith(String suffix)`以给定`String` 后缀结尾的参数。     |
| `static boolean`           | `eq(boolean value)`等于给定`boolean` 值的参数。              |
| `static byte`              | `eq(byte value)`等于给定`byte` 值的参数。                    |
| `static char`              | `eq(char value)`等于给定`char` 值的参数。                    |
| `static double`            | `eq(double value)`等于给定`double` 值的参数。                |
| `static float`             | `eq(float value)`等于给定`float` 值的参数。                  |
| `static int`               | `eq(int value)`等于给定`int` 值的参数。                      |
| `static long`              | `eq(long value)`等于给定`long` 值的参数。                    |
| `static short`             | `eq(short value)`等于给定`short` 值的参数。                  |
| `static <T> T`             | `eq(T value)`等于给定值的对象参数。                          |
| `static float`             | `floatThat(ArgumentMatcher<Float> matcher)`允许创建自定义`float`参数匹配器。 |
| `static int`               | `intThat(ArgumentMatcher<Integer> matcher)`允许创建自定义`int`参数匹配器。 |
| `static <T> T`             | `isA(Class<T> type)`实现给定`Object` 类的参数。              |
| `static <T> T`             | `isNotNull()`不是`null`的判断。                              |
| `static <T> T`             | `isNotNull(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static <T> T`             | `isNull()`是`null` 的判断。                                  |
| `static <T> T`             | `isNull(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static long`              | `longThat(ArgumentMatcher<Long> matcher)`允许创建自定义`long`参数匹配器。 |
| `static String`            | `matches(Pattern pattern)`与给定正则表达式`Pattern` 匹配的参数。 |
| `static String`            | `matches(String regex)`与给定正则表达式匹`String` 配的参数。 |
| `static <T> T`             | `notNull()`不是`null`的判断。                                |
| `static <T> T`             | `notNull(Class<T> clazz)`已弃用。 在 Java 8 中，此方法将在 Mockito 4.0 中删除。此方法仅用于通用友好性以避免强制转换，Java 8 中不再需要此方法。 |
| `static <T> T`             | `nullable(Class<T> clazz)`参数要么是`null`或者给定类型的。   |
| `static <T> T`             | `refEq(T value, String... excludeFields)` `反射性等于`(使用反射判断是否相等)给定值的对象参数，支持从类中排除所选字段。 |
| `static <T> T`             | `same(T value)`与给定值相同的对象参数。                      |
| `static short`             | `shortThat(ArgumentMatcher<Short> matcher)`允许创建自定义`short`参数匹配器。 |
| `static String`            | `startsWith(String prefix)`以给定`String` 前缀开头的参数。   |

#### Hamcrest 匹配器

允许使用 hamcrest 匹配器匹配参数。 在类路径上**需要**包含 [hamcrest](http://hamcrest.org/JavaHamcrest/)依赖，Mockito**不**依赖于 hamcrest！请注意下面描述的自动拆箱过程中的**NullPointerException**警告。

在实现或重用现有的 hamcrest 匹配器之前，请阅读如何处理[`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html).

Mockito 2.1.0 与 Hamcrest 分离了，以避免过去影响我们用户的版本不兼容。Mockito 提供了一个专用的 API [`ArgumentMatcher`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/ArgumentMatcher.html) 来匹配参数。并提供了 Hamcrest 集成，以便用户可以利用现有的 Hamcrest 匹配器。

例子：

```java
     import static org.mockito.hamcrest.MockitoHamcrest.argThat;

     //进行存根
     when(mock.giveMe(argThat(new MyHamcrestMatcher())));

     //验证
     verify(mock).giveMe(argThat(new MyHamcrestMatcher()));
 
```

自动拆箱过程中的`NullPointerException`警告：在极少数情况下，当匹配原始参数类型时，您**必须**使用 intThat()、floatThat() 等相关的方法。这样你就可以避免在自动拆箱过程中的`NullPointerException`。由于 java 的工作方式，我们实际上并没有一种干净的方法来检测这种情况并保护用户免受此问题的影响。希望这段文字能很好地描述问题和解决方案。如果您知道如何解决问题，请通过邮件列表或问题跟踪器告诉我们。

| 修饰符和类型     | 方法和说明                                                   |
| ---------------- | ------------------------------------------------------------ |
| `static <T> T`   | `argThat(org.hamcrest.Matcher<T> matcher)`允许使用 hamcrest 匹配器匹配参数。 |
| `static boolean` | `booleanThat(org.hamcrest.Matcher<Boolean> matcher)`启用与原始`boolean`参数匹配的集成 hamcrest 匹配器。 |
| `static byte`    | `byteThat(org.hamcrest.Matcher<Byte> matcher)`启用与原始`byte`参数匹配的集成 hamcrest 匹配器。 |
| `static char`    | `charThat(org.hamcrest.Matcher<Character> matcher)`启用与原始`char`参数匹配的集成 hamcrest 匹配器。 |
| `static double`  | `doubleThat(org.hamcrest.Matcher<Double> matcher)`启用与原始`double`参数匹配的集成 hamcrest 匹配器。 |
| `static float`   | `floatThat(org.hamcrest.Matcher<Float> matcher)`启用与原始`float`参数匹配的集成 hamcrest 匹配器。 |
| `static int`     | `intThat(org.hamcrest.Matcher<Integer> matcher)`启用与原始`int`参数匹配的集成 hamcrest 匹配器。 |
| `static long`    | `longThat(org.hamcrest.Matcher<Long> matcher)`启用与原始`long`参数匹配的集成 hamcrest 匹配器。 |
| `static short`   | `shortThat(org.hamcrest.Matcher<Short> matcher)`启用与原始`short`参数匹配的集成 hamcrest 匹配器。 |

## 六：最佳实践

- 1.doXXX()系列方法和thenXXX()系列方法我们该使用哪种方式？
  - **推荐所有的对方法的存根操作使用doXXX()系列方法**。
  - 第一：do系列方法可以正确的处理void方法的存根操作。
  - 第二：do系列方法在存根spy对象时可以有好的进行。（避免被存根方法的实际代码逻辑会在存根时被调用一次的问题，上文中有描述此问题发生的原因）