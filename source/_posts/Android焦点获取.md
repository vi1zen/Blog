---
title: Android焦点获取
date: 2017-06-01 10:12:50
categories:
 - Android
tags: 
 - 焦点
 
---

- **什么是焦点？**

在非触屏手机时代或电脑上，我们通常需要用键盘、 鼠标、轨迹球（trackball）与界面进行交互，当交互的时候必须使目标控件获得焦点（比如高亮起来），这样用户才会注意到是什么控件接受输入。而如果是在触屏时代，用户可以直接用手指点击控件，这个时候就没必要将目标高亮了（即获取焦点）。这也就是接下来我们要讲的触摸模式（Touch Mode）。

- **触摸模式**

当用户使用方向键或轨迹球导航用户界面时，必须聚焦到可操作项目上（如按钮），以便用户看到将接受输入的对象。 但是，如果设备具有触摸功能且用户开始通过触摸界面与之交互，则不再需要突出显示项目或聚焦到特定视图对象上。 因此，有一种交互模式称为“触摸模式”（Touch Mode）。

触摸模式是用户和手机进行交互时view层次结构的一个状态。代表了最近一次的交互是否是通过触摸屏发生的，因为在Android设备上还存在别的交互方式，比如键盘、等等。

对于支持触摸功能的设备，当用户触摸屏幕时，设备会立即进入触摸模式。无论何时，只要用户点击方向键或滚动轨迹球，设备就会退出触摸模式并找到一个视图使其获得焦点。 现在，用户可在不触摸屏幕的情况下继续与用户界面交互。

整个系统（所有窗口和 Activity）都将保持触摸模式状态。要查询当前状态，您可以调用View.isInTouchMode() 来检查设备目前是否处于触摸模式。

<!-- more -->

- **焦点和触摸模式**

在触摸模式下，没有焦点，没有选择。同样，任何已聚焦的控件当进入触摸模式时都变为未聚焦状态。而当使用轨迹球和键盘时，就会立即离开触摸模式，控件就会变成聚焦的状态。

现在我们知道了焦点不可以存在于触摸模式了吧，但是这并不完全正确，焦点其实是可以一种特殊的方式存在于触摸模式中的，我们称之为聚焦（ focusable）。这种特殊的模式是专门为可接收文字输入的控件创建的，比如EditText。这就是为什么用户可以直接输入文字到文本框中，而不必先用手指选择其文本框的原因。

在触摸模式中，任何控件只要是可聚焦（focusable ）的状态，当用户点击其控件时，该控件就会接收到其焦点。如果是不可聚焦的，点击控件将不会接收到焦点。

对于支持触摸功能的设备，当用户触摸屏幕时，设备会立即进入触摸模式。 自此以后，只有 isFocusableInTouchMode() 为 true 的视图才可聚焦，如文本编辑小部件EditText。其他可触摸的视图（如按钮Button）在用户触摸时不会获得焦点；按下时它们只是触发点击侦听器。

在触摸模式下，只有少部分的控件默认是可聚焦的状态，例如EditText。可以通过setFocusableInTouchMode或xml中android:focusableInTouchMode设置控件是否可聚焦。

- **setFocusableInTouchMode 和 setFocusable**

很多人对这两个方法有疑问其实很简单：

> setFocusable：设置控件是否能获取焦点。可以通过isFocusable()获取其状态。
> setFocusableInTouchMode：在触摸模式下，设置控件是否允许聚焦。可以通过isFocusableInTouchMode() 获取其状态。

在使用键盘或轨迹球的情况下，只有setFocusable为true的控件，才可以获取焦点（选中时高亮）。而在触摸模式下，setFocusable为true，并无法保证控件可以获取焦点。setFocusable为true只能保证在非触摸模式下，该控件可以允许获取焦点。如果想在在触摸模式中，改变控件是否允许聚焦，请使用setFocusableInTouchMode进行更改。

从上面我们也可以看出，不管是否在触摸模式下，控件获取焦点的前提是isFocusable()为true。**而在触摸模式下，只有isFocusable()和isFocusableInTouchMode()都为ture的情况下，控件才允许聚焦。**

通过上面我们发现如下的规律： 
> setFocusableInTouchMode为true，会使isFocusable也变为true，而setFocusableInTouchMode为false并不影响isFocusable。 
> setFocusable为false，会使isFocusableInTouchMode变为false，而setFocusable为true并不影响isFocusableInTouchMode。

**因此，如要获取焦点需要setFocusableInTouchMode和setFocusable同时为true才能保证获取到焦点，注意在表单中想设置某一个如EditText获取焦点，光设置这个是不行的，需要将这个EditText前面的focusable都设置为false才行。**