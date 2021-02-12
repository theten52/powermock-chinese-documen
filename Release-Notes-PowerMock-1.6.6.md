# Release Notes for PowerMock 1.6.6 #

## Highlights ##
  * PowerMock can work with JaCoCo offline instrumenting to get code coverage.
  * Jacoco offline instrumentation incompatibility with powermock byte-code manipulation when using `@SuppressStaticInitializationFor` (issue [645](https://github.com/jayway/powermock/issues/645))
  * Fixed "TooManyActualInvocations when class is prepared for test" (issue [656](https://github.com/jayway/powermock/issues/656))
  * Fixed `ClassNotFoundException` when loading cglib enhanced classes created by Spring (issue [695](https://github.com/jayway/powermock/issues/695))
  * `@Mock` fields are now injectable again (1.6.5 regression) (issue [668](https://github.com/jayway/powermock/issues/668))
  * Using Powermock with Roo/AspectJ no longer throws `NullPointerException` (issue [676](https://github.com/jayway/powermock/issues/676))
  * Added support for specifying constructor parameters for Mockito for whenNew (Only mock specific calls to new)
  * Added support for setting private static final fields (thanks to Andrei Petcu @andreicristianpetcu for pull request)

## Other changes ##
See [change log](https://raw.githubusercontent.com/jayway/powermock/master/changelog.txt) for more details