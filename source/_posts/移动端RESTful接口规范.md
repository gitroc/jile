---
title: 移动端RESTful接口规范
date: 2017-07-12 16:29:40
tags: 技术规范
---


1. ## 设计背景

* 此文为实践经验，并结合公司现状进行总结。
* 不要为了RESTful而RESTful。
* 能表达清楚的情况下，尽可能简单表述。
* 是的，本规范仅针对移动端设计。

---
2. ## 接口路径

- ### 设计原则
* URI 指向的是唯一的资源

>     http://~/$version/accounts/yanbo

* URI 指向的是唯一的集合列表

>     http://~/$version/trades/(list)

* URI 指向的是聚合资源

>     http://~/$version/users/123456/profiles

* URI 指向的是组合资源

>     http://~/$version/logic

- ### Http Methods

操作方法 | 说明 | 值 | 使用状态
---|---|---|---
GET | 获取，查找 | 1 | 未使用
POST | 新增，创建 | 2 | 使用
PUT | 更新 | 3 | 未使用
PATCH | 部分更新 | 4 | 未使用
DELETE | 删除 | 5 | 未使用
OPTIONS | 特定资源请求 | 6 | 未使用
TRACE | 测试或诊断 | 7 | 未使用
CONNECT | 管道方式 | 8 | 未使用

- ### URL组成
    >     http://{server}/{application}/{version}/{logic}?{body}

序号 | 名称 | 说明 
---|---|---
1 | server | 服务域名
2 | application | 应用名
3 | version | 接口版本号
4 | logic | 业务逻辑
5 | body | 请求数据

- ### URL定义限制
    - 不使用大写字母
    - 使用中线-代替下划线_
    - 参数列表应该被encode过

- ### 接口分类
    - Microservices

    Microservices是一个全新的概念，它主要的观点是将一个大型的服务系统分解成多个微型系统。每个微型系统都能独立工作，并且提供各种不同的服务。独立运行的特点使微型系统之间不会产生相互影响，其中的一个微型系统宕机并不会牵连到其他的微型系统。这种架构使分布式系统的节点数量大大提升。因为RESTful服务是无状态的，所以这种分解并不会带来状态共享的问题。

    - 路由规则(逻辑)

    当我们需要对不同属性的接口做路由规则的时候，按功能划分接口是一个很好的方案。例如：我们要对系统设置接口设置增加更严格的调用限制。

- ### 缓存
    网络接口相对于堆栈接口来说数据传输极其不稳定，尽可能地减少数据传输不仅能控制这种风险还能减少流量。使用缓存还能有效地提高后台的吞吐量。
    
    后台在响应请求时使用响应头E-Tag或Last-Modified来标记数据的版本，前台在发送请求时将数据版本通过请求头If-Match帮助后台判断缓存的使用。
    
    鉴于项目实际情况，暂时不使用缓存。
    
Http Request Header |  描述
---|---
If-Match: 2390239059405940 | 只有请求内容与实体相匹配才有效

Http Response Headers |  描述
---|---
E-Tag: 2390239059405940 | 只有请求内容与实体相匹配才有效
Last-Modified: 2014-04-05T14:30Z | 请求资源的最后修改时间

- ### Bookmarker
    在实际的环境中，有大量的查询需求是相同的。将这些搜索需求标签化能降低使用难度也可以达到重用的目的。
    
    鉴于项目实际情况，暂时不使用Bookmarker。

- ### HATEOAS
    HATEOAS通过Web Linking的方式来描述程序的状态信息，鉴于项目实际情况，暂时不使用HATEOAS。

- ### 分页
    >     {
    >       "header":
    >       {
    >           "version": "1.0.1", //应用版本号，字符串
    >           "build": "101", //第几次编译，整形字符串
    >           "app_type": "1", //应用类型，整形字符串 1 ios用户端， 2 android用户端， 3 ios商户， 4 android商户
    >           "timestamp": "1494835866", // 请求时间戳
    >           "uuid": "29F5CA2A-D5CB-45A3-BE0A-D351DD02171E", //通用唯一识别码
    >           "token": "0f0600679379b1523e196e87615015f7972eda5c3c3e086b", //用户唯一标识，字符串，免密登录，唯一登录
    >           "os_info": "ios_ML7N2CH/A_10.3.1(14E304)" //操作系统信息，厂商、型号、版本
    >       },
    >       "body":
    >       {
    >           "user_id": "178772",
    >           "page_index": "1",
    >           "page_size": "10",
    >           "list": 
    >           {
    >               "data": []
    >           }
    >           ...         //其他接口请求数据
    >       }
    >     }

---
3. ## 安全设计
- ### 调用限制
    为保证服务的可用性应对服务进行调用过载保护，鉴于项目实际情况，暂不在Http Header中添加任何访问限制。

Http Response Headers |  描述
---|---
X-RateLimit-Limit: 3000 | 调用量的最大限制
X-RateLimit-Reset: 1403162176516 | 调用限制重置时间
X-RateLimit-Remaining: 299 | 剩余的调用量
    
