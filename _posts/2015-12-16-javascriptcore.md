---
layout: post
title: "用JavaScriptCore实现和H5互调"
category:  JavaScriptCore
---

自从iOS7新增了JavaScriptCore库之后，一直没有深入了解过，现在趁着实现和H5互调来研究一下这个库。
完整代码
https://github.com/victorchee/JavaScriptCoreDemo

## App调用JS

### 老方法

UIWebView有一个方法stringByEvaluatingJavaScriptFromString，可以直接执行JS，在此就不多说了。

### 新方法

申明一个属性`var jsContext: JSContext?`，然后在`webViewDidFinishLoad`的为其赋值

```Swift
jsContext = webView.valueForKeyPath("documentView.webView.mainFrame.javaScriptContext") as? JSContext
```

```Swift
// 调用JS方法
jsContext?.objectForKeyedSubscript("show")?.callWithArguments([2])
```
H5中有个show方法，调用show方法，传入参数2。

## JS调用App

### 老方法

UIWebViewDelegate有一个回调

```Swift
optional func webView(_ webView: UIWebView, shouldStartLoadWithRequest request: NSURLRequest, navigationType navigationType: UIWebViewNavigationType) -> Bool
```
每次WebView接受到请求是都会通过此回调询问是否需要加载。
JS调用App的流程就是JS发起一个请求，然后WebView通过此回调拦截请求，解析字符串来通信，然后返回false不加载此请求。

### 新方法

JavaScriptCore引入了两种方式来让JS调用App：

#### Block

```Swift
let log: @convention(block) String -> Void = { input in
    print(input)
}
jsContext?.setObject(unsafeBitCast(log, AnyObject.self), forKeyedSubscript: "log")
```
Objective-C里面可以简单的使用Block，并可使用下标来赋值。Swift里面的话比较复杂，闭包申明要添加`@convention(block)`，绑定的时候需要用`unsafeBitCast`来转换。
另外这里要特别注意循环引用。

#### JSExport

上面的Block方法只能传递方法，不能传递对象，通过JSExport协议的话可以传递对象和方法。

```Swift
@objc protocol TestJSExports: JSExport {
    func functionInSwift()
}
```
申明JSExport协议的时候要添加@objc申明。

```Swift
// MARK: - TestJSExports
func functionInSwift() {
    print("call function in swift")
}
```
实现JSExport协议。

```Swift
jsContext?.setObject(self, forKeyedSubscript: "native")
```
绑定JSExport协议里面的所有对象和方法给JS调用，在JS中通过`native.functionInSwift();`即可调用。

新方法看起来多写了几行代码，但是如果交互很多的话，反而代码量会减少，而且逻辑清楚。还有个更大的好处，再也不用向前端解释这个交互流程了阿，告诉他调用`native.xxx()`就行了阿，简单明了。

最后，JavaScriptCore还可以加载JS库等高级应用，这里就不继续研究了，等遇到了再说吧，因为我觉得用原生的实现更好😄。

参考： http://nshipster.cn/javascriptcore/
