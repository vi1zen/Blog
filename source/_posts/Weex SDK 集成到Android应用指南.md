---
title: Weex SDK 集成到Android应用指南
date: 2018.03.23 10:10:10
categories:
 - Android
tags: 
 - Weex
 
---
### 引言

最近项目变更的比较频繁，跨平台开发也提上了议程，我也在考虑是不是需要引入一些动态化比较成熟的方案，目前就RN,Weex这两家有比较成熟的方案，RN社区很成熟，但上手门槛太高，Weex恰好相反，由于团队都是用的Vue,最后选择了Weex，下面是一些入坑指南。

### 开发环境搭建

这个网上一搜一大堆，这里就不贴了，随便附上官网链接:
> http://weex.apache.org/cn/guide/

需要注意的是在使用npm安卓Weex时如果使用最新版本的npm有可能会包下面的错误
```node
npm Error: write after end
```
这里把npm降级成5.6.0版本就可以了

```java
npm install npm@5.6.0
```

<!--more-->


### 集成到Android应用

#### 添加SDK
这里同样参考官网教程

> http://weex.apache.org/cn/guide/integrate-to-your-app.html

主要注意这个方法

```java
mWXSDKInstance.render("WXDemo", WXFileUtils.loadFileContent("js/weex_demo.js",this)
                ,null,null,-1,-1, WXRenderStrategy.APPEND_ASYNC);
```

其中的`js/weex_demo.js`表示的是在assets目录下的js文件路径,不能直接引用.we或者.vue文件，因为Android不能识别这两种文件类型，需要通过Weex编译成js文件才能正确加载。

### 编写前端文件

这一步和写普通的前端没什么区别，不熟悉vue语法的建议先了解下vue.js[官网][1]
```html
<template>
    <div>
        <text class="text1">Hello World!</text>
        <slider class="slide" interval="{{intervalTime}}" auto-play="true">
            <div class="slide_pages" repeat="{{itemDatas}}">
                <image class="image" src="{{picUrl}}"></image>
                <text class="title">{{title}}</text>
            </div>
        </slider>
    </div>

</template>

<script>
    module.exports = {
        data: {
            intervalTime: "2000",
            itemDatas: [
                {
                    title: '七牛云',
                    picUrl: 'https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=bf02b32ba2ec8a1300175fb2966afaea/b58f8c5494eef01fd2078f01ebfe9925bd317de7.jpg'
                },
                {
                    title: '腾讯云',
                    picUrl: 'https://gss2.bdstatic.com/9fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike116%2C5%2C5%2C116%2C38/sign=3a6b0e32a018972bb737089887a410ec/78310a55b319ebc45d3844bf8926cffc1e171657.jpg'
                },
                {
                    title: '阿里云',
                    picUrl: 'https://gss0.bdstatic.com/-4o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike72%2C5%2C5%2C72%2C24/sign=097ea254a751f3ded7bfb136f5879b7a/aec379310a55b3197face8eb4ba98226cefc17d2.jpg'
                }
            ]
        },
        methods: {}
    }
</script>

<style>
    .text1{
        font-size: 50px;
        text-align: center;
        color: deepskyblue;
        background: black;
    }
    .title {
        font-size: 50px;
        text-align: center;
        background: white;
    }

    .slide_pages {
        width: 750px;
        height: 400px;
    }

    .slide {
        width: 750px;
        height: 400px;
        flex-direction: row;
    }
    .image{
        width: 750px;
        height: 300px;
    }
</style>
```

weex中.we或者.vue文件由`<template>，<script>，<style>`三部分构成，`<template>`为必选，`<script>，<style>`为非必选，这里类比html文件就很清楚了，需要注意的是weex默认宽度是750px

编写好.we或者.vue文件后，通过weex本地编译命令转成js文件

```shell
weex compile vue js
```

最后运行Android工程就能看到效果了
![此处输入图片的描述][2]


  [1]: https://cn.vuejs.org/
  [2]: http://ol9j5v5dg.bkt.clouddn.com/device-2018-03-23-092232.png