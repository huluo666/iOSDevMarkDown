---
title: iOS runtime.h头文件中文翻译
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

```objectivec
#ifndef _OBJC_RUNTIME_H
#define _OBJC_RUNTIME_H

#include <objc/objc.h>
#include <stdarg.h>
#include <stdint.h>
#include <stddef.h>
#include <Availability.h>
#include <TargetConditionals.h>

#if TARGET_OS_MAC
#include <sys/types.h>
#endif
//http://www.jianshu.com/p/f900de4a1495
/* Types */
#if !OBJC_TYPES_DEFINED
```

```objectivec
/// 一种类型，代表着类定义中的一个方法
typedef struct objc_method *Method;

/// 一种类型,代表实例(对象)变量
typedef struct objc_ivar *Ivar;

/// 一种类型,代表一个分类
typedef struct objc_category *Category;

/// 一种类型,代表OC声明的属性
typedef struct objc_property *objc_property_t;

/// Class代表一个类 它在objc.h中这样定义 typedef struct objc_class *Class（一种类型，代表OC类）
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                 OBJC2_UNAVAILABLE;// 父类
    const char *name                                  OBJC2_UNAVAILABLE;// 类名
    long version                                      OBJC2_UNAVAILABLE;// 类的版本信息，默认为0，可以通过runtime函数class_setVersion和class_getVersion进行修改、读取
    long info                                         OBJC2_UNAVAILABLE;// 一些标识信息,如CLS_CLASS (0x1L) 表示该类为普通 class ，其中包含对象方法和成员变量;CLS_META (0x2L) 表示该类为 metaclass，其中包含类方法;
    long instance_size                                OBJC2_UNAVAILABLE;// 该类的实例变量大小(包括从父类继承下来的实例变量);
    struct objc_ivar_list *ivars                      OBJC2_UNAVAILABLE;// 该类的成员变量链表（用于存储每个成员变量的地址）
    struct objc_method_list **methodLists             OBJC2_UNAVAILABLE;// 方法定义的链表
    struct objc_cache *cache                          OBJC2_UNAVAILABLE;// 指向最近使用的方法的指针，用于提升效率；
    struct objc_protocol_list *protocols              OBJC2_UNAVAILABLE;// 协议链表
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

#endif

#ifdef __OBJC__
@class Protocol;
#else
typedef struct objc_object Protocol;
#endif

/// 定义方法
struct objc_method_description {
	SEL name;               /**< 方法名 */
	char *types;            /**< 方法参数的类型 */
};

/// Defines a property attribute -定义属性
//属性结构体
typedef struct {
    const char *name;           /**< 属性名  */
    const char *value;          /**< 属性值   (通常为空) */
} objc_property_attribute_t;

```

