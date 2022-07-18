---
layout: post
title:  "使用 Neo4j 访问数据"
date:   2022-07-15
excerpt: "本指南将引导您完成使用 Spring Data Neo4j 构建应用程序的过程，该应用程序将数据存储在基于图形的数据库 Neo4j 中并从中检索数据。"
tag:
- spring 
- java
comments: true
---

本指南将引导您完成使用 [Spring Data Neo4j](https://projects.spring.io/spring-data-neo4j/) 构建应用程序的过程，该应用程序将数据存储在基于图形的数据库 [Neo4j](https://www.neo4j.com/) 中并从中检索数据。

## 你将建造什么
您将使用 Neo4j 的 [NoSQL](https://wikipedia.org/wiki/NoSQL) 基于图形的数据存储来构建嵌入式 Neo4j 服务器、存储实体和关系以及开发查询。

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

* [下载](https://github.com/spring-guides/gs-accessing-data-neo4j/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-accessing-data-neo4j.git`
* cd 进入 `gs-accessing-data-neo4j/initial`
* 跳转到定义一个简单的实体。

完成后，您可以对照 `gs-accessing-data-neo4j/complete` 中的代码检查结果。

## 从SpringInitializr开始
您可以使用这个预先初始化的项目并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring Data Neo4j**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。

## 建立 Neo4j 服务器
在构建此应用程序之前，您需要设置 Neo4j 服务器。

Neo4j 有一个开源服务器，您可以免费安装。

在安装了 Homebrew 的 Mac 上，运行以下命令：
{% highlight bash %}
$ brew install neo4j
{% endhighlight %}
有关其他选项，请访问 [https://neo4j.com/download/community-edition/](https://neo4j.com/download/community-edition/)。

安装后，通过运行以下命令以默认设置启动它：

{% highlight bash %}
$ neo4j start
{% endhighlight %}
您应该会看到类似于以下内容的输出：
{% highlight bash %}
Starting Neo4j.
Started neo4j (pid 96416). By default, it is available at http://localhost:7474/
There may be a short delay until the server is ready.
See /usr/local/Cellar/neo4j/3.0.6/libexec/logs/neo4j.log for current status.
{% endhighlight %}

默认情况下，Neo4j 的用户名和密码为 `neo4j` 和 `neo4j`。但是，它需要更改新的帐户密码。为此，请运行以下命令：

{% highlight bash %}
curl -v -u neo4j:neo4j POST localhost:7474/user/neo4j/password -H "Content-type:application/json" -d "{\"password\":\"secret\"}"
{% endhighlight %}
这会将密码从 `neo4j` 更改为 `secret` — 在生产中不要做的事情！完成该步骤后，您应该准备好运行本指南的其余部分。

## 定义一个简单的实体
Neo4j 捕获实体及其关系，这两个方面同等重要。想象一下，您正在为一个系统建模，您在其中存储每个人的记录。但是，您还想跟踪一个人的同事（本例中的`teammates`）。使用 Spring Data Neo4j，您可以使用一些简单的注释来捕获所有这些，如以下清单（在 `src/main/java/com/example/accessingdataneo4j/Person.java` 中）所示：
{% highlight java %}
package com.example.accessingdataneo4j;

import java.util.Collections;
import java.util.HashSet;
import java.util.Optional;
import java.util.Set;
import java.util.stream.Collectors;

import org.springframework.data.neo4j.core.schema.Id;
import org.springframework.data.neo4j.core.schema.Node;
import org.springframework.data.neo4j.core.schema.Property;
import org.springframework.data.neo4j.core.schema.Relationship;
import org.springframework.data.neo4j.core.schema.GeneratedValue;

@Node
public class Person {

  @Id @GeneratedValue private Long id;

  private String name;

  private Person() {
    // Empty constructor required as of Neo4j API 2.0.5
  };

  public Person(String name) {
    this.name = name;
  }

  /**
   * Neo4j doesn't REALLY have bi-directional relationships. It just means when querying
   * to ignore the direction of the relationship.
   * https://dzone.com/articles/modelling-data-neo4j
   */
  @Relationship(type = "TEAMMATE")
  public Set<Person> teammates;

  public void worksWith(Person person) {
    if (teammates == null) {
      teammates = new HashSet<>();
    }
    teammates.add(person);
  }

  public String toString() {

    return this.name + "'s teammates => "
      + Optional.ofNullable(this.teammates).orElse(
          Collections.emptySet()).stream()
            .map(Person::getName)
            .collect(Collectors.toList());
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}
{% endhighlight %}
这里有一个 `Person` 类，它只有一个属性：`name`。

`Person` 类使用`@NodeEntity` 注释。 Neo4j 存储它时，会创建一个新节点。这个类还有一个标记为`@GraphId` 的`id`。 Neo4j 在内部使用 `@GraphId` 来跟踪数据。

下一个重要的部分是`teammates`的集合。它是一个简单的 `Set<Person>`，但被标记为 `@Relationship`。这意味着这个集合的每个成员都应该作为一个单独的 `Person` 节点存在。请注意方向如何设置为 `UNDIRECTED`。这意味着当您查询 `TEAMMATE` 关系时，Spring Data Neo4j 会忽略关系的方向。

使用`worksWith()` 方法，您可以轻松地将人们联系在一起。

最后，你有一个方便的 `toString()` 方法来打印出这个人的名字和那个人的同事。

## 创建简单查询
Spring Data Neo4j 专注于在 Neo4j 中存储数据。但它继承了 Spring Data Commons 项目的功能，包括派生查询的能力。本质上，您不需要学习 Neo4j 的查询语言。相反，您可以编写一些方法并让查询为您编写。

要了解它是如何工作的，请创建一个查询 `Person` 节点的接口。以下清单（在 `src/main/java/com/example/accessingdataneo4j/PersonRepository.java` 中）显示了这样一个查询：
{% highlight java %}
package com.example.accessingdataneo4j;

import java.util.List;
import org.springframework.data.neo4j.repository.Neo4jRepository;

public interface PersonRepository extends Neo4jRepository<Person, Long> {

  Person findByName(String name);
  List<Person> findByTeammatesName(String name);
}
{% endhighlight %}
`PersonRepository` 扩展了 `Neo4jRepository` 接口并插入了它操作的类型：`Person`。 该接口带有许多操作，包括标准的 CRUD（创建、读取、更新和删除）操作。

但是您可以通过声明它们的方法签名来定义其他查询。 在本例中，您添加了 `findByName`，它查找 `Person` 类型的节点并找到与 `name` 匹配的节点。 您还有 `findByTeamatesName`，它会查找 `Person` 节点，深入到`teammates`字段的每个条目，并根据teammates的`names`进行匹配。

## 访问 Neo4j 的权限
Neo4j 社区版需要凭据才能访问它。 您可以通过设置几个属性（在 `src/main/resources/application.properties` 中）来配置这些凭证，如以下清单所示：
{% highlight perperties %}
spring.neo4j.uri=bolt://localhost:7687
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=secret
{% endhighlight %}
这包括默认用户名 (`neo4j`) 和我们之前选择的新设置的密码 (`secret`)。

>不要将真实凭据存储在您的源存储库中。 相反，使用 [Spring Boot 的属性覆盖](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-external-config)在运行时配置它们。

有了这个，你可以把它连接起来，看看它是什么样子的！

## 创建应用程序类
Spring Initializr 为应用程序创建一个简单的类。 以下清单显示了 Initializr 为此示例创建的类（在 `src/main/java/com/example/accessingdataneo4j/AccessingDataNeo4jApplication.java` 中）：
{% highlight java %}
package com.example.accessingdataneo4j;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataNeo4jApplication {

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataNeo4jApplication.class, args);
  }

}
{% endhighlight %}
`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：
* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

只要它们包含在 `@SpringBootApplication` 类的同一个包（或子包）中，Spring Boot 就会自动处理这些存储库。要对注册过程进行更多控制，您可以使用 `@EnableNeo4jRepositories` 注释。

>默认情况下，`@EnableNeo4jRepositories` 会扫描当前包以查找扩展 Spring Data 存储库接口之一的任何接口。如果您的项目布局有多个项目并且找不到您的存储库，您可以使用它的 `basePackageClasses=MyRepository.class` 安全地告诉 Spring Data Neo4j 按类型扫描不同的根包。

显示记录输出。该服务应在几秒钟内启动并运行。

现在自动装配您之前定义的 `PersonRepository` 实例。 Spring Data Neo4j 动态实现该接口并插入所需的查询代码以满足接口的义务。

`main` 方法使用 Spring Boot 的 `SpringApplication.run()` 来启动应用程序并调用构建关系的 `CommandLineRunner`。

在本例中，您将创建三个本地 `Person` 实例：Greg、Roy 和 Craig。最初，它们只存在于内存中。请注意，没有人是任何人的队友（目前）。

起初，你找到 Greg，表明他与 Roy 和 Craig 合作，然后再次坚持他。请记住，队友关系被标记为 `UNDIRECTED`（即双向）。这意味着 Roy 和 Craig 也已更新。

这就是为什么当您需要更新 Roy 时。首先从 Neo4j 获取该记录至关重要。在将 Craig 添加到列表之前，您需要了解 Roy 队友的最新状态。

为什么没有代码可以获取 Craig 并添加任何关系？因为你已经拥有了！格雷格早些时候将克雷格标记为队友，罗伊也是如此。这意味着无需再次更新 Craig 的关系。当您遍历每个团队成员并将他们的信息打印到控制台时，您可以看到它。

最后，查看您向后看的其他查询，回答“谁与谁一起工作？”的问题。

以下清单显示了完成的 `AccessingDataNeo4jApplication` 类（位于 `src/main/java/com/example/accessingdataneo4j/AccessingDataNeo4jApplication.java`）：
{% highlight bash %}
package com.example.accessingdataneo4j;

import java.util.Arrays;
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.neo4j.repository.config.EnableNeo4jRepositories;

@SpringBootApplication
@EnableNeo4jRepositories
public class AccessingDataNeo4jApplication {

	private final static Logger log = LoggerFactory.getLogger(AccessingDataNeo4jApplication.class);

	public static void main(String[] args) throws Exception {
		SpringApplication.run(AccessingDataNeo4jApplication.class, args);
		System.exit(0);
	}

	@Bean
	CommandLineRunner demo(PersonRepository personRepository) {
		return args -> {

			personRepository.deleteAll();

			Person greg = new Person("Greg");
			Person roy = new Person("Roy");
			Person craig = new Person("Craig");

			List<Person> team = Arrays.asList(greg, roy, craig);

			log.info("Before linking up with Neo4j...");

			team.stream().forEach(person -> log.info("\t" + person.toString()));

			personRepository.save(greg);
			personRepository.save(roy);
			personRepository.save(craig);

			greg = personRepository.findByName(greg.getName());
			greg.worksWith(roy);
			greg.worksWith(craig);
			personRepository.save(greg);

			roy = personRepository.findByName(roy.getName());
			roy.worksWith(craig);
			// We already know that roy works with greg
			personRepository.save(roy);

			// We already know craig works with roy and greg

			log.info("Lookup each person by name...");
			team.stream().forEach(person -> log.info(
					"\t" + personRepository.findByName(person.getName()).toString()));

			List<Person> teammates = personRepository.findByTeammatesName(greg.getName());
			log.info("The following have Greg as a teammate...");
			teammates.stream().forEach(person -> log.info("\t" + person.getName()));
		};
	}

}
{% endhighlight %}
## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：

{% highlight bash %}
java -jar build/libs/gs-accessing-data-neo4j-0.1.0.jar
{% endhighlight %}
如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：

{% highlight bash %}
java -jar target/gs-accessing-data-neo4j-0.1.0.jar
{% endhighlight %}
>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

您应该会看到类似于以下列表的内容（还有其他内容，例如查询）：

{% highlight bash %}
Before linking up with Neo4j...
	Greg's teammates => []
	Roy's teammates => []
	Craig's teammates => []

Lookup each person by name...
	Greg's teammates => [Roy, Craig]
	Roy's teammates => [Greg, Craig]
	Craig's teammates => [Roy, Greg]
{% endhighlight %}
您可以从输出中看到（最初）没有人通过任何关系连接。然后，在您添加人员后，他们被捆绑在一起。最后，您可以看到根据队友查找人员的便捷查询。

## 概括
恭喜！您刚刚设置了一个嵌入式 Neo4j 服务器，存储了一些简单的相关实体，并开发了一些快速查询。


更多详情请访问：[IT-eyes](https://it-eyes.top)