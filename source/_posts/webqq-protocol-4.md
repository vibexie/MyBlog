---
title: WebQQ协议分析之群&讨论组
date: 2016-08-16 08:48:42
tags: QQ机器人
---
### 获取群列表
请求方式：Post
url：http://s.web2.qq.com/api/get_group_name_list_mask2
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

请求参数和获取好友列表一样，这里顺便再提供一个Java版的加密算法：
``` java
private static String hash(long x, String K) {
    int[] N = new int[4];
    for (int T = 0; T < K.length(); T++) {
        N[T % 4] ^= K.charAt(T);
    }
    String[] U = {"EC", "OK"};
    long[] V = new long[4];
    V[0] = x >> 24 & 255 ^ U[0].charAt(0);
    V[1] = x >> 16 & 255 ^ U[0].charAt(1);
    V[2] = x >> 8 & 255 ^ U[1].charAt(0);
    V[3] = x & 255 ^ U[1].charAt(1);

    long[] U1 = new long[8];

    for (int T = 0; T < 8; T++) {
        U1[T] = T % 2 == 0 ? N[T >> 1] : V[T >> 1];
    }

    String[] N1 = {"0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"};
    String V1 = "";
    for (long aU1 : U1) {
        V1 += N1[(int) ((aU1 >> 4) & 15)];
        V1 += N1[(int) (aU1 & 15)];
    }
    return V1;
}
```
<!-- more -->
请求成功后会返回的JSON格式为：
``` json
{
    "retcode": 0,
    "result": {
        "gmasklist": [],
        "gnamelist": [
            {
                "flag": 167864331,
                "name": "群名",
                "gid": 14611812014,
                "code": 3157131718
            }
        ],
        "gmarklist": [
            {
                "uin": 18796074161,
                "markname": "备注"
            }
        ]
    }
}
```
gnamelist包含群信息，其中name为群名称，gid为群编号（用于发消息），code也是群编号（用于获取群详细信息），gmarklist包含群的备注信息，markname为备注名。gmasklist目前都是空列表。

### 获取群详细信息
请求方式：Get
url：http://s.web2.qq.com/api/get_group_info_ext2?gcode=#{group_code}&vfwebqq=#{vfwebqq}&t=0.1
referer：http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1

请求参数中的gcode就是群列表中获取的code（注意不是gid）。请求成功后的JSON格式如下：
``` json
{
    "retcode": 0,
    "result": {
        "stats": [
            {
                "client_type": 1,
                "uin": 2146674552,
                "stat": 50
            }
        ],
        "minfo": [
            {
                "nick": "昵称",
                "province": "北京",
                "gender": "male",
                "uin": 3623536468,
                "country": "中国",
                "city": ""
            }
        ],
        "ginfo": {
            "face": 0,
            "memo": "群公告！",
            "class": 25,
            "fingermemo": "",
            "code": 591539174,
            "createtime": 1231435199,
            "flag": 721421329,
            "level": 4,
            "name": "群名称",
            "gid": 2419762790,
            "owner": 3509557797,
            "members": [
                {
                    "muin": 3623536468,
                    "mflag": 192
                }
            ],
            "option": 2
        },
        "cards": [
            {
                "muin": 3623536468,
                "card": "群名片"
            }
        ],
        "vipinfo": [
            {
                "vip_level": 6,
                "u": 2390929289,
                "is_vip": 1
            }
        ]
    }
}
```
这个请求不光会返回群信息，还会把群里面所有成员的信息都返回，所以数据量会比较大，而且结构复杂解析写起来也麻烦。

其中ginfo包含群的基本信息，memo是群公告、name是群名称、createtime是创建时间、owner是创建者的uin、gid和code分别对应之前群列表的id和code。这里面虽然有一个members表示群成员，但是里面信息太少就不用解析了。stats表示群成员的登录状态，minfo是群成员的基本信息，cards是成员在群中的名片，vipinfo是会员信息，这些在好友列表中都介绍过，就不再重复了。

这里有一点需要注意的是，如果群里没有人改名片，cards并不会返回一个空列表，而是直接没有这个key，所以对于这个字段要进行一下判空操作，否则有可能会出错。

### 获取讨论组列表
请求方式：Get
url：http://s.web2.qq.com/api/get_discus_list?clientid=53999199&psessionid=#{psessionid}&vfwebqq=#{vfwebqq}&t=0.1
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

请求成功后遍历dnamelist，did为讨论组编号，name为讨论组名称。
``` json
{
    "retcode": 0,
    "result": {
        "dnamelist": [
            {
                "did": 167864331,
                "name": "讨论组名",
            }
        ]
    }
}
```

### 获取讨论组详细信息
请求方式：Get
url：http://d1.web2.qq.com/channel/get_discu_info?did=#{discuss_id}&psessionid=#{psessionid}&vfwebqq=#{vfwebqq}&clientid=53999199&t=0.1
referer：http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2

请求成功的JSON格式如下：
``` json
{
    "result": {
        "info": {
            "did": 236426547,
            "discu_name": "啊啊啊啊",
            "mem_list": [
                {
                    "mem_uin": 3466696377,
                    "ruin": 1301948
                }
            ]
        },
        "mem_info": [
            {
                "nick": "Hey",
                "uin": 3466696377
            }
        ],
        "mem_status": [
            {
                "client_type": 7,
                "status": "online",
                "uin": 3253160543
            }
        ]
    },
    "retcode": 0
}
```
info为讨论组资料，mem_info为讨论组成员的信息，mem_status为登录状态，基本和群信息的格式差不多。

