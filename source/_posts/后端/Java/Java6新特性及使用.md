---
title: Java6新特性及使用
date: 2018-11-11 22:40:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/11/11/java.jpg
categories: 后端
tags:
  - Java
---

## 新特性列表

以下是Java6中的引入的部分新特性，相比Java5的新特性就少了很多了。关于Java6更详细的介绍可参考[这里](http://www.oracle.com/technetwork/java/javase/features-141434.html)。

- Web Services Metadata
- Scripting
- Compiler API
- Light-weight HTTP server
- Common annotations(JSR 250)
- StAX
- JAXB2
- Console
- Java DB(Derby)
- JDBC 4.0
- 值得关注的
  - 集合框架增强
- 其它
  - GUI增强

## 一、Web Services Metadata

`WebService`是一种独立于特定语言、特定平台，基于网络的、分布式的模块化组件。是一个能够使用`xml`消息通过网络来访问的接口，这个接口描述了一组可访问的操作。在Java6中，在想要发布为`WebService`的类上加上`@WebService`的注解，这个类的方法就变为`WebService`方法了，再通过`Endpoint.publish()`方法发布这个服务。到此，一个最简单的`WebService`搞定。运行`main`方法，在浏览器里输入`http://localhost:8080/com.blinkfox.test.Hello?wsdl`，即可查看你WebService的WSDL信息。

```java
import javax.jws.WebService;
import javax.xml.ws.Endpoint;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Hello.
 * @author blinkfox on 2017-11-28.
 */
@WebService
public class Hello {

    private static final Logger log = LoggerFactory.getLogger(Hello.class);

    /**
     * sayHello.
     * @param name 名称
     * @return 结果
     */
    public String sayHello(String name) {
        return "Hello ".concat(name);
    }

    /**
     * @param args
     */
    public static void main(String[] args) {
        Endpoint.publish("http://localhost:8080/com.blinkfox.test.Hello", new Hello());
        log.info("调用成功!");
    }

}
```

Java 自从JDK5中添加了元数据功能(注解)之后,SUN几乎重构了整个J2EE体系，由于变化很大，干脆将名字也重构为Java EE，Java EE(当前版本为5.0)将元数据纳入很多规范当中，这其中就包括`Web Services`的相关规范，这显然比以前的JAX-RPC编程模型简单(当然, Axis的编程模型也很简单)。这里要谈的Web服务元数据(JSR 181)只是Java Web 服务规范中的一个,它跟Common Annotations, JAXB2, StAX, SAAJ和JAX-WS等共同构成Java EE 5的Web Services技术堆栈。

下面介绍`JSR-181`里面各个元数据的相关参数及用途。

| Annotation   | Retention | Target            | Description                              |
| ------------ | --------- | ----------------- | ---------------------------------------- |
| WebService   | Runtime   | Type              | 标注要暴露为Web Services的类或接口                  |
| WebParam     | Runtime   | Parameter         | 自定义服务方法参数到WSDL的映射                        |
| WebResult    | Runtime   | Method            | 自定义服务方法返回值到WSDL的映射                       |
| WebMethod    | Runtime   | Method            | 自定义单个服务方法到WSDL的映射                        |
| Oneway       | Runtime   | Method            | 必须与@WebMethod连用,表明被标注方法只有输入没有输出,这就要求被标注方法不能有返回值,也不能声明checked exception |
| HandlerChain | Runtime   | Type,Method,Field | 将Web服务与外部Handler chain关联起来               |
| SOAPBinding  | Runtime   | Type,Method       | 自定义`SOAPBinding`                         |



## 二、Scripting

Java6增加了对动态语言的支持，原理上是将脚本语言编译成字节码，这样脚本语言也能享用Java平台的诸多优势，包括可移植性，安全等。另外由于现在是编译成字节码后再执行，所以比原来边解释边执行效率要高很多。可以很好的利用脚本语言的动态特性，主要支持的有`JavaSrcipt`、`Ruby`、`Python`等。

以下使用`JavaScript`的脚本，代码示例如下：

```java
import javax.script.Invocable;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * JsTest.
 * @author blinkfox
 * @version 1.0
 *
 */
public class JsTest {

    private static final Logger log = LoggerFactory.getLogger(Hello.class);

    /**
     * main方法.
     * @param args 数组参数
     * @throws ScriptException 脚本异常
     * @throws NoSuchMethodException 无方法异常
     */
    public static void main(String[] args) throws ScriptException, NoSuchMethodException {
        ScriptEngineManager enjineManager = new ScriptEngineManager();
        ScriptEngine engine = enjineManager.getEngineByName("JavaScript");

        String script="function hello(name){return 'Hello ' + name}";
        engine.eval(script);
        Invocable inv=(Invocable) engine;
        String result = (String) inv.invokeFunction("hello", "blinkfox");
        log.info("脚本执行结果:{}", result);
    }

}
```

## 三、Compiler API

在Java6中提供了一套`Compiler API`，定义在`JSR199`中, 提供在运行期动态编译java代码为字节码的功能。一套API就好比是在java程序中模拟javac程序，将Java源文件编译为class文件；其提供的默认实现也正是在文件系统上进行查找、编译工作的。`Compiler API`结合反射功能就可以实现动态的产生Java代码并编译执行这些代码，有点动态语言的特征。

基本使用示例如下：

```java
public class JavaCompilerAPICompiler {

    public void compile(Path src, Path output) throws IOException {
        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        try (StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, null)) {
            Iterable<? extends JavaFileObject> compilationUnits = fileManager.getJavaFileObjects(src.toFile());
            Iterable<String> options = Arrays.asList("-d", output.toString());
            JavaCompiler.CompilationTask task = compiler.getTask(null, fileManager, null, options, null, compilationUnits);
            boolean result = task.call();
        }
    }

}
```

## 四、轻量级HTTP server

JDK6提供了一个轻量级的`Http Server API`，据此我们可以构建自己的嵌入式Http Server，它支持`Http`和`Https`协议,提供了HTTP1.1的部分实现，没有被实现的那部分可以通过扩展已有的Http Server API来实现，程序员必须自己实现`HttpHandler`接口，HttpServer会调用`HttpHandler`实现类的回调方法来处理客户端请求，在这里，我们把一个Http请求和它的响应称为一个交换,包装成`HttpExchange`类,HttpServer负责将`HttpExchange`传给`HttpHandler`实现类的回调方法。

以下是通过JDK6新特性能够实现的HttpServer的示例：

```java
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;
import com.sun.net.httpserver.spi.HttpServerProvider;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.InetSocketAddress;

/**
 * 自定义的http服务器.
 *
 * @author blinkfox on 2017-12-04.
 */
public class MyHttpServer {

    /**
     * 启动服务，监听来自客户端的请求.
     *
     * @throws IOException IO异常
     */
    private static void httpserverService() throws IOException {
        HttpServerProvider provider = HttpServerProvider.provider();
        HttpServer httpserver = provider.createHttpServer(new InetSocketAddress(8888), 200); // 监听端口8888,能同时接受100个请求
        httpserver.createContext("/mytest", new MyHttpHandler());
        httpserver.setExecutor(null);
        httpserver.start();
        System.out.println("server started");
    }

    /**
     * Http请求处理类.
     */
    private static class MyHttpHandler implements HttpHandler {

        public void handle(HttpExchange httpExchange) throws IOException {
            String responseMsg = "ok"; //响应信息
            InputStream in = httpExchange.getRequestBody(); //获得输入流
            BufferedReader reader = new BufferedReader(new InputStreamReader(in));
            String temp = null;
            while((temp = reader.readLine()) != null) {
                System.out.println("client request:" + temp);
            }
            httpExchange.sendResponseHeaders(200, responseMsg.length()); //设置响应头属性及响应信息的长度
            OutputStream out = httpExchange.getResponseBody();  //获得输出流
            out.write(responseMsg.getBytes());
            out.flush();
            httpExchange.close();
        }

    }

    public static void main(String[] args) throws IOException {
        httpserverService();
    }

}
```

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * Http服务器测试类.
 *
 * @author blinkfox on 2017-12-04.
 */
public class HttpTest {

    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        // 测试并发对MyHttpServer的影响
        for (int i = 0; i < 20; i++) {
            Runnable run = new Runnable() {
                public void run() {
                    try {
                        startWork();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            };
            exec.execute(run);
        }
        exec.shutdown();// 关闭线程池
    }

    public static void startWork() throws IOException {
        URL url = new URL("http://127.0.0.1:8888/mytest");
        HttpURLConnection urlConn = (HttpURLConnection) url.openConnection();
        urlConn.setDoOutput(true);
        urlConn.setDoInput(true);
        urlConn.setRequestMethod("POST");
        // 测试内容包
        String teststr = "this is a test message";
        OutputStream out = urlConn.getOutputStream();
        out.write(teststr.getBytes());
        out.flush();
        while (urlConn.getContentLength() != -1) {
            if (urlConn.getResponseCode() == 200) {
                InputStream in = urlConn.getInputStream();
                BufferedReader reader = new BufferedReader(new InputStreamReader(in));
                String temp = "";
                while ((temp = reader.readLine()) != null) {
                    System.err.println("server response:" + temp);// 打印收到的信息
                }
                reader.close();
                in.close();
                urlConn.disconnect();
            }
        }
    }

}
```

## 五、Common annotations

`Common annotations`原本是Java EE 5.0(JSR 244)规范的一部分，现在SUN把它的一部分放到了Java SE 6.0中.随着Annotation元数据功能(JSR 175)加入到Java SE 5.0里面，很多Java 技术(比如EJB,Web Services)都会用Annotation部分代替XML文件来配置运行参数（或者说是支持声明式编程,如EJB的声明式事务）, 如果这些技术为通用目的都单独定义了自己的Annotations,显然有点重复建设, 所以,为其他相关的Java技术定义一套公共的Annotation是有价值的，可以避免重复建设的同时，也保证Java SE和Java EE 各种技术的一致性。

下面列举出`Common Annotations 1.0`里面的10个`Annotations`：

| **Annotation** | **Retention** | **Target**                               | **Description**                          |
| :------------- | :-----------: | :--------------------------------------- | :--------------------------------------- |
| Generated      |    Source     | ANNOTATION_TYPE, CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE | 用于标注生成的源代码                               |
| Resource       |    Runtime    | TYPE, METHOD, FIELD                      | 用于标注所依赖的资源,容器据此注入外部资源依赖，有基于字段的注入和基于setter方法的注入两种方式 |
| Resources      |    Runtime    | TYPE                                     | 同时标注多个外部依赖，容器会把所有这些外部依赖注入                |
| PostConstruct  |    Runtime    | METHOD                                   | 标注当容器注入所有依赖之后运行的方法，用来进行依赖注入后的初始化工作，只有一个方法可以标注为PostConstruct |
| PreDestroy     |    Runtime    | METHOD                                   | 当对象实例将要被从容器当中删掉之前，要执行的回调方法要标注为PreDestroy |
| RunAs          |    Runtime    | TYPE                                     | 用于标注用什么安全角色来执行被标注类的方法，这个安全角色必须和Container 的Security角色一致的 |
| RolesAllowed   |    Runtime    | TYPE, METHOD                             | 用于标注允许执行被标注类或方法的安全角色，这个安全角色必须和Container 的Security角色一致的 |
| PermitAll      |    Runtime    | TYPE, METHOD                             | 允许所有角色执行被标注的类或方法                         |
| DenyAll        |    Runtime    | TYPE, METHOD                             | 不允许任何角色执行被标注的类或方法，表明该类或方法不能在Java EE容器里面运行 |
| DeclareRoles   |    Runtime    | TYPE                                     | 用来定义可以被应用程序检验的安全角色，通常用isUserInRole来检验安全角色 |

## 六、StAX

StAX(JSR 173)是JDK6中新增的除了DOM和SAX之外的又一种处理XML文档的API。

`StAX`是`The Streaming API for XML`的缩写，一种利用拉模式解析(pull-parsing)XML文档的API。StAX通过提供一种基于事件迭代器(Iterator)的API让程序员去控制xml文档解析过程,程序遍历这个事件迭代器去处理每一个解析事件，解析事件可以看做是程序拉出来的，也就是程序促使解析器产生一个解析事件然后处理该事件，之后又促使解析器产生下一个解析事件，如此循环直到碰到文档结束符；SAX也是基于事件处理xml文档，但却是用推模式解析，解析器解析完整个xml文档后，才产生解析事件，然后推给程序去处理这些事件；DOM采用的方式是将整个xml文档映射到一颗内存树，这样就可以很容易地得到父节点和子结点以及兄弟节点的数据，但如果文档很大，将会严重影响性能。

下面是这几种XML解析API的特性比较：

| Feature                      | StAX            | SAX             | DOM            | TrAX      |
| ---------------------------- | --------------- | --------------- | -------------- | --------- |
| API Type                     | Pull, streaming | Push, streaming | In memory tree | XSLT Rule |
| Ease of Use                  | High            | Medium          | High           | Medium    |
| XPath Capability             | No              | No              | Yes            | Yes       |
| CPU and Memory Efficiency    | Good            | Good            | Varies         | Varies    |
| Forward Only                 | Yes             | Yes             | No             | No        |
| Read XML                     | Yes             | Yes             | Yes            | Yes       |
| Write XML                    | Yes             | No              | Yes            | Yes       |
| Create, Read, Update, Delete | No              | No              | Yes            | No        |

下面代码演示了如何通过StAX读取xml文档和生成xml文档：

需要读取的xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalogs>
    <catalog id="001">Book</catalog>
    <catalog id="002">Video</catalog>
</catalogs>
```

读和写XML文件的Java代码：

```java
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import javax.xml.namespace.QName;
import javax.xml.stream.*;
import javax.xml.stream.events.StartElement;
import javax.xml.stream.events.XMLEvent;

/**
 * Stax测试类.
 *
 * @author blinkfox on 2017-12-04.
 */
public class StaxTester {

    /**
     * 根据StAX读取XML文件.
     *
     * @throws XMLStreamException XML流异常
     * @throws FileNotFoundException 文件未找到异常
     */
    private static void readXxmlByStax() throws XMLStreamException, FileNotFoundException {
        XMLInputFactory xmlif = XMLInputFactory.newInstance();
        XMLEventReader xmler = xmlif.createXMLEventReader(new FileInputStream("G:\\test\\test.xml"));
        XMLEvent event;
        StringBuilder sb = new StringBuilder();
        while (xmler.hasNext()) {
            event = xmler.nextEvent();
            if (event.isStartElement()) { //如果解析的是起始标记
                StartElement element = event.asStartElement();
                sb.append("<");
                sb.append(element.getName());
                if(element.getName().getLocalPart().equals("catalog")) {
                    sb.append(" id=/");
                    sb.append(element.getAttributeByName(new QName("id")).getValue());
                    sb.append("/");
                }
                sb.append(">");
            } else if (event.isCharacters()) { //如果解析的是文本内容
                sb.append(event.asCharacters().getData());
            } else if(event.isEndElement()) { //如果解析的是结束标记
                sb.append("</");
                sb.append(event.asEndElement().getName());
                sb.append(">");
            }
        }
        System.out.println(sb);
    }

    /**
     * 根据StAX写入XML文件.
     *
     * @throws XMLStreamException XML流异常
     * @throws FileNotFoundException 文件未找到异常
     */
    private static void writeXmlByStax() throws XMLStreamException, FileNotFoundException {
        XMLOutputFactory xmlof = XMLOutputFactory.newInstance();
        XMLStreamWriter xmlw = xmlof.createXMLStreamWriter(new FileOutputStream("G:\\test\\output.xml"));
        // 写入默认的 XML 声明到xml文档
        xmlw.writeStartDocument();
        xmlw.writeCharacters("\n");
        // 写入注释到xml文档
        xmlw.writeComment("testing comment");
        xmlw.writeCharacters("\n");
        // 写入一个catalogs根元素
        xmlw.writeStartElement("catalogs");
        xmlw.writeNamespace("myNS", "http://blinkfox.com");
        xmlw.writeAttribute("owner","Chinajash");
        xmlw.writeCharacters("\n");
        // 写入子元素catalog
        xmlw.writeCharacters("    ");
        xmlw.writeStartElement("http://blinkfox.com", "catalog");
        xmlw.writeAttribute("id","007");
        xmlw.writeCharacters("Apparel");
        // 写入catalog元素的结束标签
        xmlw.writeEndElement();
        // 写入catalogs元素的结束标签
        xmlw.writeCharacters("\n");
        xmlw.writeEndElement();
        // 结束 XML 文档
        xmlw.writeEndDocument();
        xmlw.close();
        System.out.println("生成xml文件成功!");
    }

    /**
     * main方法.
     *
     * @param args 数组参数
     * @throws XMLStreamException XML流异常
     * @throws FileNotFoundException 文件未找到异常
     */
    public static void main(String[] args) throws XMLStreamException, FileNotFoundException {
        readXxmlByStax();
        writeXmlByStax();
    }

}
```

运行上面程序后，控制台输出如下:

```bash
<catalogs>
    <catalog id=/001/>Book</catalog>
    <catalog id=/002/>Video</catalog>
</catalogs>
生成xml文件成功!
```

产生的`output.xml`文件如下:

```xml
<?xml version="1.0" ?>
<!--testing comment-->
<catalogs xmlns:myNS="http://blinkfox.com" owner="Chinajash">
    <myNS:catalog id="007">Apparel</myNS:catalog>
</catalogs>
```

## 七、JAXB2

`JAXB`是`Java Architecture for XML Binding`的缩写，可以将一个Java对象转变成为XML格式，反之亦然。我们把对象与关系数据库之间的映射称为ORM, 其实也可以把对象与XML之间的映射称为`OXM`(Object XML Mapping). 原来JAXB是Java EE的一部分，在JDK6中，SUN将其放到了Java SE中，这也是SUN的一贯做法。JDK6中自带的这个JAXB版本是2.0, 比起1.0(JSR 31)来，JAXB2(JSR 222)用JDK5的新特性`Annotation`来标识要作绑定的类和属性等，这就极大简化了开发的工作量。实际上，在Java EE 5.0中，EJB和Web Services也通过Annotation来简化开发工作。另外,JAXB2在底层是用StAX(JSR 173)来处理XML文档。 下面用代码演示在JDK6中如何来用`JAXB2`：

```java
/**
 * Gender性别枚举类.
 *
 * @author blinkfox on 2017-12-04.
 */
public enum Gender {

    MALE(true),

    FEMALE (false);

    private boolean code;

    /**
     * 构造方法.
     * @param code 性别值
     */
    Gender(boolean code) {
        this.code = code;
    }

}
```

```java
import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;

/**
 * Address地址类.
 *
 * @author blinkfox on 2017-12-04.
 */
public class Address {

    @XmlAttribute
    String country;

    @XmlElement
    String state;

    @XmlElement
    String city;

    @XmlElement
    String street;

    /** 由于没有添加@XmlElement,所以该元素不会出现在输出的xml中. */
    String zipcode;

    /**
     * 默认的空构造方法.
     */
    public Address() {
        super();
    }

    public Address(String country, String state, String city, String street, String zipcode) {
        this.country = country;
        this.state = state;
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }

    /**
     * country的getter方法.
     *
     * @return country
     */
    public String getCountry() {
        return country;
    }

}
```

```java
import java.util.Calendar;
import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

/**
 * Person类.
 *
 * @author blinkfox on 2017-12-04.
 */
@XmlRootElement
public class Person {

    /** birthday将作为person的子元素. */
    @XmlElement
    Calendar birthDay;

    /** name将作为person的的一个属性. */
    @XmlAttribute
    String name;

    /** address将作为person的子元素. */
    @XmlElement
    Address address;

    /** gender将作为person的子元素. */
    @XmlElement
    Gender gender;

    /** job将作为person的子元素. */
    @XmlElement
    String job;

    /**
     * 默认的空构造方法.
     */
    public Person() {
        super();
    }

    public Person(Calendar birthDay, String name, Address address, Gender gender, String job) {
        this.birthDay = birthDay;
        this.name = name;
        this.address = address;
        this.gender = gender;
        this.job = job;
    }

    /**
     * address的getter方法.
     * @return address
     */
    public Address getAddress() {
        return address;
    }

}
```

```java
import java.io.FileReader;
import java.io.FileWriter;
import java.util.Calendar;
import javax.xml.bind.JAXBContext;
import javax.xml.bind.Marshaller;
import javax.xml.bind.Unmarshaller;

import org.apache.commons.io.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * JAXB2测试类.
 *
 * @author blinkfox on 2017-12-04.
 */
public class JAXB2Test {

    private static final Logger log = LoggerFactory.getLogger(JAXB2Test.class);

    public static void main(String[] args) {
        Address address = new Address("中国", "北京", "北京", "上地", "100080");
        Person p = new Person(Calendar.getInstance(),"JAXB2", address, Gender.MALE, "软件工程师");

        FileReader reader = null;
        FileWriter writer = null;
        try {
            // 生成xml文件.
            JAXBContext context = JAXBContext.newInstance(Person.class);
            writer = new FileWriter("G:/test/person.xml");
            Marshaller m = context.createMarshaller();
            m.marshal(p, writer);
            log.info("生成person.xml文件成功!");

            // 读取xml文件.
            reader = new FileReader("G:/test/person.xml");
            Unmarshaller um = context.createUnmarshaller();
            Person p2 = (Person) um.unmarshal(reader);
            log.info("Country:{}", p2.getAddress().getCountry());
        } catch (Exception e) {
            log.error("生成和读取XML文件出错！", e);
        } finally {
            IOUtils.closeQuietly(writer);
            IOUtils.closeQuietly(reader);
        }
    }

}
```

运行该程序，我们会得到一个`person.xml`的文件，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<person name="JAXB2">
    <birthDay>2017-12-04T17:16:19.226+08:00</birthDay>
    <address country="中国">
        <state>北京</state>
        <city>北京</city>
        <street>上地</street>
    </address>
    <gender>MALE</gender>
    <job>软件工程师</job>
</person>
```

## 八、Console

JDK6中提供了`java.io.Console`类专用来访问基于字符的控制台设备。你的程序如果要与Windows下的cmd或者Linux下的Terminal交互,就可以用Console类代劳. 但我们不总是能得到可用的Console, 一个JVM是否有可用的Console依赖于底层平台和JVM如何被调用。如果JVM是在交互式命令行(比如Windows的cmd)中启动的,并且输入输出没有重定向到另外的地方，那么就可以得到一个可用的Console实例。下面代码演示了Console类的用法:

```java
import java.io.Console;

/**
 * Jdk6之Console测试类.
 *
 * @author blinkfox on 2017-12-04.
 */
public class ConsoleTest {

    public static void main(String[] args) {
        // 获得Console实例，并判断console是否可用
        Console console = System.console();
        if (console != null) {
            // 读取整行字符和密码，密码输入时不会显示
            String user = new String(console.readLine("请输入用户名:"));
            String pwd = new String(console.readPassword("再输入密码:"));
            console.printf("用户名是:" + user + "\n");
            console.printf("密码是:" + pwd + "\n");
        } else {
            System.out.println("Console不可用!");
        }
    }

}
```

编译该代码，并在命令行中输入：`java ConsoleTest`，然后即可运行，运行示例如下：

```bash
请输入用户名:张三
再输入密码:
打印出的用户名是:张三
打印出的密码是:123456
```

> **注**: 在这里可以看到输入密码时,控制台时不显示这些密码字符的,但是程序可以得到输入的密码字符串,这与Linux下面输入密码的情况是一样的。

## 九、Java DB(Derby)

从JDK6开始，JDK目录中新增了一个名为`db`的目录。这便是 Java 6 的新成员：Java DB。这是一个纯 Java 实现、开源的数据库管理系统（DBMS），源于 Apache 软件基金会（ASF）名下的项目`Derby`。它只有 2MB 大小，对比动辄上 G 的数据库来说可谓袖珍。但这并不妨碍 Derby 功能齐备，支持几乎大部分的数据库应用所需要的特性。JDK6.0里面带的这个Derby的版本是10.2.1.7,支持存储过程和触发器；有两种运行模式，一种是作为嵌入式数据库，另一种是作为网络数据库。前者的数据库服务器和客户端都在同一个JVM里面运行，后者允许数据库服务器端和客户端不在同一个JVM里面，而且允许这两者在不同的物理机器上。值得注意的是JDK6里面的这个Derby支持JDK6的新特性`JDBC 4.0`规范(JSR 221)。

下面分两种情况演示一下如何用代码操作Derby数据库，一种是嵌入式数据库，一种是网络数据库。

### 1. 嵌入式数据库

```java
import com.blinkfox.learn.jdbc.JdbcDaoHelper;

import java.sql.*;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Derby内嵌数据库测试示例.
 *
 * @author blinkfox on 2017-12-04.
 */
public class EmbeddedDerbyTest {

    private static final Logger log = LoggerFactory.getLogger(EmbeddedDerbyTest.class);

    /** Derby驱动,在derby.jar里面. */
    private static final String DRIVER = "org.apache.derby.jdbc.EmbeddedDriver";

    /** 连接Derby的url，create=true表示当数据库不存在时就创建它. */
    private static final String URL = "jdbc:derby:EmbeddedDB;create=true";

    /**
     * main方法.
     *
     * @param args 数组参数
     */
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            Class.forName(DRIVER);
            conn = DriverManager.getConnection(URL);//启动嵌入式数据库
            st = conn.createStatement();
            st.execute("create table foo (FOOID INT NOT NULL, FOONAME VARCHAR(30) NOT NULL)"); //创建foo表
            st.executeUpdate("insert into foo(FOOID,FOONAME) values (1, 'blinkfox')"); //插入一条数据
            rs = st.executeQuery("select * from foo");//读取刚插入的数据
            while (rs.next()) {
                int id = rs.getInt(1);
                String name = rs.getString(2);
                log.info("查询结果：id = {}; name = {}", id, name);
            }
        } catch (Exception e) {
            log.error("使用Derby数据库出错!", e);
        } finally {
            JdbcDaoHelper.close(rs);
            JdbcDaoHelper.close(st);
            JdbcDaoHelper.close(conn);
        }
    }

}
```

运行上面程序后，会在当前目录生成名为`EmbeddedDB`的文件夹，既是`EmbeddedDB`数据库的数据文件存放的地方，控制台将输出：

```bash
查询结果：id = 1; name = blinkfox
```

### 2. 网络数据库

```java
import java.io.PrintWriter;
import java.sql.DriverManager;

