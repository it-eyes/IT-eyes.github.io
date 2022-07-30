---
layout: post
title:  "unable to get local issuer certificate"
date:   2022-07-26
excerpt: "This guide is intended to resolve the issue of unable to get local issuer certificate."
tag:
- git
comments: true
---

when I push the code to bitbucket,the system reports that:
{% highlight bash %}
fatal: unable to access 'htps://bitbucket.net/bitbucket/xxxxx.git/': SSL certificate problem: unable to get local issuer certificate
{% endhighlight %}
This is because the server's SSL certificate is not certified by a third-pary agency.We can solve it with the following command:
{% highlight bash %}
git config --global http.sslVerify false
{% endhighlight %}


For more details, please visit：[IT-eyes](https://it-eyes.top)