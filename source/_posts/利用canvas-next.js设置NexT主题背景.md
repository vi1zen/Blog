---
title: 利用canvas-next.js设置NexT主题背景
date: 2017-02-12 20:09:21
categories:
 - NexT
tags: 
 - NexT
 - canvas-nest.js
---

  NexT的主题背景看久了会有些闷，所以想改下背景，网上搜了下发现基本都是直接设置一张静态图片，我想能不能利用HTML5的canvas特性设置一个动态背景呢，答案是可以的，详细教程如下：
![示例背景图][1]
----------
<!-- more -->
### canvas-nest.js介绍
> **canvas-nest.js**

>一个基于html5 canvas绘制的网页背景效果，非常赞！如果需要 wordpress插件，在插件库搜索 canvas-nest 或者看看项目 canvas-nest-for-wp。

#### 特性

> * 不依赖任何框架或者内库，比如不依赖 jQuery，使用原生的 javascript。
> * 非常小，只有1.6 kb，如果开启 gzip，可以更小。
> * 非常容易实现，配置简单，即使你不是web开发者，也能简单搞定。

#### 使用

使用非常简单，感觉都没有必要写这一节内容。

将下面的代码插入到 <body> 和 </body> 之间.

``` javascript
<script type="text/javascript" src="//cdn.bootcss.com/canvas-nest.js/1.0.1/canvas-nest.min.js"></script>
```

强烈建议在 </body>标签上方. 例如下面的代码结构:

``` javascript
<html>
<head>
	...
</head>
<body>
	...
	...
    <script type="text/javascript" src="//cdn.bootcss.com/canvas-nest.js/1.0.1/canvas-nest.min.js">
    </script>
</body>
</html>

```

请注意不要将代码置于`<head> </head>`里面.

然后就完成了，打开网页即可看到效果!

配置和配置项

> * color: 线条颜色, 默认: '0,0,0' ；三个数字分别为(R,G,B)，注意用,分割
> * opacity: 线条透明度（0~1）, 默认: 0.5
> * count: 线条的总数量, 默认: 150
> * zIndex: 背景的z-index属性，css属性用于控制所在层的位置, 默认: -1
> * Example:

<script type="text/javascript" color="0,0,255" opacity='0.7' zIndex="-2" count="99" src="//cdn.bootcss.com/canvas-nest.js/1.0.1/canvas-nest.min.js"></script>
这些属性配置在引用js的script标签中，作为它的一个属性值。所有的配置项都有默认值，如果你不知道怎么设置，可以先不设置这些配置项，就使用默认值看看效果也可以的。
> 以上转载自https://github.com/hustcc/canvas-nest.js/blob/master/README-zh.md

### 主题配置文件

在站点配置文件中新增以下内容

``` javascript
# Canvas-nest
canvas_nest: true

vendors://此处不设置也行
    canvas_nest: //cdn.bootcss.com/canvas-nest.js/1.0.1/canvas-nest.min.js
```

在`\themes\next\layout\_scripts\vendors.swig`文件中新增以下内容

``` javascript
{% if theme.canvas_nest %}
  {% set js_vendors.canvas_nest  = 'canvas-nest/canvas-nest.min.js' %}
  <script type="text/javascript" color="0,0,255" opacity='0.5' zIndex="-2" count="100" src="//cdn.bootcss.com/canvas-nest.js/1.0.1/canvas-nest.min.js"></script>
{% endif %}
```


  [1]: http://ol9j5v5dg.bkt.clouddn.com/NexT_background_image.png