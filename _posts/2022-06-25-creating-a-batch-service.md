---
layout: post
title:  "使用SpringBatch创建批处理任务"
date:   2022-06-25
excerpt: "本指南将引导您完成创建基本批处理驱动解决方案的过程"
tag:
- spring 
- java
comments: true
---
本指南将引导您完成创建基本批处理驱动解决方案的过程。

## 你将建造什么
您将构建一个从 CSV 电子表格导入数据、使用自定义代码对其进行转换并将最终结果存储在数据库中的服务。

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

## 业务数据
通常，您的客户或业务分析师会提供电子表格。 对于这个简单的示例，您可以在 `src/main/resources/sample-data.csv` 中找到一些虚构的数据：
{% highlight csv %}
Jill,Doe
Joe,Doe
Justin,Doe
Jane,Doe
John,Doe
{% endhighlight %}

此电子表格的每一行都包含名字和姓氏，以逗号分隔。 这是一种相当常见的模式，Spring 无需定制即可处理。

接下来，您需要编写一个 SQL 脚本来创建一个表来存储数据。 你可以在 `src/main/resources/schema-all.sql`中找到这样的脚本：
{% highlight sql %}
DROP TABLE people IF EXISTS;

CREATE TABLE people  (
    person_id BIGINT IDENTITY NOT NULL PRIMARY KEY,
    first_name VARCHAR(20),
    last_name VARCHAR(20)
);
{% endhighlight %}

>Spring Boot 在启动时会自动运行 `schema-@@platform@@.sql`。 `-all` 是所有平台的默认值。

## 从SpringInitializr开始
您可以使用[pre-initialized project](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.5.5&packaging=jar&jvmVersion=11&groupId=com.example&artifactId=batch-processing&name=batch-processing&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.batch-processing&dependencies=batch,hsql)并单击 Generate 下载 ZIP 文件。 此项目配置为适合本教程中的示例。

手动初始化项目：

