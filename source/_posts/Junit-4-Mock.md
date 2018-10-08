---
title: 'Junit(4):Mock'
date: 2018-02-01 16:29:46
tags:
 - Junit
 - TDD
 - 软件测试
 - 学习笔记
---

  就像上篇文章所说，我们还可以用mock方式模拟一个没有完成的模块，从而完成测试，事实上如果测试的业务比较繁琐，比如涉及到输入啥的，我们也可以mock一个假的输入对象，mock object是在极限编程里提出的，而今天它已经广泛用于软件测试领域。

  就如mock这个单词的词义一样，mock object即是一个模制的对象，但这个对象的行为是人为故意给定的，不包含实际的业务。这样说可能非常抽象，如果让我们考虑用**Scala**写一个在商店里用pos机刷卡的例子,比如我们刷卡并且输入密码后会构造出这样的一个CreditCard对象：

```scala
case class CreditCard(val cardId:String,val password:String)
```

 而我们的Pos机要负责转账处理：

```scala
class Pos {
  def transfer(amount:Double,creditCard: CreditCard, creditCard2: CreditCard, accountManager: AccountManager): Unit = {
    val account1: Account = accountManager.findAccount(creditCard.cardId, creditCard.password);
    val account2: Account = accountManager.findAccount(creditCard2.cardId, creditCard2.password);
    accountManager.saveAccount(account1.copy(account1.balance - amount,account1.creditCard))
    accountManager.saveAccount(account2 copy(account2.balance + amount, account2.creditCard))
  }
}
```
，但是问题是此时我们根本没有构造号银行卡账户数据库，甚至我们连dao对象都只是一个接口，那么为了测试Pos代码逻辑能不能正确执行我们该怎么办呢？

<!--more-->

```scala
trait AccountManager {
  def saveAccount(account: Account)
  def findAccount(cardId: String, password: String): Account
}
```

答案是我么可以建立一个mock object，这不仅可以很容易的测试Pos的代码逻辑，而且节省了很大的开销，设想一下，如果真正的去测试，我们不仅要确定整个容器环境无误，还要不断地去访问数据库，这肯定是不可以忍受的。

```scala
class MockAccountManager() extends AccountManager {
    var accounMap = Map (
      ("1001", Account(1000, CreditCard("1001", "123")))
      , ("1002", Account(1000, CreditCard("1002", "123")))
      )

    override def findAccount(cardId: String, password: String): Account = {
      val x=accounMap(cardId)
      x match {
        case Account(_,password)=>x
      }
    }

    override def saveAccount(account: Account): Unit = {
      accounMap = accounMap ++ Map (
        (account.creditCard.cardId, account)
        )
    }
}
```

这样我们就可以利用这个，简单的mock object去测试我们的Pos逻辑了：

```scala
@Test
def testTransfer(): Unit = {
  val pos = new Pos();
  val card = new CreditCard("1001", "123")
  val card2 = new CreditCard("1002", "123")
  val acm = new MockAccountManager()
  pos.transfer(100.2, card, card2, acm)
  assertEquals(acm.findAccount(card2.cardId, card2.password).balance,1100.2d,0);
}
```

  我在这里手动写了一个mock object仅仅是为了演示一下大概的思想，实际上我们不需要手动写mock object，有大量的框架帮助我们完成这一点，比如在jvm平台上就有Jmock，EasyMock,Mockito等等，它们借助jvm的反射机制，以代理模式动态的生成了一个新的对象，我们可以给定这个对象特定方法的特定输入输出，十分方便，比如对于猜数字游戏：

随机给定一个4位数字，且每一位的数字都不重复，然后玩家在不知道这个数字的情况下输入一个4位数字每一位都不重复，如果位置相同且数字也相同就得到一个A，如果数字存在但是位置不同就得到B。

如：

给定1234,玩家猜测为1256，则结果为2A0B

给定1234，玩家猜测为3412，则结果为0A4B

如果玩家在6次之内得到了4A0B结果，就输出玩家胜利。

这样的话，我们可以来写一个简单的程序：

首先我们需要输入和输出,以下两个类用来读取数字和输出一个字符串：

```java
//输入
public class NumberReader {
    private Scanner scanner = new Scanner(System.in);

    public String read() throws Exception {
        String inputNumber = scanner.next();
        if (StringUtils.isNumeric(inputNumber) && inputNumber.length() == 4) {
            return inputNumber;
        }
        throw new Exception("Invalid Input");
    }
}

//输出
public class Printer {
    public void print(String message) {
        System.out.println(message);
    }
}
```

然后我们需要一个随机数字生成器,它可以随机生成一个4位不同数字组成的字符串：

```java
public class NumberGenerator {
    public String generate() {
        List<String> list = Arrays.asList("0", "1", "2", "3", "4", "5", "6", "7", "8", "9");
        Collections.shuffle(list);
        return String.join("", list.subList(0, 4));
    }
}
```

然后我们需要一种数字类,它有一个比较方法，输出xAxB这样的表示：

```java
public class Number {
    private static final int NUMBER_LENGTH = 4;
    private String value;

    public Number(String value) {
        this.value = value;
    }

    public String compare(String stringToCompare) {
        int a = 0, b = 0;
        for (int i = 0; i < NUMBER_LENGTH; i++) {
            if (value.charAt(i) == stringToCompare.charAt(i)) {
                a++;
            } else if (value.contains(stringToCompare.substring(i, i + 1))) {
                b++;
            }
        }
        return a + "A" + b + "B";
    }
}
```

