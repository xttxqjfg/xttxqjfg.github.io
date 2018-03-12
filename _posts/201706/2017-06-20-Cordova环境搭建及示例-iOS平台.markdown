---
layout:     post
title:      "Cordova环境搭建及示例-iOS平台"
subtitle:   ""
date:       2017-06-20 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - Hybrid
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

![](http://www.xttxqjfg.cn/img/201706/20/20004.png)

下面命令是在cd到hello目录的前提下

    1. cordova plugin ls  查看现有插件
    2. cordova plugin add cordova-plugin-camera -ios   增加插件，如果不加-ios，则默认
        会同时加载ios和Android两个平台的资源
    3. cordova plugin remove cordova-plugin-compat   移除已有插件

#### 自定义插件开发

### 1、原生部分

Cordova的所有插件都需要继承CDVPlugin类，方法的定义也有固定的格式。另外，如果需要js能调用到插件，还需要再config.xml中申明。下面我们来实现一个自定义的类来获取设备的一些基本信息

```
#import "CDVGetDeviceInfo.h"  

@implementation CDVGetDeviceInfo  

-(void)deviceInfo:(CDVInvokedUrlCommand *) command  
{  
    [self.commandDelegate runInBackground:^{  
        //获取到调用的命令的唯一ID  
        NSString *callBackID = command.callbackId;  

        //得到传过来的数据的值  
        //    NSArray *arrs = command.arguments;  
        //    NSLog(@"js传来的参数%@",arrs);  

        UIDevice * device = [UIDevice currentDevice];  

        NSMutableDictionary * deviceMore = [NSMutableDictionary dictionary];  
        [deviceMore setObject:device.name forKey:@"name"];  
        [deviceMore setObject:device.model forKey:@"model"];  
        [deviceMore setObject:device.localizedModel forKey:@"localizedModel"];  
        [deviceMore setObject:device.systemName forKey:@"systemName"];  
        [deviceMore setObject:device.systemVersion forKey:@"systemVersion"];  
        [deviceMore setObject:device.identifierForVendor.UUIDString forKey:@"identifierForVendor"];  

        //配置返回值  
        CDVPluginResult *result = nil;  

        //判断command传递过来的数组是否有值  
        //if (command.arguments.count) {  
            result = [CDVPluginResult resultWithStatus:(CDVCommandStatus_OK) messageAsDictionary:[deviceMore copy]];  
        // } else {  
            // result = [CDVPluginResult resultWithStatus:(CDVCommandStatus_OK) messageAsString:@"获取参数错误"];  
        //}  

        //通过调用代理发送插件的结果给对应的ID  
        [self.commandDelegate sendPluginResult:result callbackId:callBackID];  
    }];  
}  

@end 

//xml文件中申明实现好的类
<feature name="GetDeviceInfo">  
    <param name="ios-package" value="CDVGetDeviceInfo" />  
</feature>
```

### 2、js实现部分

我们在默认的index.html中增加一个点击按钮，然后实现js方法去调用刚实现的原生插件。下面是按钮点击事件的实现

```
function Cordova_getDeviceInfo() {  
    cordova.exec(
    //成功回调
    function(deviceInfo) {  
        alert(deviceInfo);
    },  
    //失败回调
    function(error) {  
        alert("error"); 
    },  
    //类名、方法名
    "GetDeviceInfo", "deviceInfo", 
    //传递给原生的参数，用数组封装
    ["1","2"]  
    );  
};  
module.exports = Cordova_getDeviceInfo();
```

#### config.xml解析

config.xml是cordova的全局配置文件，详细的说明请看[官网的说明](https://cordova.apache.org/docs/en/latest/config_ref/index.html)

```
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.mydomain.hello" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0">
    <feature name="LocalStorage">
        <param name="ios-package" value="CDVLocalStorage" />
    </feature>
    <feature name="HandleOpenUrl">
        <param name="ios-package" value="CDVHandleOpenURL" />
        <param name="onload" value="true" />
    </feature>
    <feature name="IntentAndNavigationFilter">
        <param name="ios-package" value="CDVIntentAndNavigationFilter" />
        <param name="onload" value="true" />
    </feature>
    <feature name="GestureHandler">
        <param name="ios-package" value="CDVGestureHandler" />
        <param name="onload" value="true" />
    </feature>
    <name>HelloWorld</name>
    <description>
        A sample Apache Cordova application that responds to the deviceready event.
    </description>
    <author email="dev@cordova.apache.org" href="http://cordova.io">
        Apache Cordova Team
    </author>
    //起始页，默认在www目录中
    <content src="index.html" />
    //白名单，允许访问的地址。设置为*表示不对访问做限制
    <access origin="*" />
    <allow-intent href="http://*/*" />
    <allow-intent href="https://*/*" />
    <allow-intent href="tel:*" />
    <allow-intent href="sms:*" />
    <allow-intent href="mailto:*" />
    <allow-intent href="geo:*" />
    <allow-intent href="itms:*" />
    <allow-intent href="itms-apps:*" />
    //下面都是webview的属性设置
    <preference name="AllowInlineMediaPlayback" value="false" />
    <preference name="BackupWebStorage" value="cloud" />
    <preference name="DisallowOverscroll" value="false" />
    <preference name="EnableViewportScale" value="false" />
    <preference name="KeyboardDisplayRequiresUserAction" value="true" />
    <preference name="MediaPlaybackRequiresUserAction" value="false" />
    <preference name="SuppressesIncrementalRendering" value="false" />
    <preference name="SuppressesLongPressGesture" value="false" />
    <preference name="Suppresses3DTouchGesture" value="false" />
    <preference name="GapBetweenPages" value="0" />
    <preference name="PageLength" value="0" />
    <preference name="PaginationBreakingMode" value="page" />
    <preference name="PaginationMode" value="unpaginated" />
</widget>
```


