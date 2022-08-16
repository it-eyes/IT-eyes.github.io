---
layout: post
title:  "Set language level to 8"
date:   2022-08-14
excerpt: "本指南旨在解决Set language to 8的问题。"
tag:
- java
- IDEA
comments: true
---

本指南旨在解决Set language to 8的问题。

## 背景
在使用IDEA编写代码的时候，如图有段代码出现了报错并提示“Set language to 8”。
![](../img/../assets/img/post/set-language-level-to-8.png)


## 解决
### 方法一：
在pom.xml中添加如下配置
{% highlight xml %}
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>8</source>
                <target>8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
{% endhighlight %}

### 方法二：
在File->Settings->Build,Execution,Deployment->Compiler->Java Compiler,更改`Module`的`Target bytecode version`
![](../img/../assets/img/post/java-compiler-target-8.png)

在File->Project Structure,查看设置`Language level`
![](../img/../assets/img/post/module-language-level-8.png)


更多详情请访问：[IT-eyes](https://it-eyes.top)