## 功能（Functions）
```objectivec
/* Working with Instances */
#pragma amrk --- Working with Instances
/**
 *  拷贝一个对象
 *
 *  @param obj OC对象
 *  @param size 对象的大小
 *
 *  @return ID 对象
 */
OBJC_EXPORT id object_copy(id obj, size_t size)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
    OBJC_ARC_UNAVAILABLE;

/** 
 * 释放一个对象
 * 
 * @param obj An Objective-C object.
 * 
 * @return nil
 */
OBJC_EXPORT id object_dispose(id obj)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
    OBJC_ARC_UNAVAILABLE;

/** 
 * 获取一个对象的类（Class）
 * 
 * @param obj The object you want to inspect.
 * 
 * @return The class object of which \e object is an instance, 
 *  or \c Nil if \e object is \c nil.
 */
OBJC_EXPORT Class object_getClass(id obj) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * Sets the class of an object.
 * 
 * @param obj The object to modify.
 * @param cls A class object.
 * 
 * @return The previous value of \e object's class, or \c Nil if \e object is \c nil.
 */
OBJC_EXPORT Class object_setClass(id obj, Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);


/** 
 * 判断对象是否是一个类
 * 
 * @param obj An Objective-C object.
 * 
 * @return true if the object is a class or metaclass, false otherwise.
 */
OBJC_EXPORT BOOL object_isClass(id obj)
    __OSX_AVAILABLE_STARTING(__MAC_10_10, __IPHONE_8_0);


/** 
 * 根据对象获取一个类名
 * 
 * @param obj An Objective-C object.
 * 
 * @return The name of the class of which \e obj is an instance.
 */
OBJC_EXPORT const char *object_getClassName(id obj)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 获取一个实例变量的值
 * 
 * @param obj An Objective-C object.
 * 
 * @return A pointer to any extra bytes allocated with \e obj. If \e obj was
 *   not allocated with any extra bytes, then dereferencing the returned pointer is undefined.
 * 
 * @note This function returns a pointer to any extra bytes allocated with the instance
 *  (as specified by \c class_createInstance with extraBytes>0). This memory follows the
 *  object's ordinary ivars, but may not be adjacent to the last ivar.
 * @note The returned pointer is guaranteed to be pointer-size aligned, even if the area following
 *  the object's last ivar is less aligned than that. Alignment greater than pointer-size is never
 *  guaranteed, even if the area following the object's last ivar is more aligned than that.
 * @note In a garbage-collected environment, the memory is scanned conservatively.
 */
OBJC_EXPORT void *object_getIndexedIvars(id obj)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
    OBJC_ARC_UNAVAILABLE;

/** 
 * 获取一个实例变量的值
 * 
 * @param obj The object containing the instance variable whose value you want to read.
 * @param ivar The Ivar describing the instance variable whose value you want to read.
 * 
 * @return The value of the instance variable specified by \e ivar, or \c nil if \e object is \c nil.
 * 
 * @note \c object_getIvar is faster than \c object_getInstanceVariable if the Ivar
 *  for the instance variable is already known.
 */
OBJC_EXPORT id object_getIvar(id obj, Ivar ivar) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 给一个对象的实例变量设置值
 * 
 * @param obj The object containing the instance variable whose value you want to set.
 * @param ivar The Ivar describing the instance variable whose value you want to set.
 * @param value The new value for the instance variable.
 * 
 * @note \c object_setIvar is faster than \c object_setInstanceVariable if the Ivar
 *  for the instance variable is already known.
 */
OBJC_EXPORT void object_setIvar(id obj, Ivar ivar, id value) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 修改对象实例变量值
 * 
 * @param obj A pointer to an instance of a class. Pass the object containing
 *  the instance variable whose value you wish to modify.
 * @param name A C string. Pass the name of the instance variable whose value you wish to modify.
 * @param value The new value for the instance variable.
 * 
 * @return A pointer to the \c Ivar data structure that defines the type and 
 *  name of the instance variable specified by \e name.
 */
OBJC_EXPORT Ivar object_setInstanceVariable(id obj, const char *name, void *value)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
    OBJC_ARC_UNAVAILABLE;

/** 
 * 获取对象实例变量
 * 
 * @param obj A pointer to an instance of a class. Pass the object containing
 *  the instance variable whose value you wish to obtain.
 * @param name A C string. Pass the name of the instance variable whose value you wish to obtain.
 * @param outValue On return, contains a pointer to the value of the instance variable.
 * 
 * @return A pointer to the \c Ivar data structure that defines the type and name of
 *  the instance variable specified by \e name.
 */
OBJC_EXPORT Ivar object_getInstanceVariable(id obj, const char *name, void **outValue)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
    OBJC_ARC_UNAVAILABLE;


#pragma mark ---类相关函数申明
/* Obtaining Class Definitions */

/** 
 * 根据一个类名 获取一个类
 * 
 * @param name The name of the class to look up.
 * 
 * @return The Class object for the named class, or \c nil
 *  if the class is not registered with the Objective-C runtime.
 * 
 * @note \c objc_getClass is different from \c objc_lookUpClass in that if the class
 *  is not registered, \c objc_getClass calls the class handler callback and then checks
 *  a second time to see whether the class is registered. \c objc_lookUpClass does 
 *  not call the class handler callback.
 * 
 * @warning Earlier implementations of this function (prior to OS X v10.0)
 *  terminate the program if the class does not exist.
 */
OBJC_EXPORT Class objc_getClass(const char *name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 *  获取一个元类
 * 
 * @param name The name of the class to look up.
 * 
 * @return The \c Class object for the metaclass of the named class, or \c nil if the class
 *  is not registered with the Objective-C runtime.
 * 
 * @note If the definition for the named class is not registered, this function calls the class handler
 *  callback and then checks a second time to see if the class is registered. However, every class
 *  definition must have a valid metaclass definition, and so the metaclass definition is always returned,
 *  whether it’s valid or not.
 */
OBJC_EXPORT Class objc_getMetaClass(const char *name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 返回指定类的类定义
 *
 * @param name The name of the class to look up.
 * 
 * @return The Class object for the named class, or \c nil if the class
 *  is not registered with the Objective-C runtime.
 * 
 * @note \c objc_getClass is different from this function in that if the class is not
 *  registered, \c objc_getClass calls the class handler callback and then checks a second
 *  time to see whether the class is registered. This function does not call the class handler callback.
 */
OBJC_EXPORT Class objc_lookUpClass(const char *name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * Returns the class definition of a specified class.
 * 
 * @param name The name of the class to look up.
 * 
 * @return The Class object for the named class.
 * 
 * @note This function is the same as \c objc_getClass, but kills the process if the class is not found.
 * @note This function is used by ZeroLink, where failing to find a class would be a compile-time link error without ZeroLink.
 */
OBJC_EXPORT Class objc_getRequiredClass(const char *name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 获取已注册的类定义的列表
 * http://blog.jobbole.com/79566/
 * @param buffer An array of \c Class values. On output, each \c Class value points to
 *  one class definition, up to either \e bufferCount or the total number of registered classes,
 *  whichever is less. You can pass \c NULL to obtain the total number of registered class
 *  definitions without actually retrieving any class definitions.
 * @param bufferCount An integer value. Pass the number of pointers for which you have allocated space
 *  in \e buffer. On return, this function fills in only this number of elements. If this number is less
 *  than the number of registered classes, this function returns an arbitrary subset of the registered classes.
 * 
 * @return An integer value indicating the total number of registered classes.
 * 
 * @note The Objective-C runtime library automatically registers all the classes defined in your source code.
 *  You can create class definitions at runtime and register them with the \c objc_addClass function.
 * 
 * @warning You cannot assume that class objects you get from this function are classes that inherit from \c NSObject,
 *  so you cannot safely call any methods on such classes without detecting that the method is implemented first.
 */
OBJC_EXPORT int objc_getClassList(Class *buffer, int bufferCount)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 创建并返回一个指向所有已注册类的指针列表
 * 
 * @param outCount An integer pointer used to store the number of classes returned by
 *  this function in the list. It can be \c nil.
 * 
 * @return A nil terminated array of classes. It must be freed with \c free().
 * 
 * @see objc_getClassList
 */
OBJC_EXPORT Class *objc_copyClassList(unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_3_1);
     
```

