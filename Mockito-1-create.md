# Mockito 详解：第一部分：对象创建

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

------

下文示例代码会使用到的对象：

```java
public class Foo {

    private Bar bar;

    public int sum(int a, int b) {
        return bar.add(a, b);
    }
}

public class Bar {

    public int add(int a, int b) {
        return a + b;
    }
}
```

测试中我们创建的对象一般可以分为三种：**被测对象**、**mock对象**和**spy对象**。

首先我们明确一下这三种对象的概念：

1. 被测对象：即我们想要测试的对象，比如xxService、xxUtils等。
2. mock对象：一般为我们被测对象的依赖对象。典型如被测对象的成员变量。主要是一些测试中我们不关注的对象。我们只想要得到这些对象的方法的返回值。而不关注这些方法的具体执行逻辑。此时我们可以将这些对象创建为mock对象。
3. spy对象：在Mockito中它是基于**部分mock**概念提出的。spy对象也可由mock对象使用特定参数下创建。也就是说：**spy对象其实是一种特殊的mock对象。**和mock对象一样，它可以作为被测对象的依赖对象。此时它和mock对象的最大的区别是mock对象的方法如果没有被存根，调用时会返回相应对象的空值（下文有详细介绍）；而spy对象的方法被调用时则会调用真实的代码逻辑。

在基于JUnit测试框架的测试代码书写时，我们有两种方式可以激活Mockito框架对其相关注解（@InjectMocks/@Mock/@Spy）的支持。

1. 使用@RunWith(MockitoJUnitRunner.class)，如下所示

   1. ```java
      public class Foo {
      
          private Bar bar;
      
          public int sum(int a, int b) {
              return bar.add(a, b);
          }
      }
      
      public class Bar {
      
          public int add(int a, int b) {
              return a + b;
          }
      }
      
      @RunWith(MockitoJUnitRunner.class)
      public class MockitoTests {
      
          @InjectMocks
          private Foo foo;
      
          @Mock
          private Bar bar;
      
          @Test
          public void mockTest() {
              Mockito.when(bar.add(1, 2)).thenReturn(7);
              int result = foo.sum(1, 2);
              Assert.assertEquals(7, result);
          }
      }
      ```

2. 使用MockitoAnnotations.openMocks(this)，如下所示：

   1. ```java
      public class MockitoTests {
      
          @InjectMocks
          private Foo foo;
      
          @Mock
          private Bar bar;
      
          @Before
          public void beforeClass() {
              MockitoAnnotations.openMocks(this);
          }
      
          @Test
          public void mockTest() {
              Mockito.when(bar.add(1, 2)).thenReturn(7);
              int result = foo.sum(1, 2);
              Assert.assertEquals(7, result);
          }
      }
      ```

下文中若使用注解方式定义mock对象不再重复说明，优先选择使用@RunWith(MockitoJUnitRunner.class)的方式。

## 一：创建被测试对象

### 1：创建被测对象

#### 使用@InjectMocks注解

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoTest {

    @InjectMocks
    private Foo foo;
  	
    @Test
    public void mockTest() {
      	...
    }
}
```

@InjectMocks的作用：标记应执行注入的字段（指此对象内的字段应该被自动的注入，注入的值来自@Mock或@Spy注解的字段）。

- 允许快速的mock和spy注入。
- 最大限度地减少重复的mock和spy注入代码。

@InjectMock成员变量的自动注入示例：

- 组合@Mock注解使用：

- ```java
  @RunWith(MockitoJUnitRunner.class)
  public class MockitoTest {
  
      //foo 对象内部的成员变量会自动被 @Mock 注解的生成的对象注入。
      @InjectMocks
      private Foo foo;
  
      //bar 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
      @Mock
      private Bar bar;
  
      @Test
      public void mockTest() {
          //先对mock对象的待测方法进行存根
          Mockito.when(bar.add(1, 2)).thenReturn(7);
  
          int result = foo.sum(1, 2);
          //验证是否是存根返回的值
          Assert.assertEquals(7, result);
      }
  
  }
  ```

- 组合@Spy注解的使用：

- ```java
  @RunWith(MockitoJUnitRunner.class)
  public class MockitoTest {
  
      //foo 对象内部的成员变量会自动被 @Mock 注解的生成的对象注入。
      @InjectMocks
      private Foo foo;
  
      //bar 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
      @Spy
      private Bar bar;
  
      @Test
      public void mockTest() {
          //先对spy对象的待测方法进行存根
          Mockito.doReturn(7).when(bar).add(1, 2);
  
          int result = foo.sum(1, 2);
          //验证是否是存根返回的值
          Assert.assertEquals(7, result);
      }
  
  }
  ```

关于注入的一点说明：

1. 使用@InjectMocks注解时，配合使用`@RunWith(MockitoJUnitRunner.class)`或在`@Before`方法中使用`MockitoAnnotations.openMocks(this)`即可激活Mockito对注入相关的支持。
2. @Mock注解的字段会被自动注入到@InjectMocks注解生成的对象的成员变量中。
3. @Spy注解的字段会被自动注入到@InjectMocks注解生成的对象的成员变量中。

上面的两个示例也可说明注入的具体情况。另关于注入的详细逻辑请查阅下文。

### 2.创建mock对象

#### 使用Mockito.mock()方法

```java
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
```

#### 使用@Mock注解

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoTest {
  	//foo 对象内部的成员变量会自动被 @Mock 注解的生成的对象注入。
    @InjectMocks
    private Foo foo;

  	//bar 对象会自动的注入到 @InjectMocks 注解的对象的成员变量中去。
    @Mock
    private Bar bar;

    @Test
    public void mockTest() {
      	//先对mock对象的待测方法进行存根，当真正执行到mock对象的此方法时
      	//会直接返回存根的结果而不会调用mock对象的实际代码
        Mockito.when(bar.add(1, 2)).thenReturn(7);
        
      	int result = foo.sum(1, 2);
      	//验证是否是存根返回的值
        Assert.assertEquals(7, result);
    }
}
```

