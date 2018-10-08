---
title: 'Junit(1):开始我们的单元测试之旅!'
date: 2017-10-27 17:52:55
tags:
 - 软件测试
 - TDD
 - Junit
---

  作为一名程序员，虽然你可能不是测试岗，但是懂得测试是必须的，这一个系列的文章用来记录一些Junit的测试方法，首先从实现自动化测试开始，然后衍生到stub和mock测试，再来进行我们的web应用的测试和数据库的测试。当然测试并不只包含这些内容，这里只是其中的一小块，有兴趣的同学可以继续开展。

  写这个系列的文章的目的在于，由于我未来工作的公司，是结对编程，有时候我需要去在伙伴写业务代码的时候写测试代码，不过我认为，既然软件测试在软件工程课程中是基础课，那么每一个程序员都应该懂得一定的测试手段，再者写下这套文章，有利于我从新复习java的单元测试手段，当然这些手段在未来，即使是换一种语言，也会是通用的知识。这篇文章中所用的junit版本都是junit4,用注解来标识一些测试声明，而且测试类不再需要继承TestCase,也不需要遵循它的测试方法命名规则，我们可以把测试方法命名的的更加直观。

<!--more-->

## 断言

  可能很多情况下，我们会把@Test标识的方法作为一个主方法来使用，如果在没有专门测试的项目中，你可能会使用sysout插桩，然后用一个个去查看输入输出是不是和你内心预期的相同来检验我们的代码是不是存在某些问题。

  如果有成百上千个方法需要测试，而每一个方法都需要覆盖到它所有的分支，我们就不能再用上面提到过的原始方式来对测试用例的结果进行判断了。在以@Test标识的测试用例，我们引入断言的方式来对我们的预期结果和实际结果的匹配进行判断，如果断言验证预期和实际不同则该用例执行时就会标红，这样其实我们的测试代码就和实际的业务代码分离了。

```java
package services;

import static org.junit.Assert.*;

import org.junit.Before;
import org.junit.Test;
import xin.saul.app.services.Calculate;

public class CalculateTest {
    Calculate calculate;

    @Before
    public void init() {
        calculate = new Calculate();
    }

    @Test
    public void addTest() {
        assertEquals(3, calculate.add(1, 2));
    }

    @Test
    public void minusTest() {
        assertEquals(-1, calculate.minus(1, 2));
    }

    @Test
    public void multiplyTest() {
        assertEquals(2, calculate.multiply(1, 2));
    }

    @Test
    public void divideTest() {
        assertEquals(2, calculate.divide(4, 2));
    }
    
}
```

![QQ图片20171028140338](\img\QQ图片20171028140338.png)

  从上面可以看到，我的测试用例都通过了断言，当然些测试用例是不完备的，它没有覆盖到我们有可能输入的所有情况，或者我给定的输入没有覆盖到被测试方法的所有分支。

  当然如果我们的项目是一个maven项目我们可以用mvn test来进行测试。比如对于该项目mvn test输出如下报告，当然如果你对maven熟悉你可以去详细看其生成的测试报告，它在**target->surefire-reports**文件夹。

```shell
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running services.CalculateTest
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.042 sec

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
```

## 异常测试

  现在我们对这个项目做一些小小的改变，我们加一个用例来测试:

```java
@Test
public void arithmeticExceptionTest() {
	calculate.divide(4, 0);
}
```

  你可以知道这个方法用例明显是会抛出一个ArithmeticException的

![QQ图片20171028141519](\img\QQ图片20171028141519.png)

  我们可以看到，正如我们所料，新加的测试用例直接抛出了异常，直觉告诉我们这肯定是不符合标准的，测试是要有预期的，对异常的预期也是一种预期，实际上我们可以测试在某种情况下会抛出某种异常。**这需要我们添加@Test注解的一个属性--expected,它接受一个Throwable的子类型来作为预期的异常，默认值是一个定义在Test中的静态类型None，标识预期不抛出任何异常。**如果你定义的一个预期的异常，但是实际运行过程中并没有抛出该异常，不会标红，但是会标黄，并提示没有抛出该异常，实际上这样测试用例依然是没有通过测试，会抛出一个**java.lang.AssertionError**，比如如下代码：

```
/**
 * 异常测试
 */
@Test(expected = ArithmeticException.class)
public void arithmeticExceptionTest() {
    calculate.divide(4, 1);
}
```

![QQ图片20171028142613](\img\QQ图片20171028142613.png)

Maven中你会得到如下的报告：

```shell
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running services.CalculateTest
Tests run: 6, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 1.047 sec <<< FAILURE!
arithmeticExceptionTest(services.CalculateTest)  Time elapsed: 0.004 sec  <<< FAILURE!
java.lang.AssertionError: Expected exception: java.lang.ArithmeticException
...(省略异常栈内容)
Results :

Failed tests:   arithmeticExceptionTest(services.CalculateTest): Expected exception: java.lang.ArithmeticException

Tests run: 6, Failures: 1, Errors: 0, Skipped: 0
```

## 时间预期测试

  业务量大了以后，或者涉及到复杂的算法或线程，我们需要在给定的时间之类测试我们的业务是否能完成，比如我们需要每次正常查询数据库的时间都在10毫秒之内，当然由于服务器资源是有限的，我们的测试并不完备，我们做时间的单元测试并没有考虑到可能会有服务器过载的情况导致运行时间的拉长，但是我们可以考虑一个比较合理的时间（更短），来中和掉这些误差，比如一个业务在70ms正常内完成是被允许的，但是在服务器过载的情况下需要100ms，我们可以将测试的时间预期压缩到30毫秒，以要求开放人员设计出的算法能够达到这个水平，以至于即使在服务器过载情况下，我们也能在时间允许的范围内完成大多数业务请求。

  引入时间测试的另一个理由是，**junit并非是多线程去运行每一个测试用例的，而是按顺序往下执行**，所以我们如果某个业务很耗时（比如一个或多个网络请求），或者直接就阻塞了，严重影响我们的自动化测试过程，所以如果超时我们就直接报错，把这部分代码的问题提交给开发人员去处理。

  我们需要引入@Test注解的另一个属性timeout,它接受一个long类型，默认值是0L，表示不考虑最大等待时间。

  我们在我们的单元测试类中引入一个新方法：

```java
    /**
     * 时间测试
     */
    @Test
    public void timeoutTest() {
        try {
            Thread.sleep(100000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        calculate.divide(4, 1);
    }
```

你可以预料到这个方法要注释100秒之久，在这个过程中，还没执行的测试用例都得不到测试。

![QQ图片20171028145933](\img\QQ图片20171028145933.png)



现在我们加入timeout属性

```java
    /**
     * 时间测试
     */
    @Test(timeout = 1000)
    public void timeoutTest() {
        try {
            Thread.sleep(100000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        calculate.divide(4, 1);
    }
```

现在可以看到，阻塞一秒之后并没有等到任务完成，我们的代码直接报错了，用例没有在预期时间类完成，这该线程会被interrupted，你可以知道线程会抛出一个异常打断sleep，从而让测试继续往下执行。
![QQ图片20171028150227](\img\QQ图片20171028150227.png)

















