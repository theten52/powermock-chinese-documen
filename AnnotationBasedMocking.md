# 基于注解的模拟

## 快速总结 ##

  1. 在测试的类级别使用 `@PowerMockTestListener(AnnotationEnabler.class)` 以开启注解支持。
  1. 在字段上使用 `@Mock` 以允许PowerMock创建和注入与该字段相同类型的模拟（类）。
  1. 在字段上使用 `@MockNice` 以允许PowerMock创建和注入与 fieldprivate（TODO）构造函数 相同类型的漂亮模拟类（nice mock）。
  1. 在字段上使用 `@MockStrict` 以允许PowerMock创建和注入与该字段相同类型的严格模拟类。
  1. 在字段上使用 `@Mock("methodName")` 以允许PowerMock创建和注入与该字段相同类型的部分模拟，仅模拟`methodName`方法的调用。这也适用于`@MockNice`和`@MockStrict`。

## 参考 ##

  * [ServiceHolderTest](https://github.com/powermock/powermock-examples-maven/tree/master/DocumentationExamples/src/test/java/powermock/examples/bypassencapsulation/ServiceHolderTest.java)