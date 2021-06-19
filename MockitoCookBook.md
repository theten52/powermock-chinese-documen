# Mockito 使用手册

## 零：概述

## 一：创建mock/spy对象

### 创建mock对象

- 使用Mockito.mock()方法。

  ```java
  {
          //org.mockito.Mockito.mock(java.lang.Class<T>)
          //org.mockito.Mockito.mock(java.lang.Class<T>, java.lang.String)
          //org.mockito.Mockito.mock(java.lang.Class<T>, org.mockito.stubbing.Answer)
          //org.mockito.Mockito.mock(java.lang.Class<T>, org.mockito.MockSettings)
          
          //方式1
          Foo foo1 = Mockito.mock(Foo.class);
          //方式2：给mock指定名称
          Foo foo2 = Mockito.mock(Foo.class, "mock_1");
          //方式3：给mock指定自定义返回逻辑
          Foo foo3 = Mockito.mock(Foo.class, new Answer() {
              @Override
              public Object answer(InvocationOnMock invocation) throws Throwable {
                  //自定义返回逻辑
                  return null;
              }
          });
          //方式4：使用MockSettings配置当前mock
          Foo foo4 = Mockito.mock(Foo.class, Mockito.withSettings().defaultAnswer(Mockito.RETURNS_SMART_NULLS));
  
      }
  ```
  
- 使用@InjectMocks和@Mock注解。

  - 被@InjectMocks注解的对象一般就是我们被测的对象。它会被自动的实例化。且其包含的成员变量会被相应的@Mock注解的对象自动赋值。注意观察下面的示例：

  ```java
  
  public class Foo {
  
      private Bar bar;
  
      public int sum(int a, int b) {
          return bar.add(a, b);
      }
  }
  ```

  ```java
  
  public class Bar {
  
      public int add(int a, int b) {
          return a + b;
      }
  }
  ```

  ```java
  public class MockitoTest {
      @InjectMocks
      private Foo foo;
  
      @Mock(
          answer = Answers.RETURNS_DEFAULTS,
          stubOnly = false,
          name = "bar",
          extraInterfaces = {},
          serializable = false,
          lenient = false
      )
      private Bar bar;
  
      @Test
      public void mockTest() {
          Mockito.when(bar.add(1, 2)).thenReturn(7);
          int result = foo.sum(1, 2);
          Assert.assertEquals(7, result);
      }
  }
  ```
  
- 注意事项:
  
  - 默认情况下，对于所有方法的返回值，mock 视情况而定将返回 null、原始/原始包装值或空集合。例如 int/Integer 返回0，布尔值/布尔值返回false。我们可以在创建mock时通过指定默认的Answer策略来更改此返回。支持的Answer策略：
  
    - Mockito.RETURNS_DEFAULTS;
      - 这个实现首先尝试全局配置，如果没有全局配置，那么它将使用一个返回0、空集合、空值等的默认Answer。
    - Mockito.RETURNS_DEEP_STUBS;
      - 链式调用避免空指针。
    - Mockito.RETURNS_MOCKS;
      - 首先尝试返回普通值(0、空集合、空字符串等)，然后尝试返回mock。如果返回类型不能被mock(例如final)，则返回普通的null。
    - Mockito.RETURNS_SELF;
      - 允许Builder 的mock在调用方法时返回其本身，该方法返回的Type等于类或父类。 请记住，这个Answer使用方法的返回类型。如果此类型可分配给mock类，则它将返回mock。因此，如果您有一个返回超类(例如Object)的方法，它将匹配并返回mock。
    - Mockito.RETURNS_SMART_NULLS;
      - 此实现在处理遗留代码时很有帮助。非存根方法通常返回null。如果代码使用非存根调用返回的对象，则会得到NullPointerException。这个Answer的实现返回SmartNull而不是null。SmartNull给出了比NPE更好的异常消息，因为它指出了调用无存根方法的那一行。您只需单击堆栈即可跟踪。 SmartNull首先尝试返回普通值(0，空集合，空字符串等)，然后尝试返回SmartNull。如果返回类型是final，则返回纯null。 在Mockito 4.0.0中，ReturnsSmartNulls可能是默认的返回值策略。
    - Mockito.CALLS_REAL_METHODS
      - 一个调用实际方法(用于部分mock)的Answer。

