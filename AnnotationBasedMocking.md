# Annotation-based Mocking #

## Quick summary ##

  1. Use `@PowerMockTestListener(AnnotationEnabler.class)` at the class-level of your test to enable the annotation support.
  1. Use `@Mock` on a field to allow PowerMock to create and inject a mock of the same type as the field.
  1. Use `@MockNice` on a field to allow PowerMock to create and inject a nice mock of the same type as the fieldprivate constructor.
  1. Use `@MockStrict` on a field to allow PowerMock to create and inject a strict mock of the same type as the field
  1. Use `@Mock("methodName")` on a field to allow PowerMock to create and inject a partial mock of the same type as the field mocking only the method called `methodName`. This also work for `@MockNice` and `@MockStrict`.

## References ##

  * [ServiceHolderTest](https://github.com/powermock/powermock-examples-maven/tree/master/DocumentationExamples/src/test/java/powermock/examples/bypassencapsulation/ServiceHolderTest.java)