优点：

- 最大限度地减少重复的mock创建代码。
- 使测试类更具可读性。
- 使验证错误更易于阅读，因为**字段名称**会被用于标识mock。``

注意事项:

- 默认情况下，对于所有方法的返回值，mock 对象视情况而定将返回 null、原始/原始包装值或空集合。例如 int/Integer 返回0，布尔值/布尔值返回false。我们可以在创建mock时通过指定默认的Answer策略来更改此返回。支持的Answer策略请查看下文。


### 3：创建spy对象

#### 使用Mockito.spy()方法

```java
//org.mockito.Mockito.spy(java.lang.Class<T>)
//org.mockito.Mockito.spy(T)

//方式1：spy一个具体的类型
List spy1 = Mockito.spy(List.class);
//方式2：spy一个已存在的对象
List spy2 = Mockito.spy(new ArrayList<>());
```

#### 使用Mockito.mock()方法：

```java
//健壮的 API, 来自 Settings Builder:
OtherAbstract spy = mock(OtherAbstract.class, Mockito.withSettings()
    .useConstructor().defaultAnswer(CALLS_REAL_METHODS));

//Mock使用构造函数(从 2.7.14 可用)一个抽象类
SomeAbstract spy = mock(SomeAbstract.class, Mockito.withSettings()
    .useConstructor("arg1", 123).defaultAnswer(CALLS_REAL_METHODS));

//mock一个非静态内部抽象类
InnerAbstract spy = mock(InnerAbstract.class, Mockito.withSettings().useConstructor().outerInstance(outerInstance).defaultAnswer(CALLS_REAL_METHODS));
```

**由此可见，spy对象也是基于mock对象生成的，或者说spy对象和mock对象是可以互相转换的。**