## 类相关
```objectivec
/* Working with Classes */

/** 
 * 获取类名
 * 
 * @param cls A class object.
 * 
 * @return The name of the class, or the empty string if \e cls is \c Nil.
 */
OBJC_EXPORT const char *class_getName(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 判断类是否是元类
 * 
 * @param cls A class object.
 * 
 * @return \c YES if \e cls is a metaclass, \c NO if \e cls is a non-meta class, 
 *  \c NO if \e cls is \c Nil.
 */
OBJC_EXPORT BOOL class_isMetaClass(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取当前类的父类
 * 
 * @param cls A class object.
 * 
 * @return The superclass of the class, or \c Nil if
 *  \e cls is a root class, or \c Nil if \e cls is \c Nil.
 *
 * @note You should usually use \c NSObject's \c superclass method instead of this function.
 */
OBJC_EXPORT Class class_getSuperclass(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 *  给当前类设置父类
 * 
 * @param cls The class whose superclass you want to set.
 * @param newSuper The new superclass for cls.
 * 
 * @return The old superclass for cls.
 * 
 * @warning You should not use this function.
 */
OBJC_EXPORT Class class_setSuperclass(Class cls, Class newSuper) 
     __OSX_AVAILABLE_BUT_DEPRECATED(__MAC_10_5,__MAC_10_5, __IPHONE_2_0,__IPHONE_2_0);

/** 
 * 获取类的版本号
 * 
 * @param cls A pointer to a \c Class data structure. Pass
 *  the class definition for which you wish to obtain the version.
 * 
 * @return An integer indicating the version number of the class definition.
 *
 * @see class_setVersion
 */
OBJC_EXPORT int class_getVersion(Class cls)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 设置类的版本号
 * 
 * @param cls A pointer to an Class data structure. 
 *  Pass the class definition for which you wish to set the version.
 * @param version An integer. Pass the new version number of the class definition.
 *
 * @note You can use the version number of the class definition to provide versioning of the
 *  interface that your class represents to other classes. This is especially useful for object
 *  serialization (that is, archiving of the object in a flattened form), where it is important to
 *  recognize changes to the layout of the instance variables in different class-definition versions.
 * @note Classes derived from the Foundation framework \c NSObject class can set the class-definition
 *  version number using the \c setVersion: class method, which is implemented using the \c class_setVersion function.
 */
OBJC_EXPORT void class_setVersion(Class cls, int version)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 获取类的大小
 * 
 * @param cls A class object.
 * 
 * @return The size in bytes of instances of the class \e cls, or \c 0 if \e cls is \c Nil.
 */
OBJC_EXPORT size_t class_getInstanceSize(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 *  获取当前实例的成员变量的值
 * 
 * @param cls The class whose instance variable you wish to obtain.
 * @param name The name of the instance variable definition to obtain.
 * 
 * @return A pointer to an \c Ivar data structure containing information about 
 *  the instance variable specified by \e name.
 */
OBJC_EXPORT Ivar class_getInstanceVariable(Class cls, const char *name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 获取当前类的成员变量的值
 * 
 * @param cls The class definition whose class variable you wish to obtain.
 * @param name The name of the class variable definition to obtain.
 * 
 * @return A pointer to an \c Ivar data structure containing information about the class variable specified by \e name.
 */
OBJC_EXPORT Ivar class_getClassVariable(Class cls, const char *name) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 拷贝一个类的属性链表--数组
 * 
 * @param cls The class to inspect.
 * @param outCount On return, contains the length of the returned array. 
 *  If outCount is NULL, the length is not returned.
 * 
 * @return An array of pointers of type Ivar describing the instance variables declared by the class. 
 *  Any instance variables declared by superclasses are not included. The array contains *outCount 
 *  pointers followed by a NULL terminator. You must free the array with free().
 * 
 *  If the class declares no instance variables, or cls is Nil, NULL is returned and *outCount is 0.
 */
OBJC_EXPORT Ivar *class_copyIvarList(Class cls, unsigned int *outCount) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取实例的方法
 * 
 * @param cls The class you want to inspect.
 * @param name The selector of the method you want to retrieve.
 * 
 * @return The method that corresponds to the implementation of the selector specified by 
 *  \e name for the class specified by \e cls, or \c NULL if the specified class or its 
 *  superclasses do not contain an instance method with the specified selector.
 *
 * @note This function searches superclasses for implementations, whereas \c class_copyMethodList does not.
 */
OBJC_EXPORT Method class_getInstanceMethod(Class cls, SEL name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 *  获取类方法
 * 
 * @param cls A pointer to a class definition. Pass the class that contains the method you want to retrieve.
 * @param name A pointer of type \c SEL. Pass the selector of the method you want to retrieve.
 * 
 * @return A pointer to the \c Method data structure that corresponds to the implementation of the 
 *  selector specified by aSelector for the class specified by aClass, or NULL if the specified 
 *  class or its superclasses do not contain an instance method with the specified selector.
 *
 * @note Note that this function searches superclasses for implementations, 
 *  whereas \c class_copyMethodList does not.
 */
OBJC_EXPORT Method class_getClassMethod(Class cls, SEL name)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 获取类方法实现
 *
 * @param cls The class you want to inspect.
 * @param name A selector.
 * 
 * @return The function pointer that would be called if \c [object name] were called
 *  with an instance of the class, or \c NULL if \e cls is \c Nil.
 *
 * @note \c class_getMethodImplementation may be faster than \c method_getImplementation(class_getInstanceMethod(cls, name)).
 * @note The function pointer returned may be a function internal to the runtime instead of
 *  an actual method implementation. For example, if instances of the class do not respond to
 *  the selector, the function pointer returned will be part of the runtime's message forwarding machinery.
 */
OBJC_EXPORT IMP class_getMethodImplementation(Class cls, SEL name) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 返回方法的具体实现
 * Returns the function pointer that would be called if a particular
 * message were sent to an instance of a class.
 * 
 * @param cls The class you want to inspect.
 * @param name A selector.
 * 
 * @return The function pointer that would be called if \c [object name] were called
 *  with an instance of the class, or \c NULL if \e cls is \c Nil.
 */
OBJC_EXPORT IMP class_getMethodImplementation_stret(Class cls, SEL name) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0)
     OBJC_ARM64_UNAVAILABLE;

/** 
 * 判断Selector是否响应
 * 
 * @param cls The class you want to inspect.
 * @param sel A selector.
 * 
 * @return \c YES if instances of the class respond to the selector, otherwise \c NO.
 * 
 * @note You should usually use \c NSObject's \c respondsToSelector: or \c instancesRespondToSelector: 
 *  methods instead of this function.
 */
OBJC_EXPORT BOOL class_respondsToSelector(Class cls, SEL sel) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取一个类的实例方法列表
 * 
 * @param cls The class you want to inspect.
 * @param outCount On return, contains the length of the returned array. 
 *  If outCount is NULL, the length is not returned.
 * 
 * @return An array of pointers of type Method describing the instance methods 
 *  implemented by the class—any instance methods implemented by superclasses are not included. 
 *  The array contains *outCount pointers followed by a NULL terminator. You must free the array with free().
 * 
 *  If cls implements no instance methods, or cls is Nil, returns NULL and *outCount is 0.
 * 
 * @note To get the class methods of a class, use \c class_copyMethodList(object_getClass(cls), &count).
 * @note To get the implementations of methods that may be implemented by superclasses, 
 *  use \c class_getInstanceMethod or \c class_getClassMethod.
 */
OBJC_EXPORT Method *class_copyMethodList(Class cls, unsigned int *outCount) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 返回类是否实现指定的协议
 * 
 * @param cls The class you want to inspect.
 * @param protocol A protocol.
 *
 * @return YES if cls conforms to protocol, otherwise NO.
 *
 * @note You should usually use NSObject's conformsToProtocol: method instead of this function.
 */
OBJC_EXPORT BOOL class_conformsToProtocol(Class cls, Protocol *protocol) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 返回类实现的协议列表
 * 
 * @param cls The class you want to inspect.
 * @param outCount On return, contains the length of the returned array. 
 *  If outCount is NULL, the length is not returned.
 * 
 * @return An array of pointers of type Protocol* describing the protocols adopted 
 *  by the class. Any protocols adopted by superclasses or other protocols are not included. 
 *  The array contains *outCount pointers followed by a NULL terminator. You must free the array with free().
 * 
 *  If cls adopts no protocols, or cls is Nil, returns NULL and *outCount is 0.
 */
OBJC_EXPORT Protocol * __unsafe_unretained *class_copyProtocolList(Class cls, unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取指定的属性
 * 
 * @param cls The class you want to inspect.
 * @param name The name of the property you want to inspect.
 * 
 * @return A pointer of type \c objc_property_t describing the property, or
 *  \c NULL if the class does not declare a property with that name, 
 *  or \c NULL if \e cls is \c Nil.
 */
OBJC_EXPORT objc_property_t class_getProperty(Class cls, const char *name)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取属性列表
 * 
 * @param cls The class you want to inspect.
 * @param outCount On return, contains the length of the returned array. 
 *  If \e outCount is \c NULL, the length is not returned.        
 * 
 * @return An array of pointers of type \c objc_property_t describing the properties 
 *  declared by the class. Any properties declared by superclasses are not included. 
 *  The array contains \c *outCount pointers followed by a \c NULL terminator. You must free the array with \c free().
 * 
 *  If \e cls declares no properties, or \e cls is \c Nil, returns \c NULL and \c *outCount is \c 0.
 */
OBJC_EXPORT objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 *  返回给定类的实例变量布局(这个布局指的是否是强引用)的描述信息
 * 
 * @param cls The class to inspect.
 * 
 * @return A description of the \c Ivar layout for \e cls.
 */
OBJC_EXPORT const uint8_t *class_getIvarLayout(Class cls)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 返回给定类的实例变量布局(这个布局指的是否是弱引用)的描述信息
 * 
 * @param cls The class to inspect.
 * 
 * @return A description of the layout of the weak \c Ivars for \e cls.
 */
OBJC_EXPORT const uint8_t *class_getWeakIvarLayout(Class cls)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 给一个类动态的添加方法
 * 
 * @param cls The class to which to add a method.
 * @param name A selector that specifies the name of the method being added.
 * @param imp A function which is the implementation of the new method. The function must take at least two arguments—self and _cmd.
 * @param types An array of characters that describe the types of the arguments to the method. 
 * 
 * @return YES if the method was added successfully, otherwise NO 
 *  (for example, the class already contains a method implementation with that name).
 *
 * @note class_addMethod will add an override of a superclass's implementation, 
 *  but will not replace an existing implementation in this class. 
 *  To change an existing implementation, use method_setImplementation.
 */
OBJC_EXPORT BOOL class_addMethod(Class cls, SEL name, IMP imp, 
                                 const char *types) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 替换方法实现
 * 
 * @param cls The class you want to modify.
 * @param name A selector that identifies the method whose implementation you want to replace.
 * @param imp The new implementation for the method identified by name for the class identified by cls.
 * @param types An array of characters that describe the types of the arguments to the method. 
 *  Since the function must take at least two arguments—self and _cmd, the second and third characters
 *  must be “@:” (the first character is the return type).
 * 
 * @return The previous implementation of the method identified by \e name for the class identified by \e cls.
 * 
 * @note This function behaves in two different ways:
 *  - If the method identified by \e name does not yet exist, it is added as if \c class_addMethod were called. 
 *    The type encoding specified by \e types is used as given.
 *  - If the method identified by \e name does exist, its \c IMP is replaced as if \c method_setImplementation were called.
 *    The type encoding specified by \e types is ignored.
 */
OBJC_EXPORT IMP class_replaceMethod(Class cls, SEL name, IMP imp, 
                                    const char *types) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 给一个类动态添加一个成员变量
 * 
 * @return YES if the instance variable was added successfully, otherwise NO 
 *         (for example, the class already contains an instance variable with that name).
 *
 * @note This function may only be called after objc_allocateClassPair and before objc_registerClassPair. 
 *       Adding an instance variable to an existing class is not supported.
 * @note The class must not be a metaclass. Adding an instance variable to a metaclass is not supported.
 * @note The instance variable's minimum alignment in bytes is 1<<align. The minimum alignment of an instance 
 *       variable depends on the ivar's type and the machine architecture. 
 *       For variables of any pointer type, pass log2(sizeof(pointer_type)).
 */
OBJC_EXPORT BOOL class_addIvar(Class cls, const char *name, size_t size, 
                               uint8_t alignment, const char *types) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 动态添加一个协议
 * 
 * @param cls The class to modify.
 * @param protocol The protocol to add to \e cls.
 * 
 * @return \c YES if the method was added successfully, otherwise \c NO 
 *  (for example, the class already conforms to that protocol).
 */
OBJC_EXPORT BOOL class_addProtocol(Class cls, Protocol *protocol) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 为Class动态添加一个属性
 * 
 * @param cls The class to modify.
 * @param name The name of the property.
 * @param attributes An array of property attributes.
 * @param attributeCount The number of attributes in \e attributes.
 * 
 * @return \c YES if the property was added successfully, otherwise \c NO
 *  (for example, the class already has that property).
 */
OBJC_EXPORT BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 替换属性
 * 
 * @param cls The class to modify.
 * @param name The name of the property.
 * @param attributes An array of property attributes.
 * @param attributeCount The number of attributes in \e attributes. 
 */
OBJC_EXPORT void class_replaceProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * IvarLayout的set方法 ivarLayout 和 weakIvarLayout 分别记录了哪些 ivar 是 strong 或是 weak
 * 
 * @param cls The class to modify.
 * @param layout The layout of the \c Ivars for \e cls.
 */
OBJC_EXPORT void class_setIvarLayout(Class cls, const uint8_t *layout)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * IvarLayout的set方法 ivarLayout 和 weakIvarLayout 分别记录了哪些 ivar 是 strong 或是 weak
 *
 * @param cls The class to modify.
 * @param layout The layout of the weak Ivars for \e cls.
 */
OBJC_EXPORT void class_setWeakIvarLayout(Class cls, const uint8_t *layout)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * Used by CoreFoundation's toll-free bridging.
 * Return the id of the named class.
 * 
 * @return The id of the named class, or an uninitialized class
 *  structure that will be used for the class when and if it does 
 *  get loaded.
 * 
 * @warning Do not call this function yourself.
 */
OBJC_EXPORT Class objc_getFutureClass(const char *name) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0)
     OBJC_ARC_UNAVAILABLE;
```


