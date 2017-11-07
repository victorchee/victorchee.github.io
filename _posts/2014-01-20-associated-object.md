---
layout: post
title: "Objective-C中的关联对象"
category:  Skill
---
有的时候要在对象中存放相关信息，通常的做法是生成对象所属的类的子类，然后用这个子类的对象做存储。但是如果无法生成子类的话，就没有办法了。
幸亏Objective-C提供了另一种实现——关联对象（Associated Object）。

## 介绍

我们可以给某个对象关联许多其他对象，这些对象通过‘键’来区分。
存储对象值的时候可以指明存储策略，存储策略在objc_AssociationPolicy中定义:  

objc_AssociationPolicy | 等效的@property属性
---------------------- | -----------------
OBJC_ASSOCIATION_ASSIGN | assign
OBJC_ASSOCIATION_RETAIN | retain
OBJC_ASSOCIATION_RETAIN_NONATOMIC | retain, nonatomic
OBJC_ASSOCIATION_COPY | copy
OBJC_ASSOCIATION_COPY_NONATOMIC | copy, nonatomic  


通过以下方法管理关联对象：

```Objective-C
// 设置关联对象
void objc_setAssociatedObject(id object, void* key, id value, objc_AssociationPolicy policy)

// 获取关联对象
id objc_getAssociatedObject(id object, void* key)

// 移除所有关联对象
void objc_removeAssociatedObjects(id object)  
```  
要想两个键匹配到同一个值，则必须是完全相同的指针，所以‘键’通常采用静态全局变量。  

## 例子

多说无益，直接上[代码](https://gist.github.com/c6c789b9f6c528773986.git)：  

```Objective-C
#import "ViewController.h"
#import <objc/runtime.h>

@interface ViewController () <UIAlertViewDelegate>

@end

static void *MyAlertViewKey = "MyAlertViewKey";

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

    UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Question"
                                                    message:@"What do you want to do?"
                                                   delegate:self
                                          cancelButtonTitle:@"Cancel"
                                          otherButtonTitles:@"Continue", nil];

    void (^block)(NSInteger) = ^(NSInteger buttonIndex){
        if (buttonIndex == 0) {
            NSLog(@"cancel");
        } else {
            NSLog(@"continue");
        }
    };

    objc_setAssociatedObject(alert, MyAlertViewKey, block, OBJC_ASSOCIATION_COPY);

    [alert show];
}

#pragma mark - alert view delegate
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    void (^block)(NSInteger) = objc_getAssociatedObject(alertView, MyAlertViewKey);
    block(buttonIndex);
}

@end
```

如果一个类里面有多个alert view的话，按照传统的写法，代码会很复杂，而且delegate离的远的话，阅读代码也很麻烦。
用关联对象就简单多了，创建alert view的时候直接把处理每个按钮的逻辑都写好了，这样就无需在两部分代码中切换了。

##注意

- 上面例子中用到block的时候有个问题，block可能要retain某些对象，可能会造成循环引用；
- 使用关联对象会引入难于查找的Bug，使代码难以调试；
- 所以，通常不应选用关联对象，只有在其他做法行不通的时候再考虑使用。
