# Mocking private methods #

## Quick summary ##

  1. Use the `@RunWith(PowerMockRunner.class)` annotation at the class-level of the test case.
  1. Use the `@PrepareForTest(ClassWithPrivateMethod.class)` annotation at the class-level of the test case.
  1. Use `PowerMock.createPartialMock(ClassWithPrivateMethod.class, "nameOfTheMethodToMock")` to create a mock object that _only_ mocks the method with name `nameOfTheMethodToMock` in this class (let's call it `mockObject`).
  1. Use `PowerMock.expectPrivate(mockObject, "nameOfTheMethodToMock", argument1, argument2)` to expect the method call to `nameOfTheMethodToMock` with arguments `argument1` and `argument2`.
  1. Use `PowerMock.replay(mockObject)` to change the mock object to replay mode.
  1. Use `PowerMock.verify(mockObject)` to change the mock object to verify mode.

## Example ##

Assume that you have a class called `DataService` that you like to unit test which looks like this:

```java
public class DataService {

	public boolean replaceData(final String dataId, final byte[] binaryData) {
		return modifyData(dataId, binaryData);
	}

	public boolean deleteData(final String dataId) {
		return modifyData(dataId, null);
	}

	/**
	 * Modify the data.
	 * 
	 * @param dataId
	 *            The ID of the data slot where the binary data will be stored.
	 * @param binaryData
	 *            The binary data that will be stored. If <code>null</code>
	 *            this means that the data for the particular slot is considered
	 *            removed.
	 * @return <code>true</code> if the operation was successful,
	 *         <code>false</code> otherwise.
	 */
	private boolean modifyData(final String dataId, final byte[] binaryData) {
		/*
		 * Imagine this method doing something complex and expensive.
		 */
		return true;
	}
}
```

`DataService` has two public methods that both calls a private method called `modifyData`. This method does something that you indeed want to unit test (see the  [bypass encapsulation](BypassEncapsulation) section for information on how to test private methods) but only once. You don't want to duplicate the expectation and verification behavior for the logic performed inside the `modifyData` method when testing the `replaceData` and `deleteData` methods. One way to achieve this is by [partially mocking](MockPartial) the `DataService` class in combination with setting up the correct expectations for the `modifyData` method. PowerMock can assist you with both.

So we begin by creating a partial mock of the `DataService` class where we specify that only the `modifyData` method should be mocked, all other methods will behave as normally. This is easily done using:

```java
DataService tested = PowerMock.createPartialMock(DataService.class, "modifyData");
```

Let's say that we're writing a test case for the `replaceData` method and thus we need to expect the method invocation to `modifyData` so that we can test it in separation. Since `modifyData` is private we make use of the `PowerMock.expectPrivate(..)` functionality in PowerMock. To expect the call using a byte-array containing one byte (42) and return `true` we do like this:

```java
byte[] byteArray = new byte[]{42}
PowerMock.expectPrivate(tested, "modifyData", byteArray).andReturn(true);
```

Here PowerMock will look-up the `modifyData` method using reflection and record the expected invocation. If there are several overloaded methods matching the supplied method-name then PowerMock will try to figure out which one to expect by matching the argument types of the supplied parameters (in this case `byteArray` which is of type `byte[].class`) with the parameter types of the methods. This will work fine in most situations. For example if the `DataService` class had two methods called `modifyData` one taking a `byte[]` as last parameter and one taking a `char[]` as last parameter the expectation above would still work. However there are cases when PowerMock cannot figure out which method to expect by using this simple syntax. For example if you must use EasyMock matchers for an overloaded method or if an overloaded method is using a primitive type and the other method is using a wrapper type such as:

```java
...
private int myMethod(int id) {		
	return 2*id;
}

private int myMethod(Integer id) {		
		return 3*id;
}
...
```

This is of course a naive example but you could still use `PowerMock.expectPrivate(..)` to setup the expectation for these methods. For the `myMethod(int id)` method you'd do like this to tell it to return a value of `20` when invoked:

```java
expectPrivate(mockObject, "myMethod", new Class<?>[] { int.class },10).andReturn(20);
```

Here we explicitly tell PowerMock to expect the call to the `myMethod` that takes an `int` as parameter. But in the majority of cases you don't need to specify the parameter type since PowerMock will find the correct method automatically.

We also need to tell PowerMock which classes to mock. This is done by adding the `@PrepareForTest` annotation to the class-level of test case, in this case `@PrepareForTest(DataService.class)`. You also need to tell JUnit to execute the test using PowerMock which is done by using the `@RunWith(PowerMockRunner.class)` annotation. Without these two annotations the test will fail.

Last but not least the `tested` instance must be replayed and verified using the `PowerMock.replay(tested)` and `PowerMock.verify(tested)` methods.

The full test looks like this:

```java
@Test
public void testReplaceData() throws Exception {
        final String modifyDataMethodName = "modifyData";
        final byte[] expectedBinaryData = new byte[] { 42 };
        final String expectedDataId = "id";

        // Mock only the modifyData method
        DataService tested = createPartialMock(DataService.class, modifyDataMethodName);
      
        // Expect the private method call to "modifyData"
        expectPrivate(tested, modifyDataMethodName, expectedDataId,
                        expectedBinaryData).andReturn(true);

        replay(tested);

        assertTrue(tested.replaceData(expectedDataId, expectedBinaryData));

        verify(tested);
}
```

## References ##
  * [DataServiceTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/privatemocking/DataServiceTest.java)
  * [DataService](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/privatemocking/DataService.java)