# Release Notes for PowerMock 1.3.7 #

## Highlights ##
Support for Mockito 1.8.4 and new experimental bootstrap using JUnit 4.7+

## Bootstrapping using JUnit Rule ##
  * It's now possible to bootstrap PowerMock using a JUnit Rule instead of the RunWith annotation. This allows you to use other JUnit runners than the PowerMock one (PowerMockRunner) while still benefiting from PowerMock's functionality. It's still experimental but we encourage everyone to try it out. It'll probably be the default way of bootstrapping PowerMock in the future. See [examples](https://github.com/jayway/powermock/tree/master/modules/module-test/mockito/junit4-rule-xstream/src/test/java/org/powermock/modules/test/junit4/rule/xstream) in subversion and the [wiki page](PowerMockRule) for more info.

## Mockito API specific ##
  * Upgraded the Mockito extension to use Mockito 1.8.4
  * Fixed so that a "mock name" is set in the Mockito extension API. Fixes NullPointerException when e.g. toString is invoked on a mock. ([issue 239](https://code.google.com/p/powermock/issues/detail?id=239))

## Common ##
  * Added support for suppressing all constructors in a class using suppress(constructorsDeclaredIn(X.class))
  * Added support for suppressing all constructors and methods in a class suppress(everythingDeclaredIn(X.class))
## Minor changes ##
See [change log](https://raw.githubusercontent.com/jayway/powermock/master/changelog.txt) for more details