- ### 安全验证
    RESTful服务使用Oauth2的方式进行调用授权，使用http请求头Authorization设置授权码; 必须使用User-Agent设置客户端信息, 无User-Agent请求头的请求应该被拒绝访问。鉴于项目实际情况，暂时不使用Oauth2的方式进行调用授权。

Http Request Header |  描述
---|---
User-Agent: Data-Server-Client | 客户端信息
Authorzation: Bearer 383w9JKJLJFw4ewpie2wefmjdlJLDJF | 授权码
- ### 简单校验
    - 采用服务端生成token的办法进行简单数据请求校验。
    - 采用服务端对不同设备生成唯一token的办法保证唯一登录。

- ### UUID获取
    - Android获取办法
    >     public String getUUID(){
    >         return UUID.randomUUID().toString();
    >     }
    
    - iOS获取办法
    >     +(NSString *)getUUID
    >     {
    >         NSString * strUUID = (NSString *)[KeyChainStore load:@"com.company.app.usernamepassword"];
    >         
    >         //首次执行该方法时，uuid为空
    >         if ([strUUID isEqualToString:@""] || !strUUID)
    >         {
    >             //生成一个uuid的方法
    >             CFUUIDRef uuidRef = CFUUIDCreate(kCFAllocatorDefault);
    >             
    >             strUUID = (NSString *)CFBridgingRelease(CFUUIDCreateString (kCFAllocatorDefault,uuidRef));
    >             
    >             //将该uuid保存到keychain
    >             [KeyChainStore save:KEY_USERNAME_PASSWORD data:strUUID];
    >             
    >         }
    >         return strUUID;
    >     }

---
4. ## 数据设计
- ### 数据格式
    JSON 是一种可以跨平台高扩展的轻量级的数据交换格式。易于人阅读和编写，同时也易于机器解析和生成。
- ### 数据类型

数据类型 |  描述
---|---
Number | 整数或者浮点数
String | 字符串
Boolean | true 或 false
Array  | 数组包含在方括号[]中
Object | 对象包含在大括号{}中
Null | 空类型（没有Value，就不要传Key）

- ### 属性定义
    - 不能使用大写
    - 使用下划线_命名(连接两个单词)
    - 属性和字符串值必须使用双引号 ""。

---
5. ## 状态定义

状态值 |  错误描述
---|---
0 | 成功
100 | 移动端错误
101 | ...
102 | ...
... | ...
199 | 移动端错误
200 | 服务端错误
201 | ...
202 | ...
... | ...
299 | ...

---
6. ## 附加选项
    - 格式化(Pettyprint)
    - gzip压缩

---
7. ## 应用举例
    - ### 数据请求
        >     {
        >       "header":
        >       {
        >           "version": "1.0.1", //应用版本号，字符串
        >           "build": "101", //第几次编译，整形字符串
        >           "app_type": "1", //应用类型，整形字符串 1 ios用户端， 2 android用户端， 3 ios商户， 4 android商户
        >           "timestamp": "1494835866", // 请求时间戳
        >           "uuid": "29F5CA2A-D5CB-45A3-BE0A-D351DD02171E",  //通用唯一识别码
        >           "token": "0f0600679379b1523e196e87615015f7972eda5c3c3e086b", //用户唯一标识，字符串，免密登录，唯一登录
        >           "os_info": "ios_ML7N2CH/A_10.3.1(14E304)" //操作系统信息，厂商、型号、版本
        >       },
        >       "body":
        >       {
        >           "key1": "value1",
        >           "key2": "value2",
        >           "key3": "value3",
        >           "list": 
        >           {
        >               "data": [...]
        >           }
        >           ...         //其他接口请求数据
        >       }
        >     }

    - ### 数据返回
        **成功获取服务端数据**:

        >     {
        >       "code": "0",
        >       "message": "succeed message",
        >       "header":
        >       {
        >           "timestamp": "1494835866", // 响应时间戳
        >           "token": "0f0600679379b1523e196e87615015f7972eda5c3c3e086b", //用户唯一标识，字符串，免密登录，唯一登录
        >       },
        >       "body":
        >       {
        >           "key1": "value1",
        >           "key2": "value2",
        >           "key3": "value3",
        >           "list": 
        >           {
        >               "size": "28",
        >               "data": []
        >           }
        >           ...         //其他接口返回数据
        >       }
        >     }
        
        **获取服务端数据失败**:
    
        >     {
        >       "code": "0",
        >       "message": "fail message",
        >       "header":
        >       {
        >           "timestamp": "1494835866", // 响应时间戳
        >           "token": "0f0600679379b1523e196e87615015f7972eda5c3c3e086b", //用户唯一标识，字符串，免密登录，唯一登录
        >       },
        >       "body":
        >       {
        >           "doc": "https://developer.github.com/v3/gists/" //错误详细文档链接，可不返回
        >       }
        >     }
        
    - ### 登录注册