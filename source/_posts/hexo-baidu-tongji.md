---
title: Hexo添加百度统计
date: 2016-08-09 20:19:29
tags: 工具
---
百度统计官方没有对hexo网站添加统计代码的引导，这里以yilia主题添加百度统计代码为例为hexo网站添加百度统计功能。

* 编辑文件themes/yilia/_config.yml,注释原来的 google_analytics: '' 一行，改为 baidu_tongji: true
``` xml
#google_analytics: ''
baidu_tongji: true
```
* 新建 themes/yilia/layout/_partial/baidu_tongji.ejs，以我从百度统计获取到的统计代码为例。
``` javascript
<% if (theme.baidu_tongji) { %>
<script type="text/javascript">
//在百度统计获取到的统计代码
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "//hm.baidu.com/hm.js?xxxxxxxxxxxxxxxxx";
  var s = document.getElementsByTagName("script")[0];
  s.parentNode.insertBefore(hm, s);
})();
</script>
<% } %>
```
* 编辑themes/yilia/layout/_partial/head.ejs 在 </head> 前添加<%- partial("baidu_tongji") %>
``` html
<%- partial("baidu_tongji") %>
</head>
```
* 敲hexo g和hexo d部署你的代码，再在百度统计中检查统计代码。

至此，hexo添加百度统计成功。
