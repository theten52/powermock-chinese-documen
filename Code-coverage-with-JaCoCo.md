# JaCoCo的代码覆盖率

## 概述 ## 
[JaCoCo](https://github.com/jacoco/jacoco)是根据Eclipse Public License发布的免费Java代码覆盖库。

JaCoCo工具收集代码覆盖率信息。JaCoCo支持两种类型的类检测方式：使用Java Agent即时运行，以及在构建阶段准备类时脱机使用。您可以在[JaCoCo文档中](http://www.eclemma.org/jacoco/trunk/doc/index.html)找到更多信息。

## 即时检测 ##

使用JaCoCo的最简单方法是-使用JaCoCo Java Agent进行即时检测。在这种情况下，在[加载](https://docs.oracle.com/javase/7/docs/api/java/lang/instrument/Instrumentation.html)时修改一个类。您可以只使用JaCoCo agent运行应用程序，然后计算代码覆盖率。Eclemma和Intellij Idea使用这种方式。但是有一个大问题。PowerMock检测类也是这种方式。用Javassist修改类。主要问题是Javassist从磁盘读取类，并且所有JaCoCo更改都[丢失](https://github.com/jacoco/jacoco/issues/51)了。结果，PowerMock类加载器加载了零代码覆盖率的类。

我们将用ByteBuddy（＃727）替换Javassist，它应该有助于解决这个老问题。但现在有**没有办法在PowerMock与JaCoCo在即时检测中使用**。并没有解决方法来获得IDE中的代码覆盖率。

##  

## 离线检测 ##

使用JaCoCo获得代码覆盖率的第二种方法-使用[脱机检测](http://www.eclemma.org/jacoco/trunk/doc/offline.html)。在这种情况下，已检测的类存储在磁盘上，并且可以用Javassist读取。这种方法的缺点是您必须更改构建过程，并且只能从构建工具（[例如maven](http://www.eclemma.org/jacoco/trunk/doc/examples/build/pom-offline.xml)）运行测试才能收集代码覆盖率。

JaCoCo 脱机检测仅适用于PowerMock 1.6.6及更高版本。

您可以在我们的存储库中找到将PowerMock与JaCoCo Offline Instrumentation和Maven结合使用的示例：[jacoco-offline example](https://github.com/powermock/powermock-examples-maven/tree/master/jacoco-offline)。

