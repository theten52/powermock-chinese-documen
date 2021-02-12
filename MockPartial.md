# Partial Mocking #
## Quick summary ##
  1. Use the `@RunWith(PowerMockRunner.class)` annotation at the class-level of the test case.
  1. Use the `@PrepareForTest(ClassToPartiallyMock.class)` annotation at the class-level of the test case.
  1. se `PowerMock.createPartialMock(ClassToPartiallyMock.class, "nameOfTheFirstMethodToMock", "nameOfTheSecondMethodToMock")` to create a mock object that only mocks the methods with name `nameOfTheFirstMethodToMock` and `nameOfTheSecondMethodToMock` in this class (let's call it `mockObject`).
  1. Use `PowerMock.replay(mockObject)` to change the mock object to replay mode.
  1. Use `PowerMock.verify(mockObject)` to change the mock object to verify mode.