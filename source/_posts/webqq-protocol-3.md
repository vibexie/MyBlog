---
title: WebQQ协议分析之好友
date: 2016-08-16 00:30:16
tags: QQ机器人
---
### 获取好友列表
请求方式：Post
url：http://s.web2.qq.com/api/get_user_friends2
referer：http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1

请求参数只有一个r，值是JSON，内容为：
``` json
{
    "vfwebqq": "${vfwebqq}",
    "hash": "${hash}"
}
```
<!-- more -->
vfwebqq依旧是登录后获得的参数，hash是uin和ptwebqq进行加密后的数据，最新的加密算法如下：
``` bash
def self.hash(uin, ptwebqq)
  n = Array.new(4, 0)
  ptwebqq.chars.each_index { |i| n[i % 4] ^= ptwebqq[i].ord }
  u = ['EC', 'OK']
  v = Array.new(4)
  v[0] = uin >> 24 & 255 ^ u[0][0].ord;
  v[1] = uin >> 16 & 255 ^ u[0][1].ord;
  v[2] = uin >> 8 & 255 ^ u[1][0].ord;
  v[3] = uin & 255 ^ u[1][1].ord;
  u = Array.new(8)
  (0...8).each { |i| u[i] = i.odd? ? v[i >> 1] : n[i >> 1] }
  n = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']
  v = ''
  u.each do |i|
    v << n[(i >> 4) & 15]
    v << n[i & 15]
  end
  v
end
```
请求成功后会返回一个如下格式的JSON：
``` json
{
    "retcode": 0,
    "result": {
        "friends": [
            {
                "flag": 4,
                "uin": 41837138855,
                "categories": 1
            }
        ],
        "marknames": [
            {
                "uin": 37761915054,
                "markname": "备注",
                "type": 0
            }
        ],
        "categories": [
            {
                "index": 0,
                "sort": 2,
                "name": "同学"
            }
        ],
        "vipinfo": [
            {
                "vip_level": 0,
                "u": 20191343597,
                "is_vip": 0
            }
        ],
        "info": [
            {
                "face": 603,
                "flag": 4751942,
                "nick": "昵称",
                "uin": 41837138855
            }
        ]
    }
}
```
其中所有的uin和u都代表着用户的编号。categories保存分组信息，index是编号、sort是顺序、name是名称。marknames的markname是备注，vipinfo的is_vip和vip_level分别代表用户是否为会员和会员等级，info的nick为用户昵称，friends中的categories表示所属分组。

在这里我也很疑惑为什么拆的这么细，解析的时候着实麻烦，最后也只能归结于应该是一个历史遗留问题。

### 获取好友在线状态
刚才刚提到好友列表的JSON拆的很复杂，这里又突然冒出来个单独的接口仅仅是为了获取在线状态。如此设计的原因可能是因为这两个接口的调用频率不同（好友列表很少改变，但是好友的在线状态时时都会改变），或者又是一个历史遗留问题。

请求方式：Get
url：http://d1.web2.qq.com/channel/get_online_buddies2?vfwebqq=#{vfwebqq}&clientid=53999199&psessionid=#{psessionid}&t=0.1
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

url中的vfwebqq和psessionid还是一样，就不多说了。请求成功会返回在线的用户，以及终端类型（client_type）：
``` json
{
    "result": [
        {
            "client_type": 1,
            "status": "online",
            "uin": 3017767504
        }
    ],
    "retcode": 0
}
```

### 通过uin获得QQ号
之前也提到过uin只是一个临时的用户编号，随时都会发生改变。所以如果你想真正知道和你聊天的人是谁，最好还是通过这个接口获得Ta的QQ号。

请求方式：Get
url：http://s.web2.qq.com/api/get_friend_uin2?tuid=#{uin}&type=1&vfwebqq=#{vfwebqq}&t=0.1
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2
tuid就是对方的uin。请求成功后取出result.account即可：
``` json
{
    "retcode": 0,
    "result": {
        "uiuin": "",
        "account": 3524125,
        "uin": 1382902354
    }
}
```
### 获取好友详细信息
请求方式：Get
url：http://s.web2.qq.com/api/get_friend_info2?tuin=#{uin}&vfwebqq=#{vfwebqq}&clientid=53999199&psessionid=#{psessionid}&t=0.1
referer：http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1

请求参数没什么特别的就不重复了，请求成功后会返回一堆数据，基本上都是字面上的意思就不多加解释了：
``` json
{
    "retcode": 0,
    "result": {
        "face": 603,
        "birthday": {
            "month": 8,
            "year": 1895,
            "day": 15
        },
        "occupation": "其他",
        "phone": "110",
        "allow": 1,
        "college": "aaa",
        "uin": 1382902354,
        "constel": 7,
        "blood": 5,
        "homepage": "木有",
        "stat": 20,
        "vip_info": 6,
        "country": "乍得",
        "city": "",
        "personal": "这是简介",
        "nick": "ABCD",
        "shengxiao": 11,
        "email": "352323245@qq.com",
        "province": "",
        "gender": "female",
        "mobile": "139********"
    }
}
```