#### 使用@Spy注解

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
        //对spy对象的add方法进行存根
        Mockito.doReturn(7).when(bar).add(1, 2);
        
      	int result = foo.sum(1, 2);
        Assert.assertEquals(7, result);
    }

    @Test
    public void mockTest2() {
        //不对spy对象存根，则调用实际的方法。
        //注意同mock对象的区别：mock对象默认是返回类型的空值。而spy对象是默认执行实际方法并返回。
        int result = foo.sum(1, 2);
        Assert.assertEquals(3, result);
    }
}
```

**请注意**，spy对象与mock对象的区别：

- 默认的返回策略不同：

  - mock对象的默认返回类型的空值（可以配置返回策略），不执行实际方法。
  - spy对象是默认执行实际方法并返回，可以对spy对象的某个方法进行存根以指定返回值且避免调用此方法实际逻辑。

- 存根的方式不同：

  - 存根的方式有两种：

    - then方式：Mockito.when(foo.sum()).thenXXX(...);
    - do方式：Mockito.doXXX(...).when(foo).sum();

  - 对于mock对象，两种方式都可以使用。但是void方法必须使用do方式的存根。

  - 对于spy对象，只能使用do方式的存根（因为使用then方式会使得被存根方法的实际代码）。

    - ```java
      List list = new LinkedList();
      List spy = spy(list);
      
      //以下代码是不合适的: spy.get(0)方法实际的代码会被调用， 会抛出
      //IndexOutOfBoundsException (list仍然是空的)。
      
      //为什么在存根的时候就会调用方法实际的代码？这样不就无法对此方法进行存根了吗？
      
      //因为在执行when(spy.get(0))的时候首先执行的是when()方法内的spy.get(0)；
      //而此时spy.get(0)还没有进行存根。故此方法的实际代码会被调用。
      
      //要解决这个问题，请使用doXXX()方法配合when()进行存根。
      when(spy.get(0)).thenReturn("foo");
      
      //你需要用doReturn()去存根
      doReturn("foo").when(spy).get(0);
      ```

  - **因此我们可以得到一条最佳实践：对方法的存根只是用do式而不是then方式。**



Mockito **不会**将调用传递给的真实实例，实际上是时创建了它的副本。因此，如果您保留真实实例并与之交互，则不要指望监视的人会知道这些交互及其对真实实例状态的影响。相应的，当**unstubbed**（没有进行存根）的方法在**spy对象上**调用但**不在真实实例上调用时**，您将看不到对真实实例的任何影响。

- 意思就是spy对象其实是真实对象的一个**复制体**；对spy中的方法的调用不会影响到真实对象。



注意 `final方法`。Mockito 不会 mock `final方法`，所以底线是：当您spy一个真实对象时（使用spy()方法） + 尝试存根 `final方法` = 麻烦。您也将无法验证这些方法。

像往常一样，您将看到**部分mock的警告**：面向对象编程通过将复杂性划分为单独的、特定的 SRPy 对象来减少复杂性。部分部分mock如何适应这种范式？嗯，它只是没有......部分mock通常意味着复杂性已转移到同一对象上的不同方法。在大多数情况下，这不是您想要设计应用程序的方式。

但是，在极少数情况下，部分mock会派上用场：处理您无法轻松更改的代码（第 3 方接口、遗留代码的临时重构等）但是，我不会将部分mock用于新的、测试驱动的 & 良好设计的代码。

## 二：返回策略（Answer策略）

### 1：Mockito自带的返回策略

请参考`org.mockito.Answers`枚举：

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

这不是在Mockito中可用的Answer的完整列表。在`org.mockito.stubbs.answers`包中可以找到一些有趣的Answer。

### 2：自定义返回策略

另外：我们还可以自定义Answer策略。如下：

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

也可以尝试使用Java 8 Lambda 表达式：由于[`Answer`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/stubbing/Answer.html)接口只有一种方法，因此可以在 Java 8 中使用 lambda 表达式非常简单的实现它。越需要使用方法调用的参数，就越需要使用[`InvocationOnMock`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/invocation/InvocationOnMock.html).

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

## 三：注入逻辑

【~~太长了这一段可以不看~~：主要说了@InjectMocks注解的对象如何自动注入@Mock和@Spy字段】

**Mockito 将尝试仅通过构造函数注入、setter 注入或属性注入依次注入mock对象，如下所述。如果以下任一策略失败，则 Mockito**不会报告失败**；即您必须自己提供依赖项。

1. 构造函数注入：选择最大的构造函数（方法参数最多），然后使用仅在测试类中声明的mock来解析参数。如果使用构造函数成功创建了对象，则 Mockito 不会尝试其他策略。Mockito 决定不破坏具有参数构造函数的对象。

   注意：如果找不到参数，则传递 null。如果需要不可mock的类型，则不会发生构造函数注入。在这些情况下，您必须自己满足依赖项。

2. 属性设置器注入（setter方法）：Mockito 将首先按类型解析（如果单个类型匹配则不管名称如何都会发生注入），然后，如果有多个相同类型的属性，则通过属性名称和mock名称匹配注入。

   注意1：如果你有相同类型（或类型擦除后相同）的属性，最好用匹配的属性命名所有@Mock注解的字段，否则 Mockito 可能会产生混淆并且不会注入。

   注意2：如果@InjectMocks 对象之前没有初始化并且有一个无参数构造函数，那么它将用这个构造函数初始化。

3. 字段注入：Mockito 将首先按类型解析（如果单个类型匹配则不管名称如何都会发生注入），然后，如果有多个相同类型的属性，则通过字段名称和mock对象名称的匹配进行注入。

   注意1：如果你有相同类型（或类型擦除后相同）的字段，最好用匹配的字段命名所有@Mock注解的字段，否则 Mockito 可能会产生淆并且不会注入。

   注意2：如果@InjectMocks 对象之前没有初始化并且有一个无参数构造函数，那么它将用这个构造函数初始化。

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



再次注意，@InjectMocks 只会注入使用 @Spy 或 @Mock 注解创建的mock/spy对象。

必须调用**`MockitoAnnotations.openMocks(this)`**方法来初始化带注解的对象。在上面的例子中，`openMocks()`在测试基类的@Before (JUnit4) 方法中调用。对于 JUnit3，`openMocks()`可以转到基类的`setup()`方法。 **相反，**您也可以将 openMocks() 放在 JUnit 运行程序 (@RunWith) 中或使用内置的 [`MockitoJUnitRunner`](https://javadoc.io/static/org.mockito/mockito-core/3.11.1/org/mockito/junit/MockitoJUnitRunner.html). 另外，请确保在使用相应的钩子处理测试类后释放任何mock。

Mockito 不是依赖注入框架，不要指望这个快速实用程序可以注入复杂的对象图，无论是mock/spy对象还是真实对象。

## 
