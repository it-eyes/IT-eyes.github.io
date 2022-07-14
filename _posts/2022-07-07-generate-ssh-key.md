---
layout: post
title:  "generate SSH key "
date:   2022-07-07
excerpt: "this guide walks you through the process of generate SSH key."
tag:
- ssh
comments: true
---
this guide walks you through the process of generate SSH key.


## Generate SSH Key
Run the following command,the system will ask for the file name and password of the key.You can press Enter to generate two files: `id_rsa` and `id_rsa.pub`.
{% highlight bash %}
PS C:\Users\dream\it-eyes> ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\dream/.ssh/id_rsa):
Created directory 'C:\Users\dream/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\dream/.ssh/id_rsa.
Your public key has been saved in C:\Users\dream/.ssh/id_rsa.pub.
{% endhighlight %}

Using the RSA algorithm by default,you can also use the following parameters:
* `-b`: set the key length.
* `-t`: Set the key type.Default is `rsa` and can be omitted.
* `-C`: Set remarks,such as email.
* `-f`: Location of the key directory.By default,it is the `.ssh` hidden directory in the current user directory.

Use `-m PEM` with ssh-keygen to generate private kyes in PEM format:
{% highlight bash %}
ssh-keygen -t rsa -m PEM
{% endhighlight %}


For more details, please visit：[IT-eyes](https://it-eyes.top)