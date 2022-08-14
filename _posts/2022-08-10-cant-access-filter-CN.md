---
layout: post
title:  "无法访问javax.servlet.Filter"
date:   2022-08-10
excerpt: "本指南旨在解决无法访问javax.servlet.Filter的问题。"
tag:
- spring
- springSecurity
comments: true
---

本指南旨在解决无法访问javax.servlet.Filter的问题。

## 背景
启动SpringBoot项目时，控制台抛出如图异常：
![](../assets/img/post/cant-access-javax-servlet-filter.png)

## 解决
因为`WebSecurityConfigurerAdapter`内部依赖`spring-boot-starter-web`,但项目中没有引入。需要在pom.xml中引入`spring-boot-starter-web`。
{% highlight java %}
ssh-keygen -t rsa -m PEM
{% endhighlight %}