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

### 2.方法执行时间的验证

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

### 3.方法调用顺序验证

- `calls(int wantedNumberOfInvocations)`允许按顺序进行非贪婪调用的验证。
- `inOrder(Object... mocks)`创建[`InOrder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html)对象，允许按顺序验证mock的对象。

### 4.参考：

- 1：验证mock对象的行为（方法是否被调用以及调用返回值）
- 4：验证确切的调用次数/至少调用x次/从未调用
- 6：调用顺序验证
- 7.确保在mock对象从未发生交互
- 8.寻找多余的调用
- 15.为进一步的断言[捕获参数](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#captors)（自 1.8.0 起）
  - 一些关于捕获参数进行断言的警告。
- 22.[超时验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verification_timeout)（自 1.8.5 起）
- 35.[自定义验证失败信息](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Custom_verification_failure_message)（自 2.1.0 起）
- 40.[ 使用“更严格”的 Mockito 提高生产力和编写更简明的测试](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#strict_mockito)（自 2.+ 起）

### 5.验证方法参考：

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

# 四：Mockito中有用的操作

- 17.[重置mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#resetting_mocks)（自 1.8.0 起）

  - Mockito会忘掉了所有的交互和存根。

- 18.[故障排除和验证框架的使用](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#framework_validation)（自 1.8.0 起）

- 19.[行为驱动开发的别名](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#bdd_mockito)（自 1.8.0 起）

- 20.[可序列化的mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#serializable_mocks)（自 1.8.1 起）

- 24.[单行存根](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#one_liner_stub)（自 1.9.0 起）

- 25.[忽略存根的验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#ignore_stubs_verification)（自 1.9.0 起）

- 26.[mockingDetails](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mocking_details)（2.2.x 改进）

- 27.[将调用委托给真实对象](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#delegating_call_to_real_instance)（自 1.9.5 起）

- 28.[`MockMaker` API](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mock_maker_plugin)（从 1.9.5 开始）

- 29.[BDD 风格验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#BDD_behavior_verification)（自 1.10.0 起）

- 31.[Mockito 模拟可以跨类加载器*序列化*/*反序列化*（自 1.10.0 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#serilization_across_classloader)

- 34.[切换插件的*启用*或*禁用*（15年10月1日以来）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#plugin_switch)

- 36.[Java 8 Lambda 匹配器支持](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Java_8_Lambda_Matching)（自 2.1.0 起）

- 41.[ 用于框架集成的高级公共 API（自 2.10.+ 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#framework_integrations_api)

- 42.[ 用于集成的新 API：监听验证开始事件（自 2.11.+ 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verifiation_started_listener)

- 43.[ 用于集成的新 API：可用于测试框架的`MockitoSession`（自 2.15.+ 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_session_testing_frameworks)

- 44.[ 已弃用，`org.mockito.plugins.InstantiatorProvider`因为它会泄漏内部 API。它被替换为`org.mockito.plugins.InstantiatorProvider2 (Since 2.15.4)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_instantiator_provider_deprecation)

