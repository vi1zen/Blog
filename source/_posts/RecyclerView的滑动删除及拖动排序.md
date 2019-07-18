---
title: RecyclerView的滑动删除及拖动排序
date: 2017.11.27 17:14:23
categories:
 - Android
tags: 
 - ItemTouchHelper
 
---
### 引言
在安卓中，有许多关于如何使用RecyclerView实现“drag & drop”与swipe-to-dismiss”的教程，库和例子。即使现在已经有了新的，更优的实现方式，大多数仍然是使用老旧的View.OnDragListener以及Roman Nurik在SwipeToDismiss中所使用的方法。很少有人使用新的api，反而要么经常依赖于GestureDetectors和onInterceptTouchEvent，要么实现方式很复杂。实际上，在RecyclerView上添加拖动特性有一个非常简单的方法。这个方法只需要一个类，并且它也是Android 兼容包的一部分，它就是：ItemTouchHelper

> ItemTouchHelper是一个强大的工具，它处理好了关于在RecyclerView上添加拖动排序与滑动删除的所有事情。它是RecyclerView.ItemDecoration的子类，也就是说它可以轻易的添加到几乎所有的LayoutManager和Adapter中。它还可以和现有的item动画一起工作，提供受类型限制的拖放动画等等.

### 使用 ItemTouchHelper 和 ItemTouchHelper.Callback

要使用ItemTouchHelper，你需要创建一个ItemTouchHelper.Callback。这个接口可以让你监听“move”与 “swipe”事件。这里还是控制view被选中的状态以及重写默认动画的地方。如果你只是想要一个基本的实现，有一个帮助类可以使用：SimpleCallback,但是为了了解其工作机制，我们还是自己实现。

启用基本的拖动排序与滑动删除需要重写的主要回调方法是：

```java
getMovementFlags(RecyclerView, ViewHolder)
onMove(RecyclerView, ViewHolder, ViewHolder)
onSwiped(ViewHolder, int)
```

两个帮助方法

```java
isLongPressDragEnabled()
isItemViewSwipeEnabled()
```

<!--more-->

### 核心代码

```java
public class DefaultItemTouchHelper extends ItemTouchHelper.Callback {

    private List T;
    private RecyclerView.Adapter adapter;

    public DefaultItemTouchHelper(RecyclerView.Adapter adapter, List T) {
        this.adapter = adapter;
        this.T = T;
    }

    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        /**
         * 获取触摸响应的方向 包含两个(0表示不做处理)
         * 1.拖动dragFlags
         * 2.侧滑删除swipeFlags
         */

        //侧滑处理
        int swipeFlags = 0;
        //拖动处理
        int dragFlags = 0;

        if(recyclerView.getLayoutManager() instanceof GridLayoutManager){
            dragFlags = ItemTouchHelper.LEFT | ItemTouchHelper.UP |
                    ItemTouchHelper.RIGHT| ItemTouchHelper.DOWN;
            swipeFlags = 0;
        }else if(recyclerView.getLayoutManager() instanceof LinearLayoutManager){
            LinearLayoutManager linearLayoutManager = (LinearLayoutManager) recyclerView.getLayoutManager();
            if(linearLayoutManager.getOrientation() == LinearLayoutManager.HORIZONTAL){
                dragFlags = ItemTouchHelper.LEFT|ItemTouchHelper.RIGHT;
                swipeFlags = ItemTouchHelper.DOWN;
            }else {
                dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN;
                swipeFlags = ItemTouchHelper.LEFT;
            }
        }
        return makeMovementFlags(dragFlags,swipeFlags);
    }

    /**
     * 拖动时的回调方法
     * @param recyclerView
     * @param viewHolder
     * @param target
     * @return
     */
    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder,
                          RecyclerView.ViewHolder target) {
        // 获取原来的位置
        int oldPostion = viewHolder.getAdapterPosition();
        // 目标位置
        int targetPostion = target.getAdapterPosition();
        // 改变实际的数据集
        Collections.swap(T,oldPostion,targetPostion);
        // 通知刷新数据
        adapter.notifyItemMoved(oldPostion,targetPostion);
        return true;
    }

    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        //处理侧滑操作
        int postion = viewHolder.getAdapterPosition();
        T.remove(postion);
        adapter.notifyItemRemoved(postion);
    }

    @Override
    public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState) {
        super.onSelectedChanged(viewHolder, actionState);
        if(actionState != ItemTouchHelper.ACTION_STATE_IDLE){//不是拖动或滑动状态时
            viewHolder.itemView.setBackgroundColor(Color.parseColor("#F6F6F6"));
        }
    }

    @Override
    public void clearView(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        super.clearView(recyclerView, viewHolder);
        viewHolder.itemView.setBackgroundColor(0);
    }

    /**
     * 默认返回true,可不用重写，如返回false只能拖动，会导致排序不起作用
     */
//    @Override
//    public boolean canDropOver(RecyclerView recyclerView, RecyclerView.ViewHolder current,        //                                 RecyclerView.ViewHolder target) {
//        return true;
//    }

    /**
     * 是否开启长按拖动
     * @return
     */
    @Override
    public boolean isLongPressDragEnabled() {
        return true;
    }

    //返回是否可以滑动
    @Override
    public boolean isItemViewSwipeEnabled() {
        return true;
    }
```

### 使用

```java
ItemTouchHelper itemTouchHelper = new ItemTouchHelper(new DefaultItemTouchHelper(adapter,items));
itemTouchHelper.attachToRecyclerView(recyclerView);
```

