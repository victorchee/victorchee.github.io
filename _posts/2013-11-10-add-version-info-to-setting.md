---
layout: post
title: "在设置中添加App版本信息"
category:  Skill
---
有的时候我们需要得到当前安装的App是哪个版本的，具体什么时候等遇到的时候就知道有多痛苦了。如果我们能把版本信息整合到设置里面，那就方便多了。

- 新建一个Setting.bundle
File->New->File...->Resource->Setting Bundle，名字默认就可以了。
- 修改Root.plist
Preference Items里面有默认的好几项，删掉重新添加一个Group和两个Title，参考如下：

```Xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>PreferenceSpecifiers</key>
	<array>
		<dict>
			<key>Title</key>
			<string>Info</string>
			<key>Type</key>
			<string>PSGroupSpecifier</string>
		</dict>
		<dict>
			<key>DefaultValue</key>
			<string>-</string>
			<key>Key</key>
			<string>version_preference</string>
			<key>Title</key>
			<string>Version</string>
			<key>Type</key>
			<string>PSTitleValueSpecifier</string>
		</dict>
		<dict>
			<key>Key</key>
			<string>build_preference</string>
			<key>Title</key>
			<string>Build</string>
			<key>DefaultValue</key>
			<string>-</string>
			<key>Type</key>
			<string>PSTitleValueSpecifier</string>
		</dict>
	</array>
	<key>StringsTable</key>
	<string>Root</string>
</dict>
</plist>
```

- 写入版本信息
在`- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`中向Setting.bundle中写入版本信息

```Objective-C
NSString *version = [[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleShortVersionString"];
    [[NSUserDefaults standardUserDefaults] setObject:version forKey:@"version_preference"];

    NSString *build = [[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleVersion"];
    [[NSUserDefaults standardUserDefaults] setObject:build forKey:@"build_preference"];
```

- 完成
编译运行，再到设置里面即可看到版本信息了，大功告成。