### 创建spy对象

- 使用Mockito.spy()方法

  ```java
  {
          //org.mockito.Mockito.spy(java.lang.Class<T>)
          //org.mockito.Mockito.spy(T)
  
          //方式1：spy一个具体的类型
          List spy1 = Mockito.spy(List.class);
          //方式2：spy一个已存在的对象
          List spy2 = Mockito.spy(new ArrayList<>());
      }
  ```

  

- 使用@Spy注解。

  - 被@InjectMocks注解的对象一般就是我们被测的对象。它会被自动的实例化。且其包含的成员变量会被相应的@Spy注解的对象自动赋值。注意观察下面的示例：
  - 注意使用了`@RunWith(MockitoJUnitRunner.class)`注解。

  ```java
  @RunWith(MockitoJUnitRunner.class)
  public class MockitoTest {
      @InjectMocks
      private Foo foo;
  
      @Spy
      private Bar bar;
  
      //也可以这样
      @Spy
      private Bar bar2 = new Bar();
  
      @Test
      public void mockTest() {
          //对spy对象打桩
          Mockito.when(bar.add(1, 2)).thenReturn(7);
          int result = foo.sum(1, 2);
          Assert.assertEquals(7, result);
      }
  
      @Test
      public void mockTest2() {
          //不对spy对象打桩，则调用实际的方法。
          //注意同mock对象的区别：mock对象默认是返回类型的空值。而spy对象是默认执行实际方法并返回。
          int result = foo.sum(1, 2);
          Assert.assertEquals(3, result);
      }
  }
  ```

- 注意事项：

  - 与mock对象的区别：

    - mock对象的默认返回类型的空值（可以配置返回策略），不执行实际方法。
    - spy对象是默认执行实际方法并返回。
  
  - 有时将`when(Object)`用于已经存根的spy对象是不可能或不切实际的。因此在使用spy时请考虑使用`doReturn`|`Answer`|`Throw()`存根方法族。例子：
  
    ```java
       List list = new LinkedList();
       List spy = spy(list);
    
       //以下代码是不可能: 真正的函数会被调用，spy.get(0) 会抛出 IndexOutOfBoundsException (list仍然是空的)
       when(spy.get(0)).thenReturn("foo");
    
       //你需要用 doReturn() 去存根
       doReturn("foo").when(spy).get(0);
     
    ```
  
  - Mockito **不会**将调用传递给的真实实例，实际上创建了它的副本。因此，如果您保留真实实例并与之交互，则不要指望监视的人会知道这些交互及其对真实实例状态的影响。相应的，当**unstubbed**（没有进行存根）的方法在**spy对象上**调用但**不在真实实例上时**，您将看不到对真实实例的任何影响。
  
  - 注意`final方法`。Mockito 不会 mock `final方法`，所以底线是：当您监视真实对象时 + 尝试存根 `final方法` = 麻烦。您也将无法验证这些方法

### 创建被测对象

- 使用@InjectMocks注解。

  ```java
  {
      @InjectMocks
      private Foo foo;
  
      @Mock
      private Bar bar;
  
  }
  ```

  ```java
  {
      @InjectMocks
      private Foo foo;
  
      @Spy
      private Bar bar = new Bar();
  
  }
  ```