最后我们把上面的组件组成一个游戏：

```java
public class Game {
    private static final String WIN_CODE = "4A0B";
    private Printer printer;
    private Number generatedNumber;
    private NumberReader reader;

    //我们有6次机会
    private int remainingGuessCount = 6;

    public Game(NumberGenerator numberGenerator, Printer printer, NumberReader reader) {
        this.printer = printer;
        this.reader = reader;
        this.generatedNumber = new Number(numberGenerator.generate());
    }

    public void start() {
        printer.print("please input a 4 digit number:");

        while (remainingGuessCount > 0) {
            try {
              //比较数字输入和生成的随机数
                String result = generatedNumber.compare(reader.read());
                if (WIN_CODE.equals(result)) {
                    printer.print("you win");
                    return;
                }

                printer.print(result);
                this.remainingGuessCount--;//机会使用了一次，所以减少
            } catch (Exception e) {
                printer.print(e.getMessage());
            }
        }

        printer.print("Game Over");
    }

}
```

现在我们有一个简单的游戏了，如果玩家6次内猜对了4A0B，那么就输出you win,否则输出Game Over。现在我们要对这个游戏进行测试，但是我们可能会遇到以下的问题：

1.我们需要自动测试，而不是每次都手动输入

这是一个问题，我们的输入类NumberReader，调用的是系统的System.in但是我们却要求自动化测试，而不是每次手动输入，既然如此，我们就需要一种mock object，以代理的手段自动返回我们想输入的数字，我们可以自己手写字段反射，覆盖掉输入流，比如：

```java
public class NumberReaderTest {
    private NumberReader reader = new NumberReader();

    private void setInputStream(String input) throws NoSuchFieldException, IllegalAccessException {
        Field scannerField = reader.getClass().getDeclaredField("scanner");
        scannerField.setAccessible(true);
        Scanner scannerWithMockedStream = new Scanner(new ByteArrayInputStream(input.getBytes()));
        scannerField.set(reader, scannerWithMockedStream);
    }

    @Test
    public void shouldReadNumber() throws Exception {
        setInputStream("1234");
        assertEquals("1234", reader.read());
    }

    @Test(expected = Exception.class)
    public void shouldThrowExceptionForNonNumberInput() throws Exception {
        setInputStream("123d");
        reader.read();
    }
}
```

手写是一种手段，我们可以理解框架大概是怎么做到的，当然其实可以用代理来做，现实情况中我们能够使用mock框架，这里我用的是[mockito](https://github.com/mockito/mockito),当然你也可以用其他的框架：

```java
import org.junit.Before;
import org.junit.Test;
import java.io.IOException;
import static org.mockito.Mockito.*;

public class GameTest {
    private NumberGenerator numberGenerator;
    private NumberReader numberReader;
    private Printer printer;
    private Game game;

    @Before
    public void init() {
      //我们以mock方式生成了:
        numberGenerator = mock(NumberGenerator.class);
        numberReader = mock(NumberReader.class);
        printer = mock(Printer.class);
      //我们给定generate的返回值必须是4879，因为随机生成一个数字可就不好写expected值了
        when(numberGenerator.generate()).thenReturn("4879");
        game = new Game(numberGenerator, printer, numberReader);
    }
}
```

然后我们可以写一些简单测试，来体验一下mock的作用：

```java
@Test
public void shouldPrintYouWin() throws Exception {
    when(numberReader.read()).thenReturn("4879");
    game.start();
  //使用verify可以在白盒情况下审视代码逻辑是不是正常运转
    verify(printer, times(1)).print("please input a 4 digit number:");
    verify(printer, times(1)).print("you win");
}

@Test
public void shouldPrintYouWinAfterFifthAttempts() throws Exception {
  //mockito支持我们每次返回不同的值
    when(numberReader.read()).thenReturn("7849","7849","7849","7849","7849","4879");
    game.start();
    verify(printer, times(1)).print("please input a 4 digit number:");
    verify(printer, times(5)).print("2A2B");
    verify(printer, times(1)).print("you win");
}

@Test
public void shouldPrintYouWinAfterSecondAttempts() throws Exception {
    when(numberReader.read()).thenReturn("7849","4879");
    game.start();
    verify(printer, times(1)).print("please input a 4 digit number:");
    verify(printer, times(1)).print("2A2B");
    verify(printer, times(1)).print("you win");
}

@Test
public void shouldPrintGameOver() throws Exception {
    when(numberReader.read()).thenReturn("7849");
    game.start();
    verify(printer, times(1)).print("please input a 4 digit number:");
    verify(printer, times(6)).print("2A2B");
    verify(printer, times(1)).print("Game Over");
}

@Test
public void shouldRunNormallyAfterThrowsException() throws Exception {
  //我们还可以设定抛出异常
    when(numberReader.read()).thenThrow(new IOException("IO Exception Test"))
            .thenReturn("7849","7849","7849","7849","7849","4879");
    game.start();
    verify(printer, times(1)).print("please input a 4 digit number:");
    verify(printer, times(1)).print("IO Exception Test");
    verify(printer, times(5)).print("2A2B");
    verify(printer, times(1)).print("you win");
}
```

当然上面这些简单的mock，我们也可以手动给它以代理方式写出来，有利于我们理解mockito这类框架的实际工作方式，感兴趣的朋友可以写写看，我这里就不写了~