## 实例化一个类相关
```objectivec
/* Instantiating Classes */
#pragma mark Instantiating Classes 实例化一个类相关
/** Instantiating Classes
 * Creates an instance of a class, allocating memory for the class in the 
 * default malloc memory zone.
 * 
 * @param cls The class that you wish to allocate an instance of.
 * @param extraBytes An integer indicating the number of extra bytes to allocate. 
 *  The additional bytes can be used to store additional instance variables beyond 
 *  those defined in the class definition.
 * 
 * @return An instance of the class \e cls.
 */
OBJC_EXPORT id class_createInstance(Class cls, size_t extraBytes)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0)
    OBJC_ARC_UNAVAILABLE;

/** 
 * 在指定位置创建类实例
 * 
 * @param cls The class that you wish to allocate an instance of.
 * @param bytes The location at which to allocate an instance of \e cls.
 *  Must point to at least \c class_getInstanceSize(cls) bytes of well-aligned,
 *  zero-filled memory.
 *
 * @return \e bytes on success, \c nil otherwise. (For example, \e cls or \e bytes
 *  might be \c nil)
 *
 * @see class_createInstance
 */
OBJC_EXPORT id objc_constructInstance(Class cls, void *bytes) 
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_0)
    OBJC_ARC_UNAVAILABLE;

/** 
 * 销毁类实例
 * associated references this instance might have had.
 * 
 * @param obj The class instance to destroy.
 * 
 * @return \e obj. Does nothing if \e obj is nil.
 * 
 * @warning GC does not call this. If you edit this, also edit finalize.
 *
 * @note CF and other clients do call this under GC.
 */
OBJC_EXPORT void *objc_destructInstance(id obj) 
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_0)
    OBJC_ARC_UNAVAILABLE;


#pragma mark -- Adding Classes 添加一个类
/* Adding Classes */
/**
 * 创建一个新类和元类
 * 
 * @param superclass The class to use as the new class's superclass, or \c Nil to create a new root class.
 * @param name The string to use as the new class's name. The string will be copied.
 * @param extraBytes The number of bytes to allocate for indexed ivars at the end of 
 *  the class and metaclass objects. This should usually be \c 0.
 * 
 * @return The new class, or Nil if the class could not be created (for example, the desired name is already in use).
 * 
 * @note You can get a pointer to the new metaclass by calling \c object_getClass(newClass).
 * @note To create a new class, start by calling \c objc_allocateClassPair. 
 *  Then set the class's attributes with functions like \c class_addMethod and \c class_addIvar.
 *  When you are done building the class, call \c objc_registerClassPair. The new class is now ready for use.
 * @note Instance methods and instance variables should be added to the class itself. 
 *  Class methods should be added to the metaclass.
 */
OBJC_EXPORT Class objc_allocateClassPair(Class superclass, const char *name, 
                                         size_t extraBytes) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 *  注册由objc_allocateClassPair创建的类
 * 
 * @param cls The class you want to register.
 */
OBJC_EXPORT void objc_registerClassPair(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * Used by Foundation's Key-Value Observing.
 * 
 * @warning Do not call this function yourself.
 */
OBJC_EXPORT Class objc_duplicateClass(Class original, const char *name, size_t extraBytes)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 销毁一个类及其相关联的类
 * 
 * @param cls The class to be destroyed. It must have been allocated with 
 *  \c objc_allocateClassPair
 * 
 * @warning Do not call if instances of this class or a subclass exist.
 */
OBJC_EXPORT void objc_disposeClassPair(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
```

