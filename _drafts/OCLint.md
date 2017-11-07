---
layout: post
title: "Using OCLint in Xcode"
category:  Tool
---

# 安装OCLint
`brew tap oclint/formulae`
`brew install oclint`

# 安装xcpretty
`sudo gem install -n /usr/local/bin xcpretty`

# 配置Xcode

## 新建Target
新建一个Aggregate Target，命名为OCLint
![Add Target](/images/OCLint/AddTarget.png)

## 添加运行脚本
在OCLint这个Target的Build Phases配置中，点击"Add Run Script"
![Add Run Script](/images/OCLint/AddRunScript.png)
添加以下脚本
```
source ~/.bash_profile
cd ${SRCROOT}
xcodebuild clean
xcodebuild | xcpretty -r json-compilation-database
oclint-json-compilation-database -- -report-type xcode
```

## 运行
运行OCLint这个Target对应的Scheme，就可以看到分析结果了

# 更新OCLint
`brew update`
`brew upgrade oclint`




Refrences:
[http://oclint-docs.readthedocs.io/en/stable/guide/xcode.html?highlight=xcode]
