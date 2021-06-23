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



- 参数匹配器
  - 参数匹配器一般使用在存根方法的调用时。
  - 参数匹配器也可以使用在方法的验证时。
  - 3：参数匹配
    - 有关于参数匹配器的介绍
- 示例
  - 2：添加一些存根（stub）：指定mock对象方法调用的返回值
    - 3个注意事项
  - 5.存根有异常的void方法
  - 10.[存根连续调用](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#stubbing_consecutive_calls)（iterator-style stubbing）
  - 11.[使用回调进行存根](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#answer_stubs)
  - 12.[`doReturn()`| `doThrow()`| `doAnswer()`| `doNothing()`| `doCallRealMethod()`方法族](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#do_family_methods_stubs)
  - 13.[监视真实对象：使用spy](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy)
    - 包含了一些spy使用时的提示
  - 14.更改未[存根调用的默认返回值](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#defaultreturn)（自 1.7 起）
  - 16.[真正的部分mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#partial_mocks)（自 1.8.0 起）
    - 使用spy或者mock。
    - thenCallRealMethod()方法。
- 注意事项：
  - TODO

## 3.常见问题

- 1.doXXX()系列方法和thenXXX()系列方法我们该使用哪种方式？
  - **推荐所有的对方法的存根操作使用doXXX()系列方法**。
  - 第一：do系列方法可以正确的处理void方法的存根操作。
  - 第二：do系列方法在存根spy对象时可以有好的进行。（避免被存根方法的实际代码逻辑会在存根时被调用一次的问题）