---
layout: post
title:  "unable to get local issuer certificate的解决办法"
date:   2022-07-26
excerpt: "本指南旨在解决unable to get local issuer certificate的问题。"
tag:
- git
comments: true
---

当我向git仓库推送代码的时候，系统报告如下错误：
{% highlight bash %}
fatal: unable to access 'htps://bitbucket.net/bitbucket/xxxxx.git/': SSL certificate problem: unable to get local issuer certificate
{% endhighlight %}
这是因为服务器的SSL证书没有经过第三方机构认证。我们可以使用以下命令来解决这个问题：
{% highlight bash %}
git config --global http.sslVerify false
{% endhighlight %}


更多详情请访问：[IT-eyes](https://it-eyes.top)