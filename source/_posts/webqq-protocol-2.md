---
title: WebQQ协议分析之收发消息
date: 2016-08-15 00:13:04
tags: QQ机器人
---
### 轮训消息
请求方式: Post
url: http://d1.web2.qq.com/channel/poll2
referer: http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

请求参数只有一个r，值是一个JSON，内容为：
``` json
{
    "ptwebqq": "#{ptwebqq}",
    "clientid": 53999199,
    "psessionid": "#{psessionid}",
    "key": ""
}

```
<!-- more -->
ptwebqq和psessionid都是登录后获得的参数。
请求成功后返回的内容为:
``` json
{
    "result": [
        {
            "poll_type": "message",
            "value": {
                "content": [
                    [
                        "font",
                        {
                            "color": "000000",
                            "name": "微软雅黑",
                            "size": 10,
                            "style": [
                                0,
                                0,
                                0
                            ]
                        }
                    ],
                    "好啊"
                ],
                "from_uin": 3785096088,
                "msg_id": 25477,
                "msg_type": 0,
                "time": 1450686775,
                "to_uin": 931996776
            }
        }
    ],
    "retcode": 0
}
```
poll_type为message表示这是个好友消息。from_uin是用户的编号，可以用于发消息，但不是qq号。to_uin是接受者的编号，同时也是qq号。time为消息的发送时间，content[0]为字体，后面为消息的内容。其他字段暂时不知道有何意义。

如果为群消息，返回内容为：
``` json
{
    "result": [
        {
            "poll_type": "group_message",
            "value": {
                "content": [
                    [
                        "font",
                        {
                            "color": "000000",
                            "name": "微软雅黑",
                            "size": 10,
                            "style": [
                                0,
                                0,
                                0
                            ]
                        }
                    ],
                    "好啊",
                ],
                "from_uin": 2323421101,
                "group_code": 2323421101,
                "msg_id": 50873,
                "msg_type": 0,
                "send_uin": 3680220215,
                "time": 1450687625,
                "to_uin": 931996776
            }
        }
    ],
    "retcode": 0
}
```
其中poll_type会变成group_message，group_code和from_uin都为群的编号，可以用于发群消息，但不是群号。send_uin为发信息的用户的编号。其他的字段和上面的相同。

如果是讨论组消息，poll_type会变为discu_message，did为讨论组的编号，其他的字段都和群消息相同。
``` json
{
    "result": [
        {
            "poll_type": "discu_message",
            "value": {
                "content": [
                    [
                        "font",
                        {
                            "color": "000000",
                            "name": "微软雅黑",
                            "size": 10,
                            "style": [
                                0,
                                0,
                                0
                            ]
                        }
                    ],
                    "好啊",
                ],
                "from_uin": 2322423201,
                "did": 2322423201,
                "msg_id": 50873,
                "msg_type": 0,
                "send_uin": 3680220215,
                "time": 1450687625,
                "to_uin": 931996776
            }
        }
    ],
    "retcode": 0
}
```
这里有几点需要注意：
1. 服务端收到这个请求后，如果没有新消息，会一直保持住链接，所以遇到ReadTimeout异常是正常的
2. Web QQ无法接受图片、@别人、自定义表情等消息，消息内容只有默认表情和文字
3. 如果消息内容为表情，content[1]的内容就不是String类型了，而是一个JSONArray类型，里面有表情的编号
4. 所以content的长度有可能大于2，代表着消息的内容为文字和表情的混排，content[1]开始的每一位都是分割后的文字或表情
5. 这个请求有时候会返回retcode的值为103，此时需要登录Smart QQ，确认能收到消息后点击设置-退出登录，就会恢复正常了
6. 在这里接受到的uin、group_code等并不是固定的，而是会改变的，所以不要长时间保存这些信息。

### 发送消息给好友
请求方式：Post
url：http://d1.web2.qq.com/channel/send_buddy_msg2
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

请求参数只有一个r，值是一个JSON，内容为：
``` json
{
    "to": #{user_id},
    "content": [
        "#{msg}",
        [
            "font",
            {
                "name": "宋体",
                "size": 10,
                "style": [
                    0,
                    0,
                    0
                ],
                "color": "000000"
            }
        ]
    ].to_string(),
    "face": 522,
    "clientid": 53999199,
    "msg_id": 65890001,
    "psessionid": "#{psessionid}",
}
```
psessionid是登录后获取的参数，msg是你需要发送的内容，to是用户编号，msg_id只要是一个比较大的数字即可，face暂时不知道有什么用。

这里有一点需要注意的是，我在content的值后面加了一个to_string，因为它不是一个JSONArray类型，而是String类型，在Smart QQ上抓个包也可以发现。如果这里直接提交了一个JSONArray的话，应该会返回1000001错误。

如果发送成功，会返回如下数据：
``` json
{
    "errCode": 0,
    "msg": "send ok"
}
```

### 发送群消息
请求方式：Post
url：http://d1.web2.qq.com/channel/send_qun_msg2
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

请求参数和上面几乎一样，只是将to替换成了group_uin：
``` json
{
    "group_uin": #{group_uin},
    "content": [
        "#{msg}",
        [
            "font",
            {
                "name": "宋体",
                "size": 10,
                "style": [
                    0,
                    0,
                    0
                ],
                "color": "000000"
            }
        ]
    ].to_string(),
    "face": 522,
    "clientid": 53999199,
    "msg_id": 65890001,
    "psessionid": "#{psessionid}",
}
```

### 发送讨论组消息
请求方式：Post
url：http://d1.web2.qq.com/channel/send_discu_msg2
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

格式也是一样的，只是替换为了did：
``` json
{
    "did": #{discuss_id},
    "content": [
        "#{msg}",
        [
            "font",
            {
                "name": "宋体",
                "size": 10,
                "style": [
                    0,
                    0,
                    0
                ],
                "color": "000000"
            }
        ]
    ].to_string(),
    "face": 522,
    "clientid": 53999199,
    "msg_id": 65890001,
    "psessionid": "#{psessionid}",
}
```
