# Release Notes for PowerMock 1.3.6 #

## Highlights ##
Support for Mockito 1.8.3 and EasyMock 2.5.2.

## (Possibly) Non-backward compatible changes ##
  * Fixed a bug in the Wildcard matcher which resulted in that classes located in packages containing "junit.", "java.", "sun." in their name was never prepared for test ([issue 225](https://code.google.com/p/powermock/issues/detail?id=225)).
  * The PowerMock API extension to EasyMock is not backward compatible with EasyMock class extension versions prior to 2.5.2 because of internal changes in this project.

## Mockito API specific ##
  * Upgraded the Mockito extension to use Mockito 1.8.3
  * Added support for Mockito annotations @Spy, @Captor and @InjectMocks
  * Mockito extension now supports real partial mocking for private (and private final) methods (both static and instance methods) using PowerMockito.doReturn(..), PowerMockito.doThrow(..) etc
  * Added support for MockSettings and Answers when creating class or instance mocks in Mockito extension, e.g:
> ` FinalClass mock = mock(FinalClass.class, Mockito.RETURNS_MOCKS); `

## EasyMock API specific ##
  * Upgraded the EasyMock extension to use EasyMock class extensions 2.5.2

## Minor changes ##
See [change log](https://raw.githubusercontent.com/jayway/powermock/master/changelog.txt) for more details