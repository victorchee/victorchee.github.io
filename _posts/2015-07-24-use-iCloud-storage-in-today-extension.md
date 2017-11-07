---
layout: post
title: "在TodayExtension中使用iCloud的数据库"
category:  TodayExtension
---


iOS7之后CoreData支持iCloud就变的简单多了，iOS8之后引入了TodayExtension，但是TodayExtension并不支持CoreData的iCloud数据库。以下方案就是为了解决这个问题。
## 项目结构
首先，先简单介绍下项目的结构，项目包含3个Target：
1. 主项目
2. Framework
3. TodayExtension
Framework包含了主项目和TodayExtension的共享代码，主要包括CoreData的封装，也就是说把CoreData的持久层放在了sharedGroup里面。

## 问题
CoreData支持iCloud之后，TodayExtension中就无法访问CoreData的持久层，即Sqlite数据库。

## 解决
遍历找到iCloud的Sqlite，然后已只读的方式处理。

```Swift
var iCloudStoreURL: NSURL?
var localStoreURL: NSURL?

func findStoreURL(path: String){
    let contents = NSFileManager.defaultManager().contentsOfDirectoryAtPath(path, error: nil)
    if let files = contents as? [String] {
        for file in files  {
            let filePath = path.stringByAppendingPathComponent(file)
            var isDir = false as ObjCBool
            NSFileManager.defaultManager().fileExistsAtPath(filePath, isDirectory: &isDir)
            if isDir {
                findStoreURL(filePath)
            } else {
                if filePath.hasSuffix("local/store/Procrastiatio.sqlite") {
                    localStoreURL = NSURL(fileURLWithPath: filePath)
                    continue // iCloud的优先级较高
                } else if filePath.hasSuffix("store/Procrastiatio.sqlite") {
                    iCloudStoreURL = NSURL(fileURLWithPath: filePath)
                    break // 找到iCloud的就不需要找本地的了
                }
            }
        }
    }
}

func setupCoreData() {
  // ... model

  // persistent store coordinator
  findStoreURL(applicationDocumentsDirectory.path!)
    var storeURL: NSURL
    /*
      iCloud数据库优先
      然后是本地数据库
      最后是默认数据库
    */
    if let iCloud = iCloudStoreURL {
        storeURL = iCloud
    } else {
        if let local = localStoreURL {
            storeURL = local
        } else {
            // iCloud和本地数据库都没有找到
            storeURL = applicationDocumentsDirectory.URLByAppendingPathComponent("Procrastiatio.sqlite")
        }
    }
    let option = [NSReadOnlyPersistentStoreOption: NSNumber(bool: true)]
    persistentStoreCoordinator.addPersistentStoreWithType(NSSQLiteStoreType, configuration: nil, URL: storeURL, options: option, error: nil)

    // ... context
}
```
适用：TodayExtension只做数据展示，不做任何操作。
