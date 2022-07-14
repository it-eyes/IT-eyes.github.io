---
layout: post
title:  "使用 Spring Boot 构建应用程序"
date:   2022-06-23
excerpt: "本指南提供了 Spring Boot 如何帮助您加速应用程序开发的示例。 随着您阅读更多 Spring 入门指南，您将看到更多 Spring Boot 用例。 本指南旨在让您快速了解 Spring Boot。 如果您想创建自己的基于 Spring Boot 的项目，请访问 Spring Initializr，填写您的项目详细信息，选择您的选项，然后将捆绑的项目下载为 zip 文件。"
tag:
- spring 
- java
comments: true
---

本指南提供了 [Spring Boot](https://github.com/spring-projects/spring-boot) 如何帮助您加速应用程序开发的示例。 随着您阅读更多 Spring 入门指南，您将看到更多 Spring Boot 用例。 本指南旨在让您快速了解 Spring Boot。 如果您想创建自己的基于 Spring Boot 的项目，请访问 [Spring Initializr](https://start.spring.io/)，填写您的项目详细信息，选择您的选项，然后将捆绑的项目下载为 zip 文件。

## 你将建造什么
您将使用 Spring Boot 构建一个简单的 Web 应用程序，并向其中添加一些有用的服务。

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

## 了解使用 Spring Boot 可以做什么
Spring Boot 提供了一种快速构建应用程序的方法。它查看您的类路径和您已配置的 bean，对您缺少的内容做出合理的假设，然后添加这些项目。使用 Spring Boot，您可以更多地关注业务功能，而不是基础设施。

以下示例展示了 Spring Boot 可以为您做什么：

* Spring MVC 在类路径上吗？您几乎总是需要几个特定的​​ bean，Spring Boot 会自动添加它们。 Spring MVC 应用程序还需要一个 servlet 容器，因此 Spring Boot 会自动配置嵌入式 Tomcat。
* Jetty 在类路径上吗？如果是这样，您可能不想要 Tomcat，而是想要嵌入式 Jetty。 Spring Boot 会为您处理这些问题。
* Thymeleaf 在类路径上吗？如果是这样，则必须始终将一些 bean 添加到您的应用程序上下文中。 Spring Boot 会为您添加它们。

这些只是 Spring Boot 提供的自动配置的几个示例。同时，Spring Boot 不会妨碍您。例如，如果 Thymeleaf 在您的路径上，Spring Boot 会自动将 `SpringTemplateEngine `添加到您的应用程序上下文中。但是如果你用自己的设置定义了自己的 `SpringTemplateEngine`，Spring Boot 是不会加一个的。这使您无需付出任何努力即可控制。

>Spring Boot 不会生成代码或对文件进行编辑。 相反，当您启动应用程序时，Spring Boot 会动态连接 bean 和设置并将它们应用于您的应用程序上下文。

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

## 创建一个简单的 Web 应用程序
现在您可以为一个简单的 Web 应用程序创建一个 Web 控制器，如以下清单（来自 `src/main/java/com/example/spring boot/Hello Controller.java`）所示：

{% highlight java %}
package com.example.springboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	@GetMapping("/")
	public String index() {
		return "Greetings from Spring Boot!";
	}

}
{% endhighlight %}
该类被标记为`@RestController`，这意味着它已准备好供 Spring MVC 使用来处理 Web 请求。`@GetMapping` 将 `/` 映射到 `index()` 方法。 当从浏览器调用或在命令行上使用 curl 时，该方法返回纯文本。 这是因为`@RestController` 结合了`@Controller` 和`@ResponseBody`，这两个注释会导致Web 请求返回数据而不是视图。

## 创建一个应用程序类
Spring Initializr 为您创建了一个简单的应用程序类。 但是，在这种情况下，它太简单了。 您需要修改应用程序类以匹配以下清单（来自 `src/main/java/com/example/springboot/Application.java`）：
{% highlight java %}
package com.example.springboot;

import java.util.Arrays;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Bean
	public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
		return args -> {

			System.out.println("Let's inspect the beans provided by Spring Boot:");

			String[] beanNames = ctx.getBeanDefinitionNames();
			Arrays.sort(beanNames);
			for (String beanName : beanNames) {
				System.out.println(beanName);
			}

		};
	}

}
{% endhighlight %}

