---
layout:     post
title:      "Cocoapods安装及使用"
subtitle:   ""
date:       2017-10-29 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

最近项目中在集成视频播放，采用了封装很优秀的开源播放器MobileVLCKit，这里使用的是V2.2.2版本。期间遇到了pods安装相当慢的问题，这里就记录一下cocoapods的安装方法和针对下载速度慢的pods项目如何处理

#### cocoapods安装

1、指定gem的源地址

```
//更换墙内地址
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/

//查看地址源
$ gem sources -l
https://gems.ruby-china.org

# 确保只有 gems.ruby-china.org

```

2、更新gem版本

```
gem update --system # 这里请翻墙一下

//查看更新后的版本
$ gem -v
2.6.3

```

3、安装cocoapods

```
sudo gem install -n /usr/local/bin cocoapods

//查看安装后的pods版本

pod --version
1.3.1

```

4、更新cocoapods

```
pod setup

```

#### cocoapods使用Tips

如果遇到下载速度很慢的cocoapods源，可以网络上搜索源的压缩包。这里以MobileVLCKit为例，下载之后是一个zip的压缩包，得到压缩包之后，可以上传到内网的一个下载快的地方，记住下载地址。然后打开Finder，跳转到"~/.cocoapods"这个目录，结构大致如下

![](http://www.xttxqjfg.cn/img/201710/29/01001.png)

然后搜索MobileVLCKit，找到之后编辑对应版本里面的json文件，将源地址替换成上面的下载快的地址，然后pod install就可以了

![](http://www.xttxqjfg.cn/img/201710/29/01002.png)
![](http://www.xttxqjfg.cn/img/201710/29/01003.png)
![](http://www.xttxqjfg.cn/img/201710/29/01004.png)