1. 导航到 [https://start.spring.io](https://start.spring.io/)。 该服务提取应用程序所需的所有依赖项，并为您完成大部分设置。
2. 选择 Gradle 或 Maven 以及您要使用的语言。 本指南假定您选择了 Java。
3. 单击**Dependencies**并选择**Spring Batch**和**HyperSQL Database**。
4. 单击**Generate**。
5. 下载生成的 ZIP 文件，该文件是根据您的选择配置的 Web 应用程序的存档。

>如果您的 IDE 具有 Spring Initializr 集成，您可以从您的 IDE 完成此过程。

>你也可以从 Github 上 fork 项目并在你的 IDE 或其他编辑器中打开它。

## 创建业务类
现在您可以看到数据输入和输出的格式，您可以编写代码来表示一行数据，如以下示例（来自 `src/main/java/com/example/batchprocessing/Person.java`）所示：
{% highlight java %}
package com.example.batchprocessing;

public class Person {

  private String lastName;
  private String firstName;

  public Person() {
  }

  public Person(String firstName, String lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  public void setFirstName(String firstName) {
    this.firstName = firstName;
  }

  public String getFirstName() {
    return firstName;
  }

  public String getLastName() {
    return lastName;
  }

  public void setLastName(String lastName) {
    this.lastName = lastName;
  }

  @Override
  public String toString() {
    return "firstName: " + firstName + ", lastName: " + lastName;
  }

}
{% endhighlight %}

您可以通过构造函数或通过设置属性使用名字和姓氏来实例化 `Person` 类。

## 创建中间处理器
批处理中的一个常见范例是摄取数据，对其进行转换，然后将其通过管道输出到其他地方。 在这里，您需要编写一个简单的转换器，将名称转换为大写。 以下清单（来自 src/main/java/com/example/batchprocessing/PersonItemProcessor.java）显示了如何执行此操作：
{% highlight java %}
package com.example.batchprocessing;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.batch.item.ItemProcessor;

public class PersonItemProcessor implements ItemProcessor<Person, Person> {

  private static final Logger log = LoggerFactory.getLogger(PersonItemProcessor.class);

  @Override
  public Person process(final Person person) throws Exception {
    final String firstName = person.getFirstName().toUpperCase();
    final String lastName = person.getLastName().toUpperCase();

    final Person transformedPerson = new Person(firstName, lastName);

    log.info("Converting (" + person + ") into (" + transformedPerson + ")");

    return transformedPerson;
  }

}
{% endhighlight %}

`PersonItemProcessor` 实现了 Spring Batch 的 `ItemProcessor` 接口。 这使得将代码连接到您将在本指南后面定义的批处理作业变得很容易。 根据接口，您会收到一个传入的 `Person` 对象，然后将其转换为大写的 `Person`。

>输入和输出类型不必相同。 事实上，在读取一个数据源之后，有时应用程序的数据流需要不同的数据类型。

## 将批处理作业放在一起

现在您需要将实际的批处理作业放在一起。 Spring Batch 提供了许多实用程序类来减少编写自定义代码的需要。 相反，您可以专注于业务逻辑。

要配置您的作业，您必须首先在 `src/main/java/com/exampe/batchprocessing/BatchConfiguration.java` 中创建一个 Spring `@Configuration` 类，如下例所示：
{% highlight java %}
@Configuration
@EnableBatchProcessing
public class BatchConfiguration {

  @Autowired
  public JobBuilderFactory jobBuilderFactory;

  @Autowired
  public StepBuilderFactory stepBuilderFactory;
    ...
}
{% endhighlight %}

对于初学者，`@EnableBatchProcessing` 注释添加了许多支持作业的关键 bean，并为您节省了大量的工作量。 此示例使用基于内存的数据库（由`@EnableBatchProcessing` 提供），这意味着完成后，数据就消失了。 它还自动连接下面需要的几个工厂。 现在将以下 bean 添加到 `BatchConfiguration` 类中以定义读取器、处理器和写入器：
{% highlight java %}
@Bean
public FlatFileItemReader<Person> reader() {
  return new FlatFileItemReaderBuilder<Person>()
    .name("personItemReader")
    .resource(new ClassPathResource("sample-data.csv"))
    .delimited()
    .names(new String[]{"firstName", "lastName"})
    .fieldSetMapper(new BeanWrapperFieldSetMapper<Person>(){
      {
        setTargetType(Person.class);
      }
    })
    .build();
}

@Bean
public PersonItemProcessor processor() {
  return new PersonItemProcessor();
}

@Bean
public JdbcBatchItemWriter<Person> writer(DataSource dataSource) {
  return new JdbcBatchItemWriterBuilder<Person>()
    .itemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>())
    .sql("INSERT INTO people (first_name, last_name) VALUES (:firstName, :lastName)")
    .dataSource(dataSource)
    .build();
}
{% endhighlight %}

第一段代码定义了输入、处理器和输出。

* `reader()` 创建一个 `ItemReader`。 它查找名为 `sample-data.csv` 的文件，并使用足够的信息解析每个行项目以将其转换为 `Person`。
* `processor()` 创建您之前定义的 `PersonItemProcessor` 的一个实例，用于将数据转换为大写。
* `writer(DataSource)` 创建一个 `ItemWriter`。 这个针对 JDBC 目标，并自动获取由 `@EnableBatchProcessing` 创建的数据源的副本。 它包括插入单个 `Person` 所需的 SQL 语句，由 Java bean 属性驱动。

最后一个块（来自 `src/main/java/com/example/batchprocessing/BatchConfiguration.java`）显示了实际的作业配置：
{% highlight java %}
@Bean
public Job importUserJob(JobCompletionNotificationListener listener, Step step1) {
  return jobBuilderFactory.get("importUserJob")
    .incrementer(new RunIdIncrementer())
    .listener(listener)
    .flow(step1)
    .end()
    .build();
}

@Bean
public Step step1(JdbcBatchItemWriter<Person> writer) {
  return stepBuilderFactory.get("step1")
    .<Person, Person> chunk(10)
    .reader(reader())
    .processor(processor())
    .writer(writer)
    .build();
}
{% endhighlight %}

第一种方法定义了作业，第二种方法定义了一个步骤。作业是由步骤构建的，其中每个步骤都可能涉及读取器、处理器和写入器。

在此作业定义中，您需要一个增量器，因为作业使用数据库来维护执行状态。然后列出每个步骤（尽管此作业只有一个步骤）。作业结束，Java API 生成一个完美配置的作业。

在步骤定义中，您定义一次写入多少数据。在这种情况下，它一次最多写入十个记录。接下来，您使用之前注入的 bean 配置读取器、处理器和写入器。

`chunk()` 以 `<Person,Person>` 为前缀，因为它是一个通用方法。这表示每个处理“块”的输入和输出类型，并与 `ItemReader<Person>` 和 `ItemWriter<Person>` 对齐。
批处理配置的最后一点是在作业完成时获得通知的一种方式。以下示例（来自 `src/main/java/com/example/batchprocessing/JobCompletionNotificationListener.java`）显示了这样一个类：
{% highlight java %}
package com.example.batchprocessing;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.batch.core.BatchStatus;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.listener.JobExecutionListenerSupport;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class JobCompletionNotificationListener extends JobExecutionListenerSupport {

  private static final Logger log = LoggerFactory.getLogger(JobCompletionNotificationListener.class);

  private final JdbcTemplate jdbcTemplate;

  @Autowired
  public JobCompletionNotificationListener(JdbcTemplate jdbcTemplate) {
    this.jdbcTemplate = jdbcTemplate;
  }

  @Override
  public void afterJob(JobExecution jobExecution) {
    if(jobExecution.getStatus() == BatchStatus.COMPLETED) {
      log.info("!!! JOB FINISHED! Time to verify the results");

      jdbcTemplate.query("SELECT first_name, last_name FROM people",
        (rs, row) -> new Person(
          rs.getString(1),
          rs.getString(2))
      ).forEach(person -> log.info("Found <" + person + "> in the database."));
    }
  }
}
{% endhighlight %}
`JobCompletionNotificationListener` 侦听作业何时为 `BatchStatus.COMPLETED`，然后使用 `JdbcTemplate` 检查结果。

## 使应用程序可执行
尽管批处理可以嵌入到 Web 应用程序和 WAR 文件中，但下面演示的更简单的方法可以创建一个独立的应用程序。 您将所有内容打包在一个可执行的 JAR 文件中，由一个很好的旧 Java `main()` 方法驱动。

Spring Initializr 为您创建了一个应用程序类。 对于这个简单的示例，它无需进一步修改即可工作。 以下清单（来自 `src/main/java/com/example/batchprocessing/BatchProcessingApplication.java`）显示了应用程序类：

{% highlight java %}
package com.example.batchprocessing;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BatchProcessingApplication {

  public static void main(String[] args) throws Exception {
    System.exit(SpringApplication.exit(SpringApplication.run(BatchProcessingApplication.class, args)));
  }
}
{% endhighlight %}
`@SpringBootApplication` 是一个方便的注解，它添加了以下所有内容：

* `@Configuration`：将类标记为应用程序上下文的 bean 定义源。
* `@EnableAutoConfiguration`：告诉 Spring Boot 根据类路径设置、其他 bean 和各种属性设置开始添加 bean。例如，如果 `spring-webmvc` 在类路径上，则此注释将应用程序标记为 Web 应用程序并激活关键行为，例如设置 `DispatcherServlet`。
* `@ComponentScan`：告诉 Spring 在 `com/example` 包中查找其他组件、配置和服务，让它找到控制器。

`main()` 方法使用 Spring Boot 的 `SpringApplication.run()` 方法来启动应用程序。您是否注意到没有一行 XML？也没有 `web.xml` 文件。这个 Web 应用程序是 100% 纯 Java，您不必处理任何管道或基础设施的配置。

请注意 `SpringApplication.exit()` 和 `System.exit()` 确保 JVM 在作业完成后退出。有关更多详细信息，请参阅 [Spring Boot 参考文档中的应用程序退出部分](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-application-exit)。

出于演示目的，有一些代码可以创建 `JdbcTemplate`、查询数据库并打印出批处理作业插入的人员姓名。

## 构建一个可执行的 JAR
您可以使用 Gradle 或 Maven 从命令行运行应用程序。您还可以构建一个包含所有必要依赖项、类和资源的单个可执行 JAR 文件并运行它。构建可执行 jar 可以在整个开发生命周期、跨不同环境等中轻松地作为应用程序交付、版本化和部署服务。

如果您使用 Gradle，则可以使用 `./gradlew bootRun` 运行应用程序。或者，您可以使用 `./gradlew build` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar build/libs/gs-batch-processing-0.1.0.jar
{% endhighlight %}

如果您使用 Maven，则可以使用 `./mvnw spring-boot:run` 运行应用程序。或者，您可以使用 `./mvnw clean package` 构建 JAR 文件，然后运行 ​​JAR 文件，如下所示：
{% highlight bash %}
java -jar target/gs-batch-processing-0.1.0.jar
{% endhighlight %}

>此处描述的步骤创建了一个可运行的 JAR。您还可以[构建经典的 WAR 文件](https://spring.io/guides/gs/convert-jar-to-war/)。
该作业为每个被转换的人打印一行。作业运行后，您还可以看到查询数据库的输出。它应该类似于以下输出：
{% highlight bash %}
Converting (firstName: Jill, lastName: Doe) into (firstName: JILL, lastName: DOE)
Converting (firstName: Joe, lastName: Doe) into (firstName: JOE, lastName: DOE)
Converting (firstName: Justin, lastName: Doe) into (firstName: JUSTIN, lastName: DOE)
Converting (firstName: Jane, lastName: Doe) into (firstName: JANE, lastName: DOE)
Converting (firstName: John, lastName: Doe) into (firstName: JOHN, lastName: DOE)
Found <firstName: JILL, lastName: DOE> in the database.
Found <firstName: JOE, lastName: DOE> in the database.
Found <firstName: JUSTIN, lastName: DOE> in the database.
Found <firstName: JANE, lastName: DOE> in the database.
Found <firstName: JOHN, lastName: DOE> in the database.
{% endhighlight %}

## 概括
恭喜！ 您构建了一个批处理作业，该作业从电子表格中提取数据，对其进行处理，然后将其写入数据库。


更多详情请访问：[IT-eyes](https://it-eyes.top)