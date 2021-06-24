# Mockito 详解：第三部分：结果验证

## 零：准备代码

```java
public class Foo {

    private Bar bar;
    private Tou tou;

    public int sum(int a, int b) {
        return bar.add(a, b);
    }

    public int count() {
        bar.badCode();
        return 5;
    }
}
public class Bar {

    public int add(int a, int b) {
        return a + b;
    }

    public void badCode() {
        throw new RuntimeException("bad bar code");
    }
}
public class Tou {

    public void tou() {
        throw new RuntimeException("tou");
    }
}
```

## 一：方法是否被调用/方法的调用的次数

- `atLeast(int minNumberOfInvocations)`允许至少 x 次调用的验证。 
- `atLeastOnce()`允许至少一次调用的验证。                      
- `atMost(int maxNumberOfInvocations)`允许最多 x 次调用的验证。 
- `atMostOnce()`允许最多一次调用的验证。                
- `never()` `times(0)`的别名，见[`times(int)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#times-int-) 。 
- `only()`允许检查给定的方法是否只调用一次。                   
- `times(int wantedNumberOfInvocations)`允许验证调用的确切次数。
- `verify(T mock)`验证某些行为**发生过一次**。                 
- `verify(T mock, VerificationMode mode)`验证某些行为至少发生过一次/确切的次数/从未发生过。 
- `verifyNoInteractions(Object... mocks)`验证给定的mock对象上没有发生交互。 
- `verifyNoMoreInteractions(Object... mocks)`检查任何给定的mock对象上是否有任何未经验证的交互。 
- `validateMockitoUsage()`验证测试代码中是否有书写错误的地方。

示例：

```java
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

    //tou 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
    @Mock(lenient = true)
    private Tou tou;

    @Test
    public void doReturnTest() {
        Mockito.doReturn(7).when(bar).add(1, 2);

        foo.sum(1, 2);

        //1:验证 bar.add(1,2) 方法被调用了一次。
        Mockito.verify(bar).add(1, 2);
        //2:等同于1。
        Mockito.verify(bar, Mockito.times(1)).add(1, 2);
        //3:验证 bar.add(1,2) 方法至少被调用了n次。
        Mockito.verify(bar, Mockito.atLeast(1)).add(1, 2);
        //4:验证 bar.add(1,2) 方法至少被调用了一次。
        Mockito.verify(bar, Mockito.atLeastOnce()).add(1, 2);
        //5:验证 bar.add(1,2) 方法至多被调用了n次。
        Mockito.verify(bar, Mockito.atMost(1)).add(1, 2);
        //6:验证 bar.add(1,2) 方法至多被调用了一次。
        Mockito.verify(bar, Mockito.atMostOnce()).add(1, 2);
        //7:验证 bar.badCode() 方法从没有被调用过。
        Mockito.verify(bar, Mockito.never()).badCode();
        //8:验证 bar.add(1,2) 方法只被调用了一次。
        Mockito.verify(bar, Mockito.only()).add(1, 2);
        //9:验证给定的mock对象上没有发生交互。
        Mockito.verifyNoInteractions(tou);
        //10:检查任何给定的mock对象上是否有任何未经验证的交互。（测试执行过程中和bar对象的交互都验证过了，此时验证通过）
        Mockito.verifyNoMoreInteractions(bar);
        //11:验证测试代码中是否有书写错误的地方。
        Mockito.validateMockitoUsage();
    }

}
```

## 二：方法执行时间的验证

- `after(long millis)`在给定的毫秒数后将触发验证，允许测试异步代码。
- `timeout(long millis)`验证将一遍又一遍地触发，直到给定的毫秒数，允许测试异步代码。

```java
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

    //tou 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
    @Mock(lenient = true)
    private Tou tou;

    @Test
    public void doReturnTest() {
        Mockito.doReturn(7).when(bar).add(1, 2);

        foo.sum(1, 2);

        //1:验证 bar.add(1,2) 方法在100ms内执行完成。在给定的毫秒数后将触发验证，允许测试异步代码。
        Mockito.verify(bar, Mockito.after(100)).add(1, 2);
        //2:验证 bar.add(1,2) 方法在100ms内执行完成。验证将一遍又一遍地触发，直到给定的毫秒数，允许测试异步代码。
        Mockito.verify(bar, Mockito.timeout(100)).add(1, 2);
    }

}
```

**更多关于timeout()方法的介绍**：

验证将一遍又一遍地触发，直到给定的毫秒数，允许测试异步代码。当与mock对象的交互尚未发生时很有用。`timeout()`方法的广泛使用可能是一种代码异味——有更好的方法来测试并发代码。

另请参阅[`after(long)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#after-long-)测试异步代码的方法。`timeout()`和`after`之间的差异在 [`after(long)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#after-long-) 的 Javadoc.

```java
   //当 somemethod()的执行时间不大于100毫秒时通过
   //满足验证时立即退出（例如，可能不会等待100毫秒）
   verify(mock, timeout(100)).someMethod();
   //上面代码也可以写成这样：
   verify(mock, timeout(100).times(1)).someMethod();

   //只要 someMethod() 在两次调用内时间小于100 ms则通过
   verify(mock, timeout(100).times(2)).someMethod();

   //等效于上方的代码：
   verify(mock, timeout(100).atLeast(2)).someMethod();
 
```

**更多关于after(long)方法的介绍**：

在给定的毫秒数后将触发验证，允许测试异步代码。当与mock对象的交互尚未发生时很有用。`after()`方法的广泛使用可能是一种代码异味——有更好的方法来测试并发代码。

