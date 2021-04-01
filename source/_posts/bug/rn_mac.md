title: react-native 项目 run-android 报无法安装应用
permalink: ios-safari-scroll/
date: 2017-05-27 08:46:04
tags: [bug, ios]
---



```
z at ZsMBP in ~/Projects/ReactXP/reactxp/samples/TodoList (master●)
$ react-native run-android
Scanning 740 folders for symlinks in /Users/z/Projects/ReactXP/reactxp/samples/TodoList/node_modules (7ms)
JS server already running.
Building and installing the app on the device (cd android && ./gradlew installDebug)...
Could not install the app on the device, read the error above for details.
Make sure you have an Android emulator running or a device connected and have
set up your Android development environment:
https://facebook.github.io/react-native/docs/android-setup.html
```

这个错误的解释废话太多，还是错的, 其实手动运行下命令`cd android && ./gradlew installDebug`就会发现是`./android/.gradlew` 这个文件没有执行权限导致的，运行一下
```sh
chmod +x ./android/.gradlew
```

就可以了
