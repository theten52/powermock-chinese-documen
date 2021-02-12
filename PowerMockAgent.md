# Bootstrapping using a Java agent #

```
   Feature avaliable since PowerMock 1.4.9
```

Since version 1.4.9 it's possible to bootstrap PowerMock using a Java agent instead of using the PowerMockRunner and the RunWith annotation. This allows you to use e.g. other JUnit runners while still benefiting from PowerMock's functionality. The main difference between the agent based bootstrapper and the classloading based bootstrapper is that you don't run into classloading issues when using XML frameworks etc. It's recommended to use this way of bootstrapping when using PowerMock for integration testing larger parts of a system.

## JUnit ##
To bootstrap the Agent in JUnit you can use the `PowerMockRule` in the `powermock-module-junit4-rule-agent` project. For example:

```java
@PrepareForTest(X.class);
public class MyTest {
     @Rule
     PowerMockRule rule = new PowerMockRule();

     // Tests goes here
     ...
}
```

In some cases it may be necessary to manually start the agent before the test is being run. You can do that using:

```java
public class MyTest {
   static {
       PowerMockAgent.initializeIfNeeded();
   }

   ..
}
```

It's recommended that you put `powermock-module-junit4-rule-agent` _before_ junit in the classpath.

## TestNG ##
To bootstrap the Agent in TestNG you should extend from `PowerMockTestCase` in the `powermock-module-testng-common` project and you need to have the jar file from `powermock-module-testng-agent` in classpath. For example:

```java
@PrepareForTest(X.class)
public class SomeTest extends PowerMockTestCase {
    ...
}
```

## Maven ##

### JUnit ###
```xml
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-module-junit4-rule-agent</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
```
### TestNG ###
```xml
<dependency>
  <groupId>org.powermock</groupId>
  <artifactId>powermock-module-testng-agent</artifactId>
  <version>2.0.2</version>
  <scope>test</scope>
</dependency>
```

### Eager loading of the PowerMock Agent in Maven ###
In some cases (such as mocking final classes) it may be necessary to load the PowerMock agent eagerly in Maven in order for the tests to work in Surefire. If you experience this please add the following to your pom.xml:

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.4</version>
                <configuration>
                    <argLine>
                        -javaagent:${settings.localRepository}/org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2.jar
                    </argLine>
                    <useSystemClassloader>true</useSystemClassloader>
                </configuration>
            </plugin>
        </plugins>
</build>  
```
## Non-Maven users ##
You need to download [powermock-java-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2.jar) ([source](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2-javadoc.jar)) and either [powermock-module-junit4-rule-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2.jar) ([sources](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2-javadoc.jar)) if using JUnit or [powermock-module-testng-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2.jar) ([sources](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2-javadoc.jar)) if using TestNG.

## JUnit ##

#### Eager loading of the PowerMock Agent with JUnit in Eclipse ####

To load the PowerMock agent eagerly with Eclipse and JUnit you must first go into the _Run Configuration dialog_ and add the following JVM
parameter:

```bash
-javaagent: <jarpath>/powermock-module-javaagent-2.0.2.jar
```

Next must also make sure to put powermock-module-javaagent-2.0.2.jar
in the Run Configuration's classpath, _before the
default classpath_ (this is to ensure the agent loads before junit).


## Current known limitations ##
  * No way of suppressing static initializers
  * Cannot change value of static final fields

## References ##

* [JUnit Examples](https://github.com/powermock/powermock/tree/master/tests/mockito/junit4-agent/src/test/java/samples/powermockito/junit4/agent)
* Spring Integration Test with PowerMock and Mockito [example](https://github.com/jayway/powermock/tree/master/examples/spring-mockito-xml-agent).
