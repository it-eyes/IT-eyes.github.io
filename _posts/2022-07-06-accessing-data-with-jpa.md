---
layout: post
title:  "使用JPA访问数据"
date:   2022-07-06
excerpt: "本指南将引导您完成构建应用程序的过程，该应用程序使用 Spring Data JPA 在关系数据库中存储和检索数据。"
tag:
- spring 
- java
comments: true
---

本指南将引导您完成构建应用程序的过程，该应用程序使用 Spring Data JPA 在关系数据库中存储和检索数据。

## 你将建造什么
您将构建一个将`Customer` POJO（普通旧 Java 对象）存储在基于内存的数据库中的应用程序。


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

* [下载](https://github.com/spring-guides/gs-accessing-data-jpa/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-accessing-data-jpa.git`
* cd 进入 `gs-accessing-data-jpa/initial`
* 跳转到定义一个简单的实体。

完成后，您可以对照 `gs-accessing-data-jpa/complete` 中的代码检查结果。

## 从SpringInitializr开始
您可以使用这个预先初始化的项目并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring Data JPA**和**H2 Database**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。

## 定义一个简单的实体
在此示例中，您存储 `Customer` 对象，每个对象都被注释为 JPA 实体。 以下清单显示了 Customer 类（在 `src/main/java/com/example/accessingdatajpa/Customer.java` 中）：
{% highlight java %}
package com.example.accessingdatajpa;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Customer {

  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Long id;
  private String firstName;
  private String lastName;

  protected Customer() {}

  public Customer(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return String.format(
        "Customer[id=%d, firstName='%s', lastName='%s']",
        id, firstName, lastName);
  }

  public Long getId() {
    return id;
  }

  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }
}
{% endhighlight %}

在这里，您有一个具有三个属性的 `Customer` 类：`id`、`firstName` 和 `lastName`。您还有两个构造函数。默认构造函数的存在只是为了 JPA。您不直接使用它，因此它被指定为`protected`。另一个构造函数是用于创建要保存到数据库的 `Customer` 实例的构造函数。

`Customer` 类使用`@Entity` 注解，表明它是一个 JPA 实体。 （因为不存在`@Table` 注解，所以假设该实体映射到名为 `Customer` 的表。）

`Customer` 对象的 `id` 属性使用 `@Id` 注释，以便 JPA 将其识别为对象的 ID。 `id` 属性还使用`@GeneratedValue` 进行注释，以指示应自动生成 ID。

其他两个属性 `firstName` 和 `lastName` 没有注释。假设它们被映射到与属性本身共享相同名称的列。

方便的 `toString()` 方法打印出客户的属性。

## 创建简单查询
Spring Data JPA 专注于使用 JPA 将数据存储在关系数据库中。它最引人注目的功能是能够在运行时从存储库接口自动创建存储库实现。

要了解它是如何工作的，请创建一个与`Customer`实体一起使用的存储库接口，如以下清单（在 `src/main/java/com/example/accessingdatajpa/CustomerRepository.java` 中）所示：
{% highlight java %}
package com.example.accessingdatajpa;

import java.util.List;

import org.springframework.data.repository.CrudRepository;

public interface CustomerRepository extends CrudRepository<Customer, Long> {

  List<Customer> findByLastName(String lastName);

  Customer findById(long id);
}
{% endhighlight %}
`CustomerRepository` 扩展了 `CrudRepository` 接口。它使用的实体类型和 ID，`Customer` 和 `Long`，在 `CrudRepository` 的通用参数中指定。通过扩展 `CrudRepository`，`CustomerRepository` 继承了几种使用 `Customer` 持久性的方法，包括保存、删除和查找 `Customer` 实体的方法。

Spring Data JPA 还允许您通过声明方法签名来定义其他查询方法。例如，`CustomerRepository` 包含 `findByLastName()` 方法。

在典型的 Java 应用程序中，您可能希望编写一个实现 `CustomerRepository` 的类。然而，这正是 Spring Data JPA 如此强大的原因：您无需编写存储库接口的实现。 Spring Data JPA 在您运行应用程序时创建一个实现。

现在你可以连接这个例子，看看它是什么样子的！

