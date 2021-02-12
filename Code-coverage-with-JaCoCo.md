# Code Coverage with JaCoCo

## Overview ## 
[JaCoCo](https://github.com/jacoco/jacoco) is a free Java code coverage library distributed under the Eclipse Public License. 
JaCoCo instrumentations to collect code coverage information. JaCoCo supports two ways class instrumentation: on the fly with using Java Agent and offline when classes are prepared during build phase. More information you may find in [JaCoCo documentation](http://www.eclemma.org/jacoco/trunk/doc/index.html). 

## On-the-fly instrumentation ##

The simplest way to use JaCoCo it is — on-the-fly instrumentation with using JaCoCo Java Agent. In this case a class in modified when it is [being loaded](https://docs.oracle.com/javase/7/docs/api/java/lang/instrument/Instrumentation.html). You can just run you application with JaCoCo agent and a code coverage is calculated. This way is used by Eclemma and Intellij Idea. 
But there is a big issue. PowerMock instruments classes also. Javassist is used to modify classes. The main issue is that Javassist reads classes from disk and all JaCoCo changes are [disappeared](https://github.com/jacoco/jacoco/issues/51). As result zero code coverage for classes witch are loaded by PowerMock class loader. 

We are going to replace Javassist with ByteBuddy (#727) and it should help to resolve this old issue. But right now there is **NO WAY TO USE** PowerMock with JaCoCo On-the-fly instrumentation. And no workaround to get code coverage in IDE. 

## Offline Instrumentation ##

Second way to get code coverage with JaCoCo - use [offline Instrumentation](http://www.eclemma.org/jacoco/trunk/doc/offline.html). In this case an instrumented class is stored on disk and can be read with Javassist. Disadvantage of this approach it's that you have to change build process and code coverage can be collected only running test from build tool ([example for maven](http://www.eclemma.org/jacoco/trunk/doc/examples/build/pom-offline.xml)). 

JaCoCo Offline Instrumentation works only with PowerMock version 1.6.6 and above. 

You may find example of using PowerMock with JaCoCo Offline Instrumentation and Maven in our repository: [jacoco-offline example](https://github.com/powermock/powermock-examples-maven/tree/master/jacoco-offline). 

