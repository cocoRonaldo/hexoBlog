---
title: How to delete xcode's provisions
comments: true
tags:
  - ios
  - xcode
categories: 
    - ios
    - xcode
date: 2018-05-16 14:02:50
---

## 原因
证书过多，配置证书时候，找起来不方便，有时候需要删除掉 那么如何删除掉呢？
##删除办法

>1.删除xcode

```
cd ~/Library/MobileDevice/Provisioning Profiles
rm *

```

>2.删除 钥匙串中的证书

{% blockquote %}
    打开钥匙串访问，删除相应的证书即可
{% endblockquote %}



