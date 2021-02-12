# Release Notes for PowerMock 1.4.5 #

## Highlights ##
  * Removed PowerMock specific classes to support mocking of signed classes in the EasyMock extension API since nowadays EasyMock supports this out of the box. This makes PowerMock less dependent on a specific EasyMock version and fixes and patches in EasyMock automatically applies to PowerMock in the future as well.
  * Upgraded the TestNG module to TestNG 5.13.1

## Common ##
  * Added the "toThrow" method to the stubber API. You can now do e.g. "stub(method("methodName")).toThrow(new Exception());" ([issue 182](https://code.google.com/p/powermock/issues/detail?id=182))
  * Fixed an issue that was introduced in version 1.4 when support for partial mocking of instance methods in final system classes not having a default constructor was added. The bug caused an javassist.CannotCompileException exception when mocking certain classes such as java.lang.Console ([issue 272](https://code.google.com/p/powermock/issues/detail?id=272))

## Deprecations ##
  * Deprecated "andReturn" in the stubber API and added "toReturn" instead. The API now reads e.g. "stub(method("methodName")).toReturn(new Object());"

## Non backward compatible changes ##
  * Whitebox#setInternalStateFromContext now support field matching strategies. By default  all context fields that matches a field in the target class or instance is copied to the instance. If the strategy is changed from MATCHING to STRICT then an exception will be thrown if the context contains a field that cannot be copied to the target. Note that this change may cause backward incompatibility if you're currently using setInternalStateFromContext in your code

## Minor changes ##
See [change log](http://powermock.googlecode.com/svn/trunk/changelog.txt) for more details