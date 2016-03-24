---
title: Runtime常用方法 
tags: Runtime,模板,小书匠
grammar_cjkRuby: true
---

新建NSObject Category
如`#import "NSObject+Propertys.h"`

### 1.获取对象的所有属性
```objectivec
/* 获取对象的所有属性 */
- (NSArray *)getAllProperties
{
    Class subclass = [self class];
    if (subclass == [NSObject class]) {
        return nil;
    }
    
    // Get the list of properties
    unsigned int propertyCount;
    //如果没有属性，则propertyCount为0，properties为nil
    objc_property_t *properties = class_copyPropertyList([self class],
                                                         &propertyCount);
    NSMutableArray *result = [NSMutableArray array];
    for (int i = 0; i < propertyCount; i++)
    {
        
        objc_property_t property = properties[i];
        // 获取属性名称
        const char *propertyName = property_getName(property);
        NSString *key = @(propertyName);
        
        // Add to array
        [result addObject:key];
    }
    // 注意，这里properties是一个数组指针，是C的语法，
    // 我们需要使用free函数来释放内存，否则会造成内存泄露
    free(properties);
    return result.count ? [result copy] : nil;
}
```

### 2.获取对象的所有属性和属性值
```objectivec
/* 获取对象的所有属性和属性值 */
- (NSDictionary *)getAllPropertiesAndVaules
{
//    Class currentClass = [self class];
    NSMutableDictionary *propertyDic = [NSMutableDictionary dictionary];
    //属性个数
    unsigned int outCount;
    //属性数组
    objc_property_t *property = class_copyPropertyList([self class], &outCount);
    
    //循环取出属性并存在字典中
    for (int i = 0; i < outCount; i++) {
        objc_property_t property_t = property[i];
        //获得属性的名称
        NSString *propertyName = [NSString stringWithCString:property_getName(property_t) encoding:NSUTF8StringEncoding];
        //从对象中获得指定属性名的属性值
        id propertyValue = [self valueForKey:propertyName];
        
        //属性值不为空时，就封装进字典中
        if (propertyValue && propertyValue != nil) {
            [propertyDic setObject:propertyValue forKey:propertyName];
        }
    }
    //释放掉属性数组
    free(property);
    return propertyDic;
}
```
### 3.获取对象的所有方法
```objectivec
/* 获取对象的所有方法 */
-(void)getAllMethods
{
    unsigned int mothCout_f =0;
    Method* mothList_f = class_copyMethodList([self class],&mothCout_f);
    for(int i=0;i<mothCout_f;i++)
    {
        Method temp_f = mothList_f[i];
        //        IMP imp_f = method_getImplementation(temp_f);
        //        SEL name_f = method_getName(temp_f);
        const char* name_s =sel_getName(method_getName(temp_f));
        int arguments = method_getNumberOfArguments(temp_f);
        const char* encoding =method_getTypeEncoding(temp_f);
        NSLog(@"方法名：%@,参数个数：%d,编码方式：%@",[NSString stringWithUTF8String:name_s], arguments,              [NSString stringWithUTF8String:encoding]);
    }
    //释放内存
    free(mothList_f);
}
```

```objectivec
-(NSDictionary *)getAllPropertyNameAndAttributes{
    u_int count;
    //获取到的类的属性列表
    objc_property_t *properties  = class_copyPropertyList([self class], &count);
    NSMutableArray *propertyNameArray = [NSMutableArray arrayWithCapacity:count];
    NSMutableArray *propertyAttributesArray = [NSMutableArray arrayWithCapacity:count];
    for (int i = 0; i<count; i++)
    {
        objc_property_t property = properties[i];
        //拿到属性名字
        const char *propertyName =property_getName(property);
        //属性的是属性的特性（weak,readonly等等）
        const char *propertyAttributes = property_getAttributes(property);
        
        [propertyNameArray addObject: [NSString stringWithUTF8String: propertyName]];
        [propertyAttributesArray addObject:[NSString stringWithUTF8String: propertyAttributes]];
    }
   
    NSDictionary *propertiesDictionary = [NSDictionary dictionaryWithObjects:propertyAttributesArray forKeys:propertyNameArray];
     free(properties);
    return propertiesDictionary;
}

#pragma mark---- 以下内容适用于成员变量 ivars
-(NSArray *)allMemberVariables {
    unsigned int count = 0;
    Ivar *ivars = class_copyIvarList([self class], &count);
    NSMutableArray *result = [NSMutableArray array];
    for (NSUInteger i = 0; i < count; ++i) {
        Ivar variable = ivars[i];
        const char *name = ivar_getName(variable);
        NSString *varName = [NSString stringWithUTF8String:name];
        
        [result addObject:varName];
    }
    free(ivars);
    return result.count ? [result copy] : nil;
}



+ (NSArray *)protocols {
    unsigned int outCount;
    Protocol * const *protocols = class_copyProtocolList([self class], &outCount);
    
    NSMutableArray *result = [NSMutableArray array];
    for (int i = 0; i < outCount; i++) {
        unsigned int adoptedCount;
        Protocol * const *adotedProtocols = protocol_copyProtocolList(protocols[i], &adoptedCount);
        NSString *protocolName = [NSString stringWithCString:protocol_getName(protocols[i]) encoding:NSUTF8StringEncoding];
        
        NSMutableArray *adoptedProtocolNames = [NSMutableArray array];
        for (int idx = 0; idx < adoptedCount; idx++) {
            [adoptedProtocolNames addObject:[NSString stringWithCString:protocol_getName(adotedProtocols[idx]) encoding:NSUTF8StringEncoding]];
        }
        NSString *protocolDescription = protocolName;
        
        if (adoptedProtocolNames.count) {
            protocolDescription = [NSString stringWithFormat:@"%@ <%@>", protocolName, [adoptedProtocolNames componentsJoinedByString:@", "]];
        }
        [result addObject:protocolDescription];
        //free(adotedProtocols);
    }
    //free((__bridge void *)(*protocols));
    return result.count ? [result copy] : nil;
}

```


