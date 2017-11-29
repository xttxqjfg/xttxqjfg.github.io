---
layout:     post
title:      "Xcode9报Invalid bitcode signature错误的解决方案"
subtitle:   ""
date:       2017-11-29 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

最近由于需要做iPhone X的适配工作，不得以升级了Xcode9和macOS High Sierra。升级后使用Xcode9编译原来的项目代码就出现了问题，会报一个Invalid bitcode signature的错误，经过多方查找问题，发现是pods的配置导致的，下面是解决方案

1、修改工程中pods的设置如下，可以参考原来Xcode8的工程配置

![](http://www.xttxqjfg.cn/img/201711/29/01001.png)

![](http://www.xttxqjfg.cn/img/201711/29/01002.png)

2、替换脚本

```
diff "${PODS_ROOT}/../Podfile.lock" "${PODS_ROOT}/Manifest.lock" > /dev/null
if [ $? != 0 ] ; then
# print error to STDERR
echo "error: The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation." >&2
exit 1
fi
```

3、然后清工程重新编译即可



