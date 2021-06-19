# 使用 Java 代理引导启动

```text
   从 PowerMock 1.4.9 时功能可用
```

从 1.4.9 版本开始，可以使用 Java 代理而不是使用 PowerMockRunner 和 RunWith 注解来引导 PowerMock。这允许您使用例如其他 JUnit 运行程序，同时仍然受益于 PowerMock 的功能。基于代理的引导程序和基于类加载的引导程序之间的主要区别在于，在使用 XML 框架等时不会遇到类加载问题。建议在使用 PowerMock 对系统的较大部分进行集成测试时使用这种引导方式。

## JUnit ##
To bootstrap the Agent in JUnit you can use the `PowerMockRule` in the `powermock-module-junit4-rule-agent` project. For example:

要在 JUnit 中引导代理，您可以在`powermock-module-junit4-rule-agent`项目中使用`PowerMockRule` 。例如：

```java
@PrepareForTest(X.class);
public class MyTest {
     @Rule
     PowerMockRule rule = new PowerMockRule();

     // Tests goes here
     ...
}
```

在某些情况下，可能需要在运行测试之前手动启动代理。你可以使用：

```java
public class MyTest {
   static {
       PowerMockAgent.initializeIfNeeded();
   }

   ..
}
```

It's recommended that you put `powermock-module-junit4-rule-agent` _before_ junit in the classpath.

建议您将`powermock-module-junit4-rule-agent` *在JUnit之前*放在类路径中。

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

### 在 Maven 中预先加载 PowerMock 代理 ###
在某些情况下（例如mock最终类），可能需要在 Maven 中急切地加载 PowerMock 代理，以便测试在 Surefire 中工作。如果您遇到这种情况，请将以下内容添加到您的 pom.xml 中：

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
## 非 Maven 用户 ##
You need to download [powermock-java-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2.jar) ([source](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2-javadoc.jar)) and either [powermock-module-junit4-rule-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2.jar) ([sources](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2-javadoc.jar)) if using JUnit or [powermock-module-testng-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2.jar) ([sources](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2-javadoc.jar)) if using TestNG.

你需要下载 [powermock-java-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2.jar) ([source](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-javaagent/2.0.2/powermock-module-javaagent-2.0.2-javadoc.jar)) ；如果使用JUnit，还需要[powermock-module-junit4-rule-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2.jar) ([sources](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-junit4-rule-agent/2.0.2/powermock-module-junit4-rule-agent-2.0.2-javadoc.jar)) 如果使用TestNG，还需要[powermock-module-testng-agent](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2.jar) ([sources](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2-sources.jar), [javadoc](http://search.maven.org/remotecontent?filepath=org/powermock/powermock-module-testng-agent/2.0.2/powermock-module-testng-agent-2.0.2-javadoc.jar))。

## JUnit ##

#### 在 Eclipse 中使用 JUnit 预先加载 PowerMock 代理

要使用 Eclipse 和 JUnit 急切地加载 PowerMock 代理，您必须首先进入“*运行配置”对话框*并添加以下 JVM 参数：

```bash
-javaagent: <jarpath>/powermock-module-javaagent-2.0.2.jar
```

接下来，您还必须确保将 powermock-module-javaagent-2.0.2.jar 放在运行配置的类路径中，*在默认类*路径*之前*（这是为了确保代理在 junit 之前加载）。


## 当前的已知的限制 ##
  * 无法抑制静态初始化代码块
  * 无法更改静态final字段的值

## 参考 ##

* [JUnit 示例](https://github.com/powermock/powermock/tree/master/tests/mockito/junit4-agent/src/test/java/samples/powermockito/junit4/agent)
* 使用 PowerMock 和 Mockito[示例](https://github.com/jayway/powermock/tree/master/examples/spring-mockito-xml-agent)进行Spring 集成测试。

