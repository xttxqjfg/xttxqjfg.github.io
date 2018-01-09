---
layout:     post
title:      "__attribute__简析"
subtitle:   ""
date:       2017-10-23 15:41:00
author:     "易博"
header-img: "img/201705/18/head_bg.JPG"
tags:
    - iOS
---

\_\_attribute\_\_表示属性，是Clang提供的一种源码注释，方便开发者向编译器表达诉求，一般以\_\_attribute\_\_(***)的方式出现在代码中。为了方便使用，一些常用属性被定义成了宏，经常出现在系统头文件中。比如NS_CLASS_AVAILABLE_IOS(9_0) 就是 \_\_attribute\_\_(availability(9.0)) 这个属性的简单写法。下面介绍一些可能会频繁使用到的属性

#### objc_subclassing_restricted

这个属性表示一个类不可以被继承。如果我们希望我们写的某个类不被继承，那么只需要在@interface前面加上这个属性即可，这样如果有类继承了此类，在编译的时候就会报错

```
__attribute__((objc_subclassing_restricted))
@interface NoSon : NSObject
@end
```

#### objc_requires_super

这个属性表示子类继承这个方法时必须要调用super方法，否则编译器会给出警告。宏写法是NS_REQUIRES_SUPER

```
@interface MustCallSuper : NSObject

-(void)testMethod __attribute__((objc_requires_super));
-(void)testMethod1 NS_REQUIRES_SUPER;

@end
```

#### constructor / destructor

构造器和析构器，加上这两个属性的函数会在可执行文件load和unload的时候调用，可以简单的理解为在main()函数调用前和return后执行

```
__attribute__((constructor))
static void beforeMain(void) {
    NSLog(@"beforeMain");
}

__attribute__((destructor))
static void afterMain(void) {
    NSLog(@"afterMain");
}

int main(int argc, const char * argv[]) {
    NSLog(@"main");
return 0;
}

如果有多个constructor想控制优先级的话，可以这样写

__attribute__((constructor(101)))，里面的数字越小优先级越高，1 ~ 100 为系统保留

```

#### cleanup

这个属性的作用是申明到一个变量上，当这个变量的作用域结束时，调用指定的函数

```
// 指定一个cleanup方法，注意入参是所修饰变量的地址，类型要一样
// 对于指向objc对象的指针(id *)，如果不强制声明__strong默认是__autoreleasing，造成类型不匹配
static void stringCleanUp(__strong NSString **string) {
    NSLog(@"%@", *string);
}

// 在某个方法中：
{
    __strong NSString *string __attribute__((cleanup(stringCleanUp))) = @"test";
} 
// 当运行到这个作用域结束时，自动调用stringCleanUp
```

#### objc_runtime_name

用于 @interface 或 @protocol，将类或协议的名字在编译时指定成另一个。利用这个特性，一个简单的代码混淆的原型就出来了

```
__attribute__((objc_runtime_name("NewSon")))
@interface Son : NSObject
@end

NSLog(@"%@", NSStringFromClass([Son class])); // "NewSon"
```

#### 引用

[http://blog.sunnyxx.com/2016/05/14/clang-attributes/](http://blog.sunnyxx.com/2016/05/14/clang-attributes/)