尚未实现与 InOrder 验证一起使用。

另请参阅[`timeout(long)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#timeout-long-)测试异步代码的方法。`timeout()`和之间的差异`after()`解释如下。

```java
   //100毫秒后，somemethod()执行完毕则验证通过
   verify(mock, after(100)).someMethod();
   //上面代码也可以写成这样：
   verify(mock, after(100).times(1)).someMethod();

   //100毫秒后，somemethod()执行2次完毕则验证通过
   verify(mock, after(100).times(2)).someMethod();

   //100毫秒后，如果 someMethod() 没有执行则验证通过
   verify(mock, after(100).never()).someMethod();

   //使用给定的验证模式验证在100ms内someMethod()方法的执行，
   //在使用你自定义的验证模式时是有用的。
   verify(mock, new After(100, yourOwnVerificationMode)).someMethod();
 
```

**timeout() 与 after()**

- timeout() 验证通过后立即退出并成功
- after() 等待完整的时间来检查验证是否通过

例子：

```java
   //1.
   mock.foo();
   verify(mock, after(1000)).foo();
   //等待 1000 ms之后验证会成功

   //2.
   mock.foo();
   verify(mock, timeout(1000)).foo();
   //立即成功
```

## 三：方法调用顺序验证

- `inOrder(Object... mocks)`创建[`InOrder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html)对象，以允许按顺序验证mock的对象。
- `calls(int wantedNumberOfInvocations)`允许按顺序进行非贪婪调用的验证。

**关于inOrder**：

创建[`InOrder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html)对象以允许按顺序验证模拟的对象。

```java
   InOrder inOrder = inOrder(firstMock, secondMock);

   inOrder.verify(firstMock).add("was called first");
   inOrder.verify(secondMock).add("was called second");
 
```

按顺序验证是灵活的 -**您不必**一一**验证所有交互**，而只需按顺序验证您感兴趣的那些**交互**。

此外，您可以创建 InOrder 对象，仅传递与有序验证相关的mock对象。

`InOrder`验证是“贪婪的”，但您几乎不会注意到它。如果您想了解更多信息，请阅读 [此 wiki 页面](https://github.com/mockito/mockito/wiki/Greedy-algorithm-of-verification-InOrder)。

从 Mockito 1.8.4 开始，您可以以顺序敏感的方式使用verifyNoMoreInteractions()。阅读更多：[`InOrder.verifyNoMoreInteractions()`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html#verifyNoMoreInteractions--)。

**关于calls()**：

允许按顺序进行非贪婪验证。例如

```java
   inOrder.verify( mock, calls( 2 )).someMethod( "some arg" );
```

- 如果该方法被调用 3 次，则不会失败，与 times(2) 不同
- 与 atLeast(2) 不同，不会将第三次调用标记为已验证

这种验证方式只能用于顺序验证。

## 四：验证方法参考：

| 修饰符和类型                     | 方法和说明                                                   |
| -------------------------------- | ------------------------------------------------------------ |
| `static VerificationAfterDelay`  | `after(long millis)`在给定的毫秒数后将触发验证，允许测试异步代码。 |
| `static VerificationMode`        | `atLeast(int minNumberOfInvocations)`允许至少 x 调用的验证。 |
| `static VerificationMode`        | `atLeastOnce()`允许至少一次调用的验证。                      |
| `static VerificationMode`        | `atMost(int maxNumberOfInvocations)`允许最多 x 次调用的验证。 |
| `static VerificationMode`        | `atMostOnce()`允许最多一次调用的验证。                       |
| `static VerificationMode`        | `calls(int wantedNumberOfInvocations)`允许按顺序进行非贪婪调用的验证。 |
| `static Object[]`                | `ignoreStubs(Object... mocks)`为了验证，忽略给定mock的存根方法。 |
| `static InOrder`                 | `inOrder(Object... mocks)`创建[`InOrder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html)对象，允许按顺序验证mock的对象。 |
| `static LenientStubber`          | `lenient()`宽松存根，绕过“严格存根”验证（请参阅参考资料[`Strictness.STRICT_STUBS`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/quality/Strictness.html#STRICT_STUBS)）。 |
| `static VerificationMode`        | `never()` `times(0)`的别名，见[`times(int)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#times-int-) |
| `static VerificationMode`        | `only()`允许检查给定的方法是否只调用一次。                   |
| `static <T> void`                | `reset(T... mocks)`聪明 Mockito 用户几乎不使用此功能，因为他们知道这可能是测试不佳的迹象。 |
| `static VerificationWithTimeout` | `timeout(long millis)`验证将一遍又一遍地触发，直到给定的毫秒数，允许测试异步代码。 |
| `static VerificationMode`        | `times(int wantedNumberOfInvocations)`允许验证调用的确切次数。 |
| `static void`                    | `validateMockitoUsage()`首先，如果有任何问题，我鼓励您阅读 Mockito FAQ： [https://github.com/mockito/mockito/wiki/FAQ](https://github.com/mockito/mockito/wiki/FAQ) |
| `static <T> T`                   | `verify(T mock)`验证某些行为**发生过一次**。                 |
| `static <T> T`                   | `verify(T mock, VerificationMode mode)`验证某些行为至少发生过一次/确切的次数/从未发生过。 |
| `static void`                    | `verifyNoInteractions(Object... mocks)`验证给定的模拟上没有发生交互。 |
| `static void`                    | `verifyNoMoreInteractions(Object... mocks)`检查任何给定的模拟是否有任何未经验证的交互。 |