## 创建应用程序类
Spring Initializr 为应用程序创建一个简单的类。以下清单显示了 Initializr 为此示例创建的类（在 `src/main/java/com/example/accessingdatajpa/AccessingDataJpaApplication.java` 中）：
{% highlight java %}
package com.example.accessingdatajpa;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataJpaApplication {

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataJpaApplication.class, args);
  }

}
{% endhighlight %}
`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：
* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

现在您需要修改 Initializr 为您创建的简单类。要获得输出（在本例中为控制台），您需要设置一个记录器。然后您需要设置一些数据并使用它来生成输出。以下清单显示了完成的 `AccessingDataJpaApplication` 类（在 `src/main/java/com/example/accessingdatajpa/AccessingDataJpaApplication.java` 中）：
{% highlight java %}
package com.example.accessingdatajpa;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class AccessingDataJpaApplication {

  private static final Logger log = LoggerFactory.getLogger(AccessingDataJpaApplication.class);

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataJpaApplication.class);
  }

  @Bean
  public CommandLineRunner demo(CustomerRepository repository) {
    return (args) -> {
      // save a few customers
      repository.save(new Customer("Jack", "Bauer"));
      repository.save(new Customer("Chloe", "O'Brian"));
      repository.save(new Customer("Kim", "Bauer"));
      repository.save(new Customer("David", "Palmer"));
      repository.save(new Customer("Michelle", "Dessler"));

      // fetch all customers
      log.info("Customers found with findAll():");
      log.info("-------------------------------");
      for (Customer customer : repository.findAll()) {
        log.info(customer.toString());
      }
      log.info("");

      // fetch an individual customer by ID
      Customer customer = repository.findById(1L);
      log.info("Customer found with findById(1L):");
      log.info("--------------------------------");
      log.info(customer.toString());
      log.info("");

      // fetch customers by last name
      log.info("Customer found with findByLastName('Bauer'):");
      log.info("--------------------------------------------");
      repository.findByLastName("Bauer").forEach(bauer -> {
        log.info(bauer.toString());
      });
      // for (Customer bauer : repository.findByLastName("Bauer")) {
      //  log.info(bauer.toString());
      // }
      log.info("");
    };
  }

}
{% endhighlight %}
`AccessingDataJpaApplication` 类包含一个 `demo()` 方法，该方法使 `CustomerRepository` 通过一些测试。首先，它从 Spring 应用程序上下文中获取 `CustomerRepository`。然后它保存了一些 `Customer` 对象，演示了 `save()` 方法并设置了一些要使用的数据。接下来，它调用 `findAll()` 从数据库中获取所有 `Customer` 对象。然后它调用 `findById()` 通过其 ID 获取单个`Customer`。最后，它调用 `findByLastName()` 来查找姓氏为“Bauer”的所有客户。 `demo()` 方法返回一个 `CommandLineRunner` bean，它会在应用程序启动时自动运行代码。

>默认情况下，Spring Boot 启用 JPA 存储库支持并在 `@SpringBootApplication` 所在的包（及其子包）中查找。如果您的配置具有位于不可见包中的 JPA 存储库接口定义，则可以使用 `@EnableJpaRepositories` 及其类型安全的 `basePackageClasses=MyRepository.class` 参数指出备用包。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar build/libs/gs-accessing-data-jpa-0.1.0.jar
{% endhighlight %}
java -jar build/libs/gs-accessing-data-jpa-0.1.0.jar
如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight java %}
java -jar target/gs-accessing-data-jpa-0.1.0.jar
{% endhighlight %}
>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。
运行应用程序时，您应该会看到类似于以下内容的输出:
{% highlight java %}
== Customers found with findAll():
Customer[id=1, firstName='Jack', lastName='Bauer']
Customer[id=2, firstName='Chloe', lastName='O'Brian']
Customer[id=3, firstName='Kim', lastName='Bauer']
Customer[id=4, firstName='David', lastName='Palmer']
Customer[id=5, firstName='Michelle', lastName='Dessler']

== Customer found with findById(1L):
Customer[id=1, firstName='Jack', lastName='Bauer']

== Customer found with findByLastName('Bauer'):
Customer[id=1, firstName='Jack', lastName='Bauer']
Customer[id=3, firstName='Kim', lastName='Bauer']
{% endhighlight %}

## 概括
恭喜！ 您已经编写了一个简单的应用程序，该应用程序使用 Spring Data JPA 将对象保存到数据库并从数据库中获取它们，而无需编写具体的存储库实现。


更多详情请访问：[IT-eyes](https://it-eyes.top)