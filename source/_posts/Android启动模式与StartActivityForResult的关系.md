---
title: Android启动模式与StartActivityForResult的关系
date: 2017-08-10 16:10:35
categories:
 - Android
tags: 
 - 启动模式
 
---

### 引言

在开发过程中经常会用到`StartActivityForResult`方法启动一个Activity，然后在`onActivityResult()`方法中可以接收到上个页面的回传值，但你有可能遇到过拿不到返回值的情况，那有可能是因为Activity的LaunchMode设置为了`singleTask`。`5.0之后，android的LaunchMode与StartActivityForResult的关系发生了一些改变`。两个Activity，A和B，现在由A页面跳转到B页面，看一下LaunchMode与StartActivityForResult之间的关系：

<!-- more -->

**android5.0之前**

| A\B        | standard          | singleTop  | singleTask  |singleInstance  |
| ------------- |:-------------:|:-----:|:-----:|:-----:|
| **standard**      | &radic; | &radic; | &times; | &times; |
| **singleTop**      |&radic; | &radic; | &times; | &times; |
| **singleTask** |&radic; | &radic; | &times; | &times; |
| **singleInstance** | &times; | &times;| &times; | &times; |

源代码：

```java
if (sourceRecord == null) {
    // This activity is not being started from another...  in this
    // case we -always- start a new task.
    if ((launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
        Slog.w(TAG, "startActivity called from non-Activity context; forcing " +
                "Intent.FLAG_ACTIVITY_NEW_TASK for: " + intent);
        launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
    }
} else if (sourceRecord.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
    // The original activity who is starting us is running as a single
    // instance...  this new activity it is starting must go on its
    // own task.
    launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
} else if (r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE
        || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK) {
    // The activity being started is a single instance...  it always
    // gets launched into its own task.
    launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
}
// ......
if (r.resultTo != null && (launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
    // For whatever reason this activity is being launched into a new
    // task...  yet the caller has requested a result back.  Well, that
    // is pretty messed up, so instead immediately send back a cancel
    // and let the new task continue launched as normal without a
    // dependency on its originator.
    Slog.w(TAG, "Activity is launching as a new task, so cancelling activity result.");
    r.resultTo.task.stack.sendActivityResultLocked(-1,
            r.resultTo, r.resultWho, r.requestCode,
        Activity.RESULT_CANCELED, null);
    r.resultTo = null;
}
```

**android5.0之后**

| A\B        | standard          | singleTop  | singleTask  |singleInstance  |
| ------------- |:-------------:|:-----:|:-----:|:-----:|
| **standard**      | &radic; | &radic; | &radic; | &radic; |
| **singleTop**      |&radic; |&radic; | &radic; | &radic; |
| **singleTask** |&radic; | &radic; | &radic; | &radic; |
| **singleInstance** | &radic; | &radic; | &radic; | &radic; |

源代码：
```java
final boolean launchSingleTop = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TOP;
final boolean launchSingleInstance = r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE;
final boolean launchSingleTask = r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK;
int launchFlags = intent.getFlags();
if ((launchFlags & Intent.FLAG_ACTIVITY_NEW_DOCUMENT) != 0 &&
        (launchSingleInstance || launchSingleTask)) {
    // We have a conflict between the Intent and the Activity manifest, manifest wins.
    Slog.i(TAG, "Ignoring FLAG_ACTIVITY_NEW_DOCUMENT, launchMode is " +
            "\"singleInstance\" or \"singleTask\"");
    launchFlags &=
            ~(Intent.FLAG_ACTIVITY_NEW_DOCUMENT | Intent.FLAG_ACTIVITY_MULTIPLE_TASK);
} else {
    switch (r.info.documentLaunchMode) {
        case ActivityInfo.DOCUMENT_LAUNCH_NONE:
            break;
        case ActivityInfo.DOCUMENT_LAUNCH_INTO_EXISTING:
            launchFlags |= Intent.FLAG_ACTIVITY_NEW_DOCUMENT;
            break;
        case ActivityInfo.DOCUMENT_LAUNCH_ALWAYS:
            launchFlags |= Intent.FLAG_ACTIVITY_NEW_DOCUMENT;
            break;
        case ActivityInfo.DOCUMENT_LAUNCH_NEVER:
            launchFlags &= ~Intent.FLAG_ACTIVITY_MULTIPLE_TASK;
            break;
    }
}
final boolean launchTaskBehind = r.mLaunchTaskBehind
        && !launchSingleTask && !launchSingleInstance
        && (launchFlags & Intent.FLAG_ACTIVITY_NEW_DOCUMENT) != 0;
if (r.resultTo != null && (launchFlags & Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
    // For whatever reason this activity is being launched into a new
    // task...  yet the caller has requested a result back.  Well, that
    // is pretty messed up, so instead immediately send back a cancel
    // and let the new task continue launched as normal without a
    // dependency on its originator.
    Slog.w(TAG, "Activity is launching as a new task, so cancelling activity result.");
    r.resultTo.task.stack.sendActivityResultLocked(-1,
            r.resultTo, r.resultWho, r.requestCode,
            Activity.RESULT_CANCELED, null);
    r.resultTo = null;
}
```

### 结论

可以看出这是因为ActivityStackSupervisor类中的startActivityUncheckedLocked方法在5.0中进行了修改。在5.0之前，当启动一个Activity时，系统将首先检查Activity的launchMode，如果为A页面设置为SingleInstance或者B页面设置为singleTask或者singleInstance,则会在LaunchFlags中加入FLAG_ACTIVITY_NEW_TASK标志，而如果含有FLAG_ACTIVITY_NEW_TASK标志的话，onActivityResult将会立即接收到一个cancle的信息，而5.0之后这个方法做了修改，修改之后即便启动的页面设置launchMode为singleTask或singleInstance，onActivityResult依旧可以正常工作，也就是说无论设置哪种启动方式，StartActivityForResult和onActivityResult()这一组合都是有效的。**所以如果你目前正好基于5.0做相关开发，不要忘了向下兼容，这里有个坑请注意避让。**

