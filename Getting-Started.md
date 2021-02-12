Â# Getting Started #

## API's ##
PowerMock consists of two extension API's.

One for [EasyMock](EasyMock) and one for [Mockito](Mockito). To use PowerMock you need to depend on one of these API's as well as a test framework.

 Currently PowerMock supports JUnit and [TestNG](TestNG). There are three different JUnit test executors available, one for JUnit 4.4-4.12, one for JUnit 4.0-4.3. The test executor for JUnit 3 is not avaliable since PowerMock 2.0.

 There's one test executor for TestNG which requires version 5.11+ depending on which version of PowerMock you use.

```
   Node: Since PowerMock 2.0 Supporting jUnit 3 has been removed.
```

## Writing tests ##

Write a test like this:

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( { YourClassWithEgStaticMethod.class })
public class YourTestCase {
...
}
```

## Maven setup ##

### JUnit ###
  * [EasyMock JUnit Maven setup](EasyMock-Maven)
  * [Mockito JUnit Maven setup](Mockito-Maven)
  * [Mockito2 JUnit Maven setup](Mockito-2-Maven)

### TestNG ###
  * [EasyMock TestNG Maven setup](EasyMock-Maven)
  * [Mockito TestNG Maven setup](Mockito-Maven#testng)
  * [Mockito2 TestNG Maven setup](Mockito-2-Maven#testng)

## Non maven users ##

### EasyMock

JUnit:
Download the [EasyMock](https://dl.bintray.com/powermock/generic/distributions/powermock-easymock-junit-2.0.2.zip) zip-file with PowerMock and all its dependencies and add those to your project.

TestNG:
Download the [EasyMock](https://dl.bintray.com/powermock/generic/distributions/powermock-easymock-testng-2.0.2.zip) zip-file with PowerMock and all its dependencies and add those to your project.

### Mockito

JUnit:
Download the [Mockito](https://dl.bintray.com/powermock/generic/distributions/powermock-mockito2-junit-2.0.2.zip) zip-file with PowerMock and all its dependencies and add those to your project.

TestNG:
Download the [Mockito](https://dl.bintray.com/powermock/generic/distributions/powermock-mockito2-testng-2.0.2.zip) zip-file with PowerMock and all its dependencies and add those to your project.
## Need to combine PowerMock with another JUnit runner?
  * First try the [JUnit Delegation Runner](JUnit_Delegating_Runner) and if that doesn't work then try the [PowerMockRule](PowerMockRule) or [PowerMock Java Agent](PowerMockAgent).

## Need to bootstrap using a JUnit rule? ##
  * The [PowerMockRule](PowerMockRule) makes this happen

## Java agent based bootstrapping ##
Use the [PowerMock Java Agent](PowerMockAgent) if you're having classloading problems when using PowerMock.

## Need Tutorial ? ##

1. Clone the project from git using:

 Using SSH
  ```bash
  git clone git@github.com:powermock/powermock-examples-maven.git
  ```

  Using HTTPS
  ```bash
  git clone https://github.com/powermock/powermock-examples-maven.git
  ```


2. Check out version 1.7.1:
  ```bash
  git checkout powermock-1.7.1
  ```

3. Step into the `examples/tutorial/` folder and follow the instructions in the [readme.txt](https://raw.githubusercontent.com/powermock/powermock-examples-maven/powermock-1.7.1/examples/tutorial/readme.txt) file.
