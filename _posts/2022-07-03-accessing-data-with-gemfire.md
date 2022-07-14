---
layout: post
title:  "在 Pivotal GemFire 中访问数据"
date:   2022-07-03
excerpt: "本指南将引导您完成构建 Apache Geode 数据管理系统应用程序的过程。"
tag:
- spring 
- java
comments: true
---
本指南将引导您完成构建 Apache Geode 数据管理系统应用程序的过程。

## 你将建造什么
您将[使用 Spring Data for Apache Geode](https://spring.io/projects/spring-data-geode) 来存储和检索 POJO。

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

* [下载](https://github.com/spring-guides/gs-accessing-data-gemfire/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-accessing-data-gemfire.git`
* cd 进入 `gs-accessing-data-gemfire/initial`
* 跳转到定义一个简单的实体。

完成后，您可以对照 `gs-accessing-data-gemfire/complete` 中的代码检查结果。

## 从SpringInitializr开始
对于所有 Spring 应用程序，您应该从 Spring Initializr 开始。 Spring Initializr 提供了一种快速的方法来获取应用程序所需的所有依赖项，并为您完成大量设置。 此示例需要 Spring for Apache Geode 依赖项。

您可以使用`pre-initialized project`并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring for Apache Geode**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。

## 定义一个简单的实体
Apache Geode 是一种将数据映射到区域的内存中数据网格 (IMDG)。 您可以配置在集群中的多个节点之间分区和复制数据的分布式区域。 但是，在本指南中，我们使用 `LOCAL` 区域，因此您无需设置任何额外内容，例如整个服务器集群。

Apache Geode 是一个键/值存储，一个区域实现了 `java.util.concurrent.ConcurrentMap` 接口。 尽管您可以将区域视为 `java.util.Map`，但它比简单的 Java `Map` 要复杂得多，因为数据是在区域内分布、复制和管理的。

在此示例中，您仅使用少量注释将 `Person` 对象存储在 Apache Geode（一个区域）中。
`src/main/java/hello/Person.java`
{% highlight java %}
package hello;

import java.io.Serializable;

import org.springframework.data.annotation.Id;
import org.springframework.data.annotation.PersistenceConstructor;
import org.springframework.data.gemfire.mapping.annotation.Region;

import lombok.Getter;

@Region(value = "People")
public class Person implements Serializable {

  @Id
  @Getter
  private final String name;

  @Getter
  private final int age;

  @PersistenceConstructor
  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }

  @Override
  public String toString() {
    return String.format("%s is %d years old", getName(), getAge());
  }
}
{% endhighlight %}

这里有一个 `Person` 类，它有两个字段：`name`和`age`。在创建新实例时，您还有一个持久构造函数来填充实体。该类使用 [`Project Lombok`](https://projectlombok.org/) 来简化实现。

请注意，此类使用`@Region("People")` 进行注释。当 Apache Geode 存储此类的实例时，会在 `People` 区域内创建一个新条目。该类还用`@Id` 标记`name`字段。这表示用于识别和跟踪 Apache Geode 中的`Person`数据的标识符。本质上，`@Id` 注解的字段（如`name`）是键，而 `Person` 实例是键/值条目中的值。 Apache Geode 中没有自动生成密钥，因此您必须在将实体持久保存到 Apache Geode 之前设置 ID（`name`）。

下一个重要的部分是这个人的年龄。在本指南的后面部分，我们使用它来设计一些查询。

重写的 `toString()` 方法打印出人的姓名和年龄。

## 创建简单查询
Spring Data for Apache Geode 专注于使用 Spring 在 Apache Geode 中存储和访问数据。它还继承了 Spring Data Commons 项目的强大功能，例如派生查询的能力。本质上，您不需要学习 Apache Geode (OQL) 的查询语言。您可以编写一些方法，框架会为您编写查询。

要查看其工作原理，请创建一个接口来查询存储在 Apache Geode 中的 `Person` 对象：

`src/main/java/hello/PersonRepository.java`
{% highlight java %}
package hello;

import org.springframework.data.gemfire.repository.query.annotation.Trace;
import org.springframework.data.repository.CrudRepository;

public interface PersonRepository extends CrudRepository<Person, String> {

  @Trace
  Person findByName(String name);

  @Trace
  Iterable<Person> findByAgeGreaterThan(int age);

  @Trace
  Iterable<Person> findByAgeLessThan(int age);

  @Trace
  Iterable<Person> findByAgeGreaterThanAndAgeLessThan(int greaterThanAge, int lessThanAge);

}
{% endhighlight %}

`PersonRepository` 从 Spring Data Commons 扩展 `CrudRepository` 接口，并为 Repository 使用的 value 和 ID (key) 指定泛型类型参数的类型(分别为 `Person` 和 `String`)。 该接口带有许多操作，包括基本的 CRUD（创建、读取、更新、删除）和简单的查询数据访问操作（如 `findById(..)`）。

您可以根据需要通过声明其方法签名来定义其他查询。 在这种情况下，我们添加了 `findByName`，它实质上是搜索 `Person` 类型的对象并找到与 `name` 匹配的对象。

你还有：
* `findByAgeGreaterThan`：查找超过一定年龄的人
* `findByAgeLessThan`：查找特定年龄以下的人
* `findByAgeGreaterThanAndAgeLessThan`：查找特定年龄段的人

让我们把它连接起来，看看它是什么样子的！

## 创建应用程序类
以下示例创建一个包含所有组件的应用程序类：

`src/main/java/hello/Application.java`

{% highlight java %}
package hello;

import static java.util.Arrays.asList;
import static java.util.stream.StreamSupport.stream;

import org.apache.geode.cache.client.ClientRegionShortcut;

import org.springframework.boot.ApplicationRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.data.gemfire.config.annotation.ClientCacheApplication;
import org.springframework.data.gemfire.config.annotation.EnableEntityDefinedRegions;
import org.springframework.data.gemfire.repository.config.EnableGemfireRepositories;

@SpringBootApplication
@ClientCacheApplication(name = "AccessingDataGemFireApplication")
@EnableEntityDefinedRegions(
  basePackageClasses = Person.class,
  clientRegionShortcut = ClientRegionShortcut.LOCAL
)
@EnableGemfireRepositories
public class Application {

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

  @Bean
  ApplicationRunner run(PersonRepository personRepository) {

    return args -> {

      Person alice = new Person("Adult Alice", 40);
      Person bob = new Person("Baby Bob", 1);
      Person carol = new Person("Teen Carol", 13);

      System.out.println("Before accessing data in Apache Geode...");

      asList(alice, bob, carol).forEach(person -> System.out.println("\t" + person));

      System.out.println("Saving Alice, Bob and Carol to Pivotal GemFire...");

      personRepository.save(alice);
      personRepository.save(bob);
      personRepository.save(carol);

      System.out.println("Lookup each person by name...");

      asList(alice.getName(), bob.getName(), carol.getName())
        .forEach(name -> System.out.println("\t" + personRepository.findByName(name)));

      System.out.println("Query adults (over 18):");

      stream(personRepository.findByAgeGreaterThan(18).spliterator(), false)
        .forEach(person -> System.out.println("\t" + person));

      System.out.println("Query babies (less than 5):");

      stream(personRepository.findByAgeLessThan(5).spliterator(), false)
        .forEach(person -> System.out.println("\t" + person));

      System.out.println("Query teens (between 12 and 20):");

      stream(personRepository.findByAgeGreaterThanAndAgeLessThan(12, 20).spliterator(), false)
        .forEach(person -> System.out.println("\t" + person));
    };
  }
}
{% endhighlight %}

在配置中，您需要添加`@EnableGemfireRepositories` 注解。

* 默认情况下，`@EnableGemfireRepositories` 会扫描当前包以查找扩展 Spring Data 存储库接口之一的任何接口。您可以使用它的 `basePackageClasses = MyRepository.class` 安全地告诉 Spring Data for Apache Geode 按类型扫描不同的根包以获取特定于应用程序的存储库扩展。

需要包含一个或多个区域的 Apache Geode 缓存来存储所有数据。为此，您可以使用 Spring Data for Apache Geode 方便的基于配置的注释之一：`@ClientCacheApplication`、`@PeerCacheApplication` 或 `@CacheServerApplication`。

Apache Geode 支持不同的缓存拓扑，例如客户端/服务器、点对点 (p2p)，甚至是 WAN 安排。在 p2p 中，对等缓存实例嵌入在应用程序中，您的应用程序将能够作为对等缓存成员参与集群。但是，您的应用程序受到作为集群中对等成员的所有约束，因此这不像客户端/服务器拓扑那样常用。

在我们的例子中，我们使用`@ClientCacheApplication` 创建一个“客户端”缓存实例，它能够连接到服务器集群并与之通信。然而，为了简单起见，客户端使用`LOCAL`客户端区域在本地存储数据，无需设置或运行任何服务器。

现在，还记得您是如何使用 SDG 映射注释 `@Region("People")` 将 `Person` 标记为存储在名为 `People` 的区域中的吗？您可以在此处使用 `ClientRegionFactoryBean<String, Person>` bean 定义来定义该区域。您需要注入刚刚定义的缓存实例，同时将其命名为 `People`。

>Apache Geode 缓存实例（无论是对等方还是客户端）只是存储数据的区域容器。您可以将缓存视为 RDBMS 中的模式，将区域视为表。但是，缓存还执行其他管理功能来控制和管理您的所有区域。

>类型为 `<String, Person>`，将键类型 (`String`) 与值类型 (`Person`) 匹配。

`public static void main` 方法使用 Spring Boot 的 `SpringApplication.run()` 来启动应用程序并调用 `ApplicationRunner`（另一个 bean 定义），它使用应用程序的 Spring Data 存储库在 Apache Geode 上执行数据访问操作。

应用程序自动装配您刚刚定义的 `PersonRepository` 实例。 Spring Data for Apache Geode 动态创建一个具体的类来实现这个接口并插入所需的查询代码来满足接口的义务。 `run()` 方法使用此存储库实例来演示功能。

## 存储和获取数据
在本指南中，您将创建三个本地 `Person` 对象：**Alice**、**Baby Bob** 和 **Teen Carol**。最初，它们只存在于内存中。创建它们后，您必须将它们保存到 Apache Geode。

现在您可以运行多个查询。第一个按name查找每个人。然后，您可以使用age属性运行一些查询来查找成人、婴儿和青少年。打开日志记录后，您可以看到 Spring Data for Apache Geode 代表您编写的查询。

>要查看 SDG 生成的 Apache Geode OQL 查询，请将 `@ClientCacheApplication` 注释 `logLevel` 属性更改为 `config`。因为查询方法（例如 findByName）使用 SDG 的 `@Trace` 注释进行注释，所以这会打开 Apache Geode 的 OQL 查询跟踪（查询级日志记录），它会向您显示生成的 OQL、执行时间、是否有任何 Apache Geode 索引被收集结果的查询，以及查询返回的行数。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar build/libs/gs-accessing-data-gemfire-0.1.0.jar
{% endhighlight %}

如果您使用 Maven，则可以使用 ./mvnw spring-boot:run 运行应用程序。或者，您可以使用 ./mvnw clean package 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight java %}
java -jar target/gs-accessing-data-gemfire-0.1.0.jar
{% endhighlight %}
>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

您应该会看到类似这样的内容（带有其他内容，例如查询）
{% highlight java %}
Before linking up with {apache-geode-name}...
	Alice is 40 years old.
	Baby Bob is 1 years old.
	Teen Carol is 13 years old.
Lookup each person by name...
	Alice is 40 years old.
	Baby Bob is 1 years old.
	Teen Carol is 13 years old.
Adults (over 18):
	Alice is 40 years old.
Babies (less than 5):
	Baby Bob is 1 years old.
Teens (between 12 and 20):
	Teen Carol is 13 years old.
{% endhighlight %}


## 概括
恭喜！ 您设置了一个 Apache Geode 缓存客户端，存储了简单的实体，并开发了快速查询。


更多详情请访问：[IT-eyes](https://it-eyes.top)