## 方法相关
```objectivec
/* Working with Methods */
#pragma mark ---Working with Methods 方法相关
//http://lizhaoloveit.com/2014/11/12/Objctive-Runtime-Reference/
/**
 * 获取方法名
 * 
 * @param m The method to inspect.
 * 
 * @return A pointer of type SEL.
 * 
 * @note To get the method name as a C string, call \c sel_getName(method_getName(method)).
 */
OBJC_EXPORT SEL method_getName(Method m) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/**
 * 获取方法实现
 * 返回值：是一个 IMP 类型的函数指针
 *
 * @return A function pointer of type IMP.
 */
OBJC_EXPORT IMP method_getImplementation(Method m) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取方法的类型编码
 * 
 * @param m The method to inspect.
 * 
 * @return A C string. The string may be \c NULL.
 */
OBJC_EXPORT const char *method_getTypeEncoding(Method m) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取方法的参数数量
 * 
 * @param m A pointer to a \c Method data structure. Pass the method in question.
 * 
 * @return An integer containing the number of arguments accepted by the given method.
 */
OBJC_EXPORT unsigned int method_getNumberOfArguments(Method m)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 拷贝方法的返回类型
 * 
 * @param m The method to inspect.
 * 
 * @return A C string describing the return type. You must free the string with \c free().
 */
OBJC_EXPORT char *method_copyReturnType(Method m) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 根据索引拷贝参数类型
 * 
 * @param m The method to inspect.
 * @param index The index of the parameter to inspect.
 * 
 * @return A C string describing the type of the parameter at index \e index, or \c NULL
 *  if method has no parameter index \e index. You must free the string with \c free().
 */
OBJC_EXPORT char *method_copyArgumentType(Method m, unsigned int index) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取方法的返回类型
 * 
 * @param m The method you want to inquire about. 
 * @param dst The reference string to store the description.
 * @param dst_len The maximum number of characters that can be stored in \e dst.
 *
 * @note The method's return type string is copied to \e dst.
 *  \e dst is filled as if \c strncpy(dst, parameter_type, dst_len) were called.
 */
OBJC_EXPORT void method_getReturnType(Method m, char *dst, size_t dst_len) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 *  根据索引获取参数类型.
 * 
 * @param m The method you want to inquire about. 
 * @param index The index of the parameter you want to inquire about.
 * @param dst The reference string to store the description.
 * @param dst_len The maximum number of characters that can be stored in \e dst.
 * 
 * @note The parameter type string is copied to \e dst. \e dst is filled as if \c strncpy(dst, parameter_type, dst_len) 
 *  were called. If the method contains no parameter with that index, \e dst is filled as
 *  if \c strncpy(dst, "", dst_len) were called.
 */
OBJC_EXPORT void method_getArgumentType(Method m, unsigned int index, 
                                        char *dst, size_t dst_len) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
OBJC_EXPORT struct objc_method_description *method_getDescription(Method m) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 设置方法的实现
 * 
 * @param m The method for which to set an implementation.
 * @param imp The implemention to set to this method.
 * 
 * @return The previous implementation of the method.
 */
OBJC_EXPORT IMP method_setImplementation(Method m, IMP imp) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 交换两个方法的方法实现
 * 注意：这时原子形式的，是线程安全的。
 * @param m1 Method to exchange with second method.
 * @param m2 Method to exchange with first method.
 * 
 * @note This is an atomic version of the following:
 *  \code 
 *  IMP imp1 = method_getImplementation(m1);
 *  IMP imp2 = method_getImplementation(m2);
 *  method_setImplementation(m1, imp2);
 *  method_setImplementation(m2, imp1);
 *  \endcode
 */
OBJC_EXPORT void method_exchangeImplementations(Method m1, Method m2) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);


/* Working with Instance Variables */
#pragma mark --Working with Instance Variables   实例变量相关
/** 
 * 获取实例变量名(成员变量的名字)
 * 
 * @param v The instance variable you want to enquire about.
 * 
 * @return A C string containing the instance variable's name.
 */
OBJC_EXPORT const char *ivar_getName(Ivar v) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取成员变量的类型
 * 
 * @param v The instance variable you want to enquire about.
 * 
 * @return A C string containing the instance variable's type encoding.
 *
 * @note For possible values, see Objective-C Runtime Programming Guide > Type Encodings.
 */
OBJC_EXPORT const char *ivar_getTypeEncoding(Ivar v) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取实例变量偏移地址
 * 
 * @param v The instance variable you want to enquire about.
 * 
 * @return The offset of \e v.
 * 
 * @note For instance variables of type \c id or other object types, call \c object_getIvar
 *  and \c object_setIvar instead of using this offset to access the instance variable data directly.
 */
OBJC_EXPORT ptrdiff_t ivar_getOffset(Ivar v) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

```



