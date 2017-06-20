---
layout:     post
title:      "Cordova环境搭建及示例-iOS平台"
subtitle:   ""
date:       2017-06-20 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - Hybird
---

#### Cordova环境搭建

### 1、安装nodejs

直接官网下载安装包安装即可[nodejs官网](https://nodejs.org)

### 2、重置镜像地址

如果翻墙的话，可以略过这一步。否则可能会在下一步安装cordova时失败

打开终端，在管理员权限下分别输入下面命令，设置为国内镜像地址

```
npm config set registry http://registry.cnpmjs.org 
npm info underscore （如果上面配置正确这个命令会有字符串response）
```

### 3、安装cordova

终端中输入下面命令安装corodva，可能需要管理员权限

```
npm install -g cordova
```

### 4、创建cordova项目

打开终端，cd到桌面，输入下面命令创建cordova项目。第一个参数hello是文件目录，创建的项目会放在桌面hello的文件夹中。第二个参数是工程的BundleID。第三个参数是工程的项目名称

```
cordova create hello com.mydomain.hello HelloWorld
```

到上面的步骤，工程还没有添加任何平台信息，用下面的命令添加iOS平台。先cd到hello目录

```
cd hello
cordova platform add ios
```

![](http://www.xttxqjfg.cn/img/201706/20/20001.png)

现在就可以在项目目录中找到工程，然后双击打开了运行了

![](http://www.xttxqjfg.cn/img/201706/20/20002.png)

工程中会有两个www目录和配置文件，第一个是无效的，可以删掉，我们需要用的是第二个

![](http://www.xttxqjfg.cn/img/201706/20/20003.png)

### 5、插件的相关操作

cordova官方已经提供了很多可用的插件，我们可以去官方的github上查看。也可以在终端中输入pod search cordova搜索，每个插件的github说明页会有添加说明

下面命令是在cd到hello目录的前提下

    1. cordova plugin ls  查看现有插件
    2. cordova plugin add cordova-plugin-camera -ios   增加插件，如果不加-ios，则默认
        会同时加载ios和Android两个平台的资源
    3. cordova plugin remove cordova-plugin-compat   移除已有插件


