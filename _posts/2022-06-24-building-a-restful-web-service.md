---
layout: post
title:  "构建 RESTful Web 服务"
date:   2022-06-23
excerpt: "本指南将引导您完成使用 Spring 创建“Hello, World”RESTful Web 服务的过程。"
tag:
- spring 
- java
comments: true
---

本指南将引导您完成使用 Spring 创建“Hello, World”RESTful Web 服务的过程。

## 你将建造什么
您将构建一个在 `http://localhost:8080/greeting` 处接受 HTTP GET 请求的服务。

它将以 JSON 表示的问候进行响应，如以下清单所示：

{% highlight bash %}
{"id":1,"content":"Hello, World!"}
{% endhighlight %}

您可以使用查询字符串中的可选`name`参数自定义问候语，如以下清单所示：
{% highlight bash %}
http://localhost:8080/greeting?name=User
{% endhighlight %}

`name` 参数值覆盖 `World` 的默认值并反映在响应中，如以下清单所示：
{% highlight bash %}
{"id":1,"content":"Hello, User!"}
{% endhighlight %}

## 你需要什么

* 约15分钟
* 最喜欢的文本编辑器或 IDE
* [JDK 1.8](https://www.oracle.com/java/technologies/downloads/) 或更高版本
* [Gradle 4+](http://www.gradle.org/downloads) 或[ Maven 3.2+](https://maven.apache.org/download.cgi)
* 您还可以将代码直接导入 IDE：
  * [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
  * [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)

## 如何完成本指南
像大多数 [Spring 入门指南](https://spring.io/guides)一样，您可以从头开始并完成每个步骤，也可以绕过您已经熟悉的基本设置步骤。 无论哪种方式，您最终都会得到工作代码。

要从头开始，请继续从 [Spring Initializr](##从SpringInitializr开始) 开始。

要跳过基础知识，请执行以下操作：

* [下载](https://github.com/spring-guides/gs-spring-boot/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-spring-boot.git`
* cd 进入 `gs-spring-boot/initial`
* 跳转到创建一个简单的 Web 应用程序。

完成后，您可以对照 `gs-spring-boot/complete` 中的代码检查结果。

## 从SpringInitializr开始
您可以使用[pre-initialized project](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.5.5&packaging=jar&jvmVersion=11&groupId=com.example&artifactId=spring-boot&name=spring-boot&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.spring-boot&dependencies=web)并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring Web**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。


## 创建资源表示类
现在您已经设置了项目和构建系统，您可以创建您的 Web 服务。

从考虑服务交互开始这个过程。

该服务将处理 `/greeting` 的 `GET` 请求，可以选择在查询字符串中使用`name`参数。 `GET` 请求应返回 `200 OK` 响应，其中包含表示问候的正文中的 JSON。 它应该类似于以下输出：
{% highlight bash %}
{
    "id": 1,
    "content": "Hello, World!"
}
{% endhighlight %}
`id` 字段是问候语的唯一标识符，`content`是问候语的文本表示。

要对问候表示建模，请创建一个资源表示类。 为此，请提供一个普通的旧 Java 对象，其中包含 `id` 和`content`数据的字段、构造函数和访问器，如以下清单（来自 `src/main/java/com/example/restservice/Greeting.java`）所示：
{% highlight java %}
package com.example.restservice;

public class Greeting {

	private final long id;
	private final String content;

	public Greeting(long id, String content) {
		this.id = id;
		this.content = content;
	}

	public long getId() {
		return id;
	}

	public String getContent() {
		return content;
	}
}
{% endhighlight %}
>此应用程序使用 Jackson JSON 库自动将 Greeting 类型的实例编组为 JSON。 网络启动器默认包含 Jackson。

## 创建资源控制器
在 Spring 构建 RESTful Web 服务的方法中，HTTP 请求由控制器处理。 这些组件由 `@RestController` 注释标识，下面清单中显示的 `GreetingController`（来自 `src/main/java/com/example/restservice/GreetingController.java`）通过返回 `Greeting` 的新实例来处理 `/greeting` 的 `GET` 请求类：
{% highlight java %}
package com.example.restservice;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
{% endhighlight %}

这个控制器简洁明了，但引擎盖下有很多事情要做。我们一步一步分解。

`@GetMapping` 注解确保对 `/greeting` 的 HTTP GET 请求映射到 `greeting()` 方法。

>其他 HTTP 动词有伴随注释（例如，`@PostMapping` 用于 POST）。还有一个 `@RequestMapping` 注释，它们都派生自，并且可以用作同义词（例如 `@RequestMapping(method=GET)`）。

`@RequestParam` 将查询字符串参数 `name` 的值绑定到 `greeting()` 方法的 `name` 参数中。如果请求中没有 `name` 参数，则使用 `World` 的 `defaultValue`。

方法体的实现基于来自`counter`的下一个值创建并返回具有 `id` 和 `content` 属性的新 `Greetin`g 对象，并使用greeting `templete`格式化给定`name`。

传统 MVC 控制器和前面显示的 RESTful Web 服务控制器之间的一个关键区别是 HTTP 响应主体的创建方式。这个 RESTful Web 服务控制器不依赖于视图技术将问候数据在服务器端呈现为 HTML，而是填充并返回一个 `Greeting` 对象。对象数据将作为 JSON 直接写入 HTTP 响应。

此代码使用 Spring `@RestController` 注释，它将类标记为控制器，其中每个方法都返回域对象而不是视图。它是同时包含`@Controller` 和`@ResponseBody` 的简写。

`Greeting` 对象必须转换为 JSON。由于 Spring 的 HTTP 消息转换器支持，您无需手动进行此转换。因为 [Jackson 2](https://github.com/FasterXML/jackson) 在类路径上，所以会自动选择 Spring 的 `MappingJackson2HttpMessageConverter` 将 `Greeting` 实例转换为 JSON。

`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：

* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar build/libs/gs-rest-service-0.1.0.jar
{% endhighlight %}
如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。 或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 JAR 文件，如下所示：
{% highlight bash %}
java -jar target/gs-rest-service-0.1.0.jar
{% endhighlight %}

>此处描述的步骤创建了一个可运行的 JAR。 您还可以构建经典的 WAR 文件。
显示记录输出。 该服务应在几秒钟内启动并运行。

## 测试服务
现在服务已经启动，访问 `http://localhost:8080/greeting`，你应该会看到：
{% highlight bash %}
{"id":1,"content":"Hello, World!"}
{% endhighlight %}

通过访问 `http://localhost:8080/greeting?name=User` 提供`name`查询字符串参数。 注意 `content` 属性的值是如何从 `Hello, World!`到`Hello,User!` 改变的，如以下清单所示：
{% highlight bash %}
{"id":2,"content":"Hello, User!"}
{% endhighlight %}

此更改表明 `GreetingController` 中的 `@RequestParam` 安排按预期工作。 `name` 参数已被赋予 `World` 的默认值，但可以通过查询字符串显式覆盖。

还要注意 `id` 属性如何从 `1` 更改为 `2`。这证明您正在处理多个请求中的同一个 `GreetingController` 实例，并且它的`counter`字段在每次调用时都按预期递增。

## 概括
恭喜！ 您刚刚使用 Spring 开发了一个 RESTful Web 服务。