---
layout: post
title: "在Debug和Release模式下显示不同的Setting.bundle"
category:  Skill
---

新建Resource->Settings Bundle
Target->Build Phases->add Run Script->移动Run Script到Copy Bundle Resources的上面

```
if [ "${CONFIGURATION}" == "Debug" ] || [ "${CODE_SIGN_IDENTITY}" == "iPhone Developer" ] || [ "${PROVISIONING_PROFILE_SPECIFIER}" == "MiguYouPlay-Development" ]; then
cp -r "${PROJECT_DIR}/${PRODUCT_NAME}/Settings_Debug.bundle/Root.plist" "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/Settings.bundle/Root.plist"
echo "Debug settings bundle copied"
else
cp -r "${PROJECT_DIR}/${PRODUCT_NAME}/Settings.bundle/Root.plist" "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/Settings.bundle/Root.plist"
echo "Release settings bundle copied"
fi
```

参考：http://blog.originate.com/blog/2013/09/05/iOS-modify-settings-build-phase/
http://www.dosomethinghere.com/2013/09/21/different-settings-app-entries-for-debug-vs-release-builds/
