---
title: Flutter 签名生成Apk
date: 2022-09-15 22:57:23
tags: ['Flutter', 'Android']
---

## 问题
最近在研究一个叫[Hacki](https://github.com/Livinglist/Hacki)的Flutter应用，用Debug模式调试应用的时候应用可以在手机上运行没有问题，而当我打算生成Release版本的时候，就出现了报错。

运行`flutter build apk`，之后会得到一个错误`SigningConfig "release" is missing required property "storeFile".`. 而对于我这种没有接触过安卓编程的小白来说，完全搞不清楚为什么会出现这个问题，于是只能求助于搜索引擎。

## 问题所在与解决
一通搜索之后定位问题到`android/build.gradle`文件中，搜索`storeFile`可以找到这一个代码块：
```gradle
signingConfigs {
    release {
        keyAlias keystoreProperties['keyAlias']
        keyPassword keystoreProperties['keyPassword']
        storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
        storePassword keystoreProperties['storePassword']
    }
}
```

从命名上来看可以看出来大概应该是签名的问题，而从我对于安卓打包应用的了解，打包为Release版的时候是需要进行签名的。再往下看，`storeFile`使用了一个三目运算符来赋值，最后得到的结果只有两个：`file(keystoreProperties['storeFile'])`和`null`. 推测应该就是条件不满足导致取值为null了，问题应该就是签名的时候缺失了`keystoreProperties['storeFile']`的配置。

跳转到keystoreProperties定义的地方可以看到:
```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}
```
从这一块可以看到我们需要一个叫做`key.properties`的文件，同时应该要往里写入一些东西来作为`keystoreProperties`. 那么思路就有了，直接搜索`Flutter android key.properties`，来到官网的[Build and release an Android app](https://docs.flutter.dev/deployment/android#create-an-upload-keystore). 可以看到我们需要做以下几步：
1. 创建一个keystore文件
```bash
# -keystore后跟着的是keystore文件的存放位置，可以自定义，只要第2步在项目里指定更改后的位置就可以了
# 运行后需要输入密钥(重要, 后面要用)，姓名住址之类的信息（不重要，但是如果你要发布给别人用的话还是最好写一下）

# Linux&Mac可以使用
keytool -genkey -v -keystore ~/upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
# Windows可以使用
keytool -genkey -v -keystore %userprofile%\upload-keystore.jks -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```
2. 给我们的项目配置一下keystore的信息（密钥，路径等）
在`android/`下创建一个文件`key.properties`，填入:
```properties
storePassword = 第一步的密钥
keyPassword = 第一步的密钥
keyAlias = upload // 不是很明白这个干啥的
storeFile = 第一步生成的文件的路径
```

3. 在`build.gradle`文件中配置好`keystoreProperties`
参考上文

到这里问题就完美解决了，不过中间还是有很多安卓相关的没搞明白，留个坑以后填。
