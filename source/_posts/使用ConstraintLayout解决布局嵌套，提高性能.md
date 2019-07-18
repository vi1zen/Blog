---
title: 使用ConstraintLayout解决布局嵌套，提高性能
date: 2018.02.06 11:14:15
categories:
 - Android
tags: 
 - ConstraintLayout
 
---
### 引言

ConstraintLayout是Android Studio 2.2中主要的新增功能之一，也是Google在去年的I/O大会上重点宣传的一个功能。
ConstraintLayout最显著的优点就是它可以**有效地解决布局嵌套过多**的问题。我们平时编写界面，复杂的布局总会伴随着多层的嵌套，而嵌套越多，程序的性能也就越差。ConstraintLayout则是使用约束的方式来指定各个控件的位置和关系的，它有点类似于RelativeLayout，但远比RelativeLayout要更强大。

### 添加依赖

我们需要在app/build.gradle文件中添加ConstraintLayout的依赖

Android Studio 3.0以下
> compile 'com.android.support.constraint:constraint-layout:1.0.2'

Android Studio3.0以上
> implementation 'com.android.support.constraint:constraint-layout:1.0.2'

Android Studio 3.0增加了许多新特性，依赖关键字的变化是其中之一，使用`implementation`是新版本Android Studio默认的依赖关键字，它可以隐藏依赖的内部细节，加快编译速度。

### 示例

首先来看一个例子:
![item_list][1]
要实现上图所示的效果你一定在想这很简单啊，刷刷的就写出来了，然后发现嵌套了三四层，要是再放到列表里面你会发现页面滑动时很不流畅，这时就轮到ConstraintLayout出场了，先贴代码

<!-- more -->