## 属性相关
```objectivec
/* Working with Properties */
#pragma mark --- Working with Properties 属性相关
/** 
 * 获取属性名
 * 
 * @param property The property you want to inquire about.
 * 
 * @return A C string containing the property's name.
 */
OBJC_EXPORT const char *property_getName(objc_property_t property) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获得属性的名字和@encode编码
 * 例如：propertyAttributes:T@"NSString",&,N,thisisString
 * @param property A property.
 *
 * @return A C string containing the property's attributes.
 * 
 * @note The format of the attribute string is described in Declared Properties in Objective-C Runtime Programming Guide.
 */
OBJC_EXPORT const char *property_getAttributes(objc_property_t property) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 拷贝 property 的  attributes 列表
 * 
 * @param property The property whose attributes you want copied.
 * @param outCount The number of attributes returned in the array.
 * 
 * @return An array of property attributes; must be free'd() by the caller. 
 */
OBJC_EXPORT objc_property_attribute_t *property_copyAttributeList(objc_property_t property, unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 拷贝 property 的 attribute 列表的值
 * 
 * @param property The property whose attribute value you are interested in.
 * @param attributeName C string representing the attribute name.
 *
 * @return The value string of the attribute \e attributeName if it exists in
 *  \e property, \c nil otherwise. 
 */
OBJC_EXPORT char *property_copyAttributeValue(objc_property_t property, const char *attributeName)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);


/* Working with Protocols */
#pragma mark --Working with Protocols 协议相关
/** 
 * 根据名字获取协议
 * 
 * @param name The name of a protocol.
 * 
 * @return The protocol named \e name, or \c NULL if no protocol named \e name could be found.
 * 
 * @note This function acquires the runtime lock.
 */
OBJC_EXPORT Protocol *objc_getProtocol(const char *name)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 拷贝协议列表
 * 
 * @param outCount Upon return, contains the number of protocols in the returned array.
 * 
 * @return A C array of all the protocols known to the runtime. The array contains \c *outCount
 *  pointers followed by a \c NULL terminator. You must free the list with \c free().
 * 
 * @note This function acquires the runtime lock.
 */
OBJC_EXPORT Protocol * __unsafe_unretained *objc_copyProtocolList(unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 检查A协议是否符合另一个协议
 * 
 * @param proto A protocol.
 * @param other A protocol.
 * 
 * @return \c YES if \e proto conforms to \e other, otherwise \c NO.
 * 
 * @note One protocol can incorporate other protocols using the same syntax 
 *  that classes use to adopt a protocol:
 *  \code
 *  @protocol ProtocolName < protocol list >
 *  \endcode
 *  All the protocols listed between angle brackets are considered part of the ProtocolName protocol.
 */
OBJC_EXPORT BOOL protocol_conformsToProtocol(Protocol *proto, Protocol *other)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 判断两个协议是否相等
 * 
 * @param proto A protocol.
 * @param other A protocol.
 * 
 * @return \c YES if \e proto is the same as \e other, otherwise \c NO.
 */
OBJC_EXPORT BOOL protocol_isEqual(Protocol *proto, Protocol *other)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取协议名
 * 
 * @param p A protocol.
 * 
 * @return The name of the protocol \e p as a C string.
 */
OBJC_EXPORT const char *protocol_getName(Protocol *p)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 返回一个指定的方法的方法描述结构给定的协议。
 * 
 * @param p A protocol.
 * @param aSel A selector.
 * @param isRequiredMethod A Boolean value that indicates whether aSel is a required method.
 * @param isInstanceMethod A Boolean value that indicates whether aSel is an instance method.
 * 
 * @return An \c objc_method_description structure that describes the method specified by \e aSel,
 *  \e isRequiredMethod, and \e isInstanceMethod for the protocol \e p.
 *  If the protocol does not contain the specified method, returns an \c objc_method_description structure
 *  with the value \c {NULL, \c NULL}.
 * 
 * @note This function recursively searches any protocols that this protocol conforms to.
 */
OBJC_EXPORT struct objc_method_description protocol_getMethodDescription(Protocol *p, SEL aSel, BOOL isRequiredMethod, BOOL isInstanceMethod)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 返回一个指定的方法的方法描述结构给定的协议。
 * 
 * @param p A protocol.
 * @param isRequiredMethod A Boolean value that indicates whether returned methods should
 *  be required methods (pass YES to specify required methods).
 * @param isInstanceMethod A Boolean value that indicates whether returned methods should
 *  be instance methods (pass YES to specify instance methods).
 * @param outCount Upon return, contains the number of method description structures in the returned array.
 * 
 * @return A C array of \c objc_method_description structures containing the names and types of \e p's methods 
 *  specified by \e isRequiredMethod and \e isInstanceMethod. The array contains \c *outCount pointers followed
 *  by a \c NULL terminator. You must free the list with \c free().
 *  If the protocol declares no methods that meet the specification, \c NULL is returned and \c *outCount is 0.
 * 
 * @note Methods in other protocols adopted by this protocol are not included.
 */
OBJC_EXPORT struct objc_method_description *protocol_copyMethodDescriptionList(Protocol *p, BOOL isRequiredMethod, BOOL isInstanceMethod, unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取协议属性
 * 
 * @param proto A protocol.
 * @param name The name of a property.
 * @param isRequiredProperty A Boolean value that indicates whether name is a required property.
 * @param isInstanceProperty A Boolean value that indicates whether name is a required property.
 * 
 * @return The property specified by \e name, \e isRequiredProperty, and \e isInstanceProperty for \e proto,
 *  or \c NULL if none of \e proto's properties meets the specification.
 */
OBJC_EXPORT objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 拷贝协议的属性列表
 * 
 * @param proto A protocol.
 * @param outCount Upon return, contains the number of elements in the returned array.
 * 
 * @return A C array of pointers of type \c objc_property_t describing the properties declared by \e proto.
 *  Any properties declared by other protocols adopted by this protocol are not included. The array contains
 *  \c *outCount pointers followed by a \c NULL terminator. You must free the array with \c free().
 *  If the protocol declares no properties, \c NULL is returned and \c *outCount is \c 0.
 */
OBJC_EXPORT objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 拷贝协议列表(线程不安全)
 * 
 * @param proto A protocol.
 * @param outCount Upon return, contains the number of elements in the returned array.
 * 
 * @return A C array of protocols adopted by \e proto. The array contains \e *outCount pointers
 *  followed by a \c NULL terminator. You must free the array with \c free().
 *  If the protocol declares no properties, \c NULL is returned and \c *outCount is \c 0.
 */
OBJC_EXPORT Protocol * __unsafe_unretained *protocol_copyProtocolList(Protocol *proto, unsigned int *outCount)
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 创建一个新的协议
 * \c objc_registerProtocol()
 * 
 * @param name The name of the protocol to create.
 *
 * @return The Protocol instance on success, \c nil if a protocol
 *  with the same name already exists. 
 * @note There is no dispose method for this. 
 */
OBJC_EXPORT Protocol *objc_allocateProtocol(const char *name) 
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 注册协议
 * will be ready for use and is immutable after this.
 * 
 * @param proto The protocol you want to register.
 */
OBJC_EXPORT void objc_registerProtocol(Protocol *proto) 
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 给协议添加一个方法
 * 
 * @param proto The protocol to add a method to.
 * @param name The name of the method to add.
 * @param types A C string that represents the method signature.
 * @param isRequiredMethod YES if the method is not an optional method.
 * @param isInstanceMethod YES if the method is an instance method. 
 */
OBJC_EXPORT void protocol_addMethodDescription(Protocol *proto, SEL name, const char *types, BOOL isRequiredMethod, BOOL isInstanceMethod) 
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 添加一个协议
 * added to must still be under construction, while the additional protocol
 * must be already constructed.
 * 
 * @param proto The protocol you want to add to, it must be under construction.
 * @param addition The protocol you want to incorporate into \e proto, it must be registered.
 */
OBJC_EXPORT void protocol_addProtocol(Protocol *proto, Protocol *addition) 
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 给协议添加一个属性
 * 
 * @param proto The protocol to add a property to.
 * @param name The name of the property.
 * @param attributes An array of property attributes.
 * @param attributeCount The number of attributes in \e attributes.
 * @param isRequiredProperty YES if the property (accessor methods) is not optional. 
 * @param isInstanceProperty YES if the property (accessor methods) are instance methods. 
 *  This is the only case allowed fo a property, as a result, setting this to NO will 
 *  not add the property to the protocol at all. 
 */
OBJC_EXPORT void protocol_addProperty(Protocol *proto, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount, BOOL isRequiredProperty, BOOL isInstanceProperty)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);
```

