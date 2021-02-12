# Using PowerMock 1.2.5 with Mockito 1.7 #

```
Note: PowerMock 1.2.5 only works with Mockito 1.7. [PowerMock 1.3+ works with 1.8+](MockitoUsage)**
```

Basically, PowerMock provides a class called "PowerMockito" for creating mock object/class and initiating verification, everything else you can still use Mockito to setup and verify expectation (e.g. when(), times(), anyInt()).

All usages require `@RunWith(PowerMockRunner.class)` and `@PrepareForTest` annotated at class level.

## Mocking Static Method ##
How to mock and stub:
1. Add `@PrepareForTest` at class level.

  ```java
  @PrepareForTest(Static.class); // Static.class contains static methods
  ```
1. Call `PowerMockito.mockStatic()` to mock a static class (use `PoweMockito.mockStaticPartial(class, method)` to mock a specific method):

  ```java
  PowerMockito.mockStatic(Static.class);
  ```
1. Just use Mockito.when() to setup your expectation:

  ```java
  Mockito.when(Static.firstStaticMethod(param)).thenReturn(value);
  ```

### How to verify behavior ###
1. Call `PowerMockito.verifyStatic() `to start verifying behavior (Important: You need to call `verifyStatic()` per method verification):

  ```java
  PowerMockito.verifyStatic(Static.class);
  ```
1. Use EasyMock-like semantic to verify behavior:

  ```java
  Static.firstStaticMethod(param);
  ```

### How to use argument matchers ###
Mockito matchers are may still applied to a PowerMock mock.  For example, using custom argument matchers per mocked static method:

```java
PowerMockito.verifyStatic(Static.class);
Static.thirdStaticMethod(Mockito.anyInt());
```

### How to verify exact number of calls ###
You can still use Mockito.VerificationMode (e.g Mockito.times(x)) with `PowerMockito.verifyStatic()`:
```
PowerMockito.verifyStatic(Static.class, Mockito.times(1));
```

### How to stub void static method to throw exception ###
This is not yet supported in current release 1.2.5

### A full example for mocking, stubbing & verifying static method ###
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(Static.class)
public class YourTestCase {
    @Test
    public void testMethodThatCallsStaticMethod() {
        // mock all the static methods in a class called "Static"
        PowerMockito.mockStatic(Static.class);
        // use Mockito to set up your expectation
        Mockito.when(Static.firstStaticMethod(param)).thenReturn(value);
        Mockito.when(Static.secondStaticMethod()).thenReturn(123);

        // execute your test
        classCallStaticMethodObj.execute();

        // Different from Mockito, always use PowerMockito.verifyStatic() first
        PowerMockito.verifyStatic(Static.class, Mockito.times(2));
        // Use EasyMock-like verification semantic per static method invocation
        Static.firstStaticMethod(param);

        // Remember to call verifyStatic() again
        PowerMockito.verifyStatic(Static.class); // default is once
        Static.secondStaticMethod();

        // Again, remember to call verifyStatic()
        PowerMockito.verifyStatic(Static.class, Mockito.never());
        Static.thirdStaticMethod();
    }
}
```

## Partial Mocking ##
You can use PowerMock to partially mock a method which must be non-final.

### How to mock and stub ###
1. Use PowerMockito.mockPartial(): (You can also use Annotation `@Mock("methodToMock")` or `PowerMockito.mock()` for a method with specific parameters)

  ```java
  PartialMockClass mockObj = PowerMockito.mockPartial(PartialMockClass.class, "methodToMock");
  ```
1. Just use `Mockito.when()` to setup expectation:

  ```java
  Mockito.when(mockObj.methodToMock()).thenReturn(123);
  ```

## How to verify behavior ##

Just use Mockito.vertify() for verification:

```java
Mockito.verify(mockObj, times(2)).methodToMock();
```

## How to argument matchers ##
Mockito matchers are may still applied to a PowerMock mock:

```java
Mockito.verify(mockObj).methodToMockToo(Mockito.anyInt());  
```

## How to stub void static method to throw exception ##
Just use Mockito semantic of setting up void method stub:

```java
Mockito.doThrow(new RuntimeException("TEST")).when(mockObj).methodToMock();
```

## A full example of partial mocking ##
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(PartialMockClass.class)
public class YourTestCase {
    @Test
    public void testPartialMock() {
        // create a partially mocked object for method "methodToMock"
        PartialMockClass mockObj = PowerMockito.mockPartial(PartialMockClass.class, "methodToMock");
        // use Mockito to set up your expectation
        Mockito.when(mockObj.methodToMock()).thenReturn(value);

        // execute your test
        classCallPartialMockObj.execute();

        // Use Mockito.verify() to verify result
        Mockito.verify(mockObj, times(2)).methodToMock();
    }
}
```
