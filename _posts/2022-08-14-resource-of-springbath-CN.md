---
layout: post
title:  "SpringBatch之Resource"
date:   2022-08-14
excerpt: "本指南旨在讲解SpringBatch中读取Resource的问题。"
tag:
- spring
- springBatch
comments: true
---

本指南旨在讲解SpringBatch中读取Resource的问题。

在使用SpringBatch的`reader`读取文件的时候，会用到`reder.setResource()`来配置读取的资源。
当资源在`/resource`目录下时，我们可以是使用`reader.setResource(new ClassPathResource("source.dat"))`来读取资源。
如果你想指定一个在其他地方的资源，可以使用以下方法：
{% highlight java %}
String filePath="E:\\workspace\\project\\data\\source.dat";
Resource resource=new FileSystemResource(filePath);
sourceDatReader.setResource(resource);
{% endhighlight %}


更多详情请访问：[IT-eyes](https://it-eyes.top)