# 使用 @PowerMockRunnerDelegate  #

```text
   从PowerMock 1.6.0可用的功能。
```

从PowerMock 1.6.0版本以来，支持将测试执行委托给另一个JUnit runner而不使用JUnit Rule. 这将实际的测试执行留给您选择的另一个runner。 例如，测试可以委托给“SpringJunit4ClassRunner”，“Parameterized”或“Enclosed”runner。 用法示例：

```java
@RunWith(PowerMockRunner.class)
@PowerMockRunnerDelegate(Parameterized.class)
@PrepareForTest({FinalDemo.class, PrivateFinal.class})
public class FinalDemoTest {

    @Parameterized.Parameter(0)
    public String expected;

    @Parameterized.Parameters(name = "expected={0}")
    public static Collection<?> expections() {
        return java.util.Arrays.asList(new Object[][]{
            {"Hello altered World"}, {"something"}, {"test"}
        });
    }

    @Test
    public void assertMockFinalWithExpectationsWorks() throws Exception {
        final String argument = "hello";

        FinalDemo tested = mock(FinalDemo.class);

        when(tested.say(argument)).thenReturn(expected);

        final String actual = "" + tested.say(argument);

        verify(tested).say(argument);

        assertEquals("Expected and actual did not match", expected, actual);
    }
}
```


## 参考 ##

 * [Jayway Blog: Using another JUnit runner with PowerMock](http://www.jayway.com/2014/11/29/using-another-junit-runner-with-powermock)