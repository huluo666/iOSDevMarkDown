---
title:runtime之属性关联
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

关于Associated Objects详解，请先阅读：
http://nshipster.cn/associated-objects/


```objectivec
//
//  NSObject+AssociatedKey.h
//  RuntimeLearn
//
//  Created by luo.h on 16/3/21.
//  Copyright © 2016年 appledev. All rights reserved.
//

#import <Foundation/Foundation.h>

@interface NSObject (AssociatedKey)

//关于NSString用copy还是strong---用copy更好，strong也可以（苹果官方源码就是用的strong）
@property (nonatomic,copy)   NSString  *addtionStr;
@property (nonatomic,assign) NSInteger  addtionID;

@end

```


key 值必须保证是一个对象级别的唯一常量。一般来说，有以下三种推荐的 key 值：
1.使用 &kAssociatedObjectKey 作为 key 值
`static char kAssociatedAddtionStrKey;`

2.kAssociatedObjectKey 作为 key 值
`static void *kAssociatedAddtionStrKey = &kAssociatedAddtionStrKey;`
或者`const static NSString *kAssociatedAddtionStrKey=@"theAssociatedAddtionStrKey"` ;

3.用 selector ，使用 getter 方法的名称作为 key 值
//`_cmd`在OC的方法中表示当前方法的selector,`objc_selector`编译时会根据每个方法名字参数序列生成唯一标识,所以_cmd在编译时候就已经确定的值
```


```objectivec
//
//  NSObject+AssociatedKey.m
//  RuntimeLearn
//
//  Created by luo.h on 16/3/21.
//  Copyright © 2016年 appledev. All rights reserved.
//

#import "NSObject+AssociatedKey.h"
#import <objc/runtime.h>

@implementation NSObject (AssociatedKey)

static void *kAssociatedAddtionStrKey = &kAssociatedAddtionStrKey;
static void *kAssociatedAddtionIDKey = &kAssociatedAddtionIDKey;

#pragma mark--getter
- (NSString *)addtionStr {
    //    1.
    //    return objc_getAssociatedObject(self, &kAssociatedAddtionStrKey);
    //    2.
    //    return objc_getAssociatedObject(self, kAssociatedAddtionStrKey);
    //    3.
    return objc_getAssociatedObject(self, _cmd);
}


#pragma mark--setter
- (void)setAddtionStr:(NSString *)addtionStr {
    //    1.
    //    objc_setAssociatedObject(self, &kAssociatedAddtionStrKey, addtionStr, OBJC_ASSOCIATION_COPY_NONATOMIC);
    //    2.
    //    objc_setAssociatedObject(self, kAssociatedAddtionStrKey, addtionStr, OBJC_ASSOCIATION_COPY_NONATOMIC);
    //    3.
    objc_setAssociatedObject(self, @selector(addtionStr), addtionStr, OBJC_ASSOCIATION_COPY_NONATOMIC);
}



#pragma mark - setter && getter
-(void)setAddtionID:(NSInteger)addtionID
{
    objc_setAssociatedObject(self, kAssociatedAddtionIDKey, @(addtionID), OBJC_ASSOCIATION_ASSIGN);
}

-(NSInteger)addtionID
{
    return [objc_getAssociatedObject(self, kAssociatedAddtionIDKey) integerValue];
}

@end

```


