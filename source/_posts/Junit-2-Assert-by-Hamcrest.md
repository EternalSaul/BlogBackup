---
title: 'Junit(2):使用Hamcrest断言'
date: 2017-10-29 15:32:43
toc: true
tags:
 - Junit
 - TDD
 - 软件测试
 - 学习笔记
---

  上一篇文章中，我们以几个简单的用例测试来开篇，以assertTure,assertEquals这样的简单形式来作断言，这些方法都是org.junit.Assert来提供，如果你用的是maven或者其他的项目构建工具来引入junit包，我们会发现，它居然依赖于一个org.hamcrest的项目包，实际上，如果涉及到比较复杂的断言，我们以junit需要写一大串代码，这让我们的测试代码非常不美观，这个叫做hamcrest提供了一套断言工具，而且该项目还有ruby等等语言的版本，他们返回一个org.hamcrest.Matcher的子类，另外在org.hamcrest.MatcherAssert中提供了一套assertThat方法，它接受一个org.hamcrest.Matcher和一个计算值，从而做出该值是否符合该Matcher的判断。

  在org.junit.Assert类中也有assertThat方法，不过它其实就是包装了一下MatcherAssert中的assertThat。如下：

```java
    public static <T> void assertThat(String reason, T actual,
            Matcher<? super T> matcher) {
        MatcherAssert.assertThat(reason, actual, matcher);
    }
```

<!--more-->

## 常用的比较

### 基本

#### anything

anything会返回一个匹配任意值（包括null）的matchers，如果一个值可以是任何预期，你可以用anything来做断言

```java
    @Test
    public void anythingMatchersTest() {
        Calculate calculate = new Calculate();
        assertThat(calculate, anything());
        assertThat(null, anything("hello world"));
    }
```

#### is

is匹配两个值是否相等（equals），经常做equalsTo的简写

```java
    @Test
    public void isMatchersTest() {
        assertThat("hello world", is("hello world"));
        assertThat(new String[] {"floor", "bank"}, is(new String[] {"floor", "bank"}));
    }
```
####describedsAs

包装一个Matchers，其中的三个参数：

description:作为包装后的Matchers的失败描述

matcher:要包装的matchers

valus:选择性插入值,它会被插入到description标识的数字占位符处，该参数不允许为null

```java
public static <T> org.hamcrest.Matcher<T> describedAs(java.lang.String description, org.hamcrest.Matcher<T> matcher, java.lang.Object... values) {
    return org.hamcrest.core.DescribedAs.<T>describedAs(description, matcher, values);
  }
```

比如如下代码

```java
@Test
public void describedsAsMatchersTest() {
    Calculate calculate = new Calculate();
    Calculate calculate1 = new Calculate();
    assertThat(calculate1, describedAs("hello world %0,%1",is(calculate),new String[]{"saul","xin"}));
    //当然你也可以这样传入多值参数
    //assertThat(calculate1, describedAs("hello world %0,%1",is(calculate),"saul","xin");
}
```

测试：

![QQ图片20171029180301](\img\QQ图片20171029180301.png)



### 逻辑比较

#### allOf

![QQ图片20171029221453](\img\QQ图片20171029221453.png)

allOf在hamcrest相当于与操作，如下对于"hello world"验证它是否同时符合以下两个Matchers，显然是符合的，allOf有多个重写，接受的参数是一个可变的Matchers列表，或者一个Matchers的迭代器。

```java
@Test
public void allOfTest(){
    assertThat("hello world",allOf(endsWith("ld"),startsWith("hello")));
}
```

#### anyOf

anyOf和allOf类似，相当于或操作，既然类似当然它们的参数表也是类似了

```java
@Test
public void anyOfTest(){
    assertThat("hello world",anyOf(endsWith("mld"),startsWith("hello")));
}
```

#### not

not表否定，可以接受一个Matchers或者一个值，返回一个Matchers，表示非逻辑，如下

```java
@Test
public void notTest(){
    assertThat("hello world",not("Hello world"));
    assertThat("hello world",not(allOf(endsWith("mld"),startsWith("hello"))));
}
```

### 对象

#### equalTo

检测两个值是否相等，equals判断

```java
@Test
public void equalToTest() {
    String k="hello world";
    String s=new String("hello world");
    assertThat(k,equalTo(s));
}
```

#### hasToString

