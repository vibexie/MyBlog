---
title: Yilia使用畅言
date: 2017-04-04 17:31:32
tags: 工具
---
多说已经倒闭，其系统即将关闭了，想必很多使用多说的站长都需要去找一个多说的替代品了。
很早就有人提议Yilia的作者更新多说的替代品，但是作者说需要观望一段时间。
由于我自己也使用Yilia主题，为了尽早将多说替换成畅言，所以稍稍折腾了一下。

1. 进入[畅言](http://changyan.kuaizhan.com/)注册并完善你的账号
2. 根据你的需要，获取相关的安装代码
3. 修改Yilia主题，编辑/layout/_partial/post/duoshuo.ejs
``` JavaScript
<div class="duoshuo">

<!-- 替换为畅言 Start -->
<div id="SOHUCS" sid="<%=title%>" ></div>
<script type="text/javascript">
(function(){
var appid = '';
var conf = '';
var width = window.innerWidth || document.documentElement.clientWidth;
if (width < 960) {
window.document.write('<script id="changyan_mobile_js" charset="utf-8" type="text/javascript" src="http://changyan.sohu.com/upload/mobile/wap-js/changyan_mobile.js?client_id=' + appid + '&conf=' + conf + '"><\/script>'); } else { var loadJs=function(d,a){var c=document.getElementsByTagName("head")[0]||document.head||document.documentElement;var b=document.createElement("script");b.setAttribute("type","text/javascript");b.setAttribute("charset","UTF-8");b.setAttribute("src",d);if(typeof a==="function"){if(window.attachEvent){b.onreadystatechange=function(){var e=b.readyState;if(e==="loaded"||e==="complete"){b.onreadystatechange=null;a()}}}else{b.onload=a}}c.appendChild(b)};loadJs("http://changyan.sohu.com/upload/changyan.js",function(){window.changyan.api.config({appid:appid,conf:conf})}); } })(); </script>
<!-- 替换为畅言 End -->

</div>

<!-- 注意sid是畅言标示你文章的id，所以得生成一个唯一的sid，我自己
使用<%=title%>，以文章标题标示id -->
```

