# 魅族开放平台PUSH系统HTTP接口文档

**文档变更记录**

| 日期 | 作者 | 版本 | 变更描述
| --- | --- | --- | --- |
| 2016-10-10 | JasperXgwang | 1.0.0-RC01| 撰写文档 |

# 目录 <a name="index"/>
* [一.API接口规范](#api_standard_index)
    * [接口响应规范](#api_resp_index)
    * [接口签名规范](#api_sign_index)
* [二.API说明](#api_common_index) 
    * [前言](#preface_index)      
    * [非任务推送](#untask_push_index)  
        * [应用场景](#yycj_1_index)
        * [pushId推送接口（透传消息）](#UnVarnishedMessage_push_index)
        * [pushId推送接口（通知栏消息）](#VarnishedMessage_push_index)
    * [任务推送](#task_push_index)  
        * [pushId推送](#pushId_index)
            * [应用场景](#yycj_2_index)
            * [获取推送taskId](#getTaskId_index)
            * [pushId推送接口（透传消息）](#UnVarnishedMessage_task_push_index)
            * [pushId推送接口（通知栏消息）](#VarnishedMessage_task_push_index)
        * [全部&标签推送](#all_tag_push_index)
            * [应用场景](#yycj_3_index)
            * [应用全部推送](#pushToApp_index)
            * [应用标签推送](#pushToTag_index)
            * [取消任务推送](#cancelTask_index)

# API接口规范 <a name="api_standard_index"/>
## 接口响应规范 <a name="api_resp_index"/>
> HTTP接口遵循魅族API协议规范。返回数据格式统一如下：


```
{
	“code”:”“, //必选,返回码
	“message”:”“, //可选，返回消息，网页端接口出现错误时使用此消息展示给用户，手机端可忽略此消息，甚至服务端不传输此消息
	“value”:”“,// 必选，返回结果
	“redirect”:”“ //可选, returnCode=300 重定向时，使用此URL重新请求
}
```
> Api returnCode定义


code|value
---|---
200|正常
500|其他异常
1001|系统错误
1003|服务器忙
1005|参数错误，请参考API文档
1006|签名认证失败
110000|appId不合法
110001|appKey不合法
110002|pushId未注册
110003|pushId非法
110004|参数不能为空
110009|应用被加入黑名单


## 接口签名规范 <a name="api_sign_index"/>
> 请求参数分别是“k1”、“k2”、“k3”，它们的值分别是“v1”、“v2”、“v3”，计算方法如下所示：
> 
> 1. 将请求参数格式化为“key=value”格式，如“k1=v1”、“k2=v2”、“k3=v3”
> 1. 将格式化好的参数键值对以字典序升序排列后，拼接在一起，如“k1=v1k2=v2k3=v3”
> 1. 在拼接好的字符串末尾追加上应用的Secret Key
> 
> 上述字符串的MD5值即为签名的值。（32位小写）
> 
> 将签名值放在请求的参数中例如sign=MD5_SIGN
> 
> 服务端SDK调用API的应用的私钥Secret Key为 appSecret

# API说明 <a name="api_common_index"/>

## 前言 <a name="preface_index"/>

> 消息推送结果接口响应部分value是map集合的json格式且只返回推送非法的pushId，合法的pushId不予返回，一般情况下，pushId未注册则视为非法。

map部分code定义

code|value
---|---
201|没有权限，服务器主动拒绝
501|推送消息失败（db_error）
513|推送消息失败
518|推送超过配置的速率
519|推送消息失败服务过载
520|消息折叠（短时间内同一设备同一消息收到多次）
110002|pushId未订阅
110003|pushId非法

**注：平台使用pushId来标识每个独立的用户，每一台终端上每一个app拥有一个独立的pushId**

## 非任务推送 <a name="untask_push_index"/>
### 应用场景 <a name="yycj_1_index"/>

> 场景1：查找手机业务需要远程定位位置，可发送消息指令到对应的设备
> 
> 场景2：社区用户回帖消息提醒，用户对发表的帖子有最新回复时，消息提醒发帖者


### pushId推送接口（透传消息） <a name="UnVarnishedMessage_push_index"/>

描述|内容
---|---
接口功能|根据pushId推送
请求方法|Post
请求路径|/garcia/api/server/push/unvarnished/pushByPushId
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
appId|推送应用ID 必填
pushIds|推送设备，多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "title": 推送标题, 【string 非必填，字数显示1~32个】
    "content": 推送内容,  【string 必填，字数限制2000以内】
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【int 非必填，默认值为1】
        "validTime": 有效时长 (1- 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    }
}
```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": {}
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "110002": [
            "J0476035d625e6c64567f71487e040e7d017f0558675b",
            "J0476045d625e6c64567f71487e040e7d017f0558675b",
            "J0476035d625e6sd64567f71487e040e7d017f0558675b"
        ],
        "110003": [
            "J0476035d625e6c64567f714567e040e7d017f0558675b"
        ]
    },
    "redirect": ""
}
```

> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
    "redirect": ""
}
```



### pushId推送接口（通知栏消息） <a name="VarnishedMessage_push_index"/>

描述|内容
---|---
接口功能|根据pushId推送
请求方法|Post
请求路径|/garcia/api/server/push/varnished/pushByPushId
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
appId|推送应用ID 必填
pushIds|推送设备，多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "noticeBarInfo": {
        "noticeBarType": 通知栏样式(0, "标准")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32】
        "content": 推送内容, 【string 必填，字数限制1~100】
    },
    "noticeExpandInfo": {
        "noticeExpandType": 展开方式 (0, "标准"),(1, "文本")【int 非必填，值为0、1】
        "noticeExpandContent": 展开内容, 【string noticeExpandType为文本时，必填】
    },
    "clickTypeInfo": {
        "clickType": 点击动作 (0,"打开应用"),(1,"打开应用页面"),(2,"打开URI页面"),(3, "应用客户端自定义")【int 非必填,默认为0】
        "url": URI页面地址, 【string clickType为打开URI页面时，必填, 长度限制1000字节】
        "parameters":参数 【JSON格式】【非必填】 
        "activity":应用页面地址 【string clickType为打开应用页面时，必填, 长度限制1000字节】
        "customAttribute":应用客户端自定义【string clickType为应用客户端自定义时，必填， 输入长度为1000字节以内】
    },
    "pushTimeInfo": {
        "offLine": 是否进离线消息(0 否 1 是[validTime]) 【int 非必填，默认值为1】
        "validTime": 有效时长 (1 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    },
    "advanceInfo": {
        "suspend":是否通知栏悬浮窗显示 (1 显示  0 不显示) 【int 非必填，默认1】
        "clearNoticeBar":是否可清除通知栏 (1 可以  0 不可以) 【int 非必填，默认1】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        }
    }
}

```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": {}
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "110002": [
            "J0476035d625e6c64567f71487e040e7d017f0558675b",
            "J0476045d625e6c64567f71487e040e7d017f0558675b",
            "J0476035d625e6sd64567f71487e040e7d017f0558675b"
        ],
        "110003": [
            "J0476035d625e6c64567f714567e040e7d017f0558675b"
        ]
    },
    "redirect": ""
}

```

> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
    "redirect": ""
}
```


## 任务推送 <a name="task_push_index"/>
### pushId推送 <a name="pushId_index"/>
#### 应用场景 <a name="yycj_2_index"/>

> 场景1：浏览器对指定的某一大批量pushId用户推送活动或者新闻消息，通过先获取taskId，然后通过taskId批量推送，推送过程中可以根据taskId时时获取推送统计结果

#### 获取推送taskId <a name="getTaskId_index"/>

描述|内容
---|---
接口功能|获取推送taskId
请求方法|Post
请求路径|/garcia/api/server/push/pushTask/getTaskId
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
appId|推送应用ID 必填
pushType|消息类型 0 通知栏 1透传 必填
sign|签名 必填
messageJson|Json格式，具体如下必填

> 通知栏类型（pushType=0）  


```
{
    "noticeBarInfo": {
        "noticeBarType": 通知栏样式(0, "标准")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32】
        "content": 推送内容, 【string 必填，字数限制1~100】
    },
    "noticeExpandInfo": {
        "noticeExpandType": 展开方式 (0, "标准"),(1, "文本")【int 非必填，值为0、1】
        "noticeExpandContent": 展开内容, 【string noticeExpandType为文本时，必填】
    },
    "clickTypeInfo": {
        "clickType": 点击动作 (0,"打开应用"),(1,"打开应用页面"),(2,"打开URI页面"),(3, "应用客户端自定义")【int 非必填,默认为0】
        "url": URI页面地址, 【string clickType为打开URI页面时，必填, 长度限制1000字节】
        "parameters":参数 【JSON格式】【非必填】 
        "activity":应用页面地址 【string clickType为打开应用页面时，必填, 长度限制1000字节】
        "customAttribute":应用客户端自定义【string clickType为应用客户端自定义时，必填， 输入长度为1000字节以内】
    },
    "pushTimeInfo": {
        "offLine": 是否进离线消息(0 否 1 是[validTime]) 【int 非必填，默认值为1】
        "validTime": 有效时长 (1 -72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    },
    "advanceInfo": {
        "suspend":是否通知栏悬浮窗显示 (1 显示  0 不显示) 【int 非必填，默认1】
        "clearNoticeBar":是否可清除通知栏 (1 可以  0 不可以) 【int 非必填，默认1】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        }
    }
}
```

> 透传类型（pushType=1） 


```
{
    "title": 推送标题, 【string 必填，字数显示1~32个】
    "content": 推送内容,  【string 必填，字数限制2000以内】
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【int 非必填，默认值为1】
        "validTime": 有效时长 (1- 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    }
}
```

响应内容

> 成功情况：


```
{
    "code": "200",
    "message": "",
    "value": {
        "taskId": 20457  (任务Id)
        "pushType": 0  (推送类型 0通知栏  1 透传)
        "appId": 100999  (应用的appId)
    },
    "redirect": ""
}
```


#### pushId推送接口（透传消息） <a name="UnVarnishedMessage_task_push_index"/>

描述|内容
---|---
接口功能|根据pushId推送
请求方法|Post
请求路径|/garcia/api/server/push/task/unvarnished/pushByPushId
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
taskId|推送任务ID 必填 
appId|推送应用ID 必填
pushIds|推送设备，多个英文逗号分割必填
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": {}
}
```

> 失败情况


```
{
    "code": "110032",
    "message": "非法的taskId",
    "redirect": "",
    "value": ""
}
```


```
{
    "code": "200",
    "message": "",
    "value": {
        "110002": [
            "J0476035d625e6c64567f71487e040e7d017f0558675b",
            "J0476045d625e6c64567f71487e040e7d017f0558675b",
            "J0476035d625e6sd64567f71487e040e7d017f0558675b"
        ],
        "110003": [
            "J0476035d625e6c64567f714567e040e7d017f0558675b"
        ]
    },
    "redirect": ""
}
```


> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
    "redirect": ""
}
```

#### pushId推送接口（通知栏消息） <a name="VarnishedMessage_task_push_index"/>

描述|内容
---|---
接口功能|根据pushId推送
请求方法|Post
请求路径|/garcia/api/server/push/task/varnished/pushByPushId
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
taskId|推送任务ID 必填 
appId|推送应用ID 必填
pushIds|推送设备，多个英文逗号分割必填
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": {}
}
```

> 失败情况


```
{
    "code": "110032",
    "message": "非法的taskId",
    "redirect": "",
    "value": ""
}
```


```
{
    "code": "200",
    "message": "",
    "value": {
        "110002": [
            "J0476035d625e6c64567f71487e040e7d017f0558675b",
            "J0476045d625e6c64567f71487e040e7d017f0558675b",
            "J0476035d625e6sd64567f71487e040e7d017f0558675b"
        ],
        "110003": [
            "J0476035d625e6c64567f714567e040e7d017f0558675b"
        ]
    },
    "redirect": ""
}
```


> 超速情况

```
{
    "code": "110010",
    "message": "应用请求频率超过限制",
    "value": "",
    "redirect": ""
}
```


### 全部&标签推送 <a name="all_tag_push_index"/>
#### 应用场景 <a name="yycj_3_index"/>

> 全部推送：音乐中心搞一个全网活动，需要对所有安装此应用的用户推送消息

> 标签推送：阅读咨询应用做新闻推送，指定不同标签的用户推送不同的内容，推送不同标签用
> 户感兴趣的内容。订阅了娱乐的推送娱乐新闻，订阅了美食的推送美食信息


#### 应用全部推送 <a name="pushToApp_index"/>

描述|内容
---|---
接口功能|全部用户推送
请求方法|Post
请求路径|/garcia/api/server/push/pushTask/pushToApp
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
appId|推送应用ID 必填
pushType|消息类型 0 通知栏 1透传 必填
sign|签名 必填
messageJson|Json格式，具体如下必填

> 通知栏类型（pushType=0）  


```
{
    "noticeBarInfo": {
        "noticeBarType": 通知栏样式(0, "标准")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32】
        "content": 推送内容, 【string 必填，字数限制1~100】
    },
    "noticeExpandInfo": {
        "noticeExpandType": 展开方式 (0, "标准"),(1, "文本")【int 非必填，值为0、1】
        "noticeExpandContent": 展开内容, 【string noticeExpandType为文本时，必填】
    },
    "clickTypeInfo": {
        "clickType": 点击动作 (0,"打开应用"),(1,"打开应用页面"),(2,"打开URI页面"),(3, "应用客户端自定义")【int 非必填,默认为0】
        "url": URI页面地址, 【string clickType为打开URI页面时，必填, 长度限制1000字节】
        "parameters":参数 【JSON格式】【非必填】 
        "activity":应用页面地址 【string clickType为打开应用页面时，必填, 长度限制1000字节】
        "customAttribute":应用客户端自定义【string clickType为应用客户端自定义时，必填， 输入长度为1000字节以内】
    },
    "pushTimeInfo": {
        "offLine": 是否进离线消息(0 否 1 是[validTime]) 【int 非必填，默认值为1】
        "validTime": 有效时长 (1 72 小时内的正整数) 【int offLine 值为1时，必填，默认24】
    "pushTimeType": 定时推送 (0, "即时"),(1, "定时")【必填，默认0】
        "startTime": 任务定时开始时间(yyyy-MM-dd HH:mm:ss) 【非必填pushTimeType为1必填】
    },
    "advanceInfo": {
        "suspend":是否通知栏悬浮窗显示 (1 显示  0 不显示) 【int 非必填，默认1】
        "clearNoticeBar":是否可清除通知栏 (1 可以  0 不可以) 【int 非必填，默认1】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【string 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【string 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【string 非必填，默认1】
        }
    }
}

```

> 透传类型（pushType=1） 


```
{
    "title": 推送标题, 【string 必填，字数显示1~32个字符】
    "content": 推送内容,  【string 必填，字数限制2000byte以内】
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【int 非必填，默认值为1】
        "validTime": 有效时长 (1- 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
        "pushTimeType": 定时推送 (0, "即时"),(1, "定时")【必填，默认0】
        "startTime": 任务定时开始时间(yyyy-MM-dd HH:mm:ss) 【非必填pushTimeType为1必填】
    },
    "advanceInfo": {
        "fixSpeed": 是否定速推送 0 否  1 是 (fixSpeedRate 定速速率) 【非必填】
        "fixSpeedRate": 定速速率 【fixSpeed为1时，必填】
    }
}

```

响应内容

> 成功情况：


```
{
    "code": "200",
    "message": "",
    "value": {
        "taskId": 20457 (任务Id)
        "pushType": 0 (推送类型 0通知栏  1 透传)
        "appId": 100999 (应用appId)
    },
    "redirect": ""
}

```

#### 应用标签推送 <a name="pushToTag_index"/>

描述|内容
---|---
接口功能|应用标签推送
请求方法|Post
请求路径|/garcia/api/server/push/pushTask/pushToTag
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
appId|推送应用ID 必填
pushType|消息类型 0 通知栏 1透传 必填
tagNames|推送标签 必填 多个通过英文逗号分割
scope|标签集合 必填 0 并集 1 交集
sign|签名 必填
messageJson|Json格式，具体如下必填

> 通知栏类型（pushType=0）  


```
{
    "noticeBarInfo": {
        "noticeBarType": 通知栏类型(0, "默认") 【必填，值为0】
        "title": 推送标题, 【必填，字数限制1~32字符】
        "content": 推送内容, 【必填，字数限制1~100个字符】
    },
    "noticeExpandInfo": {
        "noticeExpandType": 展开方式 (0, "禁用"),(1, "文本") 【必填，值为0或者1】
        "noticeExpandContent": 展开内容, 【noticeExpandType为文本时，必填】
    },
    "clickTypeInfo": {
        "clickType": 点击动作 (0,"打开应用"),(1,"打开应用页面"),(2,"打开URI页面"),(3, "应用客户端自定义")【int 非必填,默认为0】
        "url": URI页面地址, 【string clickType为打开URI页面时，必填, 长度限制1000字节】
        "parameters":参数 【JSON格式】【非必填】 
        "activity":应用页面地址 【string clickType为打开应用页面时，必填, 长度限制1000字节】
        "customAttribute":应用客户端自定义【string clickType为应用客户端自定义时，必填， 输入长度为1000字节以内】
    },
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【非必填】
        "validTime": 有效时长 (0- 72 小时内的正整数) 【必填，值的范围0--72】
        "pushTimeType": 定时推送 (0, "即时"),(1, "定时")【必填，默认0】
        "startTime": 任务定时开始时间(yyyy-MM-dd HH:mm:ss) 【非必填pushTimeType为1必填】
    },
    "advanceInfo": {
        "fixSpeed": 是否定速推送 0 否  1 是 (fixSpeedRate 定速速率) 【非必填，默认0】
        "fixSpeedRate": 定速速率 【fixSpeed为是时，必填】
        "suspend":是否通知栏悬浮窗显示  1 显示  0 不显示 【非必填，默认0】
        "clearNoticeBar":是否可清除通知栏  1 可以  0 不可以 【非必填，默认1】
        "notificationType": {
            "vibrate":  震动  0关闭  1 开启 ,  【非必填，默认1】
            "lights":   闪光  0关闭  1 开启, 【非必填，默认1】
            "sound":   声音  0关闭  1 开启 【非必填，默认1】
        }
    }
}


```

> 透传类型（pushType=1） 


```
{
    "title": 推送标题, 【string 必填，字数显示1~32个字符】
    "content": 推送内容,  【string 必填，字数限制2000byte以内】
    "pushTimeInfo": {
        "offLine": 是否进离线消息 0 否 1 是[validTime] 【int 非必填，默认值为1】
        "validTime": 有效时长 (1- 72 小时内的正整数) 【int offLine值为1时，必填，默认24】
        "pushTimeType": 定时推送 (0, "即时"),(1, "定时")【必填，默认0】
        "startTime": 任务定时开始时间(yyyy-MM-dd HH:mm:ss) 【非必填pushTimeType为1必填】
    },
    "advanceInfo": {
        "fixSpeed": 是否定速推送 0 否  1 是 (fixSpeedRate 定速速率) 【非必填】
        "fixSpeedRate": 定速速率 【fixSpeed为1时，必填】
    }
}


```

响应内容

> 成功情况：


```
{
    "code": "200",
    "message": "",
    "value": {
        "taskId": 20457, 任务Id
        "pushType": 0, 推送类型 0通知栏  1 透传
        "appId": 100999推送应用Id
    },
    "redirect": ""
}


```



#### 取消任务推送 <a name="cancelTask_index"/>


描述|内容
---|---
接口功能|取消任务推送（只针对全部用户推送待推送和推送中的任务取消）
请求方法|Post
请求路径|/garcia/api/server/push/pushTask/cancel
请求HOST|api-push.meizu.com
请求头|无
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下所示 

参数|描述
---|---
appId|推送应用ID 必填
pushType|消息类型 0 通知栏 1透传 必填
taskId|取消任务ID
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": {
        "result": true 成功
    }
}
```

> 失败情况：


```
{
    "code": "110032",
    "message": "非法的taskId",
    "redirect": "",
    "value": ""
}

{
    "code": "500",
    "message": "任务已取消[已完成]，无法取消",
    "redirect": "",
    "value": ""
}
```
