# Mock construction of new objects #

## Quick summary ##

  1. Use the `@RunWith(PowerMockRunner.class)` annotation at the class-level of the test case.
  1. Use the `@PrepareForTest(ClassThatCreatesTheNewInstance.class)` annotation at the class-level of the test case.
  1. Use `PowerMock.createMock(NewInstanceClass.class)` to create a mock object of the class that should be constructed (let's call it `mockObject`).
  1. Use `PowerMock.expectNew(NewInstanceClass.class).andReturn(mockObject)` to expect a new construction of an object of type `NewInstanceClass.class` but instead return the mock object.
  1. Use `PowerMock.replay(mockObject, NewInstanceClass.class)` to change the mock object and class to replay mode, alternatively use the `PowerMock.replayAll()` method.
  1. Use `PowerMock.verify(mockObject, NewInstanceClass.class)` to change the mock object and class to verify mode, alternatively use the `PowerMock.verifyAll()` method.

## Example ##
Let's say you have a class like this:

```java
public class PersistenceManager {

	public boolean createDirectoryStructure(String directoryPath) {
		File directory = new File(directoryPath);

		if (directory.exists()) {
			throw new IllegalArgumentException("\"" + directoryPath + "\" already exists.");
		}

		return directory.mkdirs();
	}
}
```

What's interesting here is the creation of a new File object. This is interesting because in order to get 100% code coverage of this method we need to be able test the method both when the `directory.exists()` query return `true` and `false`. But in order to unit test this method one needs to substitute the File object created in this method for a mock implementation. One way to do this is to extract the call to `new File(..)` to a package private method which is overridden in the test to return a File mock. But with PowerMock you don't have to change the code, instead you can instruct PowerMock to intercept the call to `new File(..)` and return a mock object instead. To do this we start by creating a mock of type File.class as usual:

```java
File fileMock = createMock(File.class);
```
To expect the call to new File we simply do:

```java
expectNew(File.class, "directoryPath").andReturn(fileMock);
```

The "directoryPath" is the constructor parameter passed to the File constructor. That's it when it comes to the expectation behavior! There are however some things that must be explained further. First of all you need to tell PowerMock which classes to prepare for test and in this case this means the class that does the actual `new` call, in this case it's the `PersistenceManager` class that does `new File(..)`. So in order to provide PowerMock with this information we add `@PrepareForTest(PersistenceManager.class)` at the class-level of test case. The reason why we're not preparing File.class is that a new instance of the File is never created when doing `expectNew`, PowerMock just intercepts the call to new from the class that _does_ `new` and thus you need to prepare the PersistenceManager class in this example. As usually you also need to tell JUnit to execute the test using PowerMock which is done by using the `@RunWith(PowerMockRunner.class)` annotation. Without these two annotations the test will fail!

The last aspect is the replay and verify behavior. In order to correctly replay the expected behavior we need to replay both the `fileMock` object and the File class. I.e.

```java
replay(fileMock, File.class);
```
The verification procedure follows the same rules:

```java
verify(fileMock, File.class);
```
That's it!

If you're using PowerMock >= 0.9 you could alternatively use the `replayAll` and `verifyAll` methods instead. This will automatically replay and verify all class- and object mocks created by PowerMock.

The full test simulating when a directory previously didn't exist looks like this:

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( PersistenceManager.class )
public class PersistenceManagerTest {

	@Test
	public void testCreateDirectoryStructure_ok() throws Exception {
		final String path = "directoryPath";
		File fileMock = createMock(File.class);

		PersistenceManager tested = new PersistenceManager();

		expectNew(File.class, path).andReturn(fileMock);

		expect(fileMock.exists()).andReturn(false);
		expect(fileMock.mkdirs()).andReturn(true);

		replay(fileMock, File.class);

		assertTrue(tested.createDirectoryStructure(path));

		verify(fileMock, File.class);
	}
}
```

There's also a slightly shorter way to write the test. In simple situations like the example above you can create the mock object of the File class and expect the call to new File at the same time using the `createMockAndExpectNew` method in PowerMock. The test could thus be rewritten as:

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( PersistenceManager.class)
public class PersistenceManagerTest {

	@Test
	public void testCreateDirectoryStructure_usingCreateMockAndExpectNew() throws Exception {
		final String path = "directoryPath";
		File fileMock = createMockAndExpectNew(File.class, path);

		PersistenceManager tested = new PersistenceManager();

		expect(fileMock.exists()).andReturn(false);
		expect(fileMock.mkdirs()).andReturn(true);

		replay(fileMock, File.class);

		assertTrue(tested.createDirectoryStructure(path));

		verify(fileMock, File.class);
	}
}
```

## More features ##
`expectNew` works similar to the normal `expect` method in EasyMock. This means that you can setup expectations to be invoked a certain number of times or throw an exception upon instantiation etc. For example let's say that you for some reason would like to throw an IllegalStateException by the time `new File(..)` is invoked. You can do this by:

```java
expectNew(File.class, "path").andThrow(new IllegalStateException("testing!");
```

As another example imagine that a method creates two new File objects and you like to return the same mock instance for both. You could do this by:

```java
expectNew(File.class, "path").andReturn(fileMock).times(2);
```

## References ##
  * [PersistenceManagerTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/newmocking/PersistenceManagerTest.java)
  * [PersistenceManager](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/newmocking/PersistenceManager.java)
  * [ExpectNewDemoTest](https://github.com/jayway/powermock/blob/master/modules/module-test/easymock/junit4-test/src/test/java/samples/junit4/expectnew/ExpectNewDemoTest.java)
  * [ExpectNewDemo](https://github.com/jayway/powermock/blob/master/tests/utils/src/main/java/samples/expectnew/ExpectNewDemo.java)