测试某个对象的toString方法的返回值是否符合某个Matchers或等于某个值

```java
@Test
public void hasToStringTest() {
    assertThat(new Integer(1), hasToString("1"));
    assertThat(new Integer(1), hasToString(endsWith("1")));
}
```

#### instanceOf

判断某个对象是否是某种类型的实例

```java
@Test
public void instanceOfTest() {
    assertThat(new Integer(1), instanceOf(Number.class));
}
```

#### isCompatibleType

测试一个类是否是可兼容另一个类

```java
@Test
public void isCompatibleTypeTest() {
    assertThat(Integer.class, typeCompatibleWith(Number.class));
}
```

#### notNullValue

检测一个值是否不为null

```java
@Test
public void notNullValueTest() {
    assertThat(null, not(notNullValue()));
}
```

#### nullValue

检测一个值是否为null

```java
@Test
public void nullValueTest() {
    assertThat(null, nullValue());
}
```

#### sameInstance

检测两个对象是否相同,==判断

```java
@Test
public void sameInstanceTest() {
    String k="hello";
    String s="hello";
    assertThat(s,sameInstance(k));
}
```

#### hasProperty

判断一个Bean对象是否有某个属性，这个Matchers会使用get方法来判断，没有get方法就是没有该属性

```java
@Test
public void hasPropertyTest() {
    User user = new User();
    assertThat(user, hasProperty("name"));
}
```

### 集合

#### array

生成一个Machers，其包含一组Machers，接着对一个数组进行遍历匹配，每个元素匹配他们的对应位置

```java
@Test
public void arrayTest() {
    String ls[] = {"hello", "world", "love", "bcd"};
    assertThat(ls, array(startsWith("hel"), startsWith("wor"), startsWith("lo"), endsWith("d")));
}
```

#### hasEntry,hasKey,hasValue

返回一个Matchers，匹配键值对

```java
@Test
public void MapTest() {
    Map<String, String> map = new HashMap<String, String>();
    map.put("name", "tom");
    assertThat(map, hasEntry("name", "tom"));
    assertThat(map, hasEntry(startsWith("nam"), endsWith("om")));
    assertThat(map, hasKey("name"));
    assertThat(map, hasValue(endsWith("om")));
}
```

#### hasItem,hasItems,hasItemInArray

返回一个Mathers，对集合中的每一个元素进行匹配，要求至少有一个元素满足给定条件

```java
@Test
public void ItemTest() {
    Set<String> set = new HashSet<String>();
    set.add("some");
    set.add("one");
    set.add("in here");
    assertThat(set, hasItem(startsWith("some")));
    assertThat(set, hasItems(anyOf(startsWith("can"), endsWith("here"))));
    assertThat(set.toArray(new String[]{}), hasItemInArray(startsWith("so")));
}
```

### 数字

#### 近似值匹配

 匹配某个值是否在误差允许的范围内接近一个精确值

```java
@Test
public void closeToTest() {
    assertThat(5.3, closeTo(5, 0.3));
    BigDecimal bigDecimal = new BigDecimal("99999999999999999999999999999999999999");
    BigDecimal error = new BigDecimal(0.001);
    assertThat(new BigDecimal("99999999999999999999999999999999999999.0009"), closeTo(bigDecimal, error));
}
```

#### 大小匹配

 返回一个Matchers做值的大小匹配

```java
@Test
public void compareToTest() {
    assertThat(5, greaterThan(4));
    assertThat(5, lessThan(6));
    assertThat(5, greaterThanOrEqualTo(5));
    assertThat(4, lessThanOrEqualTo(4));
}
```

### 文本

```java
@Test
public void stringTest(){
    assertThat("jay",equalTo("jay"));
    assertThat("hello",equalToIgnoringCase("HelLO"));
    //忽略头尾所有空白字符和空字符，且非头尾空白字符，空字符都认为是1个空白字符进行匹配
    //"helloworld"匹配"  helloworld"和"helloworld  "，但是不匹配"hello world"，因为非头尾空白字符总被认为是一个需要匹配的空字符或空白字符
    assertThat("hello  wor\nld",equalToIgnoringWhiteSpace("  \t\bhello wor  ld    "));
    assertThat("hello",startsWith("hel"));
    assertThat("name",endsWith("me"));
    assertThat("hello world!",containsString("!"));
}
```













