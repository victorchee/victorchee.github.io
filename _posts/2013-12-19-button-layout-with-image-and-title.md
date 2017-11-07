---
layout: post
title: "如何布局包含Image和Title的UIButton"
category:  Skill
---
UIButton中的titleEdgeInsets和imageEdgeInsets可以管理button中image和title的布局。
如果对其理解不够深入，用纯数字进行布局管理，经过不断的调试，还是能试出来的，但是如果改变了图片大小或文字长度，又要再来一遍。
作为程序猿，我们不应该放弃任何一个偷懒的机会。

- 默认情况下，是图片在左，文字在右，垂直居中显示，如下图：

```Objective-C
button.titleEdgeInsets = UIEdgeInsetsZero;
button.imageEdgeInsets = UIEdgeInsetsZero;
```  

![1](/images/button-layout-with-image-and-title/1.png)

- 设置如下布局后，图片和文字都居中显示。

```Objective-C
button.titleEdgeInsets = UIEdgeInsetsMake(0, -button.imageView.frame.size.width, 0, 0);
// button.imageEdgeInsets = UIEdgeInsetsMake(0, 0, 0, -button.titleLabel.frame.size.width);
// 由于iOS8中titleLabel的size为0，用上面这样设置有问题，修改一下即可
button.imageEdgeInsets = UIEdgeInsetsMake(0, 0, 0, -button.titleLabel.intrinsicContentSize.width);
```

![2](/images/button-layout-with-image-and-title/2.png)

- 如果想图片在上，文字在下，水平居中显示，则按下面设置即可：  

```Objective-C
button.titleEdgeInsets = UIEdgeInsetsMake(0, -button.imageView.frame.size.width, -button.imageView.frame.size.height, 0);
// button.imageEdgeInsets = UIEdgeInsetsMake(-button.titleLabel.frame.size.height, 0, 0, -button.titleLabel.frame.size.width);
// 由于iOS8中titleLabel的size为0，用上面这样设置有问题，修改一下即可
button.imageEdgeInsets = UIEdgeInsetsMake(-button.titleLabel.intrinsicContentSize.height, 0, 0, -button.titleLabel.intrinsicContentSize.width);
```

![3](/images/button-layout-with-image-and-title/3.png)

如果觉得图片和文字离的太近了，稍微分开一点：  

```Objective-C
CGFloat offset = 40.0f;
button.titleEdgeInsets = UIEdgeInsetsMake(0, -button.imageView.frame.size.width, -button.imageView.frame.size.height-offset/2, 0);
// button.imageEdgeInsets = UIEdgeInsetsMake(-button.titleLabel.frame.size.height-offset/2, 0, 0, -button.titleLabel.frame.size.width);
// 由于iOS8中titleLabel的size为0，用上面这样设置有问题，修改一下即可
button.imageEdgeInsets = UIEdgeInsetsMake(-button.titleLabel.intrinsicContentSize.height-offset/2, 0, 0, -button.titleLabel.intrinsicContentSize.width);
```

![4](/images/button-layout-with-image-and-title/4.png)

- 文字左对齐，图片右对齐

```Objective-C
button.titleEdgeInsets = UIEdgeInsetsMake(0, -button.imageView.frame.size.width - button.frame.size.width + button.titleLabel.intrinsicContentSize.width, 0, 0);
    button.imageEdgeInsets = UIEdgeInsetsMake(0, 0, 0, -button.titleLabel.frame.size.width - button.frame.size.width + button.imageView.frame.size.width);
```

![5](/images/button-layout-with-image-and-title/5.png)
