---
layout: post
title: "AdaptivePhotos: Using UIKit Traits and Size Classes"
category:  Learn Apple Sample Code
---

这次来研究一下AdaptivePhotos: Using UIKit Traits and Size Classes这个项目，可以在Apple Sample Code中找到这个项目。
这个项目分两部分，通过两种方式来处理屏幕旋转：代码和Storyboard。UI结构就是UISplitViewController + UINavigationController。

# 知识点

## User.swift
1. flatMap方法，flatMap和map相似，map是对数组中的每个元素进行转换返回一个数组结果，flatMap也是对数组中的每个元素进行转换，但是会对转换结果进行解包操作，并过滤掉为nil的结果。
说到map就要提一下reduce，reduce操作是对数组中的元素组合计算为一个值。


## OverlayView.swift
1. 当用代码创建View的时候translatesAutoresizingMaskIntoConstraints属性默认为true，当需要用代码创建约束的时候需要把此属性置为false；当在Storyboard中创建的View时，此属性默认为false。
2. 所有的约束都创建好的时候，掉用NSLayoutConstraint.activateConstraints来激活约束。
3. UIFont.preferredFontForTextStyle(UIFontTextStyleBody)
4. 监听UIContentSizeCategoryDidChangeNotification，当用户改变系统的字体大小的时候会发送此通知，以此来改变label的字体。注意在deinit的时候移除通知。
5. 改变label字体的时候，最后要调用invalidateIntrinsicContentSize()来使AutoLayout系统下次布局的时候采用新的intrinsicContentSize。
6. 覆写traitCollectionDidChange来处理旋转操作。


## RatingControl.swift
1. 设置UIImageView的accessibilityLabel：imageView.accessibilityLabel = String.localizedStringWithFormat(NSLocalizedString("%d stars", comment: "X stars"), 1)
2. hitTest返回响应该事件的View。
3.

```Swift
override var isAccessibilityElement: Bool {
        set { /* ignore value */ }
        get { return false }
    }
```


## AboutViewController.swift
1. iOS9之后View添加了一个readableContentGuide属性

```Swift
let readableContentGuide = view.readableContentGuide
constraints += [
      label.leadingAnchor.constraintEqualToAnchor(readableContentGuide.leadingAnchor),
      label.trailingAnchor.constraintEqualToAnchor(readableContentGuide.trailingAnchor),
      headlineLabel.leadingAnchor.constraintEqualToAnchor(readableContentGuide.leadingAnchor),
      headlineLabel.trailingAnchor.constraintEqualToAnchor(readableContentGuide.trailingAnchor)
]
```
2. 覆写willTransitionToTraitCollection来处理旋转。


## Profile Scene
1. AutoLayout的时候UILabel有默认高度，可以不用再约束高度。
