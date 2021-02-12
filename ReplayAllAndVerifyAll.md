# Replay and verify all  #

## Quick summary ##
  1. Use `PowerMock.replayAll` to change all mocks object and classes maintained by PowerMock to replay mode.
  1. Use `PowerMock.verifyAll` verify all mock objects and classes maintained by PowerMock.
  1. Use `PowerMock.resetAll` reset all mock objects and classes maintained by PowerMock.

## Example ##
Since version 0.9 of PowerMock you can use `PowerMock.replayAll(..)` and `PowerMock.verifyAll()` to replay and verify all classes and objects created (or in some cases used) by PowerMock.

So for example let's say you have some code like:

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
the test could look like:

```java
@Test
public void testCreateDirectoryStructure_ok() throws Exception {
	final String path = "directoryPath";

        // Using PowerMock's createMock method.
	File fileMock = createMock(File.class);

	PersistenceManager tested = new PersistenceManager();

	expectNew(File.class, path).andReturn(fileMock);

	expect(fileMock.exists()).andReturn(false);
	expect(fileMock.mkdirs()).andReturn(true);

        // Using replay all
	replayAll();

	assertTrue(tested.createDirectoryStructure(path));

        // Using verify all
	verifyAll();
}
```
Since all classes and mocks were created using the PowerMock API methods PowerMock can figure out which classes and objects that need to be replayed and verified automatically. But sometimes this is not possible, for example when you're mixing the EasyMock and PowerMock API's. Let's say that you create the `fileMock` using EasyMock class extensions `createMock` method instead, e.g.

```java
File fileMock = EasyMock.createMock(File.class);
```

Now the test will fail because PowerMock doesn't know of the `fileMock` because it was not created by PowerMock. What you could do then is to pass the mock object into the replayAll method of PowerMock, i.e.

```java
replayAll(fileMock);
```

The test will now pass again. Note that you don't have to modify the `verifyAll` method because mocks passsed to the `replayAll` method are automatically verified as well. The example above would look like:

```java
@Test
public void testCreateDirectoryStructure_ok_usingEasyMockToCreateTheFileMock() throws Exception {
	final String path = "directoryPath";
	File fileMock = EasyMock.createMock(File.class);

	PersistenceManager tested = new PersistenceManager();

	expectNew(File.class, path).andReturn(fileMock);

	expect(fileMock.exists()).andReturn(false);
	expect(fileMock.mkdirs()).andReturn(true);

	replayAll(fileMock);

	assertTrue(tested.createDirectoryStructure(path));

	verifyAll();
}
```
## References ##
  * [PersistenceManagerWithReplayAllAndVerifyAllTest](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/test/java/powermock/examples/newmocking/PersistenceManagerWithReplayAllAndVerifyAllTest.java)
  * [PersistenceManager](https://github.com/powermock/powermock-examples-maven/blob/master//DocumentationExamples/src/main/java/powermock/examples/newmocking/PersistenceManager.java)