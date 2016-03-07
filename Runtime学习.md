---
title: Runtime学习
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
More: http://www.jianshu.com/p/e071206103a4
http://www.jianshu.com/p/e071206103a4?sukey=014c68f407f2d3e1b30a5017d05a292c6160fcf26c8fdd9f2501f38e0642ef4a67b16b873baa1868d4abb92b98a07de1

RunTime简介
 - RunTime简称运行时.OC就是运行时机制，其中最主要的是消息机制。
 - Objective-C是一门简单的语言，95%是C。只是在语言层面上加了些关键字和语法。真正让Objective-C如此强大的是它的运行时。它很小但却很强大。它的核心是消息分发。

类与对象基础数据结构
`#import <objc/runtime.h>`
打开objc.h头文件




RunTime用法介绍
1. 动态的添加对象的成员变量和方法
2. 动态交换两个方法的实现
3. 实现分类也可以添加属性
4. 实现NSCoding的自动归档和解档
5. 实现字典转模型的自动转换

 iOS：学习runtime的理解和心得 
 http://mp.weixin.qq.com/s?__biz=MjM5OTM0MzIwMQ==&mid=209557726&idx=3&sn=0d751d6ae78e7f48496b07a27dfc8a68&3rd=MzA3MDU4NTYzMw==&scene=6#rd
http://www.jianshu.com/p/8916ad5662a2#

 Object-c runtime 能做什么 
 http://mp.weixin.qq.com/s?__biz=MzAxNDA3ODg0MA==&mid=200816603&idx=1&sn=ee978628a05f24e9a9b8839b56864337&3rd=MzA3MDU4NTYzMw==&scene=6#rd

OC方法调用的本质就是发送消息,使用消息机制必须导入
 `#import <objc/message.h>` 
 
1.发送消息
```objectivec
    Person *p = [[Person alloc] init];
    [p eat];

    // 本质：让对象发送消息
    objc_msgSend(p, @selector(eat));

    // 调用类方法的方式：两种
    [Person eat];
    [[Person class] eat];

    // 用类名调用类方法，底层会自动把类名转换成类对象调用
    // 本质：让类对象发送消息
    objc_msgSend([Person class], @selector(eat));
```
 - 消息机制的原理就是对象根据方法编号SEL去映射表查找对应的方法实现

2.交换方法
```objectivec
// 加载分类到内存的时候调用
+ (void)load
{
    // 获取imageWithName方法地址
    Method imageWithName = class_getClassMethod(self, @selector(imageWithName:));

    // 获取imageWithName方法地址
    Method myImageName = class_getClassMethod(self, @selector(myImageWithName));

    // 交换方法地址，相当于交换实现方式
    method_exchangeImplementations(imageWithName, myImageWithName);
}
```
3.动态添加方法
    作用:如果一个类有很多方法,那么类加载到内存比较消耗性能,需要给每个方法生成映射表，可以使用动态给某个类，添加方法解决
    常见面试题中会问是否使用过performSelector
```objectivec
- (void)viewDidLoad 
{
    [super viewDidLoad];
    Person *p = [[Person alloc] init];
    [p performSelector:@selector(eat)];
}
// 默认方法都有两个隐式参数，
void eat(id self,SEL sel)
{
    NSLog(@"%@ %@",self,NSStringFromSelector(sel));
}
```

// 当一个对象调用未实现的方法，会调用这个方法处理,并且会把对应的方法列表传过来.
// 刚好可以用来判断，未实现的方法是不是我们想要动态添加的方法
```
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    if (sel == @selector(eat)) 
{
        // 第一个参数：给哪个类添加方法
        // 第二个参数：添加方法的方法编号
        // 第三个参数：添加方法的函数实现（函数地址）
        // 第四个参数：函数的类型，(返回值+参数类型) v:void @:对象->self :表示SEL->_cmd
        class_addMethod(self, @selector(eat), eat, "v@:");
    }
    return [super resolveInstanceMethod:sel];
}
```
### 4.给分类添加属性
原理:给一个类声明属性,本质是给这个类添加关联，并不是直接把这个值的内存空间添加到类存空间


```objectivec
- (void)viewDidLoad {
    [super viewDidLoad];

    NSObject *objc = [[NSObject alloc] init];
    objc.name = @".....";
}

// 定义关联的key
static const char *key = "name";
- (NSString *)name
{
    // 根据关联的key，获取关联的值。
    return objc_getAssociatedObject(self, key);
}
- (void)setName:(NSString *)name
{
    // 第一个参数：给哪个对象添加关联
    // 第二个参数：关联的key，通过这个key获取
    // 第三个参数：关联的value
    // 第四个参数:关联的策略
    objc_setAssociatedObject(self, key, name, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

###利用Runtime特性的开源项目
[SVPullToRefresh](https://github.com/samvermette/SVPullToRefresh)
SVPullToRefresh原理上和EGOTableViewPullRefresh差不多，但通过Runtime特性和KVO的运用，把原来客户端实现的逻辑放到了自身来实现，最终呈现一个简单的接口。 
MJRefresh 