## 镜像相关
```objectivec
/* Working with Libraries */
#pragma mark ----Working with Libraries 镜像相关
/** 
 * 获取所有加载的Objective-C框架和动态库的名称
 * 
 * @param outCount The number of names returned.
 * 
 * @return An array of C strings of names. Must be free()'d by caller.
 */
OBJC_EXPORT const char **objc_copyImageNames(unsigned int *outCount) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取指定类所在动态库
 * 
 * @param cls The class you are inquiring about.
 * 
 * @return The name of the library containing this class.
 */
OBJC_EXPORT const char *class_getImageName(Class cls) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 获取指定库或框架中所有类的类名
 * 
 * @param image The library or framework you are inquiring about.
 * @param outCount The number of class names returned.
 * 
 * @return An array of C strings representing the class names.
 */
OBJC_EXPORT const char **objc_copyClassNamesForImage(const char *image, 
                                                     unsigned int *outCount) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

```

## Selector 选择器相关
```objectivec
/* Working with Selectors */
#pragma mark ----Working with Selectors 选择器相关
/** 
 * 获取方法名
 * 返回：C字符串 const char *name = sel_getName(methodSEL);
 * @param sel A pointer of type \c SEL. Pass the selector whose name you wish to determine.
 * 
 * @return A C string indicating the name of the selector.
 */
OBJC_EXPORT const char *sel_getName(SEL sel)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 获取一个Selectors的Uid
 * 
 * @param str A pointer to a C string. Pass the name of the method you wish to register.
 * 
 * @return A pointer of type SEL specifying the selector for the named method.
 * 
 * @note The implementation of this method is identical to the implementation of \c sel_registerName.
 * @note Prior to OS X version 10.0, this method tried to find the selector mapped to the given name
 *  and returned \c NULL if the selector was not found. This was changed for safety, because it was
 *  observed that many of the callers of this function did not check the return value for \c NULL.
 */
OBJC_EXPORT SEL sel_getUid(const char *str)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 注册Selector
 * name to a selector, and returns the selector value.
 * 
 * @param str A pointer to a C string. Pass the name of the method you wish to register.
 * 
 * @return A pointer of type SEL specifying the selector for the named method.
 * 
 * @note You must register a method name with the Objective-C runtime system to obtain the
 *  method’s selector before you can add the method to a class definition. If the method name
 *  has already been registered, this function simply returns the selector.
 */
OBJC_EXPORT SEL sel_registerName(const char *str)
    __OSX_AVAILABLE_STARTING(__MAC_10_0, __IPHONE_2_0);

/** 
 * 注册Selectors.
 * 
 * @param lhs The selector to compare with rhs.
 * @param rhs The selector to compare with lhs.
 * 
 * @return \c YES if \e rhs and \e rhs are equal, otherwise \c NO.
 * 
 * @note sel_isEqual is equivalent to ==.
 */
OBJC_EXPORT BOOL sel_isEqual(SEL lhs, SEL rhs) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);
```

