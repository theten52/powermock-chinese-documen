# Mock系统类 #
注意：尽管此处显示的所有示例都是使用EasyMock API扩展制作的，但相同的技术也适用于Mocktio API扩展。

## 总览 ##

  1. 在类级别使用 `@RunWith(PowerMockRunner.class)` 注解。
  1. 在类级别使用 `@PrepareForTest({ClassThatCallsTheSystemClass.class})` 注解。
  1. 使用 `mockStatic(SystemClass.class)` 去Mock系统类，然后设置正常的期望值。 then setup the expectations as normally.
  1. 仅EasyMock only: 使用 `PowerMock.replayAll()` 切换到replay模式。
  1. 仅EasyMock only: 使用 `PowerMock.verifyAll()` 切换到verify模式。

## 示例 ##

PowerMock 1.2.5及更高版本支持Mock Java系统类（例如位于java.lang和java.net等中的类）中的方法。此方法无需修改JVM或IDE设置即可！模拟这些类的方式与平常有些不同。通常，您将准备包含要模拟的静态方法的类（我们称其为X），但是因为PowerMock无法准备要测试的系统类，所以必须采用另一种方法。因此，不用准备X，而是准备在X中调用静态方法的类！让我们看一个简单的例子：

```java
public class SystemClassUser {

	public String performEncode() throws UnsupportedEncodingException {
		return URLEncoder.encode("string", "enc");
	}
}
```

Here we'd like to mock the static method call to `java.net.URLEncoder#encode(..)` which is normally not possible. Since the URLEncoder class is a system class we should prepare the `SystemClassUser` for test since this is the class calling the encode method of the `URLEncoder`. 在这里，我们要模拟通常无法进行的静态方法调用的`java.net.URLEncoder#encode(..)`。由于URLEncoder类是系统类，因此我们应该准备`SystemClassUser`进行测试，因为这是调用`URLEncoder`中encode方法的类。例如：

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( { SystemClassUser.class })
public class SystemClassUserTest {

	@Test
	public void assertThatMockingOfNonFinalSystemClassesWorks() throws Exception {
		mockStatic(URLEncoder.class);

		expect(URLEncoder.encode("string", "enc")).andReturn("something");
		replayAll();

		assertEquals("something", new SystemClassUser().performEncode());

		verifyAll();
	}
}
```