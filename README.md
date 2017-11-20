# 透传限制说明
  * 为优化flyme系统整体功耗，推送平台决定本周五 **(6月16号)** 起限制透传消息推送的使用，不排除关闭透传推送类型。使用透传推送的业务请尽快切换到通知栏推送，以避免消息推送失败。新接入的应用请使用通知栏推送
 
  * 受影响的接口及功能：
   1. pushId推送接口（透传消息）
   2. 别名推送接口（透传消息)
   3. 获取taskId的透传推送(getTaskId)
   4. 应用全部推送（透传消息)
   5. 应用标签推送（透传消息)
   6. 在平台上进行的透传推送

# 魅族开放平台PUSH系统HTTP接口文档

* [更新日志](https://github.com/MEIZUPUSH/PushAPI/releases)


# 目录 <a name="index"/>
* [一.API接口规范](#api_standard_index)
    * [接口响应规范](#api_resp_index)
    * [接口签名规范](#api_sign_index)
    * [接口请求示例](#api_demo_index)
* [二.API说明](#api_common_index) 
    * [前言](#preface_index)      
    * [非任务推送](#untask_push_index)  
        * [应用场景](#yycj_1_index)
        * [pushId推送接口（透传消息）](#UnVarnishedMessage_push_index)
        * [pushId推送接口（通知栏消息）](#VarnishedMessage_push_index)
        * [别名推送接口（透传消息）](#UnVarnishedMessage_alias_push_index)
        * [别名推送接口（通知栏消息）](#VarnishedMessage_alias_push_index)
    * [任务推送](#task_push_index)  
        * [非调度推送](#pushId_index)
            * [应用场景](#yycj_2_index)
            * [获取推送taskId](#getTaskId_index)
            * [pushId推送接口（透传消息）](#UnVarnishedMessage_task_push_index)
            * [pushId推送接口（通知栏消息）](#VarnishedMessage_task_push_index)
            * [别名推送接口（透传消息）](#UnVarnishedMessage_alias_task_push_index)
            * [别名推送接口（通知栏消息）](#VarnishedMessage_alias_task_push_index)
        * [调度推送](#all_tag_push_index)
            * [应用场景](#yycj_3_index)
            * [应用全部推送](#pushToApp_index)
            * [应用标签推送](#pushToTag_index)
            * [取消任务推送](#cancelTask_index)
    * [推送统计](#task_statistics_index) 
        * [获取任务推送统计](#getTaskStatistics_index)
        * [获取应用推送统计](#dailyPushStatics_index)
    * [高级功能](#super_index)
        * [消息送达与回执](#callback_index)
    * [订阅服务](#sub_index) 
        * [获取订阅开关状态](#sub_index_1)
        * [修改订阅开关状态](#sub_index_2)
        * [修改所有开关状态](#sub_index_3)
        * [别名订阅](#sub_index_4)
        * [取消别名订阅](#sub_index_5)
        * [获取订阅别名](#sub_index_6)
        * [标签订阅](#sub_index_7)
        * [取消标签订阅](#sub_index_8)
        * [获取订阅标签](#sub_index_9)
        * [取消订阅所有标签](#sub_index_10)
# API接口规范 <a name="api_standard_index"/>
## 接口响应规范 <a name="api_resp_index"/>
> HTTP接口遵循魅族API协议规范。返回数据格式统一如下：


```
{
    “code”:”“, //必选,返回码
    “message”:”“, //可选，返回消息，网页端接口出现错误时使用此消息展示给用户，手机端可忽略此消息，甚至服务端不传输此消息
    “value”:”“,// 必选，返回结果
    “redirect”:”“ //可选, returnCode=300 重定向时，使用此URL重新请求
    “msgId”: ”“//可选，消息推送msgId
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
110004|参数不能为空
110009|应用被加入黑名单
110010|应用推送速率过快
110053|透传超过限制


## 接口签名规范 <a name="api_sign_index"/>
> 请求参数分别是“k1”、“k2”、“k3”，它们的值分别是“v1”、“v2”、“v3”，计算方法如下所示：
> 
> 1. 将参数以其参数名的字典序升序进行排序,如 对 k1 k2 k3 排序
> 1. 遍历排序后的字典，将所有参数按"key=value"格式拼接在一起，如“k1=v1k2=v2k3=v3”
> 1. 在拼接好的字符串末尾追加上应用的Secret Key
> 
> 上述字符串的MD5值即为签名的值。（32位小写）
> 
> 将签名值放在请求的参数中例如sign=MD5_SIGN
> 
> 服务端SDK调用API的应用的私钥Secret Key为 appSecret

```java
   /**
     * @param paramMap 请求参数
     * @param secret   密钥
     * @return md5摘要
     */
    public static String getSignature(Map<String, String> paramMap, String secret) {
        // 先将参数以其参数名的字典序升序进行排序
        Map<String, String> sortedParams = new TreeMap<String, String>(paramMap);
        Set<Entry<String, String>> entrys = sortedParams.entrySet();

        // 遍历排序后的字典，将所有参数按"key=value"格式拼接在一起
        StringBuilder basestring = new StringBuilder();
        for (Entry<String, String> param : entrys) {
            basestring.append(param.getKey()).append("=").append(param.getValue());
        }
        basestring.append(secret);

        logger.debug("basestring is:{}", new Object[]{basestring.toString()});

        // 使用MD5对待签名串求签
        return MD5Util.MD5Encode(basestring.toString(),"UTF-8");
    }
    
   //示例，注意是针对接口中所有参数做签名，并且是原始字符串（非urlencode）
    public static void main(String[] args) {
        //本示例为三个参数 appId、pushIds、messageJson
        Map<String, String> paramMap = new HashMap<String, String>();
        paramMap.put("appId", "10000");
        paramMap.put("pushIds", "RA50c6348036344485d01776773577c64740465480a6b");
        paramMap.put("messageJson", "{\"title\":\"title\",\"content\":\"content\",\"pushTimeInfo\":{\"offLine\":1,\"validTime\":24}}");
        String sign = SignUtils.getSignature(paramMap, "<APP_SECRET>");
    }
    //MD5原始字符串为
    appId=10000messageJson={"title": "title","content": "content","pushTimeInfo": {"offLine": 1,"validTime": 24}}pushIds=RA50c6348036344485d01776773577c64740465480a6b<APP_SECRET>
    //MD5摘要 sign为
    ac076ff25d9900015a681cb5172aa53b
```
## 接口请求示例 <a name="api_demo_index"/>

```
POST http://server-api-push.meizu.com/garcia/api/server/push/unvarnished/pushByAlias HTTP/1.1
Host: server-api-push.meizu.com
Connection: keep-alive
Content-Length: 226
Cache-Control: no-cache
Content-Type: application/x-www-form-urlencoded
Accept: */*
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8

alias=xxx&appId=xxx&messageJson=%7B%22title%22%3A%22title%22%2C%22content%22%3A%22hello+test%22%2C%22pushTimeInfo%22%3A%7B%22offLine%22%3A1%2C%22validTime%22%3A24%7D%7D&sign=a68b75e5d5b30e35536f130cf1cae14a


HTTP/1.1 200 OK
Server: nginx
Date: Wed, 28 Dec 2016 03:34:53 GMT
Content-Type: application/json; charset=UTF-8
Content-Length: 87
Connection: keep-alive
Content-Language: zh-CN
Set-Cookie: JSESSIONID=1wl3nhcfqroiicj6pvxwdvjx6;Path=/
Expires: Thu, 01 Jan 1970 00:00:00 GMT


{"code":"200","message":"","value":{"110005":["xxxxxx"]},"redirect":""}
```



# API说明 <a name="api_common_index"/>

## 前言 <a name="preface_index"/>

> 消息推送结果接口响应部分value是map集合的json格式且只返回推送非法的pushId，合法的pushId不予返回，一般情况下，pushId未注册则视为非法。

map部分code定义

code|value
---|---
201|没有权限，服务器主动拒绝
501|推送消息失败（db_error）
513|推送消息失败
519|推送消息失败服务过载
520|消息折叠（1分钟内同一设备同一应用消息收到多次，默认5次）
110002|pushId失效(pushId未订阅或者消息开关关闭)
110003|pushId非法
110005|alias失效(alias未订阅或者消息开关关闭)

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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushIds|推送设备，一批最多不能超过1000个 多个英文逗号分割必填
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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
pushIds|推送设备，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "noticeBarInfo": {
        "noticeBarType": 通知栏样式(0, "标准"),(2, "安卓原生")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32字符】
        "content": 推送内容, 【string 必填，字数限制1~100字符】
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
        "validTime": 有效时长 (1到72 小时内的正整数) 【int offLine值为1时，必填，默认24】
    },
    "advanceInfo": {
        "suspend":是否通知栏悬浮窗显示 (1 显示  0 不显示) 【int 非必填，默认1】
        "clearNoticeBar":是否可清除通知栏 (1 可以  0 不可以) 【int 非必填，默认1】
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        },
        "notifyKey": "" // 非必填 默认空 分组合并推送的key,凡是带有此key的通知栏消息只会显示最后到达的一条。由数字([0-9]), 大小写字母([a-zA-Z]), 下划线(_)和中划线(-)组成, 长度不大于8个字符
    },
    //需要启用回执，设置extra，无需回执则可不设置
    "extra":{
       "callback":"http://flyme.callback",//String(必填字段), 第三方接收回执的Http接口, 最大长度128字节
       "callback.param":"param",//String(可选字段), 第三方自定义回执参数, 最大长度64字节
       "callback.type":"3 //int(可选字段), 回执类型(1-送达回执, 2-点击回执, 3-送达与点击回执), 默认3
   }
}

```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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

### 别名推送接口（透传消息） <a name="UnVarnishedMessage_alias_push_index"/>

描述|内容
---|---
接口功能|根据别名推送
请求方法|Post
请求路径|/garcia/api/server/push/unvarnished/pushByAlias
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
alias|推送别名，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "title": 推送标题, 【string 非必填，字数显示1~32个字符】
    "content": 推送内容,  【string 必填，字数限制2000字节以内】
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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
    "value": {}
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "110005": [
            "alias1",
            "alias2"
        ]
    },
   "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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


### 别名推送接口（通知栏消息） <a name="VarnishedMessage_alias_push_index"/>

描述|内容
---|---
接口功能|根据别名推送
请求方法|Post
请求路径|/garcia/api/server/push/varnished/pushByAlias
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
alias|推送别名，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填
messageJson|Json格式，具体如下必填

```
{
    "noticeBarInfo": {
        "noticeBarType": 通知栏样式(0, "标准"),(2, "安卓原生")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32字符】
        "content": 推送内容, 【string 必填，字数限制1~100字符】
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
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        },
        "notifyKey": "" // 非必填 默认空 分组合并推送的key,凡是带有此key的通知栏消息只会显示最后到达的一条。由数字([0-9]), 大小写字母([a-zA-Z]), 下划线(_)和中划线(-)组成, 长度不大于8个字符
    },
    //需要启用回执，设置extra，无需回执则可不设置
    "extra":{
       "callback":"http://flyme.callback",//String(必填字段), 第三方接收回执的Http接口, 最大长度128字节
       "callback.param":"param",//String(可选字段), 第三方自定义回执参数, 最大长度64字节
       "callback.type":"3 //int(可选字段), 回执类型(1-送达回执, 2-点击回执, 3-送达与点击回执), 默认3
   }
}

```

响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
    "value": {}
}
```

> 失败情况


```
{
    "code": "200",
    "message": "",
    "value": {
        "110005": [
            "alias1",
            "alisa2"
        ]
    },
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

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
        "noticeBarType": 通知栏样式(0, "标准"),(2, "安卓原生")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32字符】
        "content": 推送内容, 【string 必填，字数限制1~100字符】
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
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【int 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【int 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【int 非必填，默认1】
        },
        "notifyKey": "" // 非必填 默认空 分组合并推送的key,凡是带有此key的通知栏消息只会显示最后到达的一条。由数字([0-9]), 大小写字母([a-zA-Z]), 下划线(_)和中划线(-)组成, 长度不大于8个字符
    }
}
```

> 透传类型（pushType=1） 


```
{
    "title": 推送标题, 【string 必填，字数显示1~32个字符】
    "content": 推送内容,  【string 必填，字数限制2000字节以内】
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
taskId|推送任务ID 必填 
appId|推送应用ID 必填
pushIds|推送设备，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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

#### 别名推送接口（透传消息） <a name="UnVarnishedMessage_alias_task_push_index"/>

描述|内容
---|---
接口功能|根据别名推送
请求方法|Post
请求路径|/garcia/api/server/push/task/unvarnished/pushByAlias
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
taskId|推送任务ID 必填 
appId|推送应用ID 必填
alias|推送别名，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
        "110005": [
            "alias1",
            "alias2"
        ]
    },
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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

#### 别名推送接口（通知栏消息） <a name="VarnishedMessage_alias_task_push_index"/>

描述|内容
---|---
接口功能|根据别名推送
请求方法|Post
请求路径|/garcia/api/server/push/task/varnished/pushByAlias
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
taskId|推送任务ID 必填 
appId|推送应用ID 必填
alias|推送别名，一批最多不能超过1000个 多个英文逗号分割必填
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
        "110005": [
            "alias1",
            "alias2"
        ]
    },
    "msgId": "c2ee5c3bf00448cfbceb7fdf68c3c8eb"
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

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
        "noticeBarType": 通知栏样式(0, "标准"),(2, "安卓原生")【int 非必填，值为0】
        "title": 推送标题, 【string 必填，字数限制1~32字符】
        "content": 推送内容, 【string 必填，字数限制1~100字符】
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
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】   
        "notificationType": {
            "vibrate":  震动 (0关闭  1 开启) ,  【string 非必填，默认1】
            "lights":   闪光 (0关闭  1 开启), 【string 非必填，默认1】
            "sound":   声音 (0关闭  1 开启), 【string 非必填，默认1】
        },
        "notifyKey": "" // 非必填 默认空 分组合并推送的key,凡是带有此key的通知栏消息只会显示最后到达的一条。由数字([0-9]), 大小写字母([a-zA-Z]), 下划线(_)和中划线(-)组成, 长度不大于8个字符
    }
}

```

> 透传类型（pushType=1） 


```
{
    "title": 推送标题, 【string 必填，字数显示1~32个字符】
    "content": 推送内容,  【string 必填，字数限制2000字节以内】
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

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
       "noticeBarType": 通知栏样式(0, "标准"),(2, "安卓原生")【int 非必填，值为0】
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
        "fixDisplay":是否定时展示 (1 是  0 否) 【int 非必填，默认0】
        "fixStartDisplayTime": 定时展示开始时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】
        "fixEndDisplayTime ": 定时展示结束时间(yyyy-MM-dd HH:mm:ss) 【str 非必填】   
        "notificationType": {
            "vibrate":  震动  0关闭  1 开启 ,  【非必填，默认1】
            "lights":   闪光  0关闭  1 开启, 【非必填，默认1】
            "sound":   声音  0关闭  1 开启 【非必填，默认1】
        },
        "notifyKey": "" // 非必填 默认空 分组合并推送的key,凡是带有此key的通知栏消息只会显示最后到达的一条。由数字([0-9]), 大小写字母([a-zA-Z]), 下划线(_)和中划线(-)组成, 长度不大于8个字符
    }
}


```

> 透传类型（pushType=1） 


```
{
    "title": 推送标题, 【string 必填，字数显示1~32个字符】
    "content": 推送内容,  【string 必填，字数限制2000字节以内】
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
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

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
## 推送统计 <a name="task_statistics_index"/>
### 获取任务推送统计 <a name="getTaskStatistics_index"/>


描述|内容
---|---
接口功能|获取任务推送统
请求方法|Get
请求路径|/garcia/api/server/push/statistics/getTaskStatistics
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
taskId|任务ID
sign|签名 必填


响应内容

> 成功情况：

```
 {
    "code": "200",
    "message": "",
    "redirect": "",
    "value": {
        "taskId": 任务Id,
        "targetNo": 目标数,
        "validNo": 有效数,
        "pushedNo": 推送数,
        "acceptNo ": 接收数,
        "displayNo": 展示数,
        "clickNo": 点击数
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
```
### 获取应用推送统计 <a name="dailyPushStatics_index"/>


描述|内容
---|---
接口功能|获取应用推送统计（最长跨度30天）
请求方法|Get
请求路径|/garcia/api/server/push/statistics/dailyPushStatics
请求HOST|server-api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的 

参数|描述
---|---
appId|推送应用ID 必填
startTime|开始日期, 如20140214 必填
endTime|结束日期, 如20140218 必填
sign|签名  必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
    "value": [
        {
            "acceptNo": 609,//接收数
            "clickNo": 30,//点击数
            "date": "2017-05-03",//推送日期
            "displayNo": 241,//展示数
            "pushedNo": 691287,//推送总数
            "targetNo": 1741833,//推送目标数
            "validNo": 636257//推送有效数
        },
        {
            "acceptNo": 228,
            "clickNo": 31,
            "date": "2017-05-02",
            "displayNo": 39,
            "pushedNo": 228463,
            "targetNo": 879102,
            "validNo": 210962
        }
    ]
}

```

> 失败情况：


```
{
    "code": "500",
    "message": "结束时间不能早于开始时间",
    "redirect": "",
    "value": ""
}

{
    "code": "500",
    "message": "开始时间和结束时间不能相差30天以上",
    "redirect": "",
    "value": ""
}
```

## 高级功能 <a name="super_index"/>
### 消息送达与回执  <a name="callback_index"/>

- 支持回执接口

[pushId推送接口（通知栏消息）](#VarnishedMessage_push_index)

[别名推送接口（通知栏消息）](#VarnishedMessage_alias_push_index)


- 开发者通过设置通知栏消息json格式中增加extra来指定消息的送达和点击回执规则
- 回执地址请登录推送平台【配置管理】->【回执管理】注册回执地址

```
 "extra":{
     "callback":"http://flyme.callback",//String(必填字段), 第三方接收回执的Http接口, 最大长度128字节
     "callback.param":"param",//String(可选字段), 第三方自定义回执参数, 最大长度64字节
     "callback.type":"3 //int(可选字段), 回执类型(1-送达回执, 2-点击回执, 3-送达与点击回执), 默认3
 }
```
```
魅族推送服务器每隔1s将已送达或已点击的消息ID和对应设备的pushId或alias通过调用开发者http接口传给开发者(每次调用后, 魅族推送服务器会清空这些数据,下次传给业务方将是新一拨数据)

注: 

消息的送达回执只支持向pushId或alias发送的消息

单个应用注册不同回执地址累计上限不能超过100个
```

- 回执响应内容

key|value
---|---
cb|回执明细内容 如下所述（Json数据）
access_token|回执接口访问令牌

```
回执明细格式说明: 外层key代表相应的消息id和回执类型（msgId-type）, value是一个JSONObject, 包含了下面的参数值

param： 业务上传的自定义参数值
type： callback类型
targets： 一批alias或者pushId集合
```

```
{
    "msgId2-1": {
        "param": "param2",
        "type": 2,
        "targets": [
            "pushId3",
            "pushId2",
            "pushId1"
        ]
    },
    "msgId1-2": {
        "param": "param1",
        "type": 1,
        "targets": [
            "alias2",
            "alias",
            "alias1"
        ]
    }
}
```
## 订阅服务 <a name="sub_index"/>
### 获取订阅开关状态  <a name="sub_index_1"/>


描述|内容
---|---
接口功能|获取订阅开关状态
请求方法|Get
请求路径|/garcia/api/server/message/getRegisterSwitch
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "value": {
 "barTypeSwitch": int 类型 通知栏消息开关 0 关 1 开,
 "directTypeSwitch":int 类型 透传消息开关 0 关 1 开,
 "pushId": string 类型 注册 push 后唯一标识
 }
}

```

### 修改订阅开关状态 <a name="sub_index_2"/>


描述|内容
---|---
接口功能|修改订阅开关状态
请求方法|Post
请求路径|/garcia/api/server/message/changeRegisterSwitch
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
msgType|int 类型 (0,"状态栏推送"),(1,"透传消息"); 必填
subSwitch|开关状态 0 关 1 开 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "barTypeSwitch": int 类型 通知栏消息开关 0 关 1 开,
 "directTypeSwitch":int 类型 透传消息开关 0 关 1 开,
 "pushId": string 类型 注册 push 后唯一标识
 }
}

```

### 修改所有开关状态  <a name="sub_index_3"/>


描述|内容
---|---
接口功能|修改所有开关状态
请求方法|Post
请求路径|/garcia/api/server/message/changeAllSwitch
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
subSwitch|开关状态 0 关 1 开 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "barTypeSwitch": int 类型 通知栏消息开关 0 关 1 开,
 "directTypeSwitch":int 类型 透传消息开关 0 关 1 开,
 "pushId": string 类型 注册 push 后唯一标识
 }
}

```

### 别名订阅  <a name="sub_index_4"/>


描述|内容
---|---
接口功能|别名订阅
请求方法|Post
请求路径|/garcia/api/server/message/subscribeAlias
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
alias|订阅别名（60字符限制）必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "pushId": string 类型 注册 push 后唯一标识
 “alias”: string 类型
 }
}

```

### 取消别名订阅  <a name="sub_index_5"/>


描述|内容
---|---
接口功能|取消别名订阅
请求方法|Post
请求路径|/garcia/api/server/message/unSubscribeAlias
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "pushId": string 类型 注册 push 后唯一标识
 “alias”: string 类型
 }
}

```

### 获取订阅别名  <a name="sub_index_6"/>


描述|内容
---|---
接口功能|获取订阅别名
请求方法|Get
请求路径|/garcia/api/server/message/getSubAlias
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "pushId": string 类型 注册 push 后唯一标识
 “alias”: string 类型
 }
}

```

### 标签订阅  <a name="sub_index_7"/>


描述|内容
---|---
接口功能|标签订阅
请求方法|Post
请求路径|/garcia/api/server/message/subscribeTags
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
tags|多个标签用英文逗号分割，单个标签20个字符限制，100个标签数量限制 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "pushId": "83a9d4ad369d46eabba3e280366474eb",
 "tags": [
 {
 "tagId": 1,
 "tagName": "体育"
 },
 {
 "tagId": 2,
 "tagName": "科技"
 }
 ]
 }
}

```

### 取消标签订阅  <a name="sub_index_8"/>


描述|内容
---|---
接口功能|取消标签订阅
请求方法|Post
请求路径|/garcia/api/server/message/unSubscribeTags
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
tags|多个标签用英文逗号分割，单个标签20个字符限制，100个标签数量限制 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "pushId": "83a9d4ad369d46eabba3e280366474eb",
 "tags": [
 {
 "tagId": 1,
 "tagName": "体育"
 },
 {
 "tagId": 2,
 "tagName": "科技"
 }
 ]
 }
}

```

### 获取订阅标签  <a name="sub_index_9"/>


描述|内容
---|---
接口功能|获取订阅标签
请求方法|Get
请求路径|/garcia/api/server/message/getSubTags
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|如下

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
sign|签名 必填


响应内容

> 成功情况：返回取消后订阅的标签

```
{
 "code": "200",
 "message": "",
 "redirect": "",
 "value": {
 "pushId": "83a9d4ad369d46eabba3e280366474eb",
 "tags": [
 {
 "tagId": 1,
 "tagName": "体育"
 },
 {
 "tagId": 2,
 "tagName": "科技"
 }
 ]
 }
}

```

### 取消订阅所有标签  <a name="sub_index_10"/>


描述|内容
---|---
接口功能|取消订阅所有标签
请求方法|Post
请求路径|/garcia/api/server/message/unSubAllTags
请求HOST|api-push.meizu.com
请求头|Content-Type:application/x-www-form-urlencoded;charset=UTF-8
备注|签名参数 sign=MD5_SIGN
请求内容|无
响应码|200
响应头|无
请求参数|按POST提交表单的标准，你的任何值字符串是需要 urlencode 编码的

参数|描述
---|---
appId|推送应用ID 必填
pushId|订阅pushID 必填
sign|签名 必填


响应内容

> 成功情况：

```
{
    "code": "200",
    "message": "",
    "redirect": "",
     "value": true 成功
}

```
