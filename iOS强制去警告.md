title: iOS清除代码警告
date: 2016-02-24 12:15:19
tags:
type: "categories"
description: Xcode去警告
---
项目中存在很多黄色感叹号时，首先视觉上就觉得不舒服，并且有些警告可能产生Bug。当有些警告不想看到黄色感叹号时，可以采取一些特殊的手段强制消除警告！
<!--more-->
### 一、去警告基本语法
```objectivec
#pragma clang diagnostic push
#pragma clang diagnostic ignored "警告名称"
//被夹在这中间的代码针对于此警告都会无视并且不显示出来
#pragma clang diagnostic pop
```

**这段代码的基本流程:**    
 1. push 当前警告入栈
 2. 忽略我们要消除的警告
 3. 执行会产生警告的代码
 4. pop 警告出栈——恢复之前的状态

例如：		    

```objectivec
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
    return [@"contentText" sizeWithFont:[UIFont systemFontOfSize:14] constrainedToSize:CGSizeMake(320, CGFLOAT_MAX) lineBreakMode:0];
#pragma clang diagnostic pop
```

### 二、常见编译警告类型	

```objectivec
-Wincompatible-pointer-types    指针类型不匹配
-Wincomplete-implementation     没有实现已声明的方法
-Wprotocol                      没有实现协议的方法
-Wimplicit-function-declaration 尚未声明的函数(通常指c函数)
-Warc-performSelector-leaks     使用performSelector可能会出现泄漏(该警告在xcode4.3.1中没出现过,网上流传说4.2使用performselector:withObject: 就会得到该警告)
-Wdeprecated-declarations       使用了不推荐使用的方法(如[UILabel setFont:(UIFont*)])
-Wunused-variable               含有没有被使用的变量
-Wundeclared-selector           未定义selector方法
```

阅读更多
警告类型大全：http://fuckingclangwarnings.com/



### 	三、去警告宏定义
如果需要忽略的警告有很多，可以定义一个宏，简化使用

```objectivec
//忽略PerformSelector警告
#define SUPPRESS_PerformSelectorLeak_WARNING(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Warc-performSelector-leaks\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0)

//忽略未定义方法警告
#define  SUPPRESS_Undeclaredselector_WARNING(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Wundeclared-selector\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0)

//忽略过期API警告
#define SUPPRESS_DEPRECATED_WARNING(Stuff) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Wdeprecated-declarations\"") \
Stuff; \
_Pragma("clang diagnostic pop") \
} while (0
```
### 四、获取警告类型
![警告][1]
  -W    是前缀,表示打开这种类型的警告，默认开启
  -Wno- 关闭某种类型的警告
关闭警告如：`-Wno-unused-variable`、`-Wno-shorten-64-to-32`

### 五、关闭工程中指定类型的警告
1、屏蔽项目指定文件指定类型警告
`-Wno-unused-variable`
![屏蔽指定文件警告][2]				  
全部类型`-w`


2、屏蔽工程指定类型警告
Taget- buld Setting -other warning Flags -Wno-unused-variable
![屏蔽工程指定类型警告][3]

重新编译,文件中的指定类型警告全部消失了!!!!

### 六、屏蔽cocoapod引入的第三方警告
对于我们使用cocoapod引入的第三方,我们可以在podfile文件中增加一句`inhibit_all_warnings!` 来屏蔽pod的工程的任何警告。

```objectivec
# platform :ios, '8.0'
# Uncomment this line if you're using Swift
# use_frameworks!

inhibit_all_warnings!
target 'RuntimeLearn' do
pod 'UITextView+Placeholder', '~> 1.1.1'
pod 'MJRefresh', '~> 3.1.0'
#也可以单独设置打开编译警告就好了
pod 'Alamofire', '~> 3.0.0-beta.3', :inhibit_warnings => true

end
```
### 七、工程常见警告
1. 苹果app支持arm64以后会有一个问题：NSInteger变成64位了，和原来的int （%d）不匹配，会报如下warning:
`Values of type 'NSInteger' should not be used as format arguments;`
`add an explicit cast to 'long' instead`

解决办法：
```objectivec
1、系统推荐方法               [NSString stringWithFormat:@“%ld", (long)number];
2、类型强制                  [NSString stringWithFormat:@"%d", (int)number];
3、使用NSNumber类型          [NSString stringWithFormat:@“%@", @(number)];
4、对应数字是一个size_t的值    [NSString stringWithFormat:@“%zd",number];
```

建议使用第4种方式。
`use %zd for signed, %tu for unsigned, and %tx for hex.`
关于格式化规范:[String Format Specifiers。](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Strings/Articles/formatSpecifiers.html)
[Stackoverflow](http://stackoverflow.com/questions/18893880/alternatives-to-type-casting-when-formatting-nsuinteger-on-32-and-64-bit-archi?rq=1)

    PS：切记，警告屏蔽只针对于无关紧要的警告，有些警告是代码编写本身有误，可能
    引起Bug，需要及时修复。屏蔽掉无关紧要的Bug可以让我们快速找到需要修复的警告，
    让工程更整洁，让App更稳定。
    
### 参考：

http://nshipster.cn/clang-diagnostics/
http://oleb.net/blog/2013/04/compiler-warnings-for-objective-c-developers/

  [1]: http://7xr7vj.com1.z0.glb.clouddn.com/findwarning.png
  [2]: http://7xr7vj.com1.z0.glb.clouddn.com/2016/03/01/FoS6FJiNkETzlbFkUp0NHPkQm2PY908.png
  [3]: http://7xr7vj.com1.z0.glb.clouddn.com/2016/03/01/FmNscVylZviDEH2QfAsVRhy86DA7298.png