## Objective-C Language 特征
```objectivec
/* Objective-C Language Features */
#pragma mRK --- Objective-C Language 特征
/** 
 * This function is inserted by the compiler when a mutation
 * is detected during a foreach iteration. It gets called 
 * when a mutation occurs, and the enumerationMutationHandler
 * is enacted if it is set up. A fatal error occurs if a handler is not set up.
 *
 * @param obj The object being mutated.
 * 
 */
OBJC_EXPORT void objc_enumerationMutation(id obj) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * Sets the current mutation handler. 
 * 
 * @param handler Function pointer to the new mutation handler.
 */
OBJC_EXPORT void objc_setEnumerationMutationHandler(void (*handler)(id)) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * Set the function to be called by objc_msgForward.
 * 
 * @param fwd Function to be jumped to by objc_msgForward.
 * @param fwd_stret Function to be jumped to by objc_msgForward_stret.
 * 
 * @see message.h::_objc_msgForward
 */
OBJC_EXPORT void objc_setForwardHandler(void *fwd, void *fwd_stret) 
     __OSX_AVAILABLE_STARTING(__MAC_10_5, __IPHONE_2_0);

/** 
 * 创建一个指针函数的指针，该函数调用时会调用特定的block
 * 参数block的签名必须是method_return_type ^(id self, method_args …)形式的。该方法能让我们使用block作为IMP。
 * 
 * @param block The block that implements this method. Its signature should
 *  be: method_return_type ^(id self, method_args...). 
 *  The selector is not available as a parameter to this block.
 *  The block is copied with \c Block_copy().
 * 
 * @return The IMP that calls this block. Must be disposed of with
 *  \c imp_removeBlock.
 */
OBJC_EXPORT IMP imp_implementationWithBlock(id block)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 返回与IMP(使用imp_implementationWithBlock创建的)相关的block
 * \c imp_implementationWithBlock.
 * 
 * @param anImp The IMP that calls this block.
 * 
 * @return The block called by \e anImp.
 */
OBJC_EXPORT id imp_getBlock(IMP anImp)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * 解除block与IMP(使用imp_implementationWithBlock创建的)的关联关系，并释放block的拷贝
 * 
 * @param anImp An IMP that was created using \c imp_implementationWithBlock.
 * 
 * @return YES if the block was released successfully, NO otherwise. 
 *  (For example, the block might not have been used to create an IMP previously).
 */
OBJC_EXPORT BOOL imp_removeBlock(IMP anImp)
     __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_4_3);

/** 
 * This loads the object referenced by a weak pointer and returns it, after
 * retaining and autoreleasing the object to ensure that it stays alive
 * long enough for the caller to use it. This function would be used
 * anywhere a __weak variable is used in an expression.
 * 
 * @param location The weak pointer address
 * 
 * @return The object pointed to by \e location, or \c nil if \e location is \c nil.
 */
OBJC_EXPORT id objc_loadWeak(id *location)
    __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_5_0);

/** 
 * This function stores a new value into a __weak variable. It would
 * be used anywhere a __weak variable is the target of an assignment.
 * 
 * @param location The address of the weak pointer itself
 * @param obj The new object this weak ptr should now point to
 * 
 * @return The value stored into \e location, i.e. \e obj
 */
OBJC_EXPORT id objc_storeWeak(id *location, id obj) 
    __OSX_AVAILABLE_STARTING(__MAC_10_7, __IPHONE_5_0);
```

## 关联引用
```objectivec
#pragma mark --Associative References
/* Associative References */

/**http://www.cnblogs.com/ndyBlog/p/5189324.html
 * Policies related to associative references.
 * These are options to objc_setAssociatedObject()
 */
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};

    
/** 
 * 设置关联对象
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * @param value The value to associate with the key key for object. Pass nil to clear an existing association.
 * @param policy The policy for the association. For possible values, see “Associative Object Behaviors.”
 * 
 * @see objc_setAssociatedObject
 * @see objc_removeAssociatedObjects
 */
OBJC_EXPORT void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);

/** 
 * 根据Key获取关联的对象
 * 
 * @param object The source object for the association.
 * @param key The key for the association.
 * 
 * @return The value associated with the key \e key for \e object.
 * 
 * @see objc_setAssociatedObject
 */
OBJC_EXPORT id objc_getAssociatedObject(id object, const void *key)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);

/** 
 * 移除当前对象的所有关联
 * 
 * @param object An object that maintains associated objects.
 * 
 * @note The main purpose of this function is to make it easy to return an object 
 *  to a "pristine state”. You should not use this function for general removal of
 *  associations from objects, since it also removes associations that other clients
 *  may have added to the object. Typically you should use \c objc_setAssociatedObject 
 *  with a nil value to clear an association.
 * 
 * @see objc_setAssociatedObject
 * @see objc_getAssociatedObject
 */
OBJC_EXPORT void objc_removeAssociatedObjects(id object)
    __OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);
```

```objectivec
#pragma mark ---- Type Encodings 类型编码
#define _C_ID       '@'
#define _C_CLASS    '#'
#define _C_SEL      ':'
#define _C_CHR      'c'
#define _C_UCHR     'C'
#define _C_SHT      's'
#define _C_USHT     'S'
#define _C_INT      'i'
#define _C_UINT     'I'
#define _C_LNG      'l'
#define _C_ULNG     'L'
#define _C_LNG_LNG  'q'
#define _C_ULNG_LNG 'Q'
#define _C_FLT      'f'
#define _C_DBL      'd'
#define _C_BFLD     'b'
#define _C_BOOL     'B'
#define _C_VOID     'v'
#define _C_UNDEF    '?'
#define _C_PTR      '^'
#define _C_CHARPTR  '*'
#define _C_ATOM     '%'
#define _C_ARY_B    '['
#define _C_ARY_E    ']'
#define _C_UNION_B  '('
#define _C_UNION_E  ')'
#define _C_STRUCT_B '{'
#define _C_STRUCT_E '}'
#define _C_VECTOR   '!'
#define _C_CONST    'r'


//获取协议中指定条件的方法的方法描述列表
struct objc_method_description_list {
        int count;
        struct objc_method_description list[1];
};

//获取协议列表
struct objc_protocol_list {
    struct objc_protocol_list *next;
    long count;
    Protocol *list[1];
};


struct objc_category {
    char *category_name                                //分类名;
    char *class_name                                   //类名;
    struct objc_method_list *instance_methods          //实例方法列表
    struct objc_method_list *class_methods             //类方法列表
    struct objc_protocol_list *protocols               //协议列表


struct objc_ivar {
    char *ivar_name   //成员变量名                             OBJC2_UNAVAILABLE;
    char *ivar_type   //成员变量encode类型                           OBJC2_UNAVAILABLE;
    int ivar_offset   //成员变量偏移地址                        OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
}                                                            OBJC2_UNAVAILABLE;

struct objc_ivar_list {         //成员变量列表
    int ivar_count              //成员变量数量                 OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_ivar ivar_list[1]                            OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;


struct objc_method {           //对象方法
    SEL method_name            //方法名                               OBJC2_UNAVAILABLE;
    char *method_types         //方法类型                              OBJC2_UNAVAILABLE;
    IMP method_imp             //方法实现                              OBJC2_UNAVAILABLE;
}

struct objc_method_list {       //方法列表
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;

    int method_count           //方法数量                     OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;
```

