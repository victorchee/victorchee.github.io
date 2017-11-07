---
layout: post
title: "自定义UIActivity"
category: Tutorial
---

自定义UIActivity其实很简单，按照文档一步步来就可以了。完整项目参考https://github.com/victorchee/CustomActivity

先放上最终效果图

![1](/images/custom-UIActivity/1.png)

新建个类继承自UIActivity；
重载以下方法：
- activityType
返回NSStringFromClass([self class])就可以了。

```Swift
override func activityType() -> String? {
        return toString(self)
}
```

- activityTitle
显示的标题。

```Swift
override func activityTitle() -> String? {
        return "Share"
}
```

- activityImage
显示的图片。

```Swift
override func activityImage() -> UIImage? {
        return UIImage(named: "logo")
}
```

- canPerformWithActivityItems:
判断能否支持，返回NO的话这个Activity是不显示的。

```Swift
override func canPerformWithActivityItems(activityItems: [AnyObject]) -> Bool {
        return true
}
```

- prepareWithActivityItems:
预处理。

```Swift
var title: String?
    var image: UIImage?
    var url: NSURL?

override func prepareWithActivityItems(activityItems: [AnyObject]) {
        for item in activityItems {
            if item is String {
                title = item as? String
            } else if item is UIImage {
                image = item as? UIImage
            } else if item is NSURL {
                url = item as? NSURL
            }
        }
}
```

- activityCategory
Category分为两类：Share和Action。

```Swift
override class func activityCategory() -> UIActivityCategory {
        return .Share // .Action
}
```

- performActivity
点击会调用这个，在此处理业务逻辑，结束后在主线程回调activityDidFinish。

```Swift
override func performActivity() {
        print("share: \(title) & \(url)")
        dispatch_async(dispatch_get_main_queue(), { () -> Void in
            self.activityDidFinish(true)
        })
}
```


使用：
在UIActivityViewController中使用。

```Swift
let shareActivity = ShareActivity()
let actionActivity = ActionActivity()
let activityViewController = UIActivityViewController(activityItems: ["message", NSURL(string: "http://victorchee.github.io")!], applicationActivities: [shareActivity, actionActivity])
activityViewController.excludedActivityTypes = [UIActivityTypePrint, UIActivityTypeAssignToContact]
self.presentViewController(activityViewController, animated: true, completion: nil)
```
