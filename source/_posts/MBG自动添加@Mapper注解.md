---
title: MBG自动添加@Mapper注解
date: 2022-03-17 21:40:47
tags:
---

## 问题
最近开始使用MBG来生成MyBatis的Mapper和Dao, 只需要一点点设置就可以自动数据库自动生成Dao相关的代码，用起来还是挺方便的。但是因为我用的是Mybatis3和Spring，参照[Mall项目](http://www.macrozheng.com/#/architect/mall_arch_01?id=mybatis-generator)的MBG配置后发现Mapper没有带上@Mapper，在Service使用的时候发现没有注册成Bean，一番查找后发现可以通过添加MBG插件的方式来解决。

## 添加插件
在官网[Supplied Plugins](http://mybatis.org/generator/reference/plugins.html)中可以找到很多插件，但是我们需要的是一个添加@Mapper注解的插件，搜索一下可以找到一个叫做`org.mybatis.generator.plugins.MapperAnnotationPlugin`的插件，描述如下：
>This plugin has no impact and is not needed when the target runtime in use is based on MyBatis Dynamic SQL.  
This plugin adds the @Mapper annotation to generated mapper interfaces. This plugin should only be used in MyBatis3 environments.

我们只需要往`MBGConfiguration.xml`中添加`<plugin type="org.mybatis.generator.plugins.MapperAnnotationPlugin"/>`即可。