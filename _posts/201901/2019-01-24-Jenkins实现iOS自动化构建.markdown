---
layout:     post
title:      "Jenkins实现iOS自动化构建"
subtitle:   ""
date:       2019-01-24 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

本文是作为基于Jenkins的iOS企业打包自动化构建的一篇记录文章，包含过程中遇到的问题处理办法

前置：MAC、Xcode、已安装好Jenkins及其必要插件等

注意：不同版本的jenkins可能在界面上表现不一致

1、Jenkins新建job，点击进到配置，首先勾选参数化构建过程，这样能选择需要构建的版本分支等

![](http://www.xttxqjfg.cn/img/201901/24/24001.png)

2、源码管理我这里选择的是git

![](http://www.xttxqjfg.cn/img/201901/24/24002.png)

3、这里，如果机器上有多个xcode版本，在此处可以进行设置。可以在参数化构建过程中定义一个xcode版本的选择变量，然后在此处引用

![](http://www.xttxqjfg.cn/img/201901/24/24003.png)

4、如果工程采用pods管理，可以增加如下shell脚本

![](http://www.xttxqjfg.cn/img/201901/24/24004.png)

5、选择xcode插件进行编译构建

![](http://www.xttxqjfg.cn/img/201901/24/24005.png)

6、将构建好的ipa上传到蒲公英平台

![](http://www.xttxqjfg.cn/img/201901/24/24006.png)

```
#bin/bsah - l
IPANAME="${WORKSPACE}/build/Release-iphoneos/****.ipa"
LOGMSG="upload from jenkins"
curl -F "file=@${IPANAME}" -F "uKey=userkey" -F "_api_key=apikey" -F "updateDescription=${LOGMSG}" https://qiniu-storage.pgyer.com/apiv1/app/upload
```

7、ipa上传到蒲公英平台之后发送邮件通知。这里用到了sendEmail的插件，可以用一个命令发送邮件。插件下载解压到任意可执行的目录即可，官网下载地址http://caspian.dotconf.net/menu/Software/SendEmail/。[站内下载地址](http://www.xttxqjfg.cn/img/201901/24/sendEmail-v1.56.tar.gz)

![](http://www.xttxqjfg.cn/img/201901/24/24007.png)

```
#!/bin/bash
#多人接收，邮箱中用空格分开
email_reciver="***@qq.com **@qq.com"
#发送者邮箱
email_sender=***@qq.com
#邮箱用户名
email_username=***@qq.com
#邮箱密码
email_password=****

#smtp服务器地址
email_smtphost=smtp.qq.com

appBuildURL="https://www.pgyer.com/***"
qrCodeURL="https://www.pgyer.com/app/qrcode/***"

email_title="Auto:***客户端安装包更新"
email_content="<h4>有新版本的安装包更新，点击下载!</h4></br> <a style="color:red">版本更新说明:</a> </br> ${commitDesText}</br></br><a href="${appBuildURL}">下载安装链接</a> </br></br></br> <img src="${qrCodeURL}" width="250" height="250"/>"

/Users/**/Public/sendEmail-v1.56/sendEmail -f ${email_sender} -t ${email_reciver} -s ${email_smtphost} -u ${email_title} -xu ${email_username} -xp ${email_password} -m ${email_content} -o message-charset=utf-8 tls=no message-content-type=html
```
