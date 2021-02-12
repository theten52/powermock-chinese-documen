# Motivation #

PowerMock is a Java mock framework that can be used to solve testing problems that is normally considered difficult or even impossible to test. Using PowerMock, it becomes possible to mock static methods, remove static initializers, allow mocking without dependency injection, and more. PowerMock does these tricks by modifying the byte-code at run-time when the tests are being executed. PowerMock also contains some utilities that give you easier access to the internal state of an object.

Here are some scenarios when PowerMock will make sense and we also describe the alternative:

## Using a 3rd party or legacy framework ##

Communication with the framework is done through static methods (for example Java ME). PowerMock allows you to setup expectations for these static methods and simulate the behavior you need to test.

Without PowerMock: Wrap all static method calls in a separate class and use dependency injection to mock this object. This will create an extra layer of unnecessary classes in your application. However, this can of course be useful if you want to encapsulate the framework to be able to replace it.

The framework require you to subclass classes that contains static initializers (Eclipse) or constructors (Wicket). PowerMock allows you to remove static initializers and suppress constructors. It also allows you to mock classes inside a signed jar file (old Acegi, Eclipse), even if the classes are package-private.

Without PowerMock: Move all your code into a delegate and let the framework class simply delegate all method calls.

## Design ##

You really want to enforce the fact that some methods are private. PowerMock allows mocking of both private and final methods.

Without PowerMock: Private methods are not an option. Create a new class and move all private methods to this as public. Use dependency injection and mocking. This can force you to use an unwanted design.

## Performance? ##

Dependency injection is too expensive for you so you want static method calls or a static factory instead. For example if you have extremely large number objects or if you are working with Java ME. PowerMock allows you to use static methods for performance and mock.

Without PowerMock: Create Singleton for each service and somehow make sure that during a test a special instance is created. This sometimes involves breaking the singleton pattern and adding the possibility to change the instance.