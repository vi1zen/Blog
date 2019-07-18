---
title: ViewPager限定预加载的页面个数setOffscreenPageLimit(int limit)
date: 2017-03-14 17:21:57
categories:
 - Android
tags: 
 - ViewPager
---
ViewPage中方法setOffscreenPageLimit(int limit)的参数假如为1,这表示你的预告加载的页面数量是1,假设当前有四个Fragment的tab,显示一个,预先加载下一个.这样你在移动前就已经加载了下一个界面,移动时就可以看到已经加载的界面了. 

<!-- more -->

从日志里面可以看到onActivityCreated等方法在初始化第一个Fragment完成后就会初始化下一个Fragment. 
假设你想预先加载多个Fragment可以使用它提供的公共方法: 

```java
public void setOffscreenPageLimit(int limit) {
    if(limit < 1) {
        Log.w("ViewPager", "Requested offscreen page limit " + limit + " too small; defaulting to " + 1);
        limit = 1;
    }

    if(limit != this.mOffscreenPageLimit) {
        this.mOffscreenPageLimit = limit;
        this.populate();
    }

}  
```

从这个方法来看,不管你设置什么值,至少会预先加载下一个Fragment,你想预先加载几个就可以传入相应的参数,这种情况如音乐播放时,如果有自动加载歌词就可以使用了.  
  
如果你的界面需要加载一些大量的数据,但你不想预先加载下一个界面(需要网络或耗时的操作),使用ViewPager却很无耐.特别是下一个界面有可能你一段很长时间不会使用到,如微博,在显示主页后我不想立即加载下一个界面,因为都有ListView,如果我不访问它,就不必加载无用的资源.  
  
可以通过修改这个值,但有,修改后就会有一个麻烦的地方,因为移动时不会预先加载下一个界面的关系,所以会看到一片黑色的背景.

如果不介意黑色背景,可以覆盖这个类,然后定义默认的加载数量为0,private int mOffscreenPageLimit = DEFAULT_OFFSCREEN_PAGES=0;就是不预先加载下一个界面.  
  
如果想预加载，可以使用原来的ViewPager，或这里直接改为mOffscreenPageLimit=你要加载的数量。  
  
除此之外，如果我们设定mOffscreenPageLimit　＝　１，则当我们我们滑到第三个页面的时候，第一个页面的视图将被销毁，当我们滑到第一个页面时，第三个页面的视图会被销毁。但滑到第二个页面时，三个界面的视图都不会销毁。
其实这个值指的是，当前view的左右两边的预加载的页面的个数。也就是说，如果这个值mOffscreenPageLimit　＝　３，那么任何一个页面的左边可以预加载３个页面，右边也可以加载３页面。
