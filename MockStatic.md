# Mocking static methods #

## Quick summary ##

  1. Use the `@RunWith(PowerMockRunner.class)` annotation at the class-level of the test case.
  1. Use the `@PrepareForTest(ClassThatContainsStaticMethod.class)` annotation at the class-level of the test case.
  1. Use `PowerMock.mockStatic(ClassThatContainsStaticMethod.class)` to mock all methods of this class.
  1. Use `PowerMock.replay(ClassThatContainsStaticMethod.class)` to change the class to replay mode.
  1. Use `PowerMock.verify(ClassThatContainsStaticMethod.class)` to change the class to verify mode.

## Example ##

Mocking a static method in PowerMock requires the use of the "mockStatic" method in PowerMock. Let's say you have a class, ServiceRegistrator, with a method called registerService that looks like this:

```java
public long registerService(Object service) {
   final long id = IdGenerator.generateNewId();
   serviceRegistrations.put(id, service);
   return id;
}
```

The point of interest here is the static method call to IdGenerator.generateNewId() which we would like to mock. IdGenerator is a helper class that simply generates a new ID for the service. It looks like this:

```java
public class IdGenerator {

   /**
    * @return A new ID based on the current time.
    */
   public static long generateNewId() {
      return System.currentTimeMillis();
   }
}
```

With PowerMock it's possible to expect the call to IdGenerator.generateNewId() just as you would with any other method using EasyMock once you've told PowerMock to prepare the IdGenerator class for testing. You do this by adding an annotation to the class-level of test case. This is simply done be using @PrepareForTest(IdGenerator.class). You also need to tell JUnit to execute the test using PowerMock which is done by using @RunWith(PowerMockRunner.class). Without these two annotations the test will fail.

The actual test is then actually quite simple:

```java
@Test
public void testRegisterService() throws Exception {
	long expectedId = 42;

	// We create a new instance of test class under test as usually.
	ServiceRegistartor tested = new ServiceRegistartor();

	// This is the way to tell PowerMock to mock all static methods of a
	// given class
	mockStatic(IdGenerator.class);

	/*
	 * The static method call to IdGenerator.generateNewId() expectation.
	 * This is why we need PowerMock.
	 */
	expect(IdGenerator.generateNewId()).andReturn(expectedId);

	// Note how we replay the class, not the instance!
	replay(IdGenerator.class);

	long actualId = tested.registerService(new Object());

	// Note how we verify the class, not the instance!
	verify(IdGenerator.class);

	// Assert that the ID is correct
	assertEquals(expectedId, actualId);
}
```

Note that you can mock static methods in a class even though the class is final. The method can also be final. To mock only specific static methods of a class refer to the [partial mocking](MockPartial) section in the documentation.

To mock a static method in a system class you need to follow [this](MockSystem) approach.

## References ##
  * [ServiceRegistratorTest](https://github.com/powermock/powermock-examples-maven/blob/1d4dc5ca3bcc9628f3f4a3b9f0d18a55d48c17b4/tutorial/src/solution/java/demo/org/powermock/examples/tutorial/staticmocking/impl/ServiceRegistratorTest.java)
  * [ServiceRegistrator](https://github.com/powermock/powermock-examples-maven/blob/1d4dc5ca3bcc9628f3f4a3b9f0d18a55d48c17b4/tutorial/src/main/java/demo/org/powermock/examples/tutorial/staticmocking/impl/ServiceRegistrator.java)
  * [IdGenerator](https://github.com/powermock/powermock-examples-maven/blob/1d4dc5ca3bcc9628f3f4a3b9f0d18a55d48c17b4/tutorial/src/main/java/demo/org/powermock/examples/tutorial/staticmocking/impl/IdGenerator.java)