```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="@dimen/ui_180dp"
    android:background="@color/white">


    <View
        android:id="@+id/view_mark"
        android:layout_width="@dimen/ui_5dp"
        android:layout_height="@dimen/one_and_a_half_grid_unit"
        android:layout_marginLeft="@dimen/ui_padding_10dp"
        android:background="@color/blue_line_chart"
        app:layout_constraintLeft_toRightOf="parent"
        app:layout_constraintTop_toTopOf="@+id/tv_title"
        app:layout_constraintBottom_toBottomOf="@id/tv_title"/>
    
    <TextView
        android:id="@+id/tv_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintLeft_toRightOf="@id/view_mark"
        app:layout_constraintTop_toBottomOf="parent"
        android:layout_marginLeft="@dimen/ui_padding_10dp"
        android:layout_marginTop="@dimen/ui_padding_10dp"
        android:textSize="@dimen/ui_textsize_14sp"
        android:textColor="@color/black_title"
        tools:text="探鱼高新店"/>

    <TextView
        android:id="@+id/tv_leftItem"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="125,556,315"
        android:textColor="@color/black_title"
        android:paddingLeft="@dimen/ui_padding_10dp"
        android:gravity="center"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintRight_toLeftOf="@+id/guideline"/>

    <!--垂直平分线-->
    <android.support.constraint.Guideline
        android:id="@+id/guideline"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintRight_toLeftOf="@id/circlePoint1"
        app:layout_constraintGuide_percent="0.5" />

    <!--垂直右边界线-->
    <android.support.constraint.Guideline
        android:id="@+id/guideline1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        app:layout_constraintGuide_percent="1" />

    <!--水平平分线-->
    <android.support.constraint.Guideline
        android:id="@+id/guideline2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.5"/>

    <com.wangcaibao.erp.view.erpview.CirclePointView
        android:id="@+id/circlePoint1"
        android:layout_width="@dimen/ui_5dp"
        android:layout_height="@dimen/ui_5dp"
        app:fillColor="@color/gray"
        app:layout_constraintLeft_toRightOf="@+id/guideline"
        app:layout_constraintBottom_toBottomOf="@+id/tv_rightItem1"
        app:layout_constraintTop_toTopOf="@+id/tv_rightItem1" />

    <com.wangcaibao.erp.view.erpview.CirclePointView
        android:id="@+id/circlePoint2"
        android:layout_width="@dimen/ui_5dp"
        android:layout_height="@dimen/ui_5dp"
        app:fillColor="@color/gray"
        app:layout_constraintBottom_toBottomOf="@+id/tv_rightItem2"
        app:layout_constraintLeft_toRightOf="@id/guideline"
        app:layout_constraintTop_toTopOf="@+id/tv_rightItem2" />

    <com.wangcaibao.erp.view.erpview.CirclePointView
        android:id="@+id/circlePoint3"
        android:layout_width="@dimen/ui_5dp"
        android:layout_height="@dimen/ui_5dp"
        app:fillColor="@color/gray"
        app:layout_constraintBottom_toBottomOf="@+id/tv_rightItem3"
        app:layout_constraintLeft_toRightOf="@id/guideline"
        app:layout_constraintTop_toTopOf="@+id/tv_rightItem3" />

    <com.wangcaibao.erp.view.erpview.CirclePointView
        android:id="@+id/circlePoint4"
        android:layout_width="@dimen/ui_5dp"
        android:layout_height="@dimen/ui_5dp"
        app:fillColor="@color/gray"
        app:layout_constraintBottom_toBottomOf="@+id/tv_rightItem4"
        app:layout_constraintLeft_toRightOf="@id/guideline"
        app:layout_constraintTop_toTopOf="@+id/tv_rightItem4" />


    <TextView
        android:id="@+id/tv_rightItem1"
        android:layout_width="@dimen/ui_title_height"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="center_vertical|right"
        android:textColor="@color/gray_text"
        android:textSize="@dimen/ui_textsize_12sp"
        android:text="点餐:"
        app:layout_constraintLeft_toRightOf="@id/circlePoint1"
        app:layout_constraintBottom_toTopOf="@id/tv_rightItem2"/>

    <TextView
        android:id="@+id/tv_number1"
        android:layout_width="@dimen/ui_title_width"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="right|center_vertical"
        android:textColor="@color/black_title"
        android:textSize="@dimen/ui_textsize_14sp"
        tools:text="1234555"
        app:layout_constraintLeft_toRightOf="@id/tv_rightItem1"
        app:layout_constraintBottom_toTopOf="@id/tv_number2"/>

    <TextView
        android:id="@+id/tv_percent1"
        android:layout_width="@dimen/ui_55dp"
        android:layout_height="wrap_content"
        tools:background="@color/blue_line_chart"
        android:gravity="center_vertical"
        android:layout_marginLeft="@dimen/ui_padding_10dp"
        android:paddingLeft="@dimen/ui_5dp"
        android:textColor="@color/white"
        app:layout_constraintBottom_toTopOf="@+id/tv_number2"
        app:layout_constraintLeft_toRightOf="@id/tv_number1"
        app:layout_constraintTop_toTopOf="@+id/tv_number1"
        tools:text="80%" />

    <TextView
        android:id="@+id/tv_rightItem2"
        android:layout_width="@dimen/ui_title_height"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="center_vertical|right"
        android:textColor="@color/gray_text"
        android:textSize="@dimen/ui_textsize_12sp"
        app:layout_constraintBottom_toTopOf="@id/guideline2"
        app:layout_constraintLeft_toRightOf="@id/circlePoint2"
        android:text="游戏:" />

    <TextView
        android:id="@+id/tv_number2"
        android:layout_width="@dimen/ui_title_width"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="right|center_vertical"
        android:textColor="@color/black_title"
        android:textSize="@dimen/ui_textsize_14sp"
        app:layout_constraintLeft_toRightOf="@id/tv_rightItem1"
        app:layout_constraintBottom_toTopOf="@id/guideline2"
        tools:text="55" />

    <TextView
        android:id="@+id/tv_percent2"
        android:layout_width="@dimen/ui_55dp"
        android:layout_height="wrap_content"
        tools:background="@color/blue_line_chart"
        android:gravity="center_vertical"
        android:layout_marginLeft="@dimen/ui_padding_10dp"
        android:paddingLeft="@dimen/ui_5dp"
        android:textColor="@color/white"
        app:layout_constraintBottom_toBottomOf="@+id/tv_number2"
        app:layout_constraintLeft_toRightOf="@id/tv_number2"
        app:layout_constraintTop_toTopOf="@+id/tv_number2"
        tools:text="80.66%" />

    <TextView
        android:id="@+id/tv_rightItem3"
        android:layout_width="@dimen/ui_title_height"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="center_vertical|right"
        android:textColor="@color/gray_text"
        android:textSize="@dimen/ui_textsize_12sp"
        app:layout_constraintLeft_toRightOf="@id/circlePoint2"
        app:layout_constraintTop_toBottomOf="@id/guideline2"
        android:text="支付:" />

    <TextView
        android:id="@+id/tv_number3"
        android:layout_width="@dimen/ui_title_width"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="right|center_vertical"
        android:textColor="@color/black_title"
        android:textSize="@dimen/ui_textsize_14sp"
        app:layout_constraintLeft_toRightOf="@id/tv_rightItem3"
        app:layout_constraintTop_toBottomOf="@id/guideline2"
        tools:text="55" />

    <TextView
        android:id="@+id/tv_percent3"
        android:layout_width="@dimen/ui_55dp"
        android:layout_height="wrap_content"
        tools:background="@color/blue_line_chart"
        android:gravity="center_vertical"
        android:layout_marginLeft="@dimen/ui_padding_10dp"
        android:paddingLeft="@dimen/ui_5dp"
        android:textColor="@color/white"
        app:layout_constraintBottom_toBottomOf="@+id/tv_number3"
        app:layout_constraintLeft_toRightOf="@id/tv_number3"
        app:layout_constraintTop_toTopOf="@+id/tv_number3"
        tools:text="80%" />

    <TextView
        android:id="@+id/tv_rightItem4"
        android:layout_width="@dimen/ui_title_height"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="center_vertical|right"
        android:textColor="@color/gray_text"
        android:textSize="@dimen/ui_textsize_12sp"
        app:layout_constraintLeft_toRightOf="@id/circlePoint2"
        app:layout_constraintTop_toBottomOf="@id/tv_rightItem3"
        android:text="服务:" />

    <TextView
        android:id="@+id/tv_number4"
        android:layout_width="@dimen/ui_title_width"
        android:layout_height="@dimen/ui_title_height"
        android:gravity="right|center_vertical"
        android:textColor="@color/black_title"
        android:textSize="@dimen/ui_textsize_14sp"
        app:layout_constraintLeft_toRightOf="@id/tv_rightItem3"
        app:layout_constraintTop_toBottomOf="@id/tv_number3"
        tools:text="55" />

    <TextView
        android:id="@+id/tv_percent4"
        android:layout_width="@dimen/ui_55dp"
        android:layout_height="wrap_content"
        tools:background="@color/blue_line_chart"
        android:gravity="center_vertical"
        android:textColor="@color/white"
        android:layout_marginLeft="@dimen/ui_padding_10dp"
        android:paddingLeft="@dimen/ui_5dp"
        app:layout_constraintBottom_toBottomOf="@+id/tv_number4"
        app:layout_constraintLeft_toRightOf="@id/tv_number4"
        app:layout_constraintTop_toTopOf="@+id/tv_number4"
        tools:text="80%" />

    <ImageView
        android:id="@+id/iv_next"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"
        android:contentDescription="@string/view_details"
        android:src="@drawable/icon_drop_right"
        app:layout_constraintRight_toLeftOf="@id/guideline1"
        app:layout_constraintTop_toBottomOf="parent"
        app:layout_constraintBottom_toTopOf="parent"
        />

    <View
        android:layout_width="match_parent"
        android:layout_height="@dimen/ui_divider_height"
        android:background="@color/divider"
        app:layout_constraintBottom_toTopOf="parent"/>

</android.support.constraint.ConstraintLayout>
```
可以看到使用ConstraintLayout只用一层布局就可以做到，完全不用嵌套，而且性能比普通的嵌套布局好很多，可以用Hierarchy View查看布局时间，图这里就不贴了。