- 45.[新的 JUnit Jupiter (JUnit5+) 扩展](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#junit5_mockito)

- 46.[ 新的`Mockito.lenient()`和`MockSettings.lenient()`的方法（自2.20.0）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_lenient)

- 47.[用于清除内联mock中mock状态的新 API（自 2.25.0 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#clear_inline_mocks)

  

# 五：参考

Mockito Javadoc参考：

| 修饰符和类型                       | 方法和说明                                                   |
| ---------------------------------- | ------------------------------------------------------------ |
| `static VerificationAfterDelay`    | `after(long millis)`在给定的毫秒数后将触发验证，允许测试异步代码。 |
| `static VerificationMode`          | `atLeast(int minNumberOfInvocations)`允许至少 x 调用的验证。 |
| `static VerificationMode`          | `atLeastOnce()`允许至少一次调用的验证。                      |
| `static VerificationMode`          | `atMost(int maxNumberOfInvocations)`允许最多 x 次调用的验证。 |
| `static VerificationMode`          | `atMostOnce()`允许最多一次调用的验证。                       |
| `static VerificationMode`          | `calls(int wantedNumberOfInvocations)`允许按顺序进行非贪婪调用的验证。 |
| `static void`                      | `clearAllCaches()`清除所有mock、类型缓存和检测。             |
| `static <T> void`                  | `clearInvocations(T... mocks)`仅在存根不重要时使用此方法清除调用。 |
| `static VerificationMode`          | `description(String description)`添加要在验证失败时打印的说明。 |
| `static Stubber`                   | `doAnswer(Answer answer)`当你想要和通用的 [`Answer`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/stubbing/Answer.html) 一起存根void方法时使用`doAnswer()`。 |
| `static Stubber`                   | `doCallRealMethod()`使用`doCallRealMethod()`时会调用（执行）真正的方法。 |
| `static Stubber`                   | `doNothing()`使用`doNothing()`设置void方法什么也不做。       |
| `static Stubber`                   | `doReturn(Object toBeReturned)`在那些极少数情况下，你不能使用[`when(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#when-T-)时，使用`doReturn()`。 |
| `static Stubber`                   | `doReturn(Object toBeReturned, Object... toBeReturnedNext)`与[`doReturn(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doReturn-java.lang.Object-)相同，但可以设置连续的返回值。 |
| `static Stubber`                   | `doThrow(Class<? extends Throwable> toBeThrown)` 要存根void方法并抛出异常时使用`doThrow()`。 |
| `static Stubber`                   | `doThrow(Class<? extends Throwable> toBeThrown, Class<? extends Throwable>... toBeThrownNext)`与[`doThrow(Class)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#doThrow-java.lang.Class-)相同，但可以设置连续的异常。 |
| `static Stubber`                   | `doThrow(Throwable... toBeThrown)`要存根void方法并抛出异常时使用`doThrow()`,支持连续抛出异常。 |
| `static MockitoFramework`          | `framework()`为高级用户或框架集成商提供。                    |
| `static Object[]`                  | `ignoreStubs(Object... mocks)`为了验证，忽略给定mock的存根方法。 |
| `static InOrder`                   | `inOrder(Object... mocks)`创建[`InOrder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html)对象，允许按顺序验证mock的对象。 |
| `static LenientStubber`            | `lenient()`宽松存根，绕过“严格存根”验证（请参阅 参考资料[`Strictness.STRICT_STUBS`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/quality/Strictness.html#STRICT_STUBS)）。 |
| `static <T> T`                     | `mock(Class<T> classToMock)`创建给定类或接口的mock对象。     |
| `static <T> T`                     | `mock(Class<T> classToMock, Answer defaultAnswer)`使用指定的Answer策略创建mock以交互。 |
| `static <T> T`                     | `mock(Class<T> classToMock, MockSettings mockSettings)`创建具有一些非标准设置的mock。 |
| `static <T> T`                     | `mock(Class<T> classToMock, String name)`指定mock名称。      |
| `static <T> MockedConstruction<T>` | `mockConstruction(Class<T> classToMock)`为给定类的所有构造器创建线程本地mock控制器。 |
| `static <T> MockedConstruction<T>` | `mockConstruction(Class<T> classToMock, java.util.function.Function<MockedConstruction.Context,MockSettings> mockSettingsFactory)`为给定类的所有构造器创建线程本地mock控制器。 |
| `static <T> MockedConstruction<T>` | `mockConstruction(Class<T> classToMock, java.util.function.Function<MockedConstruction.Context,MockSettings> mockSettingsFactory, MockedConstruction.MockInitializer<T> mockInitializer)`为为给定类的所有构造器创建线程本地mock控制器。 |
| `static <T> MockedConstruction<T>` | `mockConstruction(Class<T> classToMock, MockedConstruction.MockInitializer<T> mockInitializer)`为给定类的所有构造器创建线程本地mock控制器。 |
| `static <T> MockedConstruction<T>` | `mockConstruction(Class<T> classToMock, MockSettings mockSettings)`为给定类的所有构造器创建线程本地mock控制器。 |
| `static <T> MockedConstruction<T>` | `mockConstruction(Class<T> classToMock, MockSettings mockSettings, MockedConstruction.MockInitializer<T> mockInitializer)`为给定类的所有构造器创建线程本地mock控制器。 |
| `static <T> MockedConstruction<T>` | `mockConstructionWithAnswer(Class<T> classToMock, Answer defaultAnswer, Answer... additionalAnswers)`为给定类的所有构造器创建线程本地mock控制器。 |
| `static MockingDetails`            | `mockingDetails(Object toInspect)`返回一个 MockingDetails 实例，该实例允许检查特定对象以获取 Mockito 相关信息。 |
| `static MockitoSessionBuilder`     | `mockitoSession()` `MockitoSession` 是一个可选的、强烈推荐的功能，它通过消除样板代码和添加额外的验证来帮助推动更清晰的测试。 |
| `static <T> MockedStatic<T>`       | `mockStatic(Class<T> classToMock)`为给定类或接口的所有静态方法创建线程本地mock控制器。 |
| `static <T> MockedStatic<T>`       | `mockStatic(Class<T> classToMock, Answer defaultAnswer)`为给定类或接口的所有静态方法创建线程本地mock控制器。 |
| `static <T> MockedStatic<T>`       | `mockStatic(Class<T> classToMock, MockSettings mockSettings)`为给定类或接口的所有静态方法创建线程本地mock控制器。 |
| `static <T> MockedStatic<T>`       | `mockStatic(Class<T> classToMock, String name)`为给定类或接口的所有静态方法创建线程本地mock控制器。 |
| `static VerificationMode`          | `never()` `times(0)`的别名，见[`times(int)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#times-int-) |
| `static VerificationMode`          | `only()`允许检查给定的方法是否只调用一次。                   |
| `static <T> void`                  | `reset(T... mocks)`聪明 Mockito 用户几乎不使用此功能，因为他们知道这可能是测试不佳的迹象。 |
| `static <T> T`                     | `spy(Class<T> classToSpy)`请参阅 的文档[`spy(Object)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy-T-)。 |
| `static <T> T`                     | `spy(T object)`创建真实对象的监视。                          |
| `static VerificationWithTimeout`   | `timeout(long millis)`验证将一遍又一遍地触发，直到给定的毫秒数，允许测试异步代码。 |
| `static VerificationMode`          | `times(int wantedNumberOfInvocations)`允许验证调用的确切次数。 |
| `static void`                      | `validateMockitoUsage()`首先，如果有任何问题，我鼓励您阅读 Mockito FAQ：[https](https://github.com/mockito/mockito/wiki/FAQ) : [//github.com/mockito/mockito/wiki/FAQ](https://github.com/mockito/mockito/wiki/FAQ) |
| `static <T> T`                     | `verify(T mock)`验证某些行为**发生过一次**。                 |
| `static <T> T`                     | `verify(T mock, VerificationMode mode)`验证某些行为至少发生过一次/确切的次数/从未发生过。 |
| `static void`                      | `verifyNoInteractions(Object... mocks)`验证给定的模拟上没有发生交互。 |
| `static void`                      | `verifyNoMoreInteractions(Object... mocks)`检查任何给定的模拟是否有任何未经验证的交互。 |
| `static void`                      | `verifyZeroInteractions(Object... mocks)`已弃用。 从 3.0.1 开始。请将您的代码迁移到[`verifyNoInteractions(Object...)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verifyNoInteractions-java.lang.Object...-) |
| `static <T> OngoingStubbing<T>`    | `when(T methodCall)`创建方法的存根。                         |
| `static MockSettings`              | `withSettings()`允许使用其他mock设置进行mock创建。           |

