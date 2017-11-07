---
layout: post
title: "创建Framework"
category: Guide
---

打开xcode,新建iOS->Framework & Library->Cocoa Touch Framework

在PROJECT->Deployment Target->iOS Deployment Target选择你需要支持的最低系统。同样的操作在TARGETS中，Deployment Info->Deployment Target

因为工程的默认设置是Dynamic Framework，需要在iOS8.0才能使用。所以我们要做的将Dynamic Framework设置为Static Library，在Build Settings->Linking->Mach-O Type->Static Library

在编译时，不要将图片文件，xib,媒体文件放在工程里面，否则编译后framework中会出现大量的零散图片文件在里面。这时需要将图片等资源放在.bundle文件中

这样打包的framework依然有问题，如果你用了Category，别人在用你的framework时会发生崩溃。这时别人在引用时需要在工程中other linker flags中添加-objC如果依然有问题，再添加-all_load

终于编译成功，但发现很多关于符号表的警告，这时需要将Generate Debug Symbols设置为NO即可关闭符号表警告

但是我需要支持bitcode，以上设置后并不能使framework支持bitcode,因此还需要进行额外的设置一个命令让其支持bit code。在TAGETS的Build setting中搜索Other C Flags，添加命令“-fembed-bitcode”。同样的设置在PROJECT中。如果不进行以上操作。别人在集成你的framework时可以编译，亦可以真机测试。唯独在打包时会发出警告并打包失败。警告为framework不支持bitcode

FrameworkDemo.h的头文件中加入需要暴露出来的头文件

无论SDK还是Framework都需要暴露公共的头文件以供使用者读取和。在TARGETS->Build Phases->Headers里面，有三种类别。Public(公共的)，这里存放公开的头文件。Private(私有的)这里存放私有的Header，虽然说是私有的，但是还是公开的头文件，编译之后仍然可以framework包里面找到。所以有些Header是不想给别人看到的，必须要放在Project中

打包。Edit Scheme->Build Configuration->选为Release然后Run即可

在你的SDK项目中新建add target -> Cross-platform -> Aggregate Target

添加script到新建的Target：Build Phases -> + -> Run Script

复制下面提供的脚本。运行之后会自动打开合并之后的framework文件，注意不要勾选Run script only when installing。

```
# Sets the target folders and the final framework product.
# 如果工程名称和Framework的Target名称不一样的话，要自定义FMKNAME
# 例如: FMK_NAME = "MyFramework"
FMK_NAME=${PROJECT_NAME}
# Install dir will be the final output to the framework.
# The following line create it in the root folder of the current project.
INSTALL_DIR=${SRCROOT}/Products/${FMK_NAME}.framework
# Working dir will be deleted after the framework creation.
WRK_DIR=build
DEVICE_DIR=${WRK_DIR}/Release-iphoneos/${FMK_NAME}.framework
SIMULATOR_DIR=${WRK_DIR}/Release-iphonesimulator/${FMK_NAME}.framework
# -configuration ${CONFIGURATION}
# Clean and Building both architectures.
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphoneos -arch armv7 -arch armv7s -arch arm64 clean build
xcodebuild -configuration "Release" -target "${FMK_NAME}" -sdk iphonesimulator -arch x86_64 clean build
# Cleaning the oldest.
if [ -d "${INSTALL_DIR}" ]
then
rm -rf "${INSTALL_DIR}"
fi
mkdir -p "${INSTALL_DIR}"
cp -R "${DEVICE_DIR}/" "${INSTALL_DIR}/"
# Uses the Lipo Tool to merge both binary files (i386 + armv6/armv7) into one Universal final product.
lipo -create "${DEVICE_DIR}/${FMK_NAME}" "${SIMULATOR_DIR}/${FMK_NAME}" -output "${INSTALL_DIR}/${FMK_NAME}"
rm -r "${WRK_DIR}"
open "${SRCROOT}/Products/"
```

Edit Scheme -> Build Configuration -> 选为Release，选择Generic iOS Device编译

新建macOS -> Framework & Library -> Bundle

在TARGTS->Build Settings->Deployment->iOS Deployment Target->选择自己需要支持的最低系统
Skip Install设置为NO
"Strip Debug Symbols During Copy" 中"Release"模式设置为 "YES"

build后会生成一个bundle包，但在包中的图片由以前的png格式全部变成tiff格式。为了防止这种格式转变。需要在Build Settings->Architectures->Base SDK->选择iOS的SDK要支持的版本。这时TARGETS中Build Setting->User-Defined中会出现一个新的Key：COMBINE_HIDPI_DEBUG_INFO,把它设置为NO

TARGTS -> Build Settings -> Build Options -> Enable Bitcode -> NO
