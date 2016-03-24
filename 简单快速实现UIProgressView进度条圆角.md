---
title: 简单快速实现UIProgressView进度条圆角 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

关键方法：
 1. 利用贝塞尔生成圆角图片
 2. 圆角不变形拉伸拉伸图片`resizableImageWithCapInsets`
![](http://7xr7vj.com1.z0.glb.clouddn.com/2016/03/24/FgyebQmfb7Xz26rka6M8clFEcAbd534.png)

代码如下
```objectivec
#import <UIKit/UIKit.h>

@interface UIProgressView (Radius)

- (void)setRadiusTrackColor:(UIColor *)trackColor ;
- (void)setRadiusProgressColor:(UIColor *)progressColor;
- (void)setRadiusTrackColor:(UIColor *)trackColor
              progressColor:(UIColor *)progressColor;

@end

```

```objectivec
#import "UIProgressView+Radius.h"

@implementation UIProgressView (Radius)

- (void)setRadiusTrackColor:(UIColor *)trackColor
{
    UIImage *trackImage = [self imageWithColor:trackColor cornerRadius:self.frame.size.height/2.0];
    [self setTrackImage:trackImage];
}

- (void)setRadiusProgressColor:(UIColor *)progressColor
{
    UIImage *progressImage = [self imageWithColor:progressColor cornerRadius:self.frame.size.height/2.0];
    [self setProgressImage:progressImage];

}

- (void)setRadiusTrackColor:(UIColor *)trackColor
              progressColor:(UIColor *)progressColor
{

    [self setRadiusTrackColor:trackColor];
    [self setRadiusProgressColor:progressColor];
}

//最小尺寸---1px
static CGFloat edgeSizeWithRadius(CGFloat cornerRadius) {
    return cornerRadius * 2 + 1;
}

- (UIImage *)imageWithColor:(UIColor *)color
               cornerRadius:(CGFloat)cornerRadius {
    CGFloat minEdgeSize = edgeSizeWithRadius(cornerRadius);
    CGRect rect = CGRectMake(0, 0, minEdgeSize, minEdgeSize);
    UIBezierPath *roundedRect = [UIBezierPath bezierPathWithRoundedRect:rect cornerRadius:cornerRadius];
    roundedRect.lineWidth = 0;
    UIGraphicsBeginImageContextWithOptions(rect.size, NO, 0.0f);
    [color setFill];
    [roundedRect fill];
    [roundedRect stroke];
    [roundedRect addClip];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return [image resizableImageWithCapInsets:UIEdgeInsetsMake(cornerRadius, cornerRadius, cornerRadius, cornerRadius)];
}

@end
```

### 使用
```
- (void)viewDidLoad {
    [super viewDidLoad];
    UIProgressView *progressView=[[UIProgressView alloc]initWithProgressViewStyle:UIProgressViewStyleDefault];
    progressView.frame=CGRectMake([UIScreen mainScreen].bounds.size.width/2-150, 50, 300, 50);
    [self.view addSubview:progressView];

    [progressView setProgress:0.68 animated:YES];
   
    //修改progressView高度
    progressView.transform=CGAffineTransformMakeScale(1.0, 8.0);
    
    //设置进度条颜色和圆角
    [progressView setRadiusTrackColor:RGBCOLOR(231, 233, 238) progressColor:RGBCOLOR(255, 153,0)];
}

```



