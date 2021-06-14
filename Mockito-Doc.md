# Mockito 简介

## Mockito 主要功能：“创建mock对象”、“验证交互”和“存根（stub）方法调用”。

名词解释：

- stub：存根。即配置mock对象的某个方法被调用时该返回什么样的结果的过程。

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
- [`spy()`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#spy-T-)/[`@Spy`](http://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Spy.html): 部分模拟, 真正的方法会被调用但是仍然可以被验证和存根。
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