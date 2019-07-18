---
title: 升级Android Studio 3.0
date: 2018.01.15 11:11:31
categories:
 - Android
tags: 
 - Android Studio3.0
 
---
### 引言
> Android Studio从3.0版本新增了许多功能，当然首当其冲就是从3.0版本新增了对 Kotlin 开发语言的支持，除此之外还有其他一些新功能，例如：Android Profiler (其中包含了： CPU Profiler、Memory Profiler、Network Profiler )，APK Debugger，Device File Explorer，Java 8 Language Features等[详情][1]。

当然作为一个程序员，我们应该积极拥抱变化，不能说好麻烦，又不是不能用之类的话，不然又和咸鱼有什么区别呢。

### 升级方式

1.应用内升级
在Android Studio内使用快捷键citr+alt+s打开设置界面，搜索updates点击Check Now按钮即可收到升级推送
![应用内升级][2]
2.通过官网下载完整安装包安装
官网地址：https://developer.android.google.cn/studio/index.html

<!-- more -->

### 更新Android Studio相关配置文件

1. 在项目级别的build.gradle中更新配置文件
- 依赖google()官方仓库
- 升级插件版本
- 升级到官方建议buildToolsVersion(非必要)
![配置文件][3]
2. 在Module级别的build.gradle中更新依赖方式（Gradle关键字依赖变化）
`Android Studio3.0后Gradle关键字依赖发生变化，创建新项目也会使用新的关键字，但是旧的关键字目前依旧可以使用。`
- compile和api

    api完全等同于compile，二者没有区别。我们大家都知道，随着Android版本的更新，有很多过时的类和方法，compile亦是如此，我们可以把compile理解成api的过去式。

- api和implementation
 
    这个指令的特点就是，对于使用了该命令编译的依赖，对该项目有依赖的项目将无法访问到使用该命令编译的依赖中的任何程序，也就是将该依赖隐藏在内部，而不对外部公开。

点击[官方链接][4]了解详情

### 安装过程中遇到的问题

每个人遇到的问题可能不一样，具体可以参考这篇文章https://mp.weixin.qq.com/s/VzMHjmKcI0hOHYrWK3Eqrw


  [1]: https://android-developers.googleblog.com/2017/10/android-studio-30.html
  [2]: http://ol9j5v5dg.bkt.clouddn.com/as_check_update.png
  [3]: http://ol9j5v5dg.bkt.clouddn.com/as_build_gradle_project.png
  [4]: https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#new_configurations