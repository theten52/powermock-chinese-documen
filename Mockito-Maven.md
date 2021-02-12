# Maven setup for the Mockito 1.x #

```
    Warning: Supporting Mockito 1 will be dropped in PowerMock 2.
```

## JUnit ##

### JUnit 4.4 or above ###

Add the following to your pom.xml if you're using JUnit 4.4 or above:

```xml
<properties>
    <powermock.version>2.0.2</powermock.version>
</properties>
<dependencies>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-module-junit4</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-api-mockito</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
</dependencies>
```

### JUnit 4.0-4.3  ###

Add the following to your pom.xml if you're using JUnit 4.0-4.3:

```xml
<properties>
    <powermock.version>2.0.2</powermock.version>
</properties>
<dependencies>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-module-junit4-legacy</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-api-mockito</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
</dependencies>
```

### JUnit 3 (deprecated) ###

Add the following to your pom.xml if you're using JUnit 3

```
   Node: Since PowerMock 2.0 Supporting jUnit 3 will be dropped
```

```xml
<properties>
    <powermock.version>1.7.1</powermock.version>
</properties>
<dependencies>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-module-junit3</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-api-mockito</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
</dependencies>
```

## TestNG ##

```xml
<properties>
    <powermock.version>2.0.2</powermock.version>
</properties>
<dependencies>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-module-testng</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>
   <dependency>
      <groupId>org.powermock</groupId>
      <artifactId>powermock-api-mockito</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>  
</dependencies>
```

## Legacy Versions ##

There is no way to get legacy version any more. Legacy versions of PowerMock (version 1.5.3 and below) was hosted on Google Code service which unfortunately [was closed in 2016](https://opensource.googleblog.com/2015/03/farewell-to-google-code.html).
