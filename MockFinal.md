# Mocking final methods or classes #

## Quick summary ##

  1. Use the `@RunWith(PowerMockRunner.class)` annotation at the class-level of the test case.
  1. Use the `@PrepareForTest(ClassWithFinal.class)` annotation at the class-level of the test case.
  1. Use `PowerMock.createMock(ClassWithFinal.class)` to create a mock object for all methods of this class (let's call it `mockObject`).
  1. Use `PowerMock.replay(mockObject)` to change the mock object to replay mode.
  1. Use `PowerMock.verify(mockObject)` to change the mock object to verify mode.

## Example ##

Let's assume you're using a third-party library that contains a class called `StateHolder`. For whatever reason this class is final and not only that, the methods are final as well. Imagine also that you need to use this class in your code and you'd still like to be able to unit test your own code in separation. `StateHolder` looks like this:

```java
public final class StateHolder extends JdbcDaoSupport {

	public final String getState() {
		/* Query a database for state, 
                   this is not something we want to do in a unit test */
		getJdbcTemplate().queryForObject("some sql", String.class);
	}
}
```


In this example we imagine having a class called `StateFormatter` that interacts with the `StateHolder` instance. It looks like this:

```java
public class StateFormatter {

	private final StateHolder stateHolder;

	public StateFormatter(StateHolder stateHolder) {
		this.stateHolder = stateHolder;
	}

	public String getFormattedState() {
		String safeState = "State information is missing";
		final String actualState = stateHolder.getState();
		if (actualState != null) {
			safeState = actualState;
		}
		return safeState;
	}
}
```

If we are to test the `getFormattedState` method in separation we need to create a mock of the `StateHolder` class and be able to expect the call to `stateHolder.getState()`. PowerMock's `createMock` method can be used to create a mock object of any class even though it's final. This means that you can use expectations as usually. A unit test of `getFormattedState` method looks something like this:

```java
@Test
public void testGetFormattedState_actualStateExists() throws Exception {
	final String expectedState = "state";

			// We use PowerMock.createMock(..) to create the mock object. 
	StateHolder stateHolderMock = createMock(StateHolder.class);
	StateFormatter tested = new StateFormatter(stateHolderMock);

	expect(stateHolderMock.getState()).andReturn(expectedState);

			// PowerMock.replay(..) must be used. 
	replay(stateHolderMock);

	final String actualState = tested.getFormattedState();

			// PowerMock.verify(..) must be used. 
	verify(stateHolderMock);

	assertEquals(expectedState, actualState);
}
```

Note that you need to tell PowerMock which classes you like to mock. This is done by adding the `@PrepareForTest` annotation to the class-level of test case, in this case `@PrepareForTest(StateHolder.class)`. You also need to tell JUnit to execute the test using PowerMock which is done by using the `@RunWith(PowerMockRunner.class)` annotation. Without these two annotations the test will fail.

See the [partial mocking](MockPartial) section for information on how to mock only specific methods of a class.

## References ##
  * [StateFormatterTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/finalmocking/StateFormatterTest.java)
  * [StateFormatter](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/finalmocking/StateFormatter.java)
  * [StateHolder](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/finalmocking/StateHolder.java)