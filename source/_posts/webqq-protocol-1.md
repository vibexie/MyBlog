---
title: WebQQ协议分析之登录
date: 2016-08-12 23:46:32
tags:  QQ机器人
---
QQ机器人，听起来是个很厉害的玩意，你可能遇到过某些QQ群里，你发送一条消息，就有一个机器人立马给你回复消息。
同样，我们可能有这么个想法，就是如何通过程序去控制某个QQ群里的账号，可以接收消息，并对消息处理后再进行回复。
现在，你的方案是什么？破解pc版QQ？破解Android版QQ?破解Mac版QQ？这些方案你都不用去考虑了，毕竟腾讯这么大公司你懂得...
其实WebQQ是一个很好的切入点，自2015年10月起，Web QQ废除了原先的用户名/密码登录，取而代之的是手机QQ扫二维码的登录方式，这也使得Github上几乎所有的相关项目都完全无法使用了。

<!-- more -->
目前版本的Web QQ协议，整个登录流程包含以下五步：
1. 获取二维码
2. 确认二维码已被扫描
3. 获取鉴权参数ptwebqq
4. 获取鉴权参数vfwebqq
5. 获取鉴权参数uin和psessionid

登录的目的是获得以下五个参数，用于之后请求其它接口：
1. ptwebqq：保存在Cookie中的鉴权信息
2. vfwebqq：类似于Token的鉴权信息
3. psessionid：类似于SessionId的鉴权信息
4. clientid：设备id，为固定值53999199
5. uin：登录用户id（其实就是当前登录的QQ号）

### 流程一：获取二维码
请求方式：Get
url：https://ssl.ptlogin2.qq.com/ptqrshow?appid=501004106&e=0&l=M&s=5&d=72&v=4&t=0.1
返回内容：二维码图片（PNG格式）

### 流程二：获取二维码扫描状态
请求方式：Get
url：https://ssl.ptlogin2.qq.com/ptqrlogin?webqq_type=10&remember_uin=1&login2qq=1&aid=501004106 &u1=http%3A%2F%2Fw.qq.com%2Fproxy.html%3Flogin2qq%3D1%26webqq_type%3D10 &ptredirect=0&ptlang=2052&daid=164&from_ui=1&pttype=1&dumy=&fp=loginerroralert &action=0-0-157510&mibao_css=m_webqq&t=1&g=1&js_type=0&js_ver=10143&login_sig=&pt_randsalt=0

referer：https://ui.ptlogin2.qq.com/cgi-bin/login?daid=164&target=self&style=16&mibao_css=m_webqq&appid=501004106&enable_qlogin=0&no_verifyimg=1 &s_url=http%3A%2F%2Fw.qq.com%2Fproxy.html&f_url=loginerroralert &strong_login=1&login_state=10&t=20131024001

返回内容：
* 扫描前（未失效）
ptuiCB('66','0','','0','二维码未失效。（3203423232）','');
* 扫描前（已失效）
ptuiCB('65','0','','0','二维码已失效。(4012918406)', '');
* 扫描后，认证前
ptuiCB('66','0','','0','二维码认证中。（3203423232）','');
* 认证后
ptuiCB('66','0','','0','http://ptlogin4.web2.qq.com/check_sig?xxxxxx','');

注：这个请求可以直接轮训请求，直到认证成功后，将返回的地址保存下来用作下次请求。

### 流程三：获取ptwebqq
请求方式：Get
url：http://s.web2.qq.com/api/getvfwebqq?ptwebqq=#{ptwebqq}&clientid=53999199&psessionid=&t=0.1

referer：http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1

url中需要填入上一步获取到的ptwebqq，请求成功后会返回一个JSON，将result.vfwebqq保存下来。

### 流程四：获取vfwebqq
请求方式：Get
url：http://s.web2.qq.com/api/getvfwebqq?ptwebqq=#{ptwebqq}&clientid=53999199&psessionid=&t=0.1

referer：http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1

url中需要填入上一步获取到的ptwebqq，请求成功后会返回一个JSON，将result.vfwebqq保存下来。

### 流程五：获取psessionid和uin
请求方式：Post
url：http://d1.web2.qq.com/channel/login2

referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

表单数据只有一个，Key为r，Value是一个JSON，内容为：
``` json
{
  "ptwebqq": "#{ptwebqq}",
  "clientid": 53999199,
  "psessionid": "",
  "status": "online",
}
```
其中真正的动态参数只有ptwebqq，请求成功后会返回一个JSON，将result.uin和result.psessionid保存下来。

需要注意的是，这里也返回了一个result.vfwebqq，但是这个值是无用的。

到此为止整个登录流程就结束了，概括一下最开始提到的五个数据：
1. ptwebqq：在流程三中通过从Cookie中获得
2. vfwebqq：在流程四中通过返回的JSON中获得（在流程五中也会返回一个，不要使用那个）
3. uin：在流程五中通过返回的JSON中获得（其实就是qq号）
4. psessionid：在流程五中通过返回的JSON中获得
5. clientid：固定值为53999199

登录成功后一般可以维持2天左右，并且如今Web QQ是不会出现顶号情况的，但是多终端登录后在接收消息等接口可能会出现冲突的情况。


