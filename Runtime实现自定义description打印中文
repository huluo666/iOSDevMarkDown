---
title: Runtime实现自定义description打印中文
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
实现原理简介：首先通过runtiime获取对象属性，然后取值进行自定义拼接，通过runtime方法交换自定义description方法，通过Unicode解码打印中文。只在Debug状态下进行方法交换

在接触runtime前，团队开发时为便于调试，常常需要在model里进行description重写，手工重写十分的繁琐不便。而通过runtime方法交换可以无需一行代码即可自定义description，十分方便。另外，从服务器请求下来的json数据包含中文，控制无法直接显示中文，通过重写
DescriptionWithLocale方法Uinicode转码后可以方便查看中文，方便开发调试。

需导入`#import <objc/runtime.h>`
 1. description----当通过NSLog函数打印一个对象时，其输出结果就是该对象的description方法的返回值
 2. debugDescription---当在LLDB中，通过诸如po命令打印一个对象时，其输出结果就是该对象的debugDescription方法的返回值。
 3. debugDescription---默认实现就是直接返回description方法的返回值。

代码如下：
```objectivec
#import "NSObject+AutoDescription.h"
#import "NSObject+Propertys.h"
#import <objc/runtime.h>

/*
 1.description----当通过NSLog函数打印一个对象时，其输出结果就是该对象的description方法的返回值
 2.debugDescription---当在LLDB中，通过诸如po命令打印一个对象时，其输出结果就是该对象的debugDescription方法的返回值。
 3.debugDescription---默认实现就是直接返回description方法的返回值。
 */

@implementation NSObject (AutoDescription)

/**
 *  C 方法交换
 */
static void  SwizzleMethods_AutoDes(Class cls, SEL originalSel, SEL newSel)
{
    //获取实例方法
    Method originalMethod = class_getInstanceMethod(cls, originalSel);
    Method newMethod = class_getInstanceMethod(cls, newSel);
    //改变两个方法的具体指针指向
    method_exchangeImplementations(originalMethod, newMethod);
}


// DEBUG输出
#ifdef DEBUG
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SwizzleMethods_AutoDes([NSObject class], @selector(description), @selector(autoDescription));
//        method_exchangeImplementations(class_getInstanceMethod([NSObject class], @selector(description)), class_getInstanceMethod([NSObject class], @selector(autoDescription)));
    });
}
#endif


- (NSString *)autoDescription
{
    if (![self isKindOfClass:[NSObject class]]) {
        return nil;
    }
    
    // 调试状态
    NSArray * allProperties = [self  getAllProperties];
    NSString * print_str = [NSString stringWithFormat:@"❗%@\n",[self class]];
    for (NSString * propertyName in allProperties) {
        id value = [self valueForKey:propertyName]?:@"nil";//默认值为nil字符串
        if (![value isKindOfClass:[NSString class]]) {
            return print_str;
        }
        print_str = [print_str stringByAppendingFormat:@"\t└%@:%@\n",propertyName,value];
    }
    return print_str;
}
```


Uicode转码方法 
参考 [StackFlow](http://stackoverflow.com/questions/2099349/using-objective-c-cocoa-to-unescape-unicode-characters-ie-u1234/)



```objectivec
+ (NSString *)stringByReplaceUnicode:(NSString *)string
{
    NSMutableString *convertedString = [string mutableCopy];
    [convertedString replaceOccurrencesOfString:@"\\U" withString:@"\\u" options:0 range:NSMakeRange(0, convertedString.length)];
    CFStringRef transform = CFSTR("Any-Hex/Java");
    CFStringTransform((__bridge CFMutableStringRef)convertedString, NULL, transform, YES);
    return convertedString;
}

@end
```

字典NSDictionary，数组NSArray Description
```objectivec
#pragma mark--- NSDictionary Description
#if DEBUG
//直接打印中文乱码问题
//http://objccn.io/issue-9-1/
@implementation NSDictionary (AutoDescription)

+ (void)load
{
    SwizzleMethods_AutoDes([NSDictionary class], @selector(descriptionWithLocale:), @selector(autoDescriptionWithLocale:));
    SwizzleMethods_AutoDes([NSDictionary class], @selector(descriptionWithLocale:indent:), @selector(autoDescriptionWithLocale:indent:));
}

- (NSString *)autoDescriptionWithLocale:(id)locale
{
    NSString *description=[self autoDescriptionWithLocale:locale];
    return [NSObject stringByReplaceUnicode:description];
}

- (NSString *)autoDescriptionWithLocale:(id)locale indent:(NSUInteger)level
{
    NSString *description=[self autoDescriptionWithLocale:locale indent:level];
    return [NSObject stringByReplaceUnicode:description];
}

@end

#pragma mark--- NSArray Description
@implementation NSArray (AutoDescription)

+ (void)load
{
    SwizzleMethods_AutoDes([NSArray class], @selector(descriptionWithLocale:), @selector(autoDescriptionWithLocale:));
    SwizzleMethods_AutoDes([NSArray class], @selector(descriptionWithLocale:indent:), @selector(autoDescriptionWithLocale:indent:));
}


- (NSString *)autoDescriptionWithLocale:(id)locale
{
    NSString *description=[self autoDescriptionWithLocale:locale];
    return [NSObject stringByReplaceUnicode:description];
}

- (NSString *)autoDescriptionWithLocale:(id)locale indent:(NSUInteger)level
{
    NSString *description=[self autoDescriptionWithLocale:locale indent:level];
    return [NSObject stringByReplaceUnicode:description];
}

@end
#endif

```