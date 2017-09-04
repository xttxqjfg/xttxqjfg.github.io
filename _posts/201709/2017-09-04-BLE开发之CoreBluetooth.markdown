---
layout:     post
title:      "BLE开发之CoreBluetooth"
subtitle:   ""
date:       2017-09-04 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

####  一、前言

BLE，全称蓝牙低能耗(Bluetooth Low Energy)技术是低成本、短距离、可互操作的鲁棒性无线技术，工作在免许可的2.4GHz ISM射频频段，隶属于蓝牙4.0规范。它从一开始就设计为超低功耗(ULP)无线技术。它利用许多智能手段最大限度地降低功耗。蓝牙低能耗技术采用可变连接时间间隔，这个间隔根据具体应用可以设置为几毫秒到几秒不等。另外，因为BLE技术采用非常快速的连接方式，因此平时可以处于“非连接”状态（节省能源），此时链路两端相互间只是知晓对方，只有在必要时才开启链路，然后在尽可能短的时间内关闭链路

iOS开发中常用的蓝牙框架也就下面这几个。GameKit.framework，一般用于游戏开发中iOS设备之间的连接;MultipeerConnectivity.framework，iOS7后推出，个人理解可以实现airdrop类似的功能;ExternalAccessory.framework，可用于基于MFI认证的设备连接;CoreBluetooth.framework，低功耗蓝牙，基于蓝牙4.0，开放度高。大部分场景下采用的都是CoreBluetooth.framework，比如现在常见的物联网设备

CoreBluetooth.framework框架结构如下图所示，基于CoreBluetooth.framework做蓝牙开发，我们首先需要了解几个概念

![](http://www.xttxqjfg.cn/img/201709/04/01001.png)

中心设备(central)、外设(peripheral)，这两个概念放在一起说明比较容易理解。比如当你用摩拜App去解锁自行车的时候，你的手机就是中心设备，摩拜车就是外设。如果是从mac利用airdrop传文件到手机，那么电脑就是中心设备，你的手机则又变成了外设。对应结构图中就有中心模式(CBCentralManager)和外设模式(CBPeripheralManager)两种

服务(service)：每个外设一般都会设计若干个服务，每个服务对应一个事件。比如摩拜车，就可能会有一个解锁的服务。每个服务都会有一个唯一的标识(UUID)用于区分

特征(characteristic)：每个服务下面可以有若干个特征，设备之间的通讯就是通过这些特征值来实现的。比如在解锁摩拜车的时候，可能会有一个车锁状态的特征，解锁或者锁车之后回去修改这个特征值。同样每个特征也会有一个唯一的标识(UUID)用于区分

特征的属性：对应特征的作用域，某些特征是只读、某些是可读可写、某些是广播等等

基于上面几个概念和结构图，在了解一下使用的流程

1、中心模式：创建对象->搜索外设->连接外设->扫描服务->获取特征->基于特征做数据交互->完成后断开连接

2、外设模式：创建对象->设置服务、特征、属性->发布广播->设置读写等委托方法