- 注意事项：

  - @InjectMocks的作用：标记应执行注入的字段。

    - 允许快速的mock和spy注入。
    - 最大限度地减少重复的mock和spy注入代码。

    Mockito 将尝试仅通过构造函数注入、setter 注入或属性注入依次注入mock对象，如下所述。如果以下任一策略失败，则 Mockito**不会报告失败**；即您必须自己提供依赖项。

    1. 构造函数注入：选择最大的构造函数（方法参数最多），然后使用仅在测试类中声明的mock来解析参数。如果使用构造函数成功创建了对象，则 Mockito 不会尝试其他策略。Mockito 决定不破坏具有参数构造函数的对象。

       注意：如果找不到参数，则传递 null。如果需要不可mock的类型，则不会发生构造函数注入。在这些情况下，您必须自己满足依赖项。

    2. 属性设置器注入（setter方法）：Mockito 将首先按类型解析（如果单个类型匹配则不管名称如何都会发生注入），然后，如果有多个相同类型的属性，则通过属性名称和mock名称匹配注入。

       注意1：如果你有相同类型（或类型擦除后相同）的属性，最好用匹配的属性命名所有@Mock注解的字段，否则 Mockito 可能会产生混淆并且不会注入。

       注2：如果@InjectMocks 对象之前没有初始化并且有一个无参数构造函数，那么它将用这个构造函数初始化。

    3. 字段注入：Mockito 将首先按类型解析（如果单个类型匹配则不管名称如何都会发生注入），然后，如果有多个相同类型的属性，则通过字段名称和mock对象名称的匹配进行注入。

       注意1：如果你有相同类型（或类型擦除后相同）的字段，最好用匹配的字段命名所有@Mock注解的字段，否则 Mockito 可能会产生淆并且不会注入。

       注2：如果@InjectMocks 对象之前没有初始化并且有一个无参数构造函数，那么它将用这个构造函数初始化。

    

    例子：

    ```java
    public class ArticleManagerTest extends SampleBaseTestCase {
    
        @Mock
        private ArticleCalculator calculator;
        
        @Mock(name = "database")
        private ArticleDatabase dbMock; // note the mock name attribute
        
        @Spy
        private UserProvider userProvider = new ConsumerUserProvider();
    
        @InjectMocks
        private ArticleManager manager;
    
        @Test
        public void shouldDoSomething() {
            manager.initiateArticle();
            verify(database).addListener(any(ArticleListener.class));
        }
    }
    
    public class SampleBaseTestCase {
    
        private AutoCloseable closeable;
    
        @Before
        public void openMocks() {
            closeable = MockitoAnnotations.openMocks(this);
        }
    
        @After
        public void releaseMocks() throws Exception {
            closeable.close();
        }
    }
    ```

    

    在上面的例子中，`@InjectMocks` 注解的字段 `ArticleManager`只能有一个参数化的构造函数或一个无参数的构造函数，或者两者都有。所有这些构造函数的可见性可以是package-protect、protected或private的，但是 Mockito 不能实例化内部类、本地类、抽象类，当然还有接口。 也要注意私有嵌套静态类。

    同样是 setter 或字段，它们的可见性声明可以是private的，Mockito 将通过反射看到它们。但是，静态或final字段将被忽略。

    所以在需要注入的领域，例如构造函数注入会在这里发生：

    ```java
       public class ArticleManager {
           ArticleManager(ArticleCalculator calculator, ArticleDatabase database) {
               // parameterized constructor
           }
       }
     
    ```

    属性setter注入将在这里发生：

    ```java
       public class ArticleManager {
           // no-arg constructor
           ArticleManager() {  }
    
           // setter
           void setDatabase(ArticleDatabase database) { }
    
           // setter
           void setCalculator(ArticleCalculator calculator) { }
       }
     
    ```

    此处将使用字段注入：

    ```java
       public class ArticleManager {
           private ArticleDatabase database;
           private ArticleCalculator calculator;
       }
     
    ```

    最后，在这种情况下，不会发生注入：

    ```java
       public class ArticleManager {
           private ArticleDatabase database;
           private ArticleCalculator calculator;
    
           ArticleManager(ArticleObserver observer, boolean flag) {
               // observer is not declared in the test above.
               // flag is not mockable anyway
           }
       }
     
    ```

    

    再次注意，@InjectMocks 只会注入使用 @Spy 或 @Mock 注解创建的mocks/spies对象。

    必须调用**`MockitoAnnotations.openMocks(this)`**方法来初始化带注解的对象。在上面的例子中，`openMocks()`在测试基类的@Before (JUnit4) 方法中调用。对于 JUnit3，`openMocks()`可以转到基类的`setup()`方法。 **相反，**您也可以将 openMocks() 放在 JUnit 运行程序 (@RunWith) 中或使用内置的 [`MockitoJUnitRunner`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnitRunner.html). 另外，请确保在使用相应的钩子处理测试类后释放任何mock。

    Mockito 不是依赖注入框架，不要指望这个快速实用程序可以注入复杂的对象图，无论是mocks/spies对象还是真实对象。

