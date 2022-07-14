---
layout: post
title:  "使用 REST 访问 MongoDB 数据"
date:   2022-07-13
excerpt: "本指南将引导您完成创建应用程序的过程，该应用程序通过基于超媒体的 RESTful 前端访问基于文档的数据。"
tag:
- spring 
- java
comments: true
---

本指南将引导您完成创建应用程序的过程，该应用程序通过基于超媒体的 RESTful 前端访问基于文档的数据。

## 你将建造什么
您将构建一个 Spring 应用程序，该应用程序允许您使用 Spring Data REST 创建和检索存储在 [MongoDB](https://www.mongodb.org/) NoSQL 数据库中的 `Person` 对象。 Spring Data REST 采用 [Spring HATEOAS](https://projects.spring.io/spring-hateoas) 和 [Spring Data MongoDB](https://projects.spring.io/spring-data-mongodb) 的特性，并自动将它们组合在一起。

>Spring Data REST 还支持 [Spring Data JPA](https://spring.io/guides/gs/accessing-data-rest)、[Spring Data Gemfire](https://spring.io/guides/gs/accessing-gemfire-data-rest) 和 [Spring Data Neo4j](https://spring.io/guides/gs/accessing-neo4j-data-rest) 作为后端数据存储，但这些不是本指南的一部分。

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

* [下载](https://github.com/spring-guides/gs-accessing-mongodb-data-rest/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-accessing-mongodb-data-rest.git`
* cd 进入 `gs-accessing-mongodb-data-rest/initial`
* 跳转到[安装和启动 MongoDB](https://spring.io/guides/gs/accessing-mongodb-data-rest/#initial)。

完成后，您可以对照 `gs-accessing-mongodb-data-rest/complete` 中的代码检查结果。

## 从SpringInitializr开始
对于所有 Spring 应用程序，您应该从 Spring Initializr 开始。 Spring Initializr 提供了一种快速的方法来获取应用程序所需的所有依赖项，并为您完成大量设置。 此示例需要 Spring for Apache Geode 依赖项。

您可以使用`pre-initialized project`并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Rest Repositories**和**Spring Data MongoDB**。
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

## 创建域对象
创建一个新的域对象来呈现一个人，如以下示例（在 `src/main/java/com/example/accessingmongodbdatarest/Person.java` 中）所示：

{% highlight java %}
package com.example.accessingmongodbdatarest;

import org.springframework.data.annotation.Id;

public class Person {

  @Id private String id;

  private String firstName;
  private String lastName;

  public String getFirstName() {
    return firstName;
  }

  public void setFirstName(String firstName) {
    this.firstName = firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public void setLastName(String lastName) {
    this.lastName = lastName;
  }
}
{% endhighlight %}
`Person` 对象具有名字和姓氏。 （还有一个ID对象，配置为自动生成，不用处理。）

创建人员存储库
接下来，您需要创建一个简单的存储库，如以下清单（在 `src/main/java/com/example/accessingmongodbdatarest/PersonRepository.java` 中）所示：
{% highlight java %}
package com.example.accessingmongodbdatarest;

import java.util.List;

import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.repository.query.Param;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource(collectionResourceRel = "people", path = "people")
public interface PersonRepository extends MongoRepository<Person, String> {

  List<Person> findByLastName(@Param("name") String name);

}
{% endhighlight %}

此存储库是一个接口，可让您执行涉及 `Person` 对象的各种操作。它通过扩展 `MongoRepository` 来获取这些操作，而 MongoRepository 又扩展了 Spring Data Commons 中定义的 `PagingAndSortingRepository` 接口。

在运行时，Spring Data REST 会自动创建此接口的实现。然后它使用 `@RepositoryRestResource` 注解来指导 Spring MVC 在 `/people` 创建 RESTful 端点。

>导出存储库不需要`@RepositoryRestResource`。它仅用于更改导出详细信息，例如使用 `/people` 代替 `/persons` 的默认值。

在这里，您还定义了一个自定义查询来检索基于 `lastName` 值的 `Person` 对象列表。您可以在本指南中进一步了解如何调用它。

>默认情况下，Spring Boot 尝试连接到本地托管的 MongoDB 实例。阅读[参考文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-mongodb)，了解如何将您的应用程序指向托管在其他地方的 MongoDB 实例。

`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：
* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：

{% highlight bash %}
java -jar build/libs/gs-accessing-mongodb-data-rest-0.1.0.jar
{% endhighlight %}
如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：

{% highlight bash %}
java -jar target/gs-accessing-mongodb-data-rest-0.1.0.jar
{% endhighlight %}
>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

显示记录输出。该服务应在几秒钟内启动并运行。

## 测试应用程序
现在应用程序正在运行，您可以对其进行测试。您可以使用任何您希望的 REST 客户端。以下示例使用 *nix 工具 `curl`。

首先，您要查看顶级服务，如以下示例所示：
{% highlight bash %}
$ curl http://localhost:8080
{
  "_links" : {
    "people" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    }
  }
}
{% endhighlight %}
前面的示例提供了此服务器必须提供的功能的第一印象。 在 [http://localhost:8080/people](http://localhost:8080/people) 有一个`people`链接。 它有一些选项，例如`?page`、`?size` 和`?sort`。

>Spring Data REST 使用 [HAL 格式](http://stateless.co/hal_specification.html)进行 JSON 输出。 它很灵活，并提供了一种方便的方式来提供与所服务数据相邻的链接。

当您使用`Person`链接时，您会看到数据库中的人员记录（目前没有）：
{% highlight bash %}
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
{% endhighlight %}
当前没有元素，因此没有页面。 是时候创建一个新的`Person`了！

>如果您多次运行本指南，可能会有剩余数据。 如果您需要重新开始，请参阅 [MongoDB shell 快速参考](https://docs.mongodb.org/manual/reference/mongo-shell/)以查找和删除数据库的命令。
以下命令创建一个名为“Frodo Baggins”的人：
{% highlight bash %}
$ curl -i -X POST -H "Content-Type:application/json" -d "{  \"firstName\" : \"Frodo\",  \"lastName\" : \"Baggins\" }" http://localhost:8080/people
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/people/53149b8e3004990b1af9f229
Content-Length: 0
Date: Mon, 03 Mar 2014 15:08:46 GMT
{% endhighlight %}

* `-i`：确保您可以看到包含标头的响应消息。 显示了新创建的 `Person` 的 URI。
* `-X POST`：表示这是一个用于创建新条目的 `POST`。
* `-H "Content-Type:application/json"`：设置内容类型，以便应用程序知道有效负载包含 JSON 对象。
* `-d '{ "firstName" : "Frodo", "lastName" : "Baggins" }'`：是正在发送的数据。

请注意前面的 `POST` 操作如何包含 `Location` 标头。 这包含新创建资源的 URI。 Spring Data REST 还有两个方法（`RepositoryRestConfiguration.setReturnBodyOnCreate(…)` 和 `setReturnBodyOnUpdate(…)`），您可以使用它们来配置框架以立即返回刚刚创建/更新的资源的表示。
从这里您可以查询所有人，如以下示例所示：
{% highlight bash %}
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
        }
      }
    } ]
  },
  "page" : {
    "size" : 20,
    "totalElements" : 1,
    "totalPages" : 1,
    "number" : 0
  }
}
{% endhighlight %}

`persons`对象包含一个带有 Frodo 的列表。 注意它是如何包含`self`链接的。 Spring Data REST 还使用 [Evo Inflector](https://www.atteo.org/2011/12/12/Evo-Inflector.html) 将实体名称复数以进行分组。

您可以直接查询单个记录，如下例所示：
{% highlight bash %}
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Frodo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
{% endhighlight %}
>这可能看起来纯粹是基于 Web 的，但在幕后，它正在与您启动的 MongoDB 数据库通信。

在本指南中，只有一个域对象。 对于更复杂的系统，域对象相互关联，Spring Data REST 呈现额外的链接以帮助导航到连接的记录。

查找所有自定义查询，如以下示例所示：
{% highlight bash %}
$ curl http://localhost:8080/people/search
{
  "_links" : {
    "findByLastName" : {
      "href" : "http://localhost:8080/people/search/findByLastName{?name}",
      "templated" : true
    }
  }
}
{% endhighlight %}

您可以看到查询的 URL，包括 HTTP 查询参数`name`。 这与嵌入在接口中的 `@Param("name")` 注释相匹配。

要使用 `findByLastName` 查询，请运行以下命令：
{% highlight bash %}
$ curl http://localhost:8080/people/search/findByLastName?name=Baggins
{
  "_embedded" : {
    "persons" : [ {
      "firstName" : "Frodo",
      "lastName" : "Baggins",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
        }
      }
    } ]
  }
}
{% endhighlight %}
因为您在代码中将其定义为返回 `List<Person>`，所以它会返回所有结果。 如果您已将其定义为仅返回 `Person`，它会选择要返回的 `Person` 对象之一。 由于这可能是不可预测的，您可能不希望对可以返回多个条目的查询执行此操作。

您还可以分别发出 `PUT`、`PATCH` 和 `DELETE` REST 调用来替换、更新或删除现有记录。 以下示例使用 `PUT` 调用：

{% highlight bash %}
$ curl -X PUT -H "Content-Type:application/json" -d "{ \"firstName\": \"Bilbo\", \"lastName\": \"Baggins\" }" http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Bilbo",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
{% endhighlight %}
以下示例使用 `PATCH` 调用：

{% highlight bash %}
$ curl -X PATCH -H "Content-Type:application/json" -d "{ \"firstName\": \"Bilbo Jr.\" }" http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people/53149b8e3004990b1af9f229
{
  "firstName" : "Bilbo Jr.",
  "lastName" : "Baggins",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people/53149b8e3004990b1af9f229"
    }
  }
}
{% endhighlight %}
>`PUT` 替换整个记录。 未提供的字段将替换为 `null`。 您可以使用 `PATCH` 更新项目的子集。

您还可以删除记录，如以下示例所示：
{% highlight bash %}
$ curl -X DELETE http://localhost:8080/people/53149b8e3004990b1af9f229
$ curl http://localhost:8080/people
{
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/people{?page,size,sort}",
      "templated" : true
    },
    "search" : {
      "href" : "http://localhost:8080/people/search"
    }
  },
  "page" : {
    "size" : 20,
    "totalElements" : 0,
    "totalPages" : 0,
    "number" : 0
  }
}
{% endhighlight %}

这个[hypermedia-driven interface](https://spring.io/understanding/HATEOAS)的一个方便的方面是您可以如何使用 curl（或您喜欢的任何 REST 客户端）发现所有 RESTful 端点。 无需与客户交换正式合同或接口文件。

## 概括
恭喜！ 您刚刚开发了一个具有基于超媒体的 REST 前端和基于 MongoDB 的后端的应用程序。


更多详情请访问：[IT-eyes](https://it-eyes.top)