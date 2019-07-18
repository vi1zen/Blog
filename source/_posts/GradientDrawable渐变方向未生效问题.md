---
title: GradientDrawable渐变方向未生效问题
date: 2019/07/18 20:52
tags: 

 - Android
---

在Android开发中使用渐变图十分常见，使用最多的就是在xml中创建渐变图，其次就是使用GradientDrawable在代码中编码实现，使用代码创建GradientDrawable代码如下(以kotlin为例)
```kotlin
val gradientrDawable = GradientDrawable(GradientDrawable.Orientation.TOP_BOTTOM,
                                        intArrayOf(Color.RED,Color.LTGRAY))
```
但是需要注意的是代码中设置使用默认的两参数构造函数渐变方向会出现未生效的问题，无论怎么更改构造函数中的Orientation，渐变的方向都是从左到右，解决办法就是不使用带有方向的构造函数方法，而使用无参构造函数生成GradientDrawable后再进行设置相关属性，代码如下：

```kotlin
val gradientrDawable = GradientDrawable().apply {
    colors = intArrayOf(Color.RED,Color.LTGRAY)
    shape = GradientDrawable.RECTANGLE
    orientation = GradientDrawable.Orientation.TOP_BOTTOM
}
```
