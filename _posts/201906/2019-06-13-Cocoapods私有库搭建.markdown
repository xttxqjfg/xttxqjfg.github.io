---
layout:     post
title:      "Cocoapods私有库搭建"
subtitle:   ""
date:       2019-06-13 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

### 前置步骤

操作之前先升级本地的cocoapods到最新版本

最新版本查询地址如下，找到最新的release版本即可

https://github.com/CocoaPods/CocoaPods/releases

### 一、新建私有仓库

1、在任意的代码托管服务器上新建一个私有仓库地址，权限为公开或者有限的，这样确保你的私有库别人能访问。下面以github为例

![](http://www.xttxqjfg.cn/img/201906/13/0613001.png)

2、在本地新建pod仓库。终端中输入下面命令

pod repo add iOS-Cocoapods-Private-Repo https://github.com/xttxqjfg/iOS-Cocoapods-Private-Repo.git

此时在本地的cocoapods中生成如下目录

![](http://www.xttxqjfg.cn/img/201906/13/0613002.png)

到这里私有仓库已经创建完毕，下面将需要私有化的项目添加到此私有库中

### 二、新建项目库

1、将需要私有化的项目工程文件等准备好。在github上新建项目库，举例叫TestSDK，然后clone到本地。

2、终端中cd到项目的根目录，输入以下命令，会在项目的根目录里生成一个podspec文件，文件默认会包含很多内容，可以将前面带#的内容全部删掉，留下仅仅需要关注的内容

pod spec create TestSDK

3、编辑podspec文件，这里我采用的是Atom编辑器，避免出现编码格式不对。文件里的部分默认项做修改，不然会无法通过检验。具体的key含义可以到官网查询，下面是一个模板示例

```
Pod::Spec.new do |s|
s.name         = "TestSDK"
s.version      = "0.0.1"
s.summary      = "A short description of TestSDK.podspec."
s.description  = "TestSDK 0.0.1"
s.homepage     = "http://EXAMPLE/TestSDK.podspec"
s.license      = "MIT"
s.author       = { "yibo" => "****@hotmail.com" }
s.platform     = :ios, "8.0"
#压缩文件形式
s.source       = { :http => "http://***/MobileVLCKit-3.1.5.tar.xz" }  
#git源码方式
s.source       = { :git => "http://***/MobileVLCKit.git", :tag => ${s.version} }  
s.vendored_frameworks = "MobileVLCKit.framework"
s.source_files  = "MobileVLCKit.framework/Headers/*.h"
s.public_header_files = "MobileVLCKit.framework/Headers/*.h"
s.frameworks = "QuartzCore","CoreText","AVFoundation","Security","CFNetwork","AudioToolbox","OpenGLES","CoreGraphics","VideoToolbox","CoreMedia"
s.libraries = "c++","xml2","z","bz2","iconv"
s.requires_arc = false
s.xcconfig = { "CLANG_CXX_LANGUAGE_STANDARD" => "c++11",
"CLANG_CXX_LIBRARY" => "libc++" }
end
```

4、将项目根目录内的内容提交到github，并设置好tag值，这里设置的tag值必须和上面文件里的s.version值一致，以后每次有修改的时候也同样需要同步修改

5、验证当前的podspec配置文件，终端cd到podspec文件所在目录下，用下面命令进行验证。如果出现错误，按照给定的提示修改即可

pod lib lint --allow-warnings

6、如果上述步骤验证通过，则可以将项目添加到私有仓库中。完成后在本地的cocoapods中即可看到刚才添加的私有项目

pod repo push iOS-Cocoapods-Private-Repo TestSDK.podspec --allow-warnings

### 三、项目集成

1、上面步骤完成后，就可以在本地xcode中使用了。在项目的Podfile中配置，然后运行pod install即可

```
source 'https://github.com/xttxqjfg/iOS-Cocoapods-Private-Repo.git'
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, ‘8.0’
inhibit_all_warnings!
target "******" do

pod 'TestSDK','~> 0.0.1'

end

```

这里需要注意一点，如果你的podfile文件中还有其他的私有库，最上面也需要将源地址配置进去，同样，Cocospod本身的源地址也需要加进去，不然会报错找不到

### 四、私有库共享

如果你需要将私有库分享出去，让小伙伴在本地输入一下命令即可添加你的私有库，其他的更新正常用pod setup即可

pod repo add iOS-Cocoapods-Private-Repo https://github.com/xttxqjfg/iOS-Cocoapods-Private-Repo.git


