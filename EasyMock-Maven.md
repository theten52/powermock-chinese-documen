# EasyMock的Maven设置

## JUnit ##

### JUnit 4.4 或更高版本 ###

如果使用的是JUnit 4.4或更高版本，则将以下内容添加到pom.xml中：

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
      <artifactId>powermock-api-easymock</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>  
</dependencies>
```
### JUnit 4.0-4.3  ###

如果使用的是JUnit 4.0-4.3，请将以下内容添加到pom.xml中：

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
      <artifactId>powermock-api-easymock</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>  
</dependencies>
```

### JUnit 3 (deprecated) ###

如果使用的是JUnit 3，则将以下内容添加到pom.xml中：

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
      <artifactId>powermock-api-easymock</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>  
</dependencies>
```

### TestNG ###

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
      <artifactId>powermock-api-easymock</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>  
</dependencies>
```

## TestNG ##

将以下内容添加到您的pom.xml中

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
      <artifactId>powermock-api-easymock</artifactId>
      <version>${powermock.version}</version>
      <scope>test</scope>
   </dependency>  
</dependencies>
```

## 旧版

无法再获取旧版本。旧版PowerMock（版本1.5.3及更低版本）托管在Google Code服务上，该服务不幸[于2016年关闭](https://opensource.googleblog.com/2015/03/farewell-to-google-code.html)。
