---
title: >-
  Spring mail SMTPSenderFailedException: 553 Mail from must equal authorized
  user 解决
date: 2022-04-26 22:00:44
tags:
---

## 版本
```xml
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.6.4</version>
```
`jdk17`

## 问题
在使用Spring Boot mail的时候，参照网上给出的配置后，出现了`com.sun.mail.smtp.SMTPSenderFailedException: 553 Mail from must equal authorized user`的错误。错误提示也很明确啊，邮箱的from要跟认证的用户一致。那么请看网上给出的配置：
```yml
mail:
    host: smtp.yeah.net
    username: email@yeah.net
    password: xxxxxxxxxxxxxxx
    properties:
        from: email@yeah.net
```
## 解决
问题就在于这里面确实写了from字段，而且也跟username一致。那么基本上可能说明字段写错了。先说答案，确实字段写错了，正确的写法应该是:
```yml
mail:
    host: smtp.yeah.net
    username: email@yeah.net
    password: xxxxxxxxxxxxxxx
    properties:
        smtp.from: wumqing@yeah.net
```

然而这个写法并不是在某个答案里面直接找到的，因为在搜索过程中即使用的是Google，第一页里面10个有8个是采集站，又或者是一些尝试过后无效的方法（可能是版本问题）。
而因为找不到可以用的解决方法，而我又基本上确定了就是字段写错的缘由，所以决定将这个字段试出来。
我在某一篇文章中发现了这么一个配置项：`properties.mail.smtp.ssl.enable: true`，于是乎在我键入`properties.mail.smtp.from: email@yeah.net`回车之后，Copilot居然给我提示`properties.mail.smtp.from: email@yeah.net`。
然后我试了一下，嗯，**Copilot牛逼!!!**

