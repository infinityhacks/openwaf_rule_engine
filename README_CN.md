Name
====

openwaf_rule_engine是[OPENWAF](https://github.com/titansec/openwaf)的规则引擎

Table of Contents
=================

* [Name](#name)
* [Version](#version)
* [Synopsis](#synopsis)
* [Description](#description)
* [Community](#community)
* [Bugs and Patches](#bugs-and-patches)
* [Changes](#changes)
* [Modules Configuration Directives](#modules-configuration-directives)
* [Rule Directives](#rule-directives)
* [Variables](#variables)
* [Transformation Functions](#transformation-functions)
* [Operators](#operators)
* [Others](#others)

Version
=======

This document describes OpenWAF Rule Engine v0.0.1.161026_beta released on 26 Oct 2016.

Synopsis
========


[Back to TOC](#table-of-contents)

Description
===========

规则引擎的启发来自[modsecurity](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual)及[freewaf(lua-resty-waf)](https://github.com/p0pr0ck5/lua-resty-waf)，将ModSecurity的规则机制用lua实现。基于规则引擎可以进行协议规范，自动工具，注入攻击，跨站攻击，信息泄露，异常请求等安全防护，支持动态添加规则，及时修补漏洞。

[Back to TOC](#table-of-contents)

Community
=========

English Mailing List
--------------------

The [OpenWAF-en](https://groups.google.com/group/openwaf-en) mailing list is for English speakers.

Chinese Mailing List
--------------------

The [OpenWAF-cn](https://groups.google.com/group/openwaf-cn) mailing list is for Chinese speakers.

Personal QQ Mail
----------------

290557551@qq.com

[Back to TOC](#table-of-contents)

Bugs and Patches
================

Please submit bug reports, wishlists, or patches by

1. creating a ticket on the [GitHub Issue Tracker](https://github.com/titansec/OpenWAF/issues),
1. or posting to the [TWAF community](#community).

[Back to TOC](#table-of-contents)

Changes
=======

[Back to TOC](#table-of-contents)


Modules Configuration Directives
================================
```txt
    "twaf_secrules":{
        "state": true,                                              -- 总开关
        "reqbody_state": true,                                      -- 请求体检测开关
        "header_filter_state": true,                                -- 响应头检测开关
        "body_filter_state": true,                                  -- 响应体检测开关
        "reqbody_limit":134217728,                                  -- 请求体检测阈值，大于阈值不检测
        "respbody_limit":524288,                                    -- 响应体检测阈值，大于阈值不检测
        "pre_path": "/opt/OpenWAF/",                                -- OpenWAF安装路径
        "path": "lib/twaf/inc/knowledge_db/twrules",                -- 特征规则库在OpenWAF中的路径
        "user_defined_rules":[                                      -- 用户自定义规则，数组
        ],
        "rules_id":{                                                -- 特征排除
            "111112": [{"REMOTE_HOST":"a.com", "URI":"^/ab"}]       -- 匹配中数组中信息则对应规则失效，数组中key为变量名称，值支持正则
            "111113": []                                            -- 特征未被排除
            "111114": [{}]                                          -- 特征被无条件排除
        }
    }
```

###state
**syntax:** *"state": true|false*

**default:** *true*

**context:** *twaf_secrules*

规则引擎总开关

###reqbody_state
**syntax:** *"reqbody_state"" true|false*

**default:** *true*

**context:** *twaf_secrules*

请求体检测开关

###header_filter_state
**syntax:** *"header_filter_state": true|false*

**default:** *true*

**context:** *twaf_secrules*

响应头检测开关

###body_filter_state
**syntax:** *"body_filter_state": true|false*

**default:** *false*

**context:** *twaf_secrules*

响应体检测开关，默认关闭，若开启需添加第三方模块[ngx_http_twaf_header_sent_filter_module暂未开源]

###reqbody_limit
**syntax:** *"reqbody_limit": number*

**default:** *134217728*

**context:** *twaf_secrules*

请求体检测大小上限，默认134217728B(128MB)，若请求体超过设置上限，则不检测

PS：reqbody_limit值要小于nginx中client_body_buffer_size的值才会生效

###respbody_limit
**syntax:** *"respbody_limit": number*

**default:** *134217728*

**context:** *twaf_secrules*

响应体检测大小上限，默认134217728B(128MB)，若响应体大小超过设置上限，则不检测

###pre_path
**syntax:** *"pre_path" string*

**default:** */opt/OpenWAF/*

**context:** *twaf_secrules*

OpenWAF的安装路径

###path
**syntax:** *"path": string*

**default:** *lib/twaf/inc/knowledge_db/twrules*

**context:** *twaf_secrules*

特征规则库在OpenWAF中的路径

###user_defined_rules
**syntax:** *"user_defined_rules": table*

**default:** *none*

**context:** *twaf_secrules*

策略下的用户自定义特征规则

系统特征规则适用于所有的策略，在引擎启动时通过加载特征库或通过API加载系统特征规则，系统特征规则一般不会动态更改

用户自定义特征在策略下生效，一般用于变动较大的特征规则，如：时域控制，修改响应头等临时性规则

```json
"user_defined_rules":[
    {
        "id": "1000001",
        "release_version": "858",
        "charactor_version": "001",
        "disable": false,
        "opts": {
            "nolog": false
        },
        "phase": "access",
        "action": "deny",
        "meta": 403,
        "severity": "high",
        "rule_name": "relative time",
        "desc": "周一至周五的8点至18点，禁止访问/test目录",
        "match": [{
            "vars": [{
                "var": "URI"
            }],
            "operator": "begins_with",
            "pattern": "/test"
        },
        {
            "vars": [{
                "var": "TIME_WDAY"
            }],
            "operator": "equal",
            "pattern": ["1", "2", "3", "4", "5"]
        },
        {
            "vars": [{
                "var": "TIME"
            }],
            "operator": "str_range",
            "pattern": ["08:00:00-18:00:00"]
        }]
    },
    {
        "id": "1000002",
        "release_version": "858",
        "charactor_version": "001",
        "disable": false,
        "opts": {
            "nolog": false
        },
        "phase": "access",
        "action": "deny",
        "meta": 403,
        "severity": "high",
        "rule_name": "iputil",
        "desc": "某ip段内不许访问",
        "match": [{
            "vars": [{
               "var": "REMOTE_ADDR"
            }],
            "operator": "ip_utils",
            "pattern": ["1.1.1.0/24","2.2.2.2-2.2.20.2"]
        }]
    }
]
```

###rules_id
**syntax:** *rules_id table*

**default:** *none*

**context:** *twaf_secrules*

用于排除特征

[Back to TOC](#table-of-contents)

Rule Directives
===============
```txt
-- lua格式
    {
        id = "xxxxxx",                             -- ID标识(唯一)，string类型
        release_version = "858",                   -- 特征库版本，string类型
        charactor_version = "001",                 -- 特征规则版本，string类型
        severity = "low",                          -- 严重等级，OPENWAF中使用"low"，"medium","high"等，string类型
        rule_name = "test",                        -- 特征名称，string类型
        disable = false,                           -- 禁用此规则，boolean类型
        opts = {                                   -- 其余动作
            nolog = false,                         -- 不记日志，true or false，默认false
            add_resp_headers = {                   -- 自定义响应头
                key_xxx = "value_xxx"              -- 响应头名称 = 响应头值
            }
            setvar = {{                            -- 设置变量，数组
                column = "test",                   -- 变量的一级key，如：TX，session等，string类型
                key = "test",                      -- 变量的二级key，如：score, id等，string类型
                incr = true,                       -- 同modsec中的=+操作， true or false，默认false
                value = 5,                         -- 变量值，number类型
                time = 3000                        -- 超时时间(ms)，number类型
            }}
        },
        phase = "test",                            -- 执行阶段（"access","header_filter","body_filter"），支持数组和字符串
        action = "test",                           -- 动作，ALLOW，DENY等，string类型
        desc = "test",                             -- 描述语句
        tags = {"test1", "test2"},                 -- 标签
        match = {                                  -- 数组，match之间是“与”的关系，相当于modsecurity的action中chain的功能
            {
                vars = {{                          -- 数组，或的关系，取代modsec中的"|"，处理多个变量，
                    var = "test",                  -- 变量名称，string类型
                    parse = {                      -- 对变量的解析，下面的操作只能出现一种
                        specific = "test",         -- 具体化，取代modsec中的":",支持数组，TODO:支持正则，如modsec的"/ /"，支持字符串和数组
                        ignore = "test",           -- 忽略某个值,取代modsec中的"!"，TODO:支持正则，如modsec的"/ /"，支持字符串和数组
                        keys = true,               -- 取出所有的key
                        values = true,             -- 取出所有的value
                        all = true                 -- 取出所有的key和value
                    }
                }},
                transform = "test",                -- 转换操作,支持字符串和数组
                operator = "test",                 -- 操作，string类型
                pattern = "test",                  -- 操作参数，支持boolean、number、字符串和数组
                parse_pattern = true|false,        -- 是否解析pattern参数（目前不支持与pf组合），如pattern参数为"%{TX.1}"
                pf = "file_path",                  -- 操作参数，文件路径
                op_negated = true                  -- 操作取反，true or false，默认false
            },
            {match_info2},
            {match_info3},
            ....
        }
    }

--json格式
    {
        "id": "xxxxxx",
        "release_version": "858",
        "charactor_version": "001",
        "severity": "test",
        "rule_name": "test",
        "disable": false, 
        "opts": {
            "nolog": false,
            "add_resp_headers": {
                "key_xxx": "value_xxx"
            },
            "setvar": [{
                "column":"test",
                "key":"test",
                "incr": true,
                "value": 5,
                "time":3000
            }]
        },
        "phase": "test",
        "action": "test",
        "desc": "test",
        "tags": ["test1", "test2"]
        "match": [
            {
                "vars": [{
                    "var": "test",
                    "storage": true,
                    "phase": "test",
                    "parse": {
                        "specific": "test",
                        "ignore": "test",
                        "keys": true,
                        "values": "test",
                        "all": "test"
                    }
                }],
                "transform": "test",
                "operator": "test",
                "pattern": "test",
                "parse_pattern": true|false,
                "pf": "file_path",
                "op_negated": true
            },
            {"match_info2"},
            {"match_info3"},
            ...
        ]
    }
```

Variables
=========
* [ARGS](#args)
* [ARGS_COMBINED_SIZE](#args_combined_size)
* [ARGS_GET](#args_get)
* [ARGS_GET_NAMES ](#args_get_names)
* [ARGS_NAMES](#args_names)
* [ARGS_POST ](#args_post)
* [ARGS_POST_NAMES ](#args_post_names)
* [BYTES_IN](#bytes_in)
* [CONNECTION_REQUESTS](#connection_requests)
* [DURATION](#duration)
* [FILES](#files)
* [FILES_NAMES](#files_names)
* [GEO](#geo)
* [GEO_CODE3](#geo_code3)
* [GEO_CODE3](#geo_code)
* [GEO_ID](#geo_id)
* [GEO_CONTINENT](#geo_continent)
* [GEO_NAME](#geo_name)
* [GZIP_RATIO](#gzip_ratio)
* [HTTP_COOKIE](#http_cookie)
* [HTTP_HOST](#http_host)
* [HTTP_REFERER](#http_referer)
* [HTTP_USER_AGENT](#http_user_agent)
* [IP_VERSION](#ip_version)
* [MATCHED_VAR](#matched_var)
* [MATCHED_VARS](#matched_vars)
* [MATCHED_VAR_NAME](#matched_var_name)
* [MATCHED_VARS_NAMES](#matched_var_names)
* [ORIGINAL_DST_ADDR](#original_dst_addr)
* [ORIGINAL_DST_PORT](#original_dst_port)
* [POLICYID](#policyid)
* [QUERY_STRING](#query_string)
* [RAW_HEADER](#raw_header)
* [RAW_HEADER_TRUE](#raw_header_true)
* [REMOTE_ADDR](#remote_addr)
* [REMOTE_HOST](#remote_host)
* [REMOTE_PORT](#remote_port)
* [REMOTE_USER](#remote_user)
* [REQUEST_BASENAME](#request_basename)
* [REQUEST_BODY](#request_body)
* [REQUEST_COOKIES](#request_cookies)
* [REQUEST_COOKIES_NAMES](#request_cookies_names)
* [REQUEST_FILENAME](#request_filename)
* [REQUEST_HEADERS](#request_headers)
* [REQUEST_HEADERS_NAMES](#request_headers_names)
* [REQUEST_LINE](#request_line)
* [REQUEST_METHOD](#request_method)
* [REQUEST_PROTOCOL](#request_protocol)
* [HTTP_VERSION](#http_version)
* [URI](#uri)
* [URL](#url)
* [REQUEST_URI](#request_uri)
* [RESPONSE_BODY](#response_body)
* [RESPONSE_HEADERS](#response_headers)
* [RESPONSE_STATUS](#response_status)
* [SCHEME](#scheme)
* [SERVER_ADDR](#server_addr)
* [SERVER_NAME](#server_name)
* [SERVER_PORT](#server_port)
* [SESSION](#session)
* [SESSION_DATA](#session_data)
* [TIME](#time)
* [TIME_DAY](#time_day)
* [TIME_EPOCH](#time_epoch)
* [TIME_HOUR](#time_hour)
* [TIME_MIN](#time_min)
* [TIME_MON](#time_mon)
* [TIME_SEC](#time_sec)
* [TIME_WDAY](#time_wday)
* [TIME_YEAR](#time_year)
* [TIME_LOCAL](#time_local)
* [TX](#tx)
* [UNIQUE_ID](#unique_id)
* [UPSTREAM_CACHE_STATUS](#upstream_cache_status)
* [USERID](#userid)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS

table类型，所有的请求参数，包含ARGS_GET和ARGS_POST

```
例如：POST http://www.baidu.com?name=miracle&age=5

请求体为：time=123456&day=365

ARGS变量值为{"name": "miracle", "age": "5", "time": "123456", "day": "365"}
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_COMBINED_SIZE

number类型，请求参数总长度，只包含key和value的长度，不包含'&'或'='等符号

```
例如：GET http://www.baidu.com?name=miracle&age=5

ARGS_COMBINED_SIZE变量值为15，而不是18
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_GET

table类型，querystring参数

```
例如：GET http://www.baidu.com?name=miracle&age=5

ARGS_GET变量值为{"name": "miracle", "age": "5"}
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_GET_NAMES

table类型，querystring参数key值

```
例如：GET http://www.baidu.com?name=miracle&age=5

ARGS_GET_NAMES变量值为["name", "age"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_NAMES

table类型，querystring参数key值及post参数key值

```
例如：POST http://www.baidu.com?name=miracle&age=5

请求体为：time=123456&day=365

ARGS_NAMES变量值为["name", "age", "time", "day"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_POST

table类型，POST参数

```
例如：

POST http://www.baidu.com/login.html

请求体为：time=123456&day=365

ARGS_POST变量值为{"time": "123456", "day": "365"}
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ARGS_POST_NAMES

table类型，POST参数key值

```
例如：

POST http://www.baidu.com/login.html

请求体为：time=123456&day=365

ARGS_POST_NAMES变量值为["time", "day"]
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##BYTES_IN

number类型，接收信息字节数

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##CONNECTION_REQUESTS

number类型，当前连接中的请求数

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##DURATION

string类型，处理事务用时时间，单位秒(s)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##FILES

table类型，从请求体中得到的原始文件名(带有文件后缀名)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##FILES_NAMES

table类型，上传文件名称（不带有后缀名）

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO

table类型，包含code3,code,id,continent,name等字段信息

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO_CODE3

string类型，3个字母长度的国家缩写

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO_CODE

string类型，2个字母长度的国家缩写

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO_ID

number类型，国家ID

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO_CONTINENT

string类型，国家所在大洲

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GEO_NAME

string类型，国家全称

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##GZIP_RATIO

string类型，压缩比率

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_COOKIE

string类型，请求头中的cookie字段

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_HOST

string类型，请求头中的host字段值，既域名:端口(80缺省)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_REFERER

string类型，请求头中的referer字段

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_USER_AGENT

string类型，请求头中的user-agent字段

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##IP_VERSION

string类型，IPv4 or IPv6

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VAR

类型不定，当前匹配中的变量

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VARS

table类型，单条规则匹配中的所有变量

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VAR_NAME

string类型，当前匹配中的变量名称

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##MATCHED_VARS_NAMES

table类型，单条规则匹配中的所有变量名称

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ORIGINAL_DST_ADDR

string类型，服务器地址，应用代理模式为WAF地址，透明模式为后端服务器地址

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##ORIGINAL_DST_PORT

string类型，服务器端口号，应用代理模式为WAF端口号，透明模式为后端服务器端口号

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##POLICYID

string类型，策略ID

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##QUERY_STRING

string类型，未解码的请求参数

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RAW_HEADER

string类型，请求头信息，带请求行

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RAW_HEADER_TRUE

string类型，请求头信息，不带请求行

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_ADDR

string类型，客户端地址

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_HOST

string类型，域名

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_PORT

number类型，端口号

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REMOTE_USER

string类型，用于身份验证的用户名

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_BASENAME

string类型，请求的文件名

```
例如: GET http://www.baidu.com/test/login.php

REQUEST_BASENAME值为/login.php
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_BODY

类型不定，请求体

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_COOKIES

table类型，请求携带的cookie

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_COOKIES_NAMES

table类型，请求携带cookie的名称

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_FILENAME

string类型，relative request URL(相对请求路径)

```
例如: GET http://www.baidu.com/test/login.php

REQUEST_FILENAME值为/test/login.php
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_HEADERS

table类型，请求头信息

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_HEADERS_NAMES

table类型，请求头key值

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_LINE

string类型，请求行

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_METHOD

string类型，请求方法

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_PROTOCOL

string类型，http请求协议，如: HTTP/1.1

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##HTTP_VERSION

string类型，http请求协议版本，如: 1.1

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##URI

string类型，请求路径，既不带域名，也不带参数

```
例如: GET http://www.baid.com/test/login.php?name=miracle

URI变量值为/test/login.php
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##URL

string类型，统一资源定位符，SCHEME与HTTP_HOST与URI的拼接

```
例如: GET http://www.baid.com/test/login.php?name=miracle

URL变量值为http://www.baid.com/test/login.php
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##REQUEST_URI

string类型，请求路径，带参数，但不带有域名

```
例如: GET http://www.baid.com/test/login.php?name=miracle

REQUEST_URI变量值为/test/login.php?name=miracle
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RESPONSE_BODY

string类型，响应体

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RESPONSE_HEADERS

table类型，响应头信息

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##RESPONSE_STATUS

function类型，响应状态码

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SCHEME

string类型，http or https

```
例如：GET http://www.baidu.com/

SCHEME变量值为http
```

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SERVER_ADDR

string类型，服务器地址

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SERVER_NAME

string类型，服务器名称

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SERVER_PORT

number类型，服务器端口号

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SESSION

table类型，第三方模块lua-resty-session提供的变量

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##SESSION_DATA

table类型，session信息，第三方模块lua-resty-session提供的变量

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME

string类型，hour:minute:second

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_DAY

number类型，天(1-31)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_EPOCH

number类型，时间戳，seconds since 1970

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_HOUR

number类型，小时(0-23)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_MIN

number类型，分钟(0-59)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_MON

number类型，月份(1-12)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_SEC

number类型，秒(0-59)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_WDAY

number类型，周(0-6)

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_YEAR

number类型，年份，four-digit，例如: 1997

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TIME_LOCAL

string类型，当前时间，例如: 26/Aug/2016:01:32:16 -0400

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##TX

table类型，用于存储当前请求信息的变量，作用域仅仅是当前请求

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##UNIQUE_ID

string类型，ID标识，随机生成的字符串，可通过配置来控制随机字符串的长度

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##UPSTREAM_CACHE_STATUS

keeps the status of accessing a response cache (0.8.3). The status can be either “MISS”, “BYPASS”, “EXPIRED”, “STALE”, “UPDATING”, “REVALIDATED”, or “HIT”.

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

##USERID

string类型，从接入规则配置得到的用于ID标识

[Back to Var](#variables)

[Back to TOC](#table-of-contents)

Transformation Functions
========================
* [base64_decode](#base64_decode)
* [sql_hex_decode](#sql_hex_decode)
* [base64_encode](#base64_encode)
* [counter](#counter)
* [compress_whitespace ](#compress_whitespace )
* [hex_decode](#hex_decode)
* [hex_encode](#hex_encode)
* [html_decode](#html_decode)
* [length](#length)
* [lowercase](#lowercase)
* [md5](#md5)
* [normalise_path](#normalise_path)
* [remove_nulls](#remove_nulls)
* [remove_whitespace](#remove_whitespace)
* [replace_comments](#replace_comments)
* [remove_comments_char](#remove_comments_char)
* [remove_comments](#remove_comments)
* [uri_decode](#uri_decode)
* [uri_encode](#uri_encode)
* [sha1](#sha1)
* [trim_left](#trim_left)
* [trim_right](#trim_right)
* [trim](#trim)

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##base64_decode

Decodes a Base64-encoded string.

Note: 注意transform的执行顺序

```
例如：
{
   "id": "xxxx",
   ...
   "transform": ["base64_decode", "lowercase"],
   ...
}

先执行base64解码，然后字符串最小化，若顺序调换，会影响结果
```

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##sql_hex_decode

Decode sql hex data.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##base64_encode

Encodes input string using Base64 encoding.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##counter

计数，相当于modsecurity中的'&'符号

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##compress_whitespace

Converts any of the whitespace characters (0x20, \f, \t, \n, \r, \v, 0xa0) to spaces (ASCII 0x20), compressing multiple consecutive space characters into one.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##hex_decode

Decodes a string that has been encoded using the same algorithm as the one used in hexEncode 

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##hex_encode

Encodes string (possibly containing binary characters) by replacing each input byte with two hexadecimal characters.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##html_decode

Decodes the characters encoded as HTML entities.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##length

Looks up the length of the input string in bytes

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##lowercase

Converts all characters to lowercase

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##md5

Calculates an MD5 hash from the data in input. The computed hash is in a raw binary form and may need encoded into text to be printed (or logged). Hash functions are commonly used in combination with hex_encode (for example: "transform": ["md5", "hex_encode").

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##normalise_path

Removes multiple slashes, directory self-references, and directory back-references (except when at the beginning of the input) from input string.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_nulls

Removes all NUL bytes from input

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_whitespace

Removes all whitespace characters from input.

移除空白字符\s，包含水平定位字符 ('\t')、归位键('\r')、换行('\n')、垂直定位字符('\v')或翻页('\f')等

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##replace_comments

用一个空格代替/*...*/注释内容

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_comments_char

Removes common comments chars (/*, */, --, #).

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##remove_comments

去掉/*...*/注释内容

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##uri_decode

Unescape str as an escaped URI component.

```
例如: 
"b%20r56+7" 使用uri_decode转换后为 b r56 7
```

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##uri_encode

Escape str as a URI component.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##sha1

Calculates a SHA1 hash from the input string. The computed hash is in a raw binary form and may need encoded into text to be printed (or logged). Hash functions are commonly used in combination with hex_encode (for example, "transform": ["sha1", "hex_encode"]).

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##trim_left

Removes whitespace from the left side of the input string.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##trim_right

Removes whitespace from the right side of the input string.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

##trim

Removes whitespace from both the left and right sides of the input string.

[Back to TFF](#transformation-functions)

[Back to TOC](#table-of-contents)

Operators
=========

* [begins_with](#begins_with)
* [contains](#contains)
* [contains_word](#contains_word)
* [detect_sqli](#detect_sqli)
* [detect_xss](#detect_xss)
* [ends_with](#ends_with)
* [equal](#equal)
* [greater_eq](#greater_eq)
* [greater](#greater)
* [ip_utils](#ip_utils)
* [less_eq](#less_eq)
* [less](#less)
* [pf](#pf)
* [regex](#regex)
* [str_match](#str_match)
* [validate_url_encoding](#validate_url_encoding)
* [num_range](#num_range)
* [str_range](#str_range)

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##begins_with

Returns true if the parameter string is found at the beginning of the input.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##contains

Returns true if the parameter string is found anywhere in the input.

operator为contains且pattern为数组，相当于modsecurity的pm

PS: modsecurity的pm忽略大小写，OpenWAF中contains不忽略大小写

```
例如:
{
    "id": "xxx",
    ...
    "operator": "contains",
    "pattern": ["abc", "def"],
    ...
}
```

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##contains_word

Returns true if the parameter string (with word boundaries) is found anywhere in the input.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##detect_sqli

This operator uses LibInjection to detect SQLi attacks.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##detect_xss

This operator uses LibInjection to detect XSS attacks.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##ends_with

Returns true if the parameter string is found at the end of the input.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##equal

Performs a string comparison and returns true if the parameter string is identical to the input string.

相当于modsecurity的eq和streq

```
例如:
{
    "id": "xxx",
    ...
    "operator": "equal",
    "pattern": [12345, "html", "23456"]
    ...
}
```

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##greater_eq

Performs numerical comparison and returns true if the input value is greater than or equal to the provided parameter.

return false, if a value is provided that cannot be converted to a number.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##greater

Performs numerical comparison and returns true if the input value is greater than the operator parameter.

return false, if a value is provided that cannot be converted to a number.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##ip_utils

Performs a fast ipv4 or ipv6 match of REMOTE_ADDR variable data. Can handle the following formats:

Full IPv4 Address: 192.168.1.100
Network Block/CIDR Address: 192.168.1.0/24
IPv4 Address Region: 1.1.1.1-2.2.2.2

ip_utils与pf的组合相当于modsecurity中的ipMatchF和ipMatchFromFile

```
例如:
规则如下：
{
    "id": "xxxx",
    ...
    "operator": "ip_utils",
    "pf": "/tmp/ip_blacklist.txt",
    ...
}
"/tmp/ip_blacklist.txt"文件内容如下：
192.168.1.100
192.168.1.0/24
1.1.1.1-2.2.2.2
```

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##less_eq

Performs numerical comparison and returns true if the input value is less than or equal to the operator parameter.

return false, if a value is provided that cannot be converted to a number.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##less

Performs numerical comparison and returns true if the input value is less than to the operator parameter.

return false, if a value is provided that cannot be converted to a number.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##pf

pattern是operator操作的参数

pf是指pattern from file，与pattern互斥（二者不可同时出现），目前仅支持绝对路径

pf与contains组合，相当于modsecurity的pmf或pmFromFile

pf与ip_utils组合，相当于modsecurity的ipMatchF或ipMatchFromFile

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##regex

Performs a regular expression match of the pattern provided as parameter. 

regex还有modecurity的capture捕获功能

modsecurity有关capture的描述如下：
When used together with the regular expression operator (@rx), the capture action will create copies of the regular expression captures and place them into the transaction variable collection.

OpenWAF中无capture指令，但使用regex默认开启capture功能

```
例如:
{
    "id": "000031",
    "release_version": "858",
    "charactor_version": "001",
    "opts": {
        "nolog": false
    },
    "phase": "access",
    "action": "deny",
    "meta": 403,
    "severity": "low",
    "rule_name": "protocol.reqHeader.c",
    "desc": "协议规范性约束，检测含有不合规Range或Request-Range值的HTTP请求",
    "match": [
        {
            "vars": [
                {
                    "var": "REQUEST_HEADERS",
                    "parse": {
                        "specific": "Range"
                    }
                },
                {
                    "var": "REQUEST_HEADERS",
                    "parse": {
                        "specific": "Request-Range"
                    }
                }
            ],
            "operator": "regex",
            "pattern": "(\\d+)\\-(\\d+)\\,"
        },
        {
            "vars": [{
                "var": "TX",
                "parse": {
                    "specific": "2"
                }
            }],
            "operator": "greater_eq",
            "pattern": "%{TX.1}",
            "parse_pattern": true,
            "op_negated": true
        }
    ]
}
```

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##str_match

等同于contains

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##validate_url_encoding

Validates the URL-encoded characters in the provided input string.

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##num_range

判断是否在数字范围内

它与transform的length组合，相当于modsecurity的validateByteRange

```
{
    "id": "xxx",
    ...
    "operator": "num_range",
    "pattern": [10, "13", "32-126"],
    "transform": "length",
    ...
}
```

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)

##str_range

判断是否在字符串范围内

```
例如时间区间判断:
{
    "id": "xxx",
    ...
    "operator": "str_range",
    "pattern": ["01:42:00-04:32:00"],
    ...
}
```

[Back to OPERATORS](#operators)

[Back to TOC](#table-of-contents)


Others
======

* [allow](#allow)
* [deny](#deny)
* [id](#id)
* [nolog](#nolog)
* [op_negated](#op_negated)
* [parse](#parse)
* [pass](#pass)
* [phase](#phase)
* [proxy_cache](#proxy_cache)
* [redirect](#redirect)
* [charactor_version](#charactor_version)
* [severity](#severity)
* [setvar](#setvar)
* [meta](#meta)
* [transform](#transform)
* [tag](#tag)
* [release_version](#release_version)
* [robot](#robot)
* [add_resp_headers](#add_resp_headers)


[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##allow

Stops rule processing of the current phase on a successful match and allows the transaction to proceed.

```
"action": "allow"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##deny

Stops rule processing and intercepts transaction.

```
"action": "deny",
"meta": 403
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##id

Stops rule processing and intercepts transaction.

```
"id": "xxxxxxx"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##nolog

不记录日志

```
"opts": {
    "nolog": true
}
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##op_negated

对operator结果的取反

```
"match": [{
    "vars": [{
        "var": "HTTP_USER_AGENT"
    }],
    "transform": "length",
    "operator": "less_eq",
    "pattern": 50,
    "op_negated": true
}]

等价于

"match": [{
    "vars": [{
        "var": "HTTP_USER_AGENT"
    }],
    "transform": "length",
    "operator": "greater",
    "pattern": 50
}]

若请求头中user_agent字段长度大于50，则匹配中此条规则
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##parse

对变量进一步解析

```
若请求GET http://www.baidu.com?name=miracle&age=5

"match": [{
    "vars": [{
        "var": "ARGS_GET"
    }]，
    ...
}]
得到的值为{"name": "miracle", "age": "5"}


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "specific": "name"
        }
    }]
}]
得到的值为["miracle"]


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "specific": ["name", "age"]
        }
    }]
}]
得到的值为["miracle", "5"]


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "ignore": "name"
        }
    }]
}]
得到的值为{"age": "5"}


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "ignore": ["name", "age"]
        }
    }]
}]
得到的值为[]


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "keys": true
        }
    }]
}]
得到的值为["name", "age"]


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "values": true
        }
    }]
}]
得到的值为["miracle", "5"]


"match": [{
    "vars": [{
        "var": "ARGS_GET",
        "parse": {
            "all": true
        }
    }]
}]
得到的值为["name", "age", "miracle", "5"]
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##pass

Continues processing with the next rule in spite of a successful match.

```
"action": "pass"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##phase

规则执行的阶段，取值可为"access","header_filter","body_filter"的组合

```
{
    "id": "xxx_01",
    "phase": "access",
    ...
}
"xxx_01"规则在access阶段执行

{
    "id": "xxx_02",
    "phase": ["access", "header_filter"],
    ...
}
"xxx_02规则在access阶段和"header_filter"阶段各执行一次
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##proxy_cache

```
{
    ...
    phase = "header_filter",         -- 缓存开关需在header_filter阶段配置
    action = "pass",                 -- 无需拦截请求
    opts = {
        nolog = true,                -- 不需记录日志
        proxy_cache = {
            state = true|false,      -- 缓存开关
            expired = 600            -- 缓存时长（单位秒）,默认600秒
        }
    }
    ...
}

若state为true，且得到的缓存状态为"MISS"或"EXPIRED"，则对响应内容进行缓存，同时设置缓存时长
若state为false，则清除对应缓存键的缓存（包含其缓存文件）
```

举例如下：
```
# nginx.conf 有关proxy cache 配置如下
http {
    proxy_cache_path  /opt/cache/OpenWAF-proxy levels=2:2 keys_zone=twaf_cache:101m max_size=100m use_temp_path=off;
    proxy_cache_key $host$uri;
    proxy_cache twaf_cache;
    proxy_ignore_headers X-Accel-Expires Cache-Control Set-Cookie;
    proxy_no_cache $twaf_cache_flag;
    
    server {
        set $twaf_cache_flag 1;         #默认不缓存
    }
}

# lua 格式 配置
{ 
    id = "test_x01",                      -- id 全局唯一
    opts = {
        nolog = true,
        proxy_cache = {
            state = true,
            expired = 300
        }
    },
    phase = "header_filter", 
    action = "pass",
    match = {{
        vars = {{
            var = "URI"
        },{
            var = "REQUEST_HEADERS",
            parse = {
                specific = "Referer"
            }
        }},
        operator = "equal",
        pattern = {"/xampp/", "%{SCHEME}://%{HTTP_HOST}/xampp/"},
        parse_pattern = true
    }}
}
此规则将缓存URI为'/xampp/'的页面，更新时间为300秒

若match中过滤条件为响应码，则相当于Nginx的proxy_cache_valid指令
若match中过滤条件为请求方法，则相当于Nginx的proxy_cache_methods指令
若macth中过滤条件为资源类型，则相当于Nginx的proxy_cache_content_type指令

PS: proxy_cache_content_type指令为官方指令，是miracle Qi修改Nginx源码扩展的功能
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##redirect

```
"action": "redirect",
"meta": "/index.html"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##charactor_version

指定此条规则的版本，同modsecurity中Action的rev功能

```
"charactor_version": "001"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##severity

Assigns severity to the rule in which it is used.

The data below is used by the OWASP ModSecurity Core Rule Set (CRS):

EMERGENCY: is generated from correlation of anomaly scoring data where there is an inbound attack and an outbound leakage.
ALERT: is generated from correlation where there is an inbound attack and an outbound application level error.
CRITICAL: Anomaly Score of 5. Is the highest severity level possible without correlation. It is normally generated by the web attack rules (40 level files).
ERROR: Error - Anomaly Score of 4. Is generated mostly from outbound leakage rules (50 level files).
WARNING: Anomaly Score of 3. Is generated by malicious client rules (35 level files).
NOTICE: Anomaly Score of 2. Is generated by the Protocol policy and anomaly files.
INFO
DEBUG

也可自定义严重等级，如:low，medium，high，critical等

```
"severity": "high"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##setvar

Creates, removes, or updates a variable. 

```
{
    "id": "xxx_01",
    "opts":{
        "nolog": false,
        "setvar": [{
            "column": "TX",
            "key": "score",
            "value": 5,
            "incr": true
        }]
    },
    ...
}
"xxx_01"规则中，给变量TX中score成员的值加5，若TX中无score成员，则初始化为0，再加5

{
    "id": "xxx_02",
    "opts":{
        "nolog": false,
        "setvar": [{
            "column": "TX",
            "key": "score",
            "value": 5
        }]
    },
    ...
}

"xxx_02"规则中，给变量TX中score成员的值赋为5
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##meta

"action"的附属信息

```
若"action"为"deny"，则"meta"为响应码
"action": "deny",
"meta": 403

若"action"为"redirect"，则"meta"为重定向地址
"action": "redirect",
"meta": "/index.html"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##transform

This action is used to specify the transformation pipeline to use to transform the value of each variable used in the rule before matching.

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##tag

Assigns a tag (category) to a rule.

```
支持数组    "tag": ["xxx_1", "xxx_2"]
支持字符串  "tag": "xxx_3"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##release_version

规则集版本，等同于modsecurity中Action的ver功能

```
"release_version": "858"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##robot

人机识别

需提前配置人机识别模块配置，此功能暂未放开

```
"action": "robot"
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)

##add_resp_headers

增删改响应头

```
例如隐藏server字段:
"opts": {
    "add"_resp_headers": {
        "server": ""
    }
}
```

[Back to OTHERS](#others)

[Back to TOC](#table-of-contents)
