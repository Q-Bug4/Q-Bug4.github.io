title: 使用burp suite抓取安卓APP包
date: '2021-05-15 15:25:00'
updated: '2021-05-16 13:26:03'
tags: [Android, 安卓, 抓包]
permalink: /articles/2021/05/15/1621063500676.html
---
![](https://b3logfile.com/bing/20190326.jpg?imageView2/1/w/960/h/540/interlace/1/q/100)

# 前记

最近需要抓取某平台的数据, 而因为目标数据只能在APP上请求, 因此就需要我心爱的安卓手机出场了. 

经过一番尝试, **建议使用安卓版本<7的手机进行抓包**, 可以少掉很多限制. 我也是用安卓6的小米3抓包成功的, 使用安卓10的小米8出现了一堆问题而且最后还是没办法成功抓包.

补充（2021.12.20)：直接在安卓上装了个`HttpCanary`，还挺好用的直接用手机就能抓包。随便推荐个工具叫`Http Tool Kit`也挺不错的。

# 环境&设备

`PC : Windows10`

`手机 : 小米3(安卓6 可以抓包)`

~~`手机 : 小米8(安卓10 无法抓取部分包)`~~

# 抓包准备流程

如果在安装用户证书流程做完后, 已经可以满足你的需求, 则不需要将证书安装到系统中.

## 安装用户证书流程

首先我们选用burp suite在PC端进行抓包, 为了抓取https包需要2,3两个步骤, 若不需要抓取https即可以跳过.

1. burp suite -> Proxy -> Options 设置好要拦截的ip以及端口.
2. 在(1.)的Options中找到`Import / export CA certificate`, 一路导出即可.
3. 将(2.)得到的证书传输到手机中进行安装( 各个手机安装方式有所差异, 但是一般都是 设置-> 安装证书 -> 选择证书 即可 )
4. 手机连接跟PC在同一局域网的Wifi, 进入Wifi设置代理, 代理ip为PC的ip, 端口即在(1.)中设置的端口.
5. 回到burp, 进入于Options平级的Intercept, 点击按钮使得`Intercept is on`
6. 手机访问网站即可抓包.

## 安装系统证书流程

此步骤是为了解决APP不信任用户证书的问题.

系统证书目录`/system/etc/security/cacerts/`

1. 如果你是安卓10并且使用Magisk, 安装Magisk模块`Move Certificates`, 其会将所有用户证书放入系统证书. (安全问题有待考虑)
2. 手动放入目录 ( 手机需要root, 非root的方法请自行查找  )
   1. 假设你已经有了burp的证书PW.cer, 执行如下Shell (我是在wsl2下执行的, Windows应该也差不多)
   2. ```bash
      sudo openssl x509 -in PW.cer -inform DER -out PW.pem -outform pem # 转格式
      sudo openssl x509 -inform PEM -subject_hash -in PW.pem|head -1 # !!!复制输出结果,假设结果是7bf17d07
      sudo cat PW.pem > 7bf17d07.0 # 7bf17d07是证书hash, 0是防止存在相同hash的证书的编号
      ```
   3. 将7bf17d07.0传输到手机中, 使用MT管理器之类的管理器将其放入`/system/etc/security/cacerts/`下 .  或者使用adb进行push.

完成以上操作, 你应该就可以愉快进行抓包啦~

# 可能出现的问题

### 手机设置代理后无法联网

我是使用校园网就不会出现这个问题, 但是用手机连PC的热点就会出现只要设置代理就无法联网, 且无法访问到PC的服务. ( 但是以前抓包是用PC的热点却可以联网 )

**这个问题一般是因为找不到代理对象而出现的.**

### 部分网站/APP无法联网或抓到包

原因 :

1. APP并没有走我们的抓包工具的代理服务器
2. APP不信任我们安装的证书

解决方法 :

1. sasdsda修改host将请求都导向代理服务器(没试过, 但是应该可行)
2. 1. 使用低版本安卓(< Android Nougat)
   2. 将证书放入系统证书目录. (需要做一些修改, 见抓包流程)

# 安卓10抓包大失败

## 出问题

我个人主力机是小米8, 也给心爱的它装上了Magisk并进行Root了. 抓个APP的包不是手到擒来? 信心十足的把小米8和PC都连上校园网, 设置好代理, 手机安装好CA证书, 打开burp suite一气呵成开始抓包. 然后在测试阶段就发现微信公众号的图片加载不出来, 最重要的是 **目标APP无法联网**. What ?!?

## 寻求解决

Google了一下才发现是因为Android10中该APP只信任系统证书, 于是又想把证书变成系统证书. 然后又发现GG了, 无论如何都无法挂载/system为读写. adb, twrp, MT管理器都试过了均没有效果.

其中twrp可以将证书放入系统证书目录, 但是一开机就又恢复原状了. 然而过了几天在更新Clash订阅的时候突然发现出现了报错信息:`/system/etc/security/cacerts/7bf17d07.0 permission denied` , 居然成功放进去了, 结果发现还是APP内依然无法联网.

于是只好宣布安卓10抓包大失败!

**山穷水复疑无路, 柳暗花明又一村**, 好在在Android10 下我们仍有备选方案 : JustTrustMe(Xposed) 与 Move Certificates(Magisk).

- JustTrustMe .v4
  - 官方版本为 .v2 , 2016年最后一次更新, 试了一下已经没有用了, 于是找到这个老哥写的[v4版本](https://github.com/gfbjngjibn/JustTrustMe), 但是, 试了一下依然没有用.
- Move Certificates
  - 留待测试.

# 安卓6 抓包大成功

在我的小米3 (Android 6)中, 只要按照抓包流程的操作来, 安装完系统证书后就可以进行抓包了, 没有一点问题.~~(唯一问题就是小米3实在是太卡了)~~
