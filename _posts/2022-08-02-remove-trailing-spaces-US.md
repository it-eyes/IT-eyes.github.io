---
layout: post
title:  "Cancel removing trailing spaces in IDEA"
date:   2022-07-31
excerpt: "This guide is intended to address the issue of removing trailing whitespace in IDEA."
tag:
- IDEA
comments: true
---

This guide is intended to address the issue of removing trailing whitespace in IDEA.

## background
Modified the code in the code repository. There are spaces at the end of some lines. After submitting, these spaces are gone. It seems that the content has been modified in the code repository, but it is not actually modified. Can be confusing for reviewing code.

## solve
IDEA will remove the trailing spaces by default when saving. If we don't want to remove the trailing spaces, we just need to cancel this function. As shown in the figure:
![](../assets/img/post/remove-trailing-spaces.png)

For more details, please visit：[IT-eyes](https://it-eyes.top)