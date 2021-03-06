---
layout:     post
title:      "常用URL Scheme"
subtitle:   ""
date:       2017-07-24 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

#### 系统相关

| 应用名称        | URL Scheme    |
| --------   | --------   |
| 短信        | sms://      |
| app store        | itms-apps://      |
| 电话        | tel://      |
| 无线局域网 | App-Prefs:root=WIFI  |
| 蓝牙 | App-Prefs:root=Bluetooth  |
| 蜂窝移动网络 | App-Prefs:root=MOBILE_DATA_SETTINGS_ID  |
| 个人热点 | App-Prefs:root=INTERNET_TETHERING  |
| 运营商 | App-Prefs:root=Carrier  |
| 通知 | App-Prefs:root=NOTIFICATIONS_ID  |
| 通用 | App-Prefs:root=General  |
| 通用-关于本机 | App-Prefs:root=General&path=About  |
| 通用-键盘 | App-Prefs:root=General&path=Keyboard  |
| 通用-辅助功能 | App-Prefs:root=General&path=ACCESSIBILITY  |
| 通用-语言与地区 | App-Prefs:root=General&path=INTERNATIONAL  |
| 通用-还原 | App-Prefs:root=Reset  |
| 墙纸 | App-Prefs:root=Wallpaper  |
| Siri | App-Prefs:root=SIRI  |
| 隐私 | App-Prefs:root=Privacy  |
| Safari | App-Prefs:root=SAFARI  |
| 音乐 | App-Prefs:root=MUSIC  |
| 音乐-均衡器 | App-Prefs:root=MUSIC&path=com.apple.Music:EQ  |
| 照片与相机 | App-Prefs:root=Photos  |
| FaceTime | App-Prefs:root=FACETIME  |

App-Prefs:root=****,此类型格式在iOS10的SDK下已经无法正常跳转，需使用NSURL URLWithString:UIApplicationOpenSettingsURLString

#### 应用

| 应用名称        | URL Scheme    |
| --------   | --------   |
| 微博        | weibo://      |
| QQ        | mqq://      |
| 支付宝        | alipay://      |
| 微信        | weixin://      |
| 微信        | wechat://      |
| 虾米音乐        | xiami://      |
| chrome        | googlechrome://      |
| 微博国际版        | weibointernational://      |
| 摩拜单车        | mobike://      |
| ofo        | ofoapp://      |
| 有道云笔记        | youdaonote://      |
| 印象笔记        | evernote://      |
| 今日头条        | snssdk141://      |
| 网易新闻        | newsapp://      |
| 网易云音乐        | orpheuswidget://      |
| QQ音乐        | qqmusic://      |
| 美团外卖        | meituanwaimai://      |
| 美团        | imeituan://      |
| Gmail        | googlegmail://      |
| 网易邮箱        | neteasemail://      |
| QQ邮箱        | qqmail://      |
| 腾讯视频        | tenvideo://      |
| 爱奇艺        | iqiyi://      |
| 12306        | cn.12306://      |
| 有道词典        | yddict://      |
| 钉钉        | dingtalk://      |
