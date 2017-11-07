---
layout: post
title: "iOS中绘制渐变视图"
category: Tip
---

- 放射渐变  

```Objective-C
CGContextRef context = UIGraphicsGetCurrentContext();

size_t num_locations = 2;
CGColorSpaceRef rgb = CGColorSpaceCreateDeviceRGB();
CGFloat components[8] = {1,1,1,1, // 中心颜色（r， g， b， alpha）
			 .2,.2,.2,1 // 边缘颜色};
CGFloat locations[2] = {0,1};

CGGradientRef gradient = CGGradientCreateWithColorComponents(rgb, components, locations, num_locations);
CGColorSpaceRelease(rgb);
CGRect bounds = self.bounds;
CGPoint center = CGPointMake(CGRectGetMidX(bounds), CGRectGetMidY(bounds));

// 中心发散渐进
CGContextDrawRadialGradient(context, gradient, center, 0, center, bounds.size.width, kCGGradientDrawsAfterEndLocation);

CGGradientRelease(gradient);  
```  

- 线形渐变  

```Objective-C
AGradientLayer *gradientLayer = [CAGradientLayer layer];
gradientLayer.frame = self.view.frame;
gradientLayer.colors = [NSArray arrayWithObjects:(id)baseColor.CGColor, (id)barColor.CGColor, nil];
[self.view.layer insertSublayer:gradientLayer atIndex:0];  
```