import org.apache.derby.drda.NetworkServerControl;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Derby网络数据库测试示例.
 *
 * @author blinkfox on 2017-12-04.
 */
public class NetworkServerDerbyTest {

    private static final Logger log = LoggerFactory.getLogger(NetworkServerDerbyTest.class);

    /** Derby驱动,在derbyclient.jar里面. */
    private static final String DRIVER = "org.apache.derby.jdbc.ClientDriver";

    /** 连接Derby的url. */
    private static final String URL = "jdbc:derby://localhost:1527/NetworkDB;create=true";

    /**
     * main方法.
     * <p>创建Derby网络服务器,默认端口是1527,也可以通过运行<Derby_Home>/frameworks/NetworkServer/bin/startNetworkServer.bat
     来创建并启动Derby网络服务器,如果是Unix,用startNetworkServer.ksh</p>
     *
     * @param args 数组参数
     */
    public static void main(String[] args) {
        NetworkServerControl derbyServer = null;
        try {
            //NetworkServerControl类在derbynet.jar里面
            derbyServer = new NetworkServerControl();
            PrintWriter pw = new PrintWriter(System.out); //用系统输出作为Derby数据库的输出
            derbyServer.start(pw); //启动Derby服务器
            Class.forName(DRIVER);
            DriverManager.getConnection(URL);
        } catch (Exception e) {
            log.error("操作Derby网络数据库异常!", e);
        } finally {
            if (derbyServer != null) {
                try {
                    derbyServer.shutdown();
                } catch (Exception e) {
                    log.error("关闭Derby网络数据库异常!", e);
                }
            }
        }
    }

}
```

运行上面程序后,会在当前目录生成名为`NetworkDB`的文件夹。关于`Derby`的详细情况,请参考[http://db.apache.org/derby](http://db.apache.org/derby)。

## 十、JDBC 4.0

在 Java SE 6 所提供的诸多新特性和改进中，值得一提的是为 Java 程序提供数据库访问机制的 JDBC 版本升级到了 4.0, 这个以 JSR-221 为代号的版本，提供了更加便利的代码编写机制及柔性，并且支持更多的数据类型。JDBC4.0 主要有以下改进和新特性。

- 自动加载`java.sql.Driver`，而不需要再调用`class.forName`；
- 添加了`java.sql.RowId`数据类型用来可以访问`sql rowid`；
- 添加了`National Character Set`的支持；
- 增强了`BLOB`和`CLOB`的支持功能；
- `SQL/XML`和`XML`支持；
- `Wrapper Pattern`；
- `SQLException`增强；
- `Connection`和`Statement`接口增强；
- `New Scalar Funtions`；
- `JDBC API changes`。

## 十一、值得关注的

### 1. 集合框架增强

Jdk6中的集合框架的API更改数量要少于JDK5，更多地关注了规范的准确性和清晰度。即使在编写旧版本的程序时，我们也建议使用Java SE 6规范。
API更改的主要主题是更好的双向收集访问。

新增了以下几个接口：

- `Deque`: 双端队列接口，继承了Queue接口，队列两头都可以实现入队和出队。
- `BlockingDeque`: 双端阻塞队列接口，继承了BlockingQueue、Deque接口。
- `NavigableSet`: 可导航Set接口，继承自SortedSet接口。
- `NavigableMap`: 可导航Map接口，继承自SortedMap接口。
- `ConcurrentNavigableMap`: 支持并发的可导航Map，继承自`ConcurrentMap`接口和`NavigableMap`接口。

新增了以下几个实现类：

- `ArrayDeque`: 底层采用了循环数组的方式来完成双端队列的实现，无限扩展且可选容量。Java已不推荐使用Stack，而是推荐使用更高效的`ArrayDeque`来实现栈的功能，非线程安全。
- `ConcurrentSkipListSet`: 底层使用跳跃列表来实现，适用于高并发的场景，内部使用了ConcurrentNavigableMap，同TreeSet功能相似，线程安全。
- `ConcurrentSkipListMap`: 底层使用跳跃列表来实现，适用于高并发的场景，内部使用了ConcurrentNavigableMap，同TreeMap功能相似，是一个并发的、可排序的Map，线程安全。因此它可以在多线程环境中弥补ConcurrentHashMap不支持排序的问题。
- `LinkedBlockingDeque`: 底层采用了双向链表实现的双端阻塞并发队列，无限扩展且可选容量。该阻塞队列同时支持FIFO和FILO两种操作方式，即可以从队列的头和尾同时操作(插入/删除)，且线程安全。
- `AbstractMap.SimpleEntry`: `Map.Entry`的简单可变实现。
- `AbstractMap.SimpleImmutableEntry`: `Map.Entry`的简单不可变实现。

以下的类已经被改进来用来实现新的接口：

- `LinkedList`: 改进以实现Deque接口。
- `TreeSet`: 改进以实现NavigableSet接口。
- `TreeMap`: 改进以实现NavigableMap接口。

新增了两个新的方法到`Collections`的工具类中：

- `newSetFromMap(Map)`: 从通用的Map实现中创建一个通用的Set实现。Java集合中有`IdentityHashMap`，但是没有`IdentityHashSet`类，我们可以通过这样的方式来实现：

```java
Set<Object> identityHashSet = Collections.newSetFromMap(new IdentityHashMap<Object, Boolean>());
```

- `asLifoQueue(Deque)`: 通过传入`Deque`得到一个后进先出(LIFO)的队列。

现在`Arrays`工具类，具有`copyOf`和`copyOfRange`方法，可以有效地调整，截断或复制所有类型的数组的子数组。

以前是这样实现的：

```java
int[] newArray = new int[newLength];
System.arraycopy(oldArray, 0, newArray, 0, oldArray.length);
```

现在可以这样实现：

```java
int[] newArray = Arrays.copyOf(a, newLength);
```

---

参考文档：

-[JavaSE6 Features and Enhancements](http://www.oracle.com/technetwork/java/javase/features-141434.html)
-[Java6的新特性](https://segmentfault.com/a/1190000004417536)
-[chinajash](http://my.csdn.net/Chinajash)