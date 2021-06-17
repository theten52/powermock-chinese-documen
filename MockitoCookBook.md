# Mockito 使用手册

## 概述

## 创建mock/spy对象

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
    - Mockito.RETURNS_DEEP_STUBS;
    - Mockito.RETURNS_MOCKS;
    - Mockito.RETURNS_SELF;
    - Mockito.RETURNS_SMART_NULLS;

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

  - 与mock对象的区别：mock对象默认是返回类型的空值。而spy对象是默认执行实际方法并返回。

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

## 验证方法调用

### 方法是否被调用

### 调用的返回值

## 存根方法调用