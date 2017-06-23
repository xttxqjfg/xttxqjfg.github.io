---
layout:     post
title:      "HealthKit简析及应用"
subtitle:   ""
date:       2017-06-22 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

感于微信和支付宝的计步捐赠，就想着能不能自己做一个小玩意儿来改变自己每天的计步量。不过事实是，现在的版本已经不行了，写入到苹果健康的计步数据都带有数据来源，不是微信和支付宝认可的数据来源，是不会计入排行的步数。既然如此，也还是初步了解了一下HealthKit，下面就主要介绍一下HealthKit里常用的应用场景

#### 授权

任何应用想访问苹果健康中的数据，就必须要得到健康应用的授权才可以。苹果健康里面有些数据是可读可写，有些则只读。所以在授权的时候对需要使用的数据项，比如步数，会有可读和可写的授权。在代码中，我们可以使用(- (HKAuthorizationStatus)authorizationStatusForType:(HKObjectType *)type;)方法来获取到某个数据项的授权状态。但是这里我们只能拿到应用是否有写数据的权限，无法获取读数据的权限，下面是苹果官方的说明

>form:https://developer.apple.com/documentation/healthkit/hkhealthstore/1614154-authorizationstatus \n
This method checks the authorization status for saving data.
To help prevent possible leaks of sensitive health information, your app cannot determine whether or not a user has granted permission to read data. If you are not given permission, it simply appears as if there is no data of the requested type in the HealthKit store. If your app is given share permission but not read permission, you see only the data that your app has written to the store. Data from other sources remains hidden.  \n
大致的意思就是为了保护健康数据的隐私，你的App无法获取是否被授予读取数据的权限状态。如果没有被授权，你同样能去仓库查询数据，但是你得到的结果就跟查询没有数据的结果是一样的。如果被授予可写权限但是没有可读权限，你只能查询到你的应用写到仓库中的数据，其他应用写到仓库的数据是不可见的



