---
title : Jekyll post title escape character
date : 2023-02-12 01:08:00 +0900
categories : [Tools, Jekyll]
tags : [jekyll] #소문자만 가능
pinned : 0
image:
    path: /assets/img/posts/title_escape.png
---

When posting in your blog powered by jekyll, you input your title at the top of you readme file.
But the formatting doesn't allow you to include colons and your blog will render a crash.

![title_escape](/assets/img/posts/title_escape.png)

Escape characters don't work as well in this case.

## Solution
A simple solution to this problem: wrap your string with quotes. 
![wrap_title](/assets/img/posts/wrap_title.png)