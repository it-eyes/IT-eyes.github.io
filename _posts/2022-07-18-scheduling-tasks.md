---
layout: post
title:  "@Scheduled调度任务"
date:   2022-07-18
excerpt: "本指南将引导您完成使用 Spring 安排任务的步骤。"
tag:
- spring 
- java
comments: true
---
本指南将引导您完成使用 Spring 安排任务的步骤。

## 你将建造什么
您将使用 Spring 的 `@Scheduled` 注解构建一个每五秒打印一次当前时间的应用程序。

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

* [下载](https://github.com/spring-guides/gs-scheduling-tasks/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-scheduling-tasks.git`
* cd 进入 `gs-scheduling-tasks/initial`
* 跳转到创建调度任务。

完成后，您可以对照 `gs-scheduling-tasks/complete` 中的代码检查结果。

## 从SpringInitializr开始
您可以使用这个预先初始化的项目并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Generate**。
4. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。


## 添加`awaitility`依赖
`complete/src/test/java/com/example/schedulingtasks/ScheduledTasksTest.java` 中的测试需要`awaitility`库。

> 更高版本的 `awaitility` 库不适用于此测试，因此您必须指定版本 3.1.2。
> 
要将`awaitility`库添加到 Maven，请添加以下依赖项：
{% highlight xml %}
<dependency>
  <groupId>org.awaitility</groupId>
  <artifactId>awaitility</artifactId>
  <version>3.1.2</version>
  <scope>test</scope>
</dependency>
{% endhighlight %}
以下清单显示了完成的 `pom.xml` 文件：
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.1</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>scheduling-tasks-complete</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>scheduling-tasks-complete</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.awaitility</groupId>
			<artifactId>awaitility</artifactId>
			<version>3.1.2</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
{% endhighlight %}
要将 `awaitility` 库添加到 Gradle，请添加以下依赖项：
{% highlight gradle %}
testImplementation 'org.awaitility:awaitility:3.1.2'
{% endhighlight %}
以下清单显示了完成的 `build.gradle` 文件：
{% highlight gradle %}
plugins {
	id 'org.springframework.boot' version '2.7.1'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.awaitility:awaitility:3.1.2'
	testImplementation('org.springframework.boot:spring-boot-starter-test')
}

test {
	useJUnitPlatform()
}
{% endhighlight %}

## 创建计划任务
现在您已经设置了项目，您可以创建计划任务。 以下清单（来自 `src/main/java/com/example/schedulingtasks/ScheduledTasks.java`）显示了如何执行此操作：
{% highlight java %}
/*
 * Copyright 2012-2015 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *	  https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.example.schedulingtasks;

import java.text.SimpleDateFormat;
import java.util.Date;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class ScheduledTasks {

	private static final Logger log = LoggerFactory.getLogger(ScheduledTasks.class);

	private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

	@Scheduled(fixedRate = 5000)
	public void reportCurrentTime() {
		log.info("The time is now {}", dateFormat.format(new Date()));
	}
}
{% endhighlight %}
`Scheduled` 注释定义了特定方法何时运行。

>此示例使用 `fixedRate`，它指定方法调用之间的间隔，从每次调用的开始时间开始测量。 还有[其他选项](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html#scheduling-annotation-support-scheduled)，例如 `fixedDelay`，它指定从任务完成开始测量的调用之间的间隔。 您还可以使用 `@Scheduled(cron=". . .")` 表达式进行更复杂的任务调度。

## 启用计划
虽然计划任务可以嵌入到 Web 应用程序和 WAR 文件中，但更简单的方法（显示在下一个清单中）会创建一个独立的应用程序。 为此，请将所有内容打包在一个可执行的 JAR 文件中，由一个很好的旧 Java `main()` 方法驱动。 以下清单（来自 `src/main/java/com/example/schedulingtasks/SchedulingTasksApplication.java`）显示了应用程序类：
{% highlight java %}
package com.example.schedulingtasks;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableScheduling
public class SchedulingTasksApplication {

	public static void main(String[] args) {
		SpringApplication.run(SchedulingTasksApplication.class);
	}
}
{% endhighlight %}
`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：
* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

`@EnableScheduling` 注解确保创建后台任务执行器。没有它，什么都不会安排。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：

{% highlight bash %}
java -jar build/libs/gs-scheduling-tasks-0.1.0.jar
{% endhighlight %}
如果您使用 Maven，则可以使用 ./mvnw spring-boot:run 运行应用程序。或者，您可以使用 ./mvnw clean package 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：

{% highlight bash %}
java -jar target/gs-scheduling-tasks-0.1.0.jar
{% endhighlight %}

>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

显示日志输出，您可以从日志中看到它在后台线程上。您应该看到您的计划任务每​​五秒触发一次。以下清单显示了典型输出：

{% highlight bash %}
...
2019-10-02 12:07:35.659  INFO 28617 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 12:07:35
2019-10-02 12:07:40.659  INFO 28617 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 12:07:40
2019-10-02 12:07:45.659  INFO 28617 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 12:07:45
2019-10-02 12:07:50.657  INFO 28617 --- [   scheduling-1] c.e.schedulingtasks.ScheduledTasks       : The time is now 12:07:50
...
{% endhighlight %}

## 概括
恭喜！ 您创建了一个带有计划任务的应用程序。 此外，此技术适用于任何类型的应用程序。


更多详情请访问：[IT-eyes](https://it-eyes.top)