### 示例

- 9.mocks的简单创建方式--[`@Mock`注解](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mock_annotation)
- 16.[真正的部分mock](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#partial_mocks)（自 1.8.0 起）
- 20.新注释：[`@Captor`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#captor_annotation), [`@Spy`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spy_annotation), [`@InjectMocks`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#injectmocks_annotation)（自 1.8.3 起）
- 23.[使用`@Spies`， `@InjectMocks`自动实例化对象](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#automatic_instantiation)并具有[良好的构造函数注入](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#constructor_injection)（1.9.0以后）
- 30.[监视或mock抽象类（自 1.10.12 起，在 2.7.13 和 2.7.14 中进一步增强）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#spying_abstract_classes)
- 32.[对深存根（deep stubs）更好的通用支持（自 1.10.0 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#better_generic_support_with_deep_stubs)
- 33.[Mockito JUnit rule（自 1.10.17 起）](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mockito_junit_rule)
- 37.[Java 8 自定义Answer支持](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Java_8_Custom_Answers)（自 2.1.0 起）
- 38.[元数据和泛型类型的保留](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Meta_Data_And_Generics)（自 2.1.0 起）
- 39.[mock final类、枚举和 final方法](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Mocking_Final)（自 2.1.0 起）
- 48.[mock静态方法](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#static_mocks)（自 3.4.0 起）
- 49.[mock对象构造](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#mocked_construction)（自 3.5.0 起）

## 二：存根方法调用

- 定义存根方法的方式:

  - Mockito.when(foo.sum()).thenXXX(...);

    - 即对foo.sum()方法存根。
    - 注意：
      - foo对象应该是一个mock对象。spy对象不建议使用此方式进行存根。因为当代码执行到when(foo.sum())时。foo.sum()方法会首先执行。导致sum()方法的实际代码逻辑被执行。（sum()的实际代码逻辑是否会被执行要看被spy对象的类型，当被spy对象是一个mock对象或者接口时不会执行-这些类型也没有实际代码逻辑可以执行。当被spy对象一个具体的对象时则实际代码逻辑会被执行）

  - Mockito.doXXX(...).when(foo).sum();

    - 即对foo.sum()方法存根。
    - 可以存根void方法。
    - foo对象可以是一个mock对象，也可以是一个spy对象。

  - Mockito.doXXX(....).when(foo.sum());

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

- 定义返回值的方式：

  - | then_xxx方法                       | do_XXX方法                                                   |                                                              |
    | ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | then(Answer\<?> answer)            | doAnswer(Answer answer)                                      | 返回值使用自定义的Answer策略。                               |
    | thenAnswer(Answer\<?> answer)      | 同上                                                         | 同上。                                                       |
    | thenReturn(T value)                | doReturn(Object toBeReturned)                                | 直接指定返回值。                                             |
    | thenReturn(T value, T... values)   | doReturn(Object toBeReturned, Object... toBeReturnedNext)    | 直接指定返回值，可以定义多个返回值。第一次调用到存根方法返回第一个返回值。以此类推。超过返回值数量的调用返回参数的最后一个返回值。 |
    | thenCallRealMethod()               | doCallRealMethod()                                           | 调用实际的代码逻辑。不指定返回值。                           |
    | thenThrow(Throwable... throwables) | doThrow(Throwable... toBeThrown)                             | 调用到存根方法时抛出异常。                                   |
    | 同上                               | doThrow(Class\<? extends Throwable> toBeThrown)              | 调用到存根方法时抛出异常。可以定义多个异常。第一次调用到存根方法返回第一个异常。以此类推。超过异常数量的调用返回参数的最后一个异常。 |
    | 同上                               | doThrow(Class\<? extends Throwable> toBeThrown, Class\<? extends Throwable>... toBeThrownNext) | 同上。                                                       |
    | 无                                 | doNothing()                                                  | void方法使用的存根方式。                                     |

- 参数匹配器

  - 参数匹配器一般使用在存根方法的调用时。
  - 参数匹配器也可以使用在方法的验证时。
  - example-3：参数匹配
    - 有关于参数匹配器的介绍

- 示例
  - example-2：添加一些存根（stub）：指定mock对象方法调用的返回值
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

## 三：验证方法调用

### 验证方法：

### 方法是否被调用/方法的调用的次数

- `atLeast(int minNumberOfInvocations)`允许至少 x 调用的验证。 
- `atLeastOnce()`允许至少一次调用的验证。                      
- `atMost(int maxNumberOfInvocations)`允许最多 x 次调用的验证。 
- `atMostOnce()`允许最多一次调用的验证。                
- `never()` `times(0)`的别名，见[`times(int)`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#times-int-) 。 
- `only()`允许检查给定的方法是否只调用一次。                   
- `times(int wantedNumberOfInvocations)`允许验证调用的确切次数。
- `verify(T mock)`验证某些行为**发生过一次**。                 
- `verify(T mock, VerificationMode mode)`验证某些行为至少发生过一次/确切的次数/从未发生过。 
- `verifyNoInteractions(Object... mocks)`验证给定的模拟上没有发生交互。 
- `verifyNoMoreInteractions(Object... mocks)`检查任何给定的模拟是否有任何未经验证的交互。 

### 方法执行的时间

- `after(long millis)`在给定的毫秒数后将触发验证，允许测试异步代码。
-  `timeout(long millis)`验证将一遍又一遍地触发，直到给定的毫秒数，允许测试异步代码。

### 调用顺序验证

- `calls(int wantedNumberOfInvocations)`允许按顺序进行非贪婪调用的验证。
- `inOrder(Object... mocks)`创建[`InOrder`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/InOrder.html)对象，允许按顺序验证mock的对象。

### 示例：

- example-1：验证mock对象的行为（方法是否被调用以及调用返回值）
- 4：验证确切的调用次数/至少调用x次/从未调用
- 6：调用顺序验证
- 7.确保在mock对象从未发生交互
- 8.寻找多余的调用
- 15.为进一步的断言[捕获参数](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#captors)（自 1.8.0 起）
  - 一些关于捕获参数进行断言的警告。
- 22.[超时验证](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#verification_timeout)（自 1.8.5 起）
- 35.[自定义验证失败信息](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#Custom_verification_failure_message)（自 2.1.0 起）
- 40.[ 使用“更严格”的 Mockito 提高生产力和编写更简明的测试](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/Mockito.html#strict_mockito)（自 2.+ 起）

### 验证方法参考：

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



        Mockito.after(100);
        Mockito.atLeast();
        Mockito.atLeastOnce();
        Mockito.atMost();
        Mockito.atMostOnce();
        Mockito.calls();
        Mockito.clearAllCaches();
        Mockito.clearInvocations();
        Mockito.description();
        Mockito.doAnswer();
        Mockito.doCallRealMethod();
        Mockito.doNothing();
        Mockito.doReturn();
        Mockito.doReturn(null,null);
        Mockito.doThrow(Object.class);
        Mockito.doThrow(null,null);
        Mockito.doThrow(new Exception())
        Mockito.framework()
        Mockito.ignoreStubs()
        Mockito.inOrder()
        Mockito.lenient()
        Mockito.mock(null)
        Mockito.mock(null,null)
        Mockito.mock(null, new MockSettings())
        Mockito.mock(Object.class,"xx")
        Mockito.mockConstruction(6)
        Mockito.mockConstructionWithAnswer()
        Mockito.mockingDetails()
        Mockito.mockitoSession()
        Mockito.mockStatic(4)
        Mockito.never()
        Mockito.only()
        Mockito.reset();
        Mockito.spy(2)
        Mockito.timeout()
        Mockito.times()
        Mockito.validateMockitoUsage();
        Mockito.verify(2)
        Mockito.verifyNoInteractions();
        Mockito.verifyNoMoreInteractions();
        Mockito.verifyZeroInteractions();
        Mockito.when()
        Mockito.withSettings()