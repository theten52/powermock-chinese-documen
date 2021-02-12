# Release Notes for PowerMock 1.3 #

PowerMock 1.3 is one of biggest releases to date and there's been lots of changes both internally and externally.

## (Possibly) Non-backward compatible changes ##
**Mockito API specific**
* Removed `mockStatic(Class<?> type, Method method, Method... methods)`, `mockStaticPartial`, `mockPartial`, `mock(Class<T> type, Method... methods)` & `mock(Class<T> type, Method... methods)` from the Mockito extension API. You should use `PowerMockito.spy` instead
* Verification behavior of static methods in the Mockito extension API has changed. Before you did `verifyStatic(MyClass.class); MyClass.methodToVerify(argument);`, now you do `verifyStatic(); MyClass.methodToVerify(argument);`. You can also pass a verification mode when verifying static methods: `verifyStatic(times(2)); MyClass.methodToVerify(argument);`. This change is NOT backward compatible with version 1.2.5.


**Common**
* PowerMock now changes the context class-loader to the MockClassloader which means that you don't need to use @PowerMockIgnore as often as before. This solves many issues with frameworks that creates new instances using reflection like log4j, hibernate and many XML frameworks. If you've been using PowerMockIgnore in your test you may need to remove it (or update the ignored package list) otherwise the test might fail.
* Test classes are now always prepared for test automatically. This means that you can use suppressing constructors and mock final system classes more easily since you don't have to prepare the actual test class for test. In some cases this change is not backward compatible with version 1.2.5. These cases are when you've tried to suppress a constructor but have forgot to prepare the test class for test as well (which should be quite rare).
* Fixed a major issue with the prepare for test algorithm. Before classes were accidentally prepared for test automatically if the fully qualified name of the class started with the same fully qualified name as a class that were prepared for test. This has now been resolved. This may lead to backward incompatibility issues in cases where tests didn't prepare all necessary artifacts for test.
* @PowerMockIgnore now accept wildcards, this means that if you want to ignore all classes in the "com.mypackage" package you now have to specify `@PowerMockIgnore("org.mypackage.*")` (before you did `@PowerMockIgnore("org.mypackage."`)).
* @PrepareForTest now accepts wildcards in the fullyQualifiedName attribute, e.g. you can now do `@PrepareForTest(fullyQualifiedName={"*name*"})` to prepare all classes in containing "name" in its fully-qualified name. If you haven't been preparing packages for test earlier then this should cause no problem unless you've forgot to prepare all necessary artifacts for test.


## Other notable changes ##
**EasyMock API specific**
* PowerMock now supports mocking instance methods of final system classes (such as java.lang.String). To do this you need to prepare the class that invokes the method of the system class. Note that partial mocking of instance methods in final system classes doesn't yet work if a constructor needs to be invoked on the mock.


**Mockito API specific**
* Upgraded the Mockito extension to use Mockito 1.8
* Mocking of static methods in final system classes now works
* When using the PowerMock Mock annotation with Mockito the method names (previously used for partial mocking) are ignored. They are no longer needed, just use PowerMockito.spy instead.
* Support for private method expectations using `PowerMockito.when(..)`
* You can now mock construction of new objects using the Mockito extension. To do so use `PowerMockito#whenNew(..)`, e.g. `whenNew(MyClass.class).withArguments(myObject1, myObject2).thenReturn(myMock)`. Verification can be done with `PowerMockito#verifyNew(..), e.g. verifyNew(MyClass.class).withArguments(myObject1, myObject2)`.
* Supports suppressing constructors, fields and methods
* Supports verification of private methods (for both static and instance methods). Use `verifyPrivate(..).invoke(..)`, e.g. `verifyPrivate(myObject, times(2)).invoke("myMethod", argument1, argument2)`.
* Supports "verifyNoMoreInteractions" and "verifyZeroInteractions" for both instance mocks, class mocks and new instance mocks
* Supports, doAnswer, doThrow, doCallRealMethod, doNothing and doReturn for void methods defined in final classes, final void methods and static methods (use the PowerMockito version of each method). Note that if a method is a private void method you should still use `PowerMockito#when`.


**Common**
* Classes prepared for test in parent test classes are now automatically prepared for test as well. I.e. if Test Case A extends Test Case B and A prepares X.class and B prepares Y.class then both X and Y will be prepared for test. Before only X was prepared for test.
* Upgraded the JUnit runner to JUnit 4.7 and also implemented support for JUnit 4.7 rules.
* Annotation support has been integrated in the test runners so there's no need to specify the AnnotationEnabler listener using `@PowerMockListener(AnnotationEnabler.class)`. It works both for the EasyMock and Mockito extension API's.
* Added a PowerMock classloading project which can be used to execute any Runnable or Callable in a different classloader. The result of a callable operation will be cloned to the the callers classloader.
* Implemented a more fluent API for suppression, replacing and stubbing (`org.powermock.api.support.membermodification.MemberModifier`). It uses matcher methods defined in `org.powermock.api.support.membermodification.MemberMatcher`. Some examples:
  
  ```java
  replace(method(X)).with(method(Y)); // Works only for static methods
  replace(method(X)).with(invocationHandler); 
  stub(method(X)).andReturn(Y);
  suppress(methodsDeclaredIn(X.class, Y.class))
  suppress(method(X.class, "methodName"))
  suppress(methods(method(X.class, "methodName"), method(Y.class, "methodName2")))
  suppress(methods(X.class, "methodName1", "methodName2"))
  suppress(field(..))
  suppress(fields(..))
  suppress(constructor(..));
  suppress(constructors(..));
  ```
* PowerMock and PowerMockito now supports stubbing methods (including private methods). Use PowerMock#stub or PowerMockito#stub to accomplish this.
* PowerMock and PowerMockito now supports proxying methods, including private methods using replace(..), e.g. `replace(method(MyClass.class, "methodToProxy")).with(myInvocationHandler)`. Every call to the "methodToProxy" method will be routed to the supplied invocation handler instead.
* PowerMock and PowerMockito now supports duck typing of static methods, including static private methods using replace(..), e.g. `replace(method(MyClass.class, "methodToDuckType")).with(method(MyOtherClass.class, "methodToInvoke"))`. Every call to the "methodToDuckType" method will be routed to the "methodToInvoke" method instead.

## Deprecation ##
* Deprecated `Whitebox#getInternalState(Object object, String fieldName, Class<?> where, Class<T> type)`. Use `"Whitebox.<Type> getInternalState(Object object, String fieldName, Class<?> where)"` instead.
* Deprecated `org.powermock.core.classloader.annotations.Mock`, you should now use the Mock annotation in respective extension API instead. For EasyMock this is `org.powermock.api.easymock.annotation.Mock` and for Mockito it's `org.mockito.Mock`.
* Deprecated `org.powermock.api.easymock.powermocklistener.AnnotationEnabler` and `org.powermock.api.mockito.powermocklistener.AnnotationEnabler`. You should just remove them because they're now integrated with the test runner.
* MockPolicy: Deprecated setSubstituteReturnValues, getSubstituteReturnValues and addSubtituteReturnValue in `MockPolicyInterceptionSettings`, you should now use setMethodsToStub, getStubbedMethods and stubMethod instead
* Deprecated all suppressMethod, suppressField and suppressConstructor methods in the EasyMock extension API. You should now use PowerMock#suppress instead. See the new suppression API above.

## Removed deprecations ##
* Removed the deprecated classes "org.powermock.PowerMock" and "org.powermock.Whitebox"

## Minor changes ##
See [change log](https://raw.githubusercontent.com/jayway/powermock/master/changelog.txt) for more details