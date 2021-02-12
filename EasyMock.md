# 将PowerMock与EasyMock一起使用

http://www.easymock.org/

EasyMock通过使用Java的动态代理机制动态生成接口，从而为接口提供MockObject。

## 扩展 ##

PowerMock通过静态Mock扩展了EasyMock并为构造函数设置了期望值。

## 支持版本 ##
<table>
<blockquote><tr><th align='left'> <b><u>EasyMock</u></b></th><th align='left'><b><u>PowerMock</u></b></th></tr>
<tr><td>3.4</td><td>1.6.3+</td></tr>
<tr><td>3.3</td><td>Supported in 1.6.0 if you depend on cglib-nodep 2.2.2</td></tr>
<tr><td>3.0 - 3.2</td><td>1.3.9 - 1.6.0</td></tr>
<tr><td>2.5.x</td><td>1.3.7 & 1.3.8</td></tr>
<tr><td>2.4.x or older</td><td>1.3.6</td></tr>
</table>

## Maven 配置 ##

### JUnit ###
  * [EasyMock JUnit Maven 配置](EasyMock-Maven)

### TestNG ###
  * [EasyMock TestNG Maven 配置](EasyMock-Maven)

## 用例 ## 
* [Mock Static](MockStatic)
* [Mock Final](MockFinal)
* [Mock Private](MockPrivate)
* [Mock New](MockConstructor)
* [Mock Partial](MockPartial)
* [Replay and verify all](ReplayAllAndVerifyAll)