# Bootstrapping using a JUnit Rule #

```
   Feature avaliable since PowerMock 1.4
```

Since version 1.4 it's possible to bootstrap PowerMock using a [JUnit Rule](http://www.infoq.com/news/2009/07/junit-4.7-rules) instead of using the PowerMockRunner and the RunWith annotation. This allows you to use other JUnit runners while still benefiting from PowerMock's functionality. You do this by specifying:

```java
@PrepareForTest(X.class);
public class MyTest {
    @Rule
    PowerMockRule rule = new PowerMockRule();

    // Tests goes here
    ...
}
```

## Using PowerMockRule with Maven ##
You need to depend on these projects:

```xml
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-module-junit4-rule</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-classloading-xstream</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
```

You can also replace `powermock-classloading-xstream` with an Objenesis version:

```xml
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-classloading-objenesis</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
```

However this version is not as stable as the xstream version but it does not require any additional dependencies.

**Warning**: Use version PowerMock 1.4 (or later) since prior to this version the rule did not actually execute any test methods (but it looked like it did).

## Using PowerMockRule without Maven

You need to download <a href='http://repo1.maven.org/maven2/org/powermock/powermock-module-junit4-rule/2.0.2/powermock-module-junit4-rule-2.0.2.jar'>powermock-module-junit4-rule</a>, <a href='http://repo1.maven.org/maven2/org/powermock/powermock-classloading-base/2.0.2/powermock-classloading-base-2.0.2.jar'>powermock-classloading-base</a> and one of<br>
<a href='http://repo1.maven.org/maven2/org/powermock/powermock-classloading-xstream/2.0.2/powermock-classloading-xstream-2.0.2.jar'>powermock-classloading-xstream</a> or <a href='http://repo1.maven.org/maven2/org/powermock/powermock-classloading-objenesis/2.0.2/powermock-classloading-objenesis-2.0.2.jar'>powermock-classloading-objenesis</a> and put it in your classpath.<br>

## References ##

* [Mockito Examples](https://github.com/powermock/powermock/tree/master/tests/mockito/junit4-rule-xstream/src/test/java/samples/powermockito/junit4/rule/xstream)
* Spring Integration Test with PowerMock and Mockito [example](https://github.com/jayway/powermock/tree/master/examples/spring-mockito).
