---
layout: post
title:  "使用MongoDB访问数据"
date:   2022-06-25
excerpt: "本指南将引导您完成使用 Spring Data MongoDB 构建应用程序的过程，该应用程序将数据存储在基于文档的数据库 MongoDB 中并从中检索数据。"
tag:
- spring 
- java
comments: true
---
本指南将引导您完成使用 Spring Data MongoDB 构建应用程序的过程，该应用程序将数据存储在基于文档的数据库 MongoDB 中并从中检索数据。

## 你将建造什么
您将使用 Spring Data MongoDB 将 `Customer` POJO（普通旧 Java 对象）存储在 MongoDB 数据库中。

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

* [下载](https://github.com/spring-guides/gs-accessing-data-gemfire/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-accessing-data-mongodb.git`
* cd 进入 `gs-accessing-data-mongodb/initial`
* 跳转到[安装和启动 MongoDB](https://spring.io/guides/gs/accessing-data-mongodb/#initial)。

完成后，您可以对照 `gs-accessing-data-mongodb/complete` 中的代码检查结果。

## 从SpringInitializr开始
对于所有 Spring 应用程序，您应该从 Spring Initializr 开始。 Spring Initializr 提供了一种快速的方法来获取应用程序所需的所有依赖项，并为您完成大量设置。 此示例需要 Spring for Apache Geode 依赖项。

您可以使用`pre-initialized project`并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring Data MongoDB**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。

## 安装和启动 MongoDB
设置好项目后，您可以安装和启动 MongoDB 数据库。

如果您使用带有 Homebrew 的 Mac，则可以运行以下命令：
{% highlight bash %}
$ brew install mongodb
{% endhighlight %}
使用 MacPorts，您可以运行以下命令：
{% highlight bash %}
$ port install mongodb
{% endhighlight %}
对于其他具有包管理的系统，例如 Redhat、Ubuntu、Debian、CentOS 和 Windows，请参阅 [https://docs.mongodb.org/manual/installation/](https://docs.mongodb.org/manual/installation/) 中的说明。

安装 MongoDB 后，您可以通过运行以下命令在控制台窗口中启动它（这也会启动服务器进程）：
{% highlight bash %}
$ mongod
{% endhighlight %}
您应该会看到类似于以下内容的输出：
{% highlight bash %}
all output going to: /usr/local/var/log/mongodb/mongo.log
{% endhighlight %}

## 定义一个简单的实体
MongoDB 是一个 NoSQL 文档存储。 在此示例中，您存储 `Customer` 对象。 以下清单显示了 Customer 类（在 `src/main/java/com/example/accessingdatamongodb/Customer.java` 中）：
{% highlight java %}
package com.example.accessingdatamongodb;

import org.springframework.data.annotation.Id;


public class Customer {

  @Id
  public String id;

  public String firstName;
  public String lastName;

  public Customer() {}

  public Customer(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return String.format(
        "Customer[id=%s, firstName='%s', lastName='%s']",
        id, firstName, lastName);
  }

}
{% endhighlight %}
在这里，您有一个具有三个属性的 `Customer` 类：`id`、`firstName` 和 `lastName`。 `id` 主要供 MongoDB 内部使用。在创建新实例时，您还有一个构造函数来填充实体。

>在本指南中，为简洁起见，省略了典型的 getter 和 setter。
`id` 符合 MongoDB ID 的标准名称，因此它不需要任何特殊注释来为 Spring Data MongoDB 标记它。

其他两个属性 `firstName` 和 `lastName` 没有注释。假设它们被映射到与属性本身共享相同名称的字段。

方便的 `toString()` 方法打印出有关客户的详细信息。

>MongoDB 将数据存储在集合中。 Spring Data MongoDB 将 `Customer` 类映射到一个名为 `customer` 的集合中。如果要更改集合的名称，可以在类上使用 Spring Data MongoDB 的 `@Document` 注释。

## 创建简单查询
Spring Data MongoDB 专注于在 MongoDB 中存储数据。它还继承了 Spring Data Commons 项目的功能，例如派生查询的能力。本质上，您不需要学习 MongoDB 的查询语言。您可以编写一些方法，并为您编写查询。

要查看其工作原理，请创建一个查询`Customer`文档的存储库接口，如以下清单（在 `src/main/java/com/example/accessingdatamongodb/CustomerRepository.java` 中）所示：
{% highlight java %}
package com.example.accessingdatamongodb;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;

public interface CustomerRepository extends MongoRepository<Customer, String> {

  public Customer findByFirstName(String firstName);
  public List<Customer> findByLastName(String lastName);

}
{% endhighlight %}
`CustomerRepository` 扩展了 `MongoRepository` 接口并插入了它使用的值类型和 ID：分别是 `Customer` 和 `String`。该接口带有许多操作，包括标准的 CRUD 操作（创建、读取、更新和删除）。

您可以通过声明其方法签名来定义其他查询。在这种情况下，添加 `findByFirstName`，它实质上是查找 `Customer` 类型的文档并查找与 `firstName` 匹配的文档。

您还有 `findByLastName`，它按last name查找人员列表。

在典型的 Java 应用程序中，您编写一个实现 `CustomerRepository` 的类并自己制作查询。 Spring Data MongoDB 如此有用的原因在于您不需要创建此实现。 Spring Data MongoDB 在您运行应用程序时动态创建它。

现在您可以连接此应用程序并查看它的外观！

## 创建应用程序类
Spring Initializr 为应用程序创建一个简单的类。以下清单显示了 Initializr 为此示例创建的类（在 `src/main/java/com/example/accessingdatamongodb/AccessingDataMongodbApplication.java` 中）：
{% highlight java %}
package com.example.accessingdatamongodb;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataMongodbApplication {

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataMongodbApplication.class, args);
  }

}
{% endhighlight %}
`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：
* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

只要它们包含在 `@SpringBootApplication` 类的同一个包（或子包）中，Spring Boot 就会自动处理这些存储库。要对注册过程进行更多控制，您可以使用 `@EnableMongoRepositories` 注释。

>默认情况下，`@EnableMongoRepositories` 会扫描当前包以查找扩展 Spring Data 存储库接口之一的任何接口。如果您的项目布局有多个项目并且找不到您的存储库，您可以使用它的 `basePackageClasses=MyRepository.class` 安全地告诉 Spring Data MongoDB 按类型扫描不同的根包。
Spring Data MongoDB 使用 `MongoTemplate` 执行 `find*` 方法后面的查询。您可以自己使用该模板进行更复杂的查询，但本指南不涵盖这一点。 （参见 [Spring Data MongoDB 参考指南](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/)）

现在您需要修改 Initializr 为您创建的简单类。您需要设置一些数据并使用它来生成输出。以下清单显示了完成的 `AccessingDataMongodbApplication` 类（在 `src/main/java/com/example/accessingdatamongodb/AccessingDataMongodbApplication.java` 中）：

{% highlight java %}
package com.example.accessingdatamongodb;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataMongodbApplication implements CommandLineRunner {

  @Autowired
  private CustomerRepository repository;

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataMongodbApplication.class, args);
  }

  @Override
  public void run(String... args) throws Exception {

    repository.deleteAll();

    // save a couple of customers
    repository.save(new Customer("Alice", "Smith"));
    repository.save(new Customer("Bob", "Smith"));

    // fetch all customers
    System.out.println("Customers found with findAll():");
    System.out.println("-------------------------------");
    for (Customer customer : repository.findAll()) {
      System.out.println(customer);
    }
    System.out.println();

    // fetch an individual customer
    System.out.println("Customer found with findByFirstName('Alice'):");
    System.out.println("--------------------------------");
    System.out.println(repository.findByFirstName("Alice"));

    System.out.println("Customers found with findByLastName('Smith'):");
    System.out.println("--------------------------------");
    for (Customer customer : repository.findByLastName("Smith")) {
      System.out.println(customer);
    }

  }

}
{% endhighlight %}

`AccessingDataMongodbApplication` 包括一个自动装配 `CustomerRepository` 实例的 `main()` 方法。 Spring Data MongoDB 动态创建一个代理并将其注入那里。我们通过一些测试使用 `CustomerRepository`。首先，它保存了一些 `Customer` 对象，演示了 `save()` 方法并设置了一些要使用的数据。接下来，它调用 `findAll()` 从数据库中获取所有 `Customer` 对象。然后它调用 `findByFirstName()` 以获取单个`Customer`的名字。最后，它调用 `findByLastName()` 来查找姓氏为 `Smith` 的所有客户。

>默认情况下，Spring Boot 尝试连接到本地托管的 MongoDB 实例。阅读[参考文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-mongodb)以获取有关将应用程序指向其他地方托管的 MongoDB 实例的详细信息。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar build/libs/gs-accessing-data-mongodb-0.1.0.jar
{% endhighlight %}
如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar 目标/gs-accessing-data-mongodb-0.1.0.jar
{% endhighlight %}

>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

由于 `AccessingDataMongodbApplication` 实现了 `CommandLineRunner`，所以在 Spring Boot 启动时会自动调用 run 方法。您应该会看到类似以下的内容（还有其他输出，例如查询）：

{% highlight bash %}
== Customers found with findAll():
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']

== Customer found with findByFirstName('Alice'):
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
== Customers found with findByLastName('Smith'):
Customer[id=51df1b0a3004cb49c50210f8, firstName='Alice', lastName='Smith']
Customer[id=51df1b0a3004cb49c50210f9, firstName='Bob', lastName='Smith']
{% endhighlight %}

## 概括
恭喜！ 您设置了一个 MongoDB 服务器并编写了一个简单的应用程序，该应用程序使用 Spring Data MongoDB 将对象保存到数据库并从数据库中获取它们，所有这些都没有编写具体的存储库实现。

如果您想轻松地使用基于超媒体的 RESTful 前端公开 MongoDB 存储库，请阅读[使用 REST 访问 MongoDB 数据](https://spring.io/guides/gs/accessing-mongodb-data-rest)。


更多详情请访问：[IT-eyes](https://it-eyes.top)