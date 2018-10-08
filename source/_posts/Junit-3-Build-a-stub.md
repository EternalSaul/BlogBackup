---
title: 'Junit(3):桩测试'
date: 2018-01-29 22:45:51
tags:
 - Junit
 - TDD
 - 软件测试
 - 学习笔记
---

  如果我们只是单纯测试一个类里的公共方法，那么使用简单的断言，就能满足我们的需求，可是我们的**测试很多时候要依赖其他的类，或者说环境**，比如你测试的方法可能要访问一些服务器获取资源，或者访问数据库获取数据，此时我们必须需要一个真实的环境么？实际上很多情况下，你根本拿不到真实的环境，比如你测试的是一个还没有完成的服务模块。既然有这种需求，我们就得想办法在开发环境中模拟一个模块来测试，这有两种策略，一种就是stub，还一种就是mock，**这两种方法都是为了代替依赖部分，不同的是mock是构建一个假的对象，而stub是真实地简单实现一个对象**，这一节我先讲这个stub，我们将以stub的方式，模拟一个对一个服务器发送http请求测试。

<!--more-->

  stub是一段代码，常在运行期间代替真实的代码，它的目的在于模拟一个简单的行为，因为真实的代码可能过于复杂，或者根本不可得，使用stub技术就可以允许独立的测试真实代码中的一部分行为，**其实你可以把stub代码理解为是对真实代码一部分行为的简单实现**。

  有时候stub代码很难写，因为他可能要再现真实代码的复杂逻辑，你知道有的系统中即使是以小块功能逻辑也是非常复杂的，所以一般情况下stub只适合代码中的粗粒度部分的测试，以stub代码来代替成熟的外部系统，比如服务器的连接或者数据库。

比如我们要测试下面这个类：

```java
package xin.saul;

import java.io.IOException;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class MyClient {

    public String getContent(URL url) {
        StringBuffer content = new StringBuffer();
        HttpURLConnection connection = null;
        try {
            InputStream stream = url.openStream();
            byte[] bs = new byte[2049];
            int len;
            while ((len = stream.read(bs)) != -1) {
                content.append(new String(bs, 0, len));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return content.toString();
    }
  
}
```

假如这个Myclient就是我们所构造应用的客户端，它可以通过url获取一个web资源，我们应该如何去测试它呢？这里会出现几种情况，一种就是我们的服务器已经构建完毕了，那么我们可以直接去测试它，但是很多时候我们的客户端开发完成后，我们的服务器并没有开发完成，那么我们测试前必须去先架构一个apache或者tomcat服务器么？答案肯定是否定的，apache和tomcat启动速度并不是很快，我们有一些代替方案，比如我们以Jetty来写一段stub代码，以模拟我们的客户端和服务器交互过程。

```java
package xin.saul;

import org.eclipse.jetty.server.Handler;
import org.eclipse.jetty.server.Request;
import org.eclipse.jetty.server.Server;
import org.junit.Assert;
import org.junit.BeforeClass;
import org.junit.Test;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;
import java.net.MalformedURLException;
import java.net.URL;

public class MyClientTest{
    @BeforeClass
    public static void initClass() throws Exception {
        Server server = new Server(8080);
        MyClientTest test = new MyClientTest();
        server.setHandler(test.new SayHelloHandler());
        server.setStopAtShutdown(true);
        server.start();
    }

    @Test
    public void testClient() throws MalformedURLException {
        MyClient client = new MyClient();
        String content = client.getContent(new URL("http://127.0.0.1:8080"));
        Assert.assertEquals("Server response",content);
    }


    private class SayHelloHandler implements Handler {

        @Override
        public void handle(String target, Request baseRequest, HttpServletRequest request, HttpServletResponse response) throws IOException, ServletException {
            response.setContentType("text/plain; charset=utf-8");
            response.setStatus(HttpServletResponse.SC_OK);
            PrintWriter writer = response.getWriter();
            writer.append("Server response");
            writer.close();
        }
      //注：此处省略了其他的方法重新，直接是ide自动生成的重写，我本人并没有改动
    }
}
```

  可以看到stub代码其实就是很傻乎乎的直接实现一个小模块小功能，当然这里是基于servlet服务,这里我以嵌入式服务器jetty来实现了这段stub，原因是它启动的很快，代码很简单。这里提一点，**我们知道对于单元测试来说每个测试的不相干性是必要的，你不能因为上一个测试的执行而改变了服务器某些状态，于是我们希望没执行一个新的测试都能够拥有一个干净的环境**，这也是诸如@Before这样的注解会做的，但是在打桩测试中你会发现一个jetty嵌入式服务器的启动时间可能是0.5秒，如果你要执行很多个stub测试，那就很有意思了，我不觉得你想在按下测试按钮以后在屏幕前等待十几分钟甚至几十分钟，然后发现某个地方测试没通过，重构之后又等一遍，于是我们可以尝试使用@BeforeClass注解，它只会执行一次，当然**你得保证我们的服务器执行的具体逻辑和测试的执行顺序无关，测试之间不会相互影响**。