`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：

* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

还有一个标记为`@Bean` 的 `CommandLineRunner` 方法，它在启动时运行。它检索由您的应用程序创建或由 Spring Boot 自动添加的所有 bean。它对它们进行分类并打印出来。

## 运行应用程序
要运行应用程序，请在终端窗口（`complete`）目录中运行以下命令：



{% highlight bash %}
./gradlew bootRun
{% endhighlight %}
如果您使用 Maven，请在终端窗口（`complete`）目录中运行以下命令：

{% highlight bash %}
./mvnw spring-boot:run
{% endhighlight %}

您应该会看到类似于以下内容的输出：
{% highlight bash %}
Let's inspect the beans provided by Spring Boot:
application
beanNameHandlerMapping
defaultServletHandlerMapping
dispatcherServlet
embeddedServletContainerCustomizerBeanPostProcessor
handlerExceptionResolver
helloController
httpRequestHandlerAdapter
messageSource
mvcContentNegotiationManager
mvcConversionService
mvcValidator
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$DispatcherServletConfiguration
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration$EmbeddedTomcat
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration
org.springframework.boot.context.embedded.properties.ServerProperties
org.springframework.context.annotation.ConfigurationClassPostProcessor.enhancedConfigurationProcessor
org.springframework.context.annotation.ConfigurationClassPostProcessor.importAwareProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.web.servlet.config.annotation.DelegatingWebMvcConfiguration
propertySourcesBinder
propertySourcesPlaceholderConfigurer
requestMappingHandlerAdapter
requestMappingHandlerMapping
resourceHandlerMapping
simpleControllerHandlerAdapter
tomcatEmbeddedServletContainerFactory
viewControllerHandlerMapping
{% endhighlight %}

您可以清楚地看到 `org.springframework.boot.autoconfigure` bean。 还有一个`tomcatEmbeddedServletContainerFactory`。

现在使用 curl 运行服务（在单独的终端窗口中），通过运行以下命令（显示其输出）：

{% highlight bash %}
$ curl localhost:8080
Greetings from Spring Boot!
{% endhighlight %}

## 添加单元测试
您将希望为您添加的端点添加一个测试，而 Spring Test 为此提供了一些机制。

如果您使用 Gradle，请将以下依赖项添加到您的 `build.gradle` 文件中：
{% highlight bash %}
testImplementation('org.springframework.boot:spring-boot-starter-test')
{% endhighlight %}

如果您使用 Maven，请将以下内容添加到您的 `pom.xml` 文件中：
{% highlight xml %}
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
{% endhighlight %}

现在编写一个简单的单元测试，通过端点模拟 servlet 请求和响应，如以下清单（来自 `src/test/java/com/example/springboot/HelloControllerTest.java`）所示：
{% highlight java %}
package com.example.springboot;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest {

	@Autowired
	private MockMvc mvc;

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("Greetings from Spring Boot!")));
	}
}
{% endhighlight %}
`MockMvc` 来自 Spring Test，它允许您通过一组方便的构建器类将 HTTP 请求发送到 `DispatcherServlet` 并对结果进行断言。 注意使用 `@AutoConfigureMockMvc` 和 `@SpringBootTest` 来注入 `MockMvc` 实例。 使用了`@SpringBootTest`，我们要求创建整个应用程序上下文。 另一种方法是要求 Spring Boot 使用 `@WebMvcTest` 仅创建上下文的 Web 层。 在任何一种情况下，Spring Boot 都会自动尝试定位应用程序的主应用程序类，但如果您想构建不同的东西，您可以覆盖它或缩小范围。

除了模拟 HTTP 请求周期外，还可以使用 Spring Boot 编写一个简单的全栈集成测试。 例如，我们可以创建以下测试（来自 `src/test/java/com/example/springboot/HelloControllerIT.java`），而不是（或同时）前面显示的模拟测试：
{% highlight java %}
package com.example.springboot;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class HelloControllerIT {

	@Autowired
	private TestRestTemplate template;

    @Test
    public void getHello() throws Exception {
        ResponseEntity<String> response = template.getForEntity("/", String.class);
        assertThat(response.getBody()).isEqualTo("Greetings from Spring Boot!");
    }
}
{% endhighlight %}
由于 `webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT`，嵌入式服务器在随机端口上启动，实际端口在 `TestRestTemplate` 的基本 URL 中自动配置。

## 添加生产级服务
如果您正在为您的企业构建网站，您可能需要添加一些管理服务。 Spring Boot 通过其执行器模块提供了几种此类服务（例如健康、审计、bean 等）。

如果您使用 Gradle，请将以下依赖项添加到您的 `build.gradle` 文件中：
{% highlight bash %}
implementation 'org.springframework.boot:spring-boot-starter-actuator'
{% endhighlight %}
如果您使用 Maven，请将以下依赖项添加到您的 `pom.xml` 文件中：
{% highlight xml %}
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
{% endhighlight %}
然后重新启动应用程序。 如果您使用 Gradle，请在终端窗口（complete目录）运行以下命令：
{% highlight bash %}
./gradlew bootRun
{% endhighlight %}
如果您使用 Maven，请在终端窗口（在完整目录中）运行以下命令：
{% highlight bash %}
./mvnw spring-boot:run
{% endhighlight %}
您应该会看到一组新的 RESTful 端点已添加到应用程序中。 这些是 Spring Boot 提供的管理服务。 以下清单显示了典型输出：
{% highlight bash %}
management.endpoint.configprops-org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointProperties
management.endpoint.env-org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointProperties
management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties
management.endpoint.logfile-org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointProperties
management.endpoints.jmx-org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointProperties
management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties
management.endpoints.web.cors-org.springframework.boot.actuate.autoconfigure.endpoint.web.CorsEndpointProperties
management.health.diskspace-org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthIndicatorProperties
management.info-org.springframework.boot.actuate.autoconfigure.info.InfoContributorProperties
management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties
management.metrics.export.simple-org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleProperties
management.server-org.springframework.boot.actuate.autoconfigure.web.server.ManagementServerProperties
{% endhighlight %}

执行器公开以下内容：
*actuator/health
*actuator

>还有一个 `/actuator/shutdown` 端点，但默认情况下，它仅通过 JMX 可见。 要将其作为 HTTP 端点启用，请将 management.endpoint.shutdown.enabled=true 添加到 `application.properties` 文件并使用 `management.endpoints.web.exposure.include=health,info,shutdown` 公开它。 但是，您可能不应该为公开可用的应用程序启用关闭端点。

您可以通过运行以下命令来检查应用程序的运行状况：
{% highlight bash %}
$ curl localhost:8080/actuator/health
{"status":"UP"}
{% endhighlight %}

因为我们没有启用它，所以请求的端点不可用（因为端点不存在）。

有关每个 REST 端点的更多详细信息以及如何使用 `application.properties` 文件（在 `src/main/resources` 中）调整它们的设置，请参阅有关端点的文档。

## 查看 Spring Boot 的 Starters
您已经看到了 Spring Boot 的一些“入门者”。 您可以在源代码中看到它们。

## JAR 支持和 Groovy 支持
最后一个示例展示了 Spring Boot 如何让您连接您可能不知道需要的 bean。 它还展示了如何打开便捷的管理服务。

然而，Spring Boot 做的远不止这些。 由于 Spring Boot 的加载器模块，它不仅支持传统的 WAR 文件部署，还允许您将可执行的 JAR 放在一起。 各种指南通过 `spring-boot-gradle-plugin` 和 `spring-boot-maven-plugin` 演示了这种双重支持。

最重要的是，Spring Boot 还支持 Groovy，让您只需一个文件即可构建 Spring MVC Web 应用程序。

创建一个名为 `app.groovy` 的新文件并将以下代码放入其中：
{% highlight java %}
@RestController
class ThisWillActuallyRun {

    @GetMapping("/")
    String home() {
        return "Hello, World!"
    }

}
{% endhighlight %}
文件在哪里并不重要。 您甚至可以在一条推文中放入这么小的应用程序！
接下来，安装 Spring Boot 的 CLI。

通过运行以下命令来运行 Groovy 应用程序：
{% highlight bash %}
$ spring run app.groovy
{% endhighlight %}

关闭之前的应用程序，以避免端口冲突。
从不同的终端窗口，运行以下 curl 命令（显示其输出）：
{% highlight bash %}
$ curl localhost:8080
Hello, World!
{% endhighlight %}

Spring Boot 通过向代码动态添加关键注释并使用 Groovy Grape 拉下使应用程序运行所需的库来实现这一点。

## 概括
恭喜！ 您使用 Spring Boot 构建了一个简单的 Web 应用程序，并了解了它如何加快您的开发速度。 您还打开了一些方便的制作服务。 这只是 Spring Boot 可以做的一小部分。 有关更多信息，请参阅 Spring Boot 的在线文档。


更多详情请访问：[IT-eyes](https://it-eyes.top)