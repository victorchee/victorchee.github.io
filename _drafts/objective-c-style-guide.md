---
layout: post
title: "Objective-C编码规范"
category: CodingStyle
---

# Objective-C编码规范

## 介绍

对于团队，如果每个人的代码风格不统一，阅读或修改其他人的代码会非常困难，这样不易于保持代码的一致性并难以进行Code Review。

对于个人，代码规范是对自身编码习惯的一种监督，如果没有这种监督，有时候因为偷懒，会写出难看的代码，时间长了自己都看不懂。这样对于代码的维护性是不利的。

## 背景

这里有些关于编码风格Apple官方文档，如果有些东西没有提及，可以在以下文档来查找更多细节：

* [The Objective-C Programming Language](http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html)
* [Cocoa Fundamentals Guide](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/Introduction.html)
* [Coding Guidelines for Cocoa](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
* [iOS App Programming Guide](http://developer.apple.com/library/ios/#documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)

## 目录

* [代码组织](#code-organization)
* [空格](#spacing)
* [注释](#comments)
* [命名](#naming)
  * [下划线](#underscores)
* [方法](#methods)
* [变量](#variables)
* [属性特性](#property-attributes)
* [字面值](#literals)
* [常量](#constants)
* [枚举类型](#enumerated-types)
* [Case语句](#case-statements)
* [私有属性](#private-properties)
* [布尔值](#booleans)
* [条件语句](#conditionals)
  * [三元操作符](#ternary-operator)
* [Init方法](#init-methods)
* [类构造方法](#class-constructor-methods)
* [CGRect函数](#cgrect-functions)
* [错误处理](#error-handling)
* [单例模式](#singletons)

<b id="code-organization"></b>
## 代码组织

在方法分组和protocol/delegate实现中使用`#pragma mark -`来分类，一般要遵循以下结构：

**应该:**

```objc
#pragma mark - Lifecycle
- (instancetype)init {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors
- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions/Event Response
- (IBAction)submitData:(id)sender {}
- (void)someButtonDidTapped:(UIButton *)button

#pragma mark - Protocol conformance
#pragma mark - UITextFieldDelegate
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate

#pragma mark - Public
- (void)publicMethod {}

#pragma mark - Private
- (void)privateMethod {}
```

<b id="spacing"></b>
## 空格

* 缩进使用**4**个空格，不要在工程里使用Tab，在`Xcode > Preferences > Text Editing > Indentation`将Tab和自动缩进都设置为默认的4个空格。
* 方法大括号和其他大括号(`if`/`else`/`switch`/`while` 等.)总是在同一行语句打开但在新行中关闭。

**应该:**

```objc
if (user.isHappy) {
    //Do something
} else {
    //Do something else
}
```

**不应该:**

```objc
if (user.isHappy)
{
  //Do something
}
else {
  //Do something else
}
```

* 在方法之间应该有且只有一行空行，这样有利于在视觉上更清晰和更易于组织。在方法内的空行应该分离功能，但通常都抽离出来成为一个新方法。
* 应该避免以冒号对齐的方式来调用方法。因为有时方法签名可能有3个以上的冒号和冒号对齐会使代码更加难读。请**不要**这样做，尽管冒号对齐的方法包含代码块，因为Xcode的对齐方式令它难以辨认。

**应该:**

```objc
// blocks are easily readable
[UIView animateWithDuration:1.0 animations:^{
  // something
} completion:^(BOOL finished) {
  // something
}];
```

**不应该:**

```objc
// colon-aligning makes the block indentation hard to read
[UIView animateWithDuration:1.0
                 animations:^{
                     // something
                 }
                 completion:^(BOOL finished) {
                     // something
                 }];
```

<b id="comments"></b>
## 注释

当需要注释时，注释应该用来解释这段特殊代码**为什么**要这样做。任何被使用的注释都必须保持最新或被删除。

一般都避免使用块注释，因为代码尽可能做到自解释，只有当断断续续或几行代码时才需要注释。*例外：这不应用在生成文档的注释*

<b id="naming"></b>
## 命名

Apple命名规则尽可能坚持，特别是与这些相关的[memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) ([NARC](http://stackoverflow.com/a/2865194/340508))。

变量命名不要使用拼音，数字，除非你无法找到合适的英文单词。当脑子里不能直接想出单词的话，查个字典绝对不会浪费时间。

**应该:**

```objc
UIButton *settingsButton;
```

**不应该:**

```objc
UIButton *setBut;
```

常量应该使用驼峰式命名规则，所有的单词首字母大写和加上与类名有关的前缀。

**应该:**

```objc
static NSTimeInterval const TutorialViewControllerNavigationFadeAnimationDuration = 0.3;
```

**不应该:**

```objc
static NSTimeInterval const fadetime = 1.7;
```

属性也是使用驼峰式，但首单词的首字母小写。

**应该:**

```objc
@property (copy, nonatomic) NSString *descriptiveVariableName;
```

**不应该:**

```objc
id varnm;
```

<b id="underscores"></b>
### 下划线

当使用属性时，实例变量应该使用`self.`来访问和改变。这就意味着所有属性将会视觉效果不同，因为它们前面都有`self.`。

但有一个特例：在初始化方法里，实例变量(例如，_variableName)应该直接被使用来避免getters/setters潜在的副作用。

局部变量不应该包含下划线。

<b id="methods"></b>
## 方法

Objective-C中的函数不同于其他语言，函数名是按参数隔开的。初看起来是有点怪，但是习惯了就会发现这样和命名方式可读性极好，就像一句话一样。所以当你的函数名不能流利的读出来的话，那么它的命名肯定就有问题。

这里面值得注意的几点：

- 最左边的（+、-）和返回类型的左括号之间有一个空格，返回类型的右括号与第一个参数之间没有空格，在方法各个段之间应该也有一个空格(符合Apple的风格)；
- 函数名中每一段描述都以小写字母开头，描述的写法要考虑清晰明确，参考变量命名；
- 从描述中可以看出对应参数的意义。

**应该:**

```objc
- (instancetype)initWithWidth:(CGFloat)width height:(CGFloat)height;
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
```

**不应该:**

```objc
// +后面应该有个空格，参数命名使用缩略语，最后一个参数描述Selector首字母大写了
+(UIButton *)createButton:(MUButtonItemAttribute *)attr target:(id)target Selector:(SEL)sel;

// 有一些多余的空格
+ (AppDelegate *) appDelegate;
- (void)hideTabbar :(BOOL)status;

// 加减号和括号之间要有空格
-(void)pushEvent:(NSDictionary*)dictionary target:(id)target;

// 参数名和参数要对应，含义清晰
- (void)setNavBackButton:(NSString *)string;
- (id)init:(NSString*)nickName phoneNumber:(NSString *)phoneNumber;
- (void)setTextFieldBackGroud:(CGRect)rect;
- (id)init:(NSInteger)flag;
- (void)requestHealthRecord:(NSString *)typeStr;

// 命名风格混乱
+ (NSInteger) GetServerBack:(NSString *)serverName
                 path_Param:(NSMutableDictionary*)path_Param
                query_Param:(NSMutableDictionary*)query_Param
                 body_Param:(NSMutableDictionary*)body_Param
                     method:(NSInteger)method
                returnValue:(NSMutableDictionary*)returnValue;

// 参数之间用and、with，多此一举
- (id)initWithFrame:(CGRect)frame andImageView:(UIImageView*)imageView withImageFrame:(CGRect)imageInitFrame withImageUrl:(NSURL*)imageUrl;
```

<b id="variables"></b>
## 变量

变量尽量以描述性的方式来命名。单个字符的变量命名应该尽量避免，最重要的是清晰其次才考虑长短，如果为了简短而含糊不清，是不可取的。当然，有时候为了语义清晰导致变量名很长，那首先应该考虑这个变量是否可以换种方式替换掉，即逻辑结构修改；如果没办法改，那长就长吧。

星号表示变量是指针。例如， `NSString *text` 既不是 `NSString* text` 也不是 `NSString * text`，除了一些特殊情况下常量。

[私有属性](#private-properties) 应该尽可能代替实例变量的使用。尽管使用实例变量是一种有效的方式，但更偏向于使用属性来保持代码一致性。

通过使用'back'属性(_variable，变量名前面有下划线)直接访问实例变量应该尽量避免，除了在初始化方法(`init`, `initWithCoder:`, 等…)，`dealloc` 方法和自定义的setters和getters。想了解关于如何在初始化方法和dealloc直接使用Accessor方法的更多信息，查看[这里](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6)

**应该:**

```objc
@interface Tutorial : NSObject

@property (copy, nonatomic) NSString *tutorialName;

@end
```

**不应该:**

```objc
@interface Tutorial : NSObject {
  NSString *tutorialName;
}
```

<b id="property-attributes"></b>
## 属性特性

所有属性特性应该显式地列出来，有助于阅读代码，不要使用retain，而应该使用strong来代替。属性特性的顺序应该是storage、atomicity，与在Interface Builder连接UI元素时自动生成代码一致。

**应该:**

```objc
@property (weak, nonatomic) IBOutlet UIView *containerView;
@property (copy, nonatomic) NSString *tutorialName;
```

**不应该:**

```objc
@property (nonatomic, weak) IBOutlet UIView *containerView;
@property (nonatomic) NSString *tutorialName;
```

NSString应该使用`copy` 而不是 `strong`的属性特性。因为即使你声明一个`NSString`的属性，有人可能传入一个`NSMutableString`的实例，然后在你没有注意的情况下修改它。

**应该:**

```objc
@property (copy, nonatomic) NSString *tutorialName;
```

**不应该:**

```objc
@property (strong, nonatomic) NSString *tutorialName;
```

<b id="literals"></b>
## 字面值

`NSString`, `NSDictionary`, `NSArray`, 和 `NSNumber`的字面值应该在创建这些类的不可变实例时被使用。请特别注意`nil`值不能传入`NSArray`和`NSDictionary`字面值，因为这样会导致crash。

**应该:**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone": @"Kate", @"iPad": @"Kamal", @"Mobile Web": @"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingStreetNumber = @10018;
```

**不应该:**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingStreetNumber = [NSNumber numberWithInteger:10018];
```
<b id="constants"></b>
## 常量

常量是容易重复被使用和无需通过查找和代替就能快速修改值。常量应该使用`static`来声明而不是使用`#define`。

**应该:**

```objc
static NSString * const AboutViewControllerCompanyName = @"company.com";

static CGFloat const ImageThumbnailHeight = 50.0;
```

**不应该:**

```objc
#define CompanyName @"company.com"

#define thumbnailHeight 2
```
<b id="enumerated-types"></b>
## 枚举类型

当使用`enum`时，推荐使用新的固定基本类型规格，因为它有更强的类型检查和代码补全。现在SDK有一个宏`enumdef`来帮助和鼓励你使用固定的基本类型。

**应该:**

```objc
typedef enum : NSUInteger {
    LeftMenuTopItemMain,
    LeftMenuTopItemShows,
    LeftMenuTopItemSchedule,
} LeftMenuTopItemType;
```

你也可以显式地赋值(展示旧的k-style常量定义)：

```objc
typedef enum : NSUInteger {
  PinSizeMin = 1,
  PinSizeMax = 5,
  PinCountMin = 100,
  PinCountMax = 500,
} GlobalConstants;
```

旧的k-style常量定义应该**避免**除非编写Core Foundation C的代码。

**不应该:**

```objc
enum GlobalConstants {
  kMaxPinSize = 5,
  kMaxPinCount = 500,
};
```

<b id="case-statements"></b>
## Case语句

大括号在case语句中并不是必须的，除非编译器强制要求。当一个case语句包含多行代码时，大括号应该加上。

```objc
switch (condition) {
  case 1:
    // ...
    break;
  case 2: {
    // ...
    // Multi-line example using braces
    break;
  }
  case 3:
    // ...
    break;
  default:
    // ...
    break;
}

```
有很多次，当相同代码被多个cases使用时，一个fall-through应该被使用。一个fall-through就是在case最后移除'break'语句，这样就能够允许执行流程跳转到下一个case值。为了代码更加清晰，一个fall-through需要注释一下。


```objc
switch (condition) {
  case 1:
    // ** fall-through! **
  case 2:
    // code executed for values 1 and 2
    break;
  default:
    // ...
    break;
}

```

当在switch使用枚举类型时，'default'是不需要的。例如：

```objc
LeftMenuTopItemType menuType = LeftMenuTopItemMain;

switch (menuType) {
  case LeftMenuTopItemMain:
    // ...
    break;
  case LeftMenuTopItemShows:
    // ...
    break;
  case LeftMenuTopItemSchedule:
    // ...
    break;
}
```

<b id="private-properties"></b>
## 私有属性

私有属性应该在类的实现文件中的类扩展(匿名分类)中声明，命名分类(比如`private`)应该从不使用除非是扩展其他类。匿名分类应该通过使用<headerfile>+Private.h文件的命名规则暴露给测试。

**例如:**

```objc
@interface DetailViewController ()

@property (strong, nonatomic) GADBannerView *googleAdView;
@property (strong, nonatomic) ADBannerView *iAdView;
@property (strong, nonatomic) UIWebView *adXWebView;

@end
```

<b id="booleans"></b>
## 布尔值

Objective-C使用`YES`和`NO`。因为`true`和`false`应该只在CoreFoundation，C或C++代码使用。既然`nil`解析成`NO`，所以没有必要在条件语句比较。不要拿某样东西直接与`YES`比较，因为`YES`被定义为1和一个`BOOL`能被设置为8位。

这是为了在不同文件保持一致性和在视觉上更加简洁而考虑。

**应该:**

```objc
if (someObject) {}
if (![anotherObject boolValue]) {}
```

**不应该:**

```objc
if (someObject == nil) {}
if ([anotherObject boolValue] == NO) {}
if (isAwesome == YES) {} // Never do this.
if (isAwesome == true) {} // Never do this.
```

如果`BOOL`属性的名字是一个形容词，属性就能忽略"is"前缀，但要指定get访问器的惯用名称。例如：

```objc
@property (assign, getter=isEditable) BOOL editable;
```

文字和例子从这里引用[Cocoa Naming Guidelines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE)

<b id="conditionals"></b>
## 条件语句

条件语句主体为了防止出错应该使用大括号包围，即使条件语句主体能够不用大括号编写(如，只用一行代码)。这些错误包括添加第二行代码和期望它成为if语句；还有，[even more dangerous defect](http://programmers.stackexchange.com/a/16530)可能发生在if语句里面一行代码被注释了，然后下一行代码不知不觉地成为if语句的一部分。除此之外，这种风格与其他条件语句的风格保持一致，所以更加容易阅读。

**应该:**

```objc
if (!error) {
  return success;
}
```

**不应该:**

```objc
if (!error)
  return success;
```

或

```objc
if (!error) return success;
```

<b id="ternary-operator"></b>
### 三元操作符

当需要提高代码的清晰性和简洁性时，三元操作符`?:`才会使用。单个条件求值常常需要它。多个条件求值时，如果使用`if`语句或重构成实例变量时，代码会更加易读。一般来说，最好使用三元操作符是在根据条件来赋值的情况下。

Non-boolean的变量与某东西比较，加上括号()会提高可读性。如果被比较的变量是boolean类型，那么就不需要括号。

**应该:**

```objc
NSInteger value = 5;
result = (value != 0) ? x : y;

BOOL isHorizontal = YES;
result = isHorizontal ? x : y;
```

**不应该:**

```objc
result = a > b ? x = c > d ? c : d : y;
```

<b id="init-methods"></b>
## Init方法

Init方法应该遵循Apple生成代码模板的命名规则。返回类型应该使用`instancetype`而不是`id`

```objc
- (instancetype)init {
  self = [super init];
  if (self) {
    // ...
  }
  return self;
}
```

查看关于instancetype的文章[Class Constructor Methods](#class-constructor-methods)

<b id="class-constructor-methods"></b>
## 类构造方法

当类构造方法被使用时，它应该返回类型是`instancetype `而不是`id`。这样确保编译器正确地推断结果类型。


```objc
@interface Airplane
+ (instancetype)airplaneWithType:(AirplaneType)type;
@end
```
关于更多instancetype信息，请查看[NSHipster.com](http://nshipster.com/instancetype/)

<b id="cgrect-functions"></b>
## CGRect函数

当访问`CGRect`里的`x`, `y`, `width`, 或 `height`时，应该使用[`CGGeometry`函数](http://developer.apple.com/library/ios/#documentation/graphicsimaging/reference/CGGeometry/Reference/reference.html)而不是直接通过结构体来访问。引用Apple的`CGGeometry `:

> 在这个参考文档中所有的函数，接受CGRect结构体作为输入，在计算它们结果时隐式地标准化这些rectangles。因此，你的应用程序应该避免直接访问和修改保存在CGRect数据结构中的数据。相反，使用这些函数来操纵rectangles和获取它们的特性。


**应该:**

```objc
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
CGRect frame = CGRectMake(0.0, 0.0, width, height);
```

**不应该:**

```objc
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
CGRect frame = (CGRect){ .origin = CGPointZero, .size = frame.size };
```

<b id="error-handling"></b>
## 错误处理

当方法通过引用来返回一个错误参数，判断返回值而不是错误变量。

**应该:**
```objc
NSError *error;
if (![self trySomethingWithError:&error]) {
  // Handle Error
}
```

**不应该:**
```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
  // Handle Error
}
```

在成功的情况下，有些Apple的APIs记录垃圾值(garbage values)到错误参数(如果non-NULL)，那么判断错误值会导致false负值和crash。

<b id="singletons"></b>
## 单例模式

单例对象应该使用线程安全模式来创建共享实例。

```objc
+ (instancetype)sharedInstance {
  static id sharedInstance = nil;

  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    sharedInstance = [[self alloc] init];
  });

  return sharedInstance;
}
```

这会防止[possible and sometimes prolific crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html).
