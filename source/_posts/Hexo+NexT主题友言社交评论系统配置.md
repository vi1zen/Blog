---
title: Hexo+NexT主题友言社交评论系统配置
date: 2017-3-24 14:03:50
categories:
 - NexT
tags: 
 - 友言
---

最近发现多说的评论系统要关闭了，[官网链接：](http://dev.duoshuo.com/threads/58d1169ae293b89a20c57241)
![duoshuo](http://ol9j5v5dg.bkt.clouddn.com/duoshuo_close.png)

没办法，只好换一个了，国内做社交评论系统的就只有多说、畅言、友言这三家，畅言的话需要ICP备案号，对于我们这些草根站长，服务器一般用github或者海外的就太不友好了，最后发现友言也不错，具体的配置如下：

<!-- more -->

1. 添加youyan_uid

> 在站点配置文件_config.yml文件中配置,xxxxxxx为你在友言注册的UID

```
youyan_uid: xxxxxxx

```
至此，友言评论系统就可以使用了
![youyan](http://ol9j5v5dg.bkt.clouddn.com/youyan_test.png)

如果还是没有的话，请[更新NexT主题版本](http://theme-next.iissnan.com/getting-started.html#install-next-theme)或者配置如下文件

2. 新增youyan.swig文件

`在如图所在的路径下建立youyan.swig文件`

![](http://ol9j5v5dg.bkt.clouddn.com/youyan_location.png)

`嵌入友言评论框通用代码`

```
{% if not (theme.duoshuo and theme.duoshuo.shortname)
  and not theme.duoshuo_shortname
  and not theme.disqus_shortname
  and not theme.hypercomments_id
  and not theme.gentie_productKey %}

  {% if theme.youyan_uid %}
    {% set uid = theme.youyan_uid %}

    {% if page.comments %}
      <!-- UY BEGIN -->
      <script type="text/javascript" src="http://v2.uyan.cc/code/uyan.js?uid={{uid}}"></script>
      <!-- UY END -->
    {% endif %}
  {% endif %}

{% endif %}

```

`在comments.swig文件中引用`

```
{% include './comments/youyan.swig' %}

```

配置完成发布到正式环境就可以使用友言了。