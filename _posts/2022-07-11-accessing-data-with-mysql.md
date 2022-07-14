---
layout: post
title:  "使用 MySQL 访问数据"
date:   2022-06-25
excerpt: "本指南将引导您完成创建连接到 MySQL 数据库的 Spring 应用程序的过程（与大多数其他指南和许多示例应用程序使用的内存中的嵌入式数据库相反）。 它使用 Spring Data JPA 来访问数据库，但这只是许多可能的选择之一（例如，您可以使用普通的 Spring JDBC）。"
tag:
- spring 
- java
comments: true
---

本指南将引导您完成创建连接到 MySQL 数据库的 Spring 应用程序的过程（与大多数其他指南和许多示例应用程序使用的内存中的嵌入式数据库相反）。 它使用 Spring Data JPA 来访问数据库，但这只是许多可能的选择之一（例如，您可以使用普通的 Spring JDBC）。

## 你将建造什么
您将创建一个 MySQL 数据库，构建一个 Spring 应用程序，并将其连接到新创建的数据库。

>MySQL 使用 GPL 许可，因此您使用它分发的任何程序二进制文件也必须使用 GPL。 请参阅 [GNU 通用公共许可证](https://www.gnu.org/licenses/gpl.html)。

## 你需要什么
* MySQL 5.6 或更高版本。 如果您安装了 Docker，则将数据库作为[容器](https://hub.docker.com/_/mysql/)运行可能会很有用。
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

* [下载](https://github.com/spring-guides/gs-accessing-data-mysql/archive/main.zip)并解压本指南的源代码库，或使用 Git 克隆它： `git clone https://github.com/spring-guides/gs-accessing-data-mysql.git`
* cd 进入 `gs-accessing-data-mysql/initial`
* 跳转到[创建数据库](https://spring.io/guides/gs/accessing-data-mysql/#initial)。

完成后，您可以对照 `gs-accessing-data-mysql/complete` 中的代码检查结果。

## 从SpringInitializr开始

您可以使用`pre-initialized project`并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring Web**，**Spring Web**和**MySQL Driver**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。

## 创建数据库
打开终端（Microsoft Windows 中的命令提示符）并以可以创建新用户的用户身份打开 MySQL 客户端。

例如，在 Linux 系统上，使用以下命令；
{% highlight bash %}
$ sudo mysql --password
{% endhighlight %}

>这以 `root` 身份连接到 MySQL，并允许从所有主机访问用户。 这不是生产服务器的推荐方式。

要创建新数据库，请在 `mysql` 提示符下运行以下命令：
{% highlight sql %}
mysql> create database db_example; -- Creates the new database
mysql> create user 'springuser'@'%' identified by 'ThePassword'; -- Creates the user
mysql> grant all on db_example.* to 'springuser'@'%'; -- Gives all privileges to the new user on the newly created database
{% endhighlight %}
## 创建 application.properties 文件
Spring Boot 为您提供所有事物的默认值。 例如，默认数据库是 `H2`。 因此，当您想使用任何其他数据库时，您必须在 `application.properties` 文件中定义连接属性。

创建一个名为 `src/main/resources/application.properties` 的资源文件，如以下清单所示：
{% highlight yml %}
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/db_example
spring.datasource.username=springuser
spring.datasource.password=ThePassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#spring.jpa.show-sql: true
{% endhighlight %}

在这里，`spring.jpa.hibernate.ddl-auto` 可以是 `none`、`update`、`create` 或 `create-drop`。有关详细信息，请参阅 [Hibernate 文档](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#configurations-hbmddl)。
* `none`：`MySQL` 的默认值。不会对数据库结构进行任何更改。
* `update`：Hibernate 根据给定的实体结构更改数据库。
* `create`：每次都创建数据库，但不会在关闭时删除它。
* `create-drop`：创建数据库并在 `SessionFactory` 关闭时将其删除。

您必须从 `create` 或 `update` 开始，因为您还没有数据库结构。第一次运行后，您可以根据程序要求将其切换为`update`或`none`。当您想对数据库结构进行一些更改时，请使用`update`。

`H2` 和其他嵌入式数据库的默认设置是 `create-drop`。对于其他数据库，例如 MySQL，默认值为 `none`。

>在您的数据库处于生产状态后，将其设置为 `none`，撤销连接到 Spring 应用程序的 MySQL 用户的所有权限，并只为 MySQL 用户提供 `SELECT`、`UPDATE`、`INSERT` 和 `DELETE`，这是一个很好的安全实践。您可以在本指南的末尾阅读更多相关信息。

## 创建`@Entity` 模型

您需要创建实体模型，如以下清单（在 `src/main/java/com/example/accessingdatamysql/User.java` 中）所示：
{% highlight java %}
package com.example.accessingdatamysql;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity // This tells Hibernate to make a table out of this class
public class User {
  @Id
  @GeneratedValue(strategy=GenerationType.AUTO)
  private Integer id;

  private String name;

  private String email;

  public Integer getId() {
    return id;
  }

  public void setId(Integer id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  public String getEmail() {
    return email;
  }

  public void setEmail(String email) {
    this.email = email;
  }
}
{% endhighlight %}
Hibernate 自动将实体转换为表格。

## 创建存储库
您需要创建包含用户记录的存储库，如以下清单（在 `src/main/java/com/example/accessingdatamysql/UserRepository.java` 中）所示：
{% highlight java %}
package com.example.accessingdatamysql;

import org.springframework.data.repository.CrudRepository;

import com.example.accessingdatamysql.User;

// This will be AUTO IMPLEMENTED by Spring into a Bean called userRepository
// CRUD refers Create, Read, Update, Delete

public interface UserRepository extends CrudRepository<User, Integer> {

}
{% endhighlight %}
Spring 自动在同名的 bean 中实现了这个存储库接口（大小写有所变化 — 称为 `userRepository`）。

## 创建控制器
您需要创建一个控制器来处理对应用程序的 HTTP 请求，如以下清单（在 `src/main/java/com/example/accessingdatamysql/MainController.java` 中）所示：
{% highlight java %}
package com.example.accessingdatamysql;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller // This means that this class is a Controller
@RequestMapping(path="/demo") // This means URL's start with /demo (after Application path)
public class MainController {
  @Autowired // This means to get the bean called userRepository
         // Which is auto-generated by Spring, we will use it to handle the data
  private UserRepository userRepository;

  @PostMapping(path="/add") // Map ONLY POST Requests
  public @ResponseBody String addNewUser (@RequestParam String name
      , @RequestParam String email) {
    // @ResponseBody means the returned String is the response, not a view name
    // @RequestParam means it is a parameter from the GET or POST request

    User n = new User();
    n.setName(name);
    n.setEmail(email);
    userRepository.save(n);
    return "Saved";
  }

  @GetMapping(path="/all")
  public @ResponseBody Iterable<User> getAllUsers() {
    // This returns a JSON or XML with the users
    return userRepository.findAll();
  }
}
{% endhighlight %}

>前面的示例为两个端点明确指定了 `POST` 和 `GET`。 默认情况下，`@RequestMapping` 映射所有 HTTP 操作。

## 创建应用程序类
Spring Initializr 为应用程序创建一个简单的类。 以下清单显示了 Initializr 为此示例创建的类（在 `src/main/java/com/example/accessingdatamysql/AccessingDataMysqlApplication.java` 中）：
{% highlight java %}
package com.example.accessingdatamysql;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccessingDataMysqlApplication {

  public static void main(String[] args) {
    SpringApplication.run(AccessingDataMysqlApplication.class, args);
  }

}
{% endhighlight %}
对于此示例，您无需修改​​ `AccessingDataMysqlApplication` 类。

`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：
* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight java %}
java -jar build/libs/gs-accessing-data-mysql-0.1.0.jar
{% endhighlight %}
如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight java %}
java -jar target/gs-accessing-data-mysql-0.1.0.jar
{% endhighlight %}

>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。

运行应用程序时，会显示日志记录输出。该服务应在几秒钟内启动并运行。

## 测试应用程序
现在应用程序正在运行，您可以使用 `curl` 或其他类似工具对其进行测试。您有两个可以测试的 HTTP 端点：

`GET localhost:8080/demo/all`：获取所有数据。 `POST localhost:8080/demo/add`：将一个用户添加到数据中。

以下 curl 命令添加了一个用户：
{% highlight bash %}
$ curl localhost:8080/demo/add -d name=First -d email=someemail@someemailprovider.com
{% endhighlight %}
答复应如下所示：
{% highlight bash %}
Saved
{% endhighlight %}
以下命令显示所有用户：
{% highlight bash %}
$ curl 'localhost:8080/demo/all'
{% endhighlight %}
答复应如下所示：
{% highlight bash %}
[{"id":1,"name":"First","email":"someemail@someemailprovider.com"}]
{% endhighlight %}

## 进行一些安全更改
当您在生产环境中时，您可能会受到 SQL 注入攻击。 黑客可能会注入 `DROP TABLE` 或任何其他破坏性 SQL 命令。 因此，作为一种安全实践，您应该在向用户公开应用程序之前对数据库进行一些更改。

以下命令撤销与 Spring 应用程序关联的用户的所有权限：
{% highlight bash %}
mysql> revoke all on db_example.* from 'springuser'@'%';
{% endhighlight %}
现在 Spring 应用程序无法在数据库中执行任何操作。

应用程序必须具有某些权限，因此使用以下命令授予应用程序所需的最低权限：

{% highlight bash %}
mysql> grant select, insert, delete, update on db_example.* to 'springuser'@'%';
{% endhighlight %}
删除所有权限并授予一些权限，为您的 Spring 应用程序提供了仅更改数据库数据而不是结构（模式）所需的权限。

当您要更改数据库时：

1. 重新授予权限。
2. 将 `spring.jpa.hibernate.ddl-auto` 更改为`update`。
3. 重新运行您的应用程序。

然后重复此处显示的两个命令，以使您的应用程序再次安全地用于生产。 更好的是，使用专用的迁移工具，例如 Flyway 或 Liquibase。

## 概括
恭喜！ 您刚刚开发了一个绑定到 MySQL 数据库并可以投入生产的 Spring 应用程序！


更多详情请访问：[IT-eyes](https://it-eyes.top)