### 使用

ConstraintLayout有两种布局方式，一种就是在视图编辑器里直接拖动，另外一种就是在代码里布局，我的建议是简单的布局可以在视图编辑器里直接拖，很方便，但是复杂布局还是建议大家在代码里布局，因为view太多了真的不好拖，还容易误操作，当然你有一个巨大无比的显示屏就当我没说。

1.在视图编辑器里直接拖动

这里引用一下郭霖郭神的blog，他的博客已经写的很清楚了。想了解的可以点击[郭霖的博客][2]

2.在代码里布局

相对于直接在视图编辑器里布局，在代码里布局就不是那么直观明了了，但是用代码布局更加准确，不容易出错。

#### 基本布局代码：

- app:layout_constraintLeft_toRightOf="parent"

上面代码`layout_constraintLeft`表示要布局的view自身的四个位置中的左边，`toRightOf="parent"`表示相对于目标view的位置右边，这行代码连起来就是要布局的view的左边在parent view的右边，也就是view在parent view的右边的意思

- app:layout_constraintBottom_toBottomOf="parent"

对于这种类似于bottom_toBottom的代码，是表示边与边对齐的意思，这行代码的意思是view的底边相对于parent view底边对齐

```java
app:layout_constraintBottom_toBottomOf="parent"
app:layout_constraintStart_toStartOf="parent"
app:layout_constraintTop_toTopOf="parent"
```

上面代码是表示垂直居中的意思

#### 引导线（可用来百分比布局）

```java
<!--水平平分线-->
<android.support.constraint.Guideline
    android:id="@+id/guideline"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    app:layout_constraintGuide_percent="0.5">
</android.support.constraint.Guideline>
```
GuideLine也叫做辅助线，分为垂直和水平方向，查看源码可以看到它的draw()方法是空实现，也就表示在它不会真正绘制，只是起辅助作用，通过辅助线可以简单实现百分比布局等功能。通过GuideLine和app:layout_constraintXX_toXXOf="XX"配合使用，基本可以实现所有复杂布局。


### 总结

ConstraintLayout作为新的布局容器，优点是能良好的解决布局嵌套带来的性能问题，在追求性能的页面很有必要使用，缺点就是便利性不够，简而言之就是稍显麻烦，一般对性能要求不高的页面不是很必要，这个希望能继续优化吧，大家在日常工作中根据实际情况来使用就行了。

  [1]: http://ol9j5v5dg.bkt.clouddn.com/constraint_item.png
  [2]: http://blog.csdn.net/guolin_blog/article/details/53122387