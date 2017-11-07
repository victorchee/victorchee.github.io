---
layout: post
title: "用命令行打AdHoc包"
category: Tutorial
---

用Xcode打包的时候，每次要点好多下鼠标，要是能一键打包该多好。
下面就来一步步实现用命令行打包，想要打包，直接运行下就好了。

本文只针对打AdHoc包，当然也可以打Development包，把相关证书替换就行了。但是不针对打Production包，我觉得可能一个月才上传一次，可能更久，用Xcode上传可能更可靠一些。

## 目标
- 支持自动配置证书
- 支持Today Extension
- 支持svn操作

## 准备工作
```
package|---|---adHocBuilder.sh
           |---Project_AdHoc.mobileprovision
           |---Extension_AdHoc.mobileprovision
           |---|Project
```
- 新建一个目录，命名为package。
- 把项目和Today的AdHoc证书放在package里面。
- 把项目checkout到package里面，我用的是trunk。
- 新建adHocBuilder.sh，然后`sudo chmod +x adHocBuilder.sh`添加执行权限。

## 项目配置
用Xcode打开项目，配置主Target里面的Provisioning Profile改为APP_PROFILE，Today Extension的Target里面的Provisioning Profile改为EXTENSION_PROFILE。

Code Signing Identity 不用改，下面会通过命令设置的。然后就可以关闭Xcode了，下面用不到了。

参考：http://stackoverflow.com/questions/27973011/xcodebuild-different-provisioning-profile-for-target-dependency

## 准备CodeSign信息
Build的时候会需要CodeSign信息，包括证书的UUID和Code Sign Identity。
Code Sign Identity必须要完整信息，在Xcode中看到的都是不完整的，完整的需要到钥匙串中取。
由于现在打的是AdHoc包，所以选择Distribution证书，如果是Development包的话，选Developer证书。
右键Get Info，找到Common Name即是Code Sign Identity信息，包括括号内的信息，千万不要漏了。
```Bash
Code_Sign_Identity="iPhone Distribution: XXXXXXXX (XXXX)"
```
至于证书的UUID，用PlistBuddy来提取就行了。PlistBuddy是用来操作Plist文件的。
```Bash
App_Profile_UUID=`/usr/libexec/plistbuddy -c Print:UUID /dev/stdin <<< \
\`security cms -D -i ./Project_AdHoc.mobileprovision\``
Extension_Profile_UUID=`/usr/libexec/plistbuddy -c Print:UUID /dev/stdin <<< \
\`security cms -D -i ./Extension_AdHoc.mobileprovision\``
echo $App_Profile_UUID
echo $Extension_Profile_UUID
```

## SVN Update
准备工作都做完了，打包之前希望能更新到最新的代码。这一步是可选的，看自己的需求了。
```Bash
cd $Project_Directory
svn update
```

## 设置Build Version
更新完代码之后，设置项目中的Build Version，希望每次打包，Build Version都会加一。当然这一步也是可选的，不过最好是有这一步，为了和以前的包有区别嘛。
同样也是用PlistBuddy来设置项目和Today中的对应的Plist文件。
```Bash
# 设置项目内的Build Version，增1
Project_Plist=${Project_Directory}/Project/Project-Info.plist
Project_Build_Version=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${Project_Plist})
Fixed_Project_Build_Version=$(expr $Project_Build_Version + 1)
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $Fixed_Project_Build_Version" ${Project_Plist}
# 设置Extension的Build Version，增1
Extension_Plist=${Project_Directory}/TodayExtension/Info.plist
Extension_Build_Version=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${Extension_Plist})
Fixed_Extension_Build_Version=$(expr $Extension_Build_Version + 1)
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $Fixed_Extension_Build_Version" ${Extension_Plist}
```

## Clean
打包之前先Clean一下。
```Bash
xcodebuild clean -project $Project_Path -configuration $Configuration -alltargets
```

## Build
然后就是用xcodebuild来Build了，Build的时候已经把证书信息放进去了，所以下一步导出的时候就不用在配置证书了。
```Bash
Project_Path="$Project_Directory/$Project_Name.xcodeproj"
Configuration="Release"
SDK="iphoneos"

xcodebuild -project $Project_Path -alltargets -sdk $SDK CODE_SIGN_IDENTITY="$Code_Sign_Identity" APP_PROFILE="$App_Profile_UUID" EXTENSION_PROFILE="$Extension_Profile_UUID" build
```

## 给要导出的包命名
这一步也是可选的，可以简单的给个名字，也可以给它一个容易辨识的名字，我选择后者。
主要步骤就是找到Build生成的app文件，然后读取里面的Plist，找到名称、版本号、Build版本号，然后加上日期作为ipa的名称。
```Bash
# Build目录
Build_Directory="$Path/trunk/build"
# Build App文件
Build_Path="$Build_Directory/Release-iphoneos/$Project_Name.app"
# app文件中Info.plist文件路径
App_Infoplist_Path=${Build_Path}/Info.plist
# 取版本号
Version=$(/usr/libexec/PlistBuddy -c "print CFBundleShortVersionString" ${App_Infoplist_Path})
# 取build值
BuildVersion=$(/usr/libexec/PlistBuddy -c "print CFBundleVersion" ${App_Infoplist_Path})
# 取displayName
DisplayName=$(/usr/libexec/PlistBuddy -c "print CFBundleDisplayName" ${App_Infoplist_Path})
# IPA名称
IPA_Name="${DisplayName}_${Version}(${BuildVersion})_$(date +"%Y%m%d")"
# 导出IPA文件
Export_Path="$Path/$IPA_Name.ipa"
```

## Export
再就是用xcrun导出ipa包了，Build_Path在上面已经找到，自己设置一下Export_Path就行了。
```Bash
/usr/bin/xcrun -sdk $SDK PackageApplication -v "$Build_Path" -o "$Export_Path"
```

## 清理
这一步是删除Build生成的`build`目录。
```Bash
rm -rf $Build_Directory
```

## Commit
最后一步就是把刚改的Build Version上传到SVN上去。
注意：其实我们也修改了项目的配置，但是并没有上传，考虑到的就是不污染开发团队的其他人配置的证书信息。
```Bash
cd $Project_Directory
svn commit -m "Build Version ++" $Project_Plist $Extension_Plist
```
