# 目录

* [功能与注意事项](#功能与注意事项)
  * [pip安装所需模块](#pip安装所需模块)
  * [服务端Usage](#服务端usage)
  * [版本](#版本)
  * [UUID1](#uuid1)
  * [Tornado 与 Motor](#tornado-与-motor)
* [服务端结构](#服务端结构)
* [数据库数据格式](#数据库数据格式)  
  * [原始数据 集合 文档结构](#原始数据-集合-文档结构)   
  * [原始数据 集合 字段要求(BSON)](#原始数据-集合-字段要求)  
  * [合并数据 集合 文档结构](#合并数据-集合-文档结构)   
  * [合并数据 集合 字段要求(BSON)](#合并数据-集合-字段要求) 
* [API](#api)  
  * [POST 上传数据 JSON格式](#post-上传数据-json格式)
    * [POST 上传数据 例子](#post-上传数据-例子)
    * [POST 上传数据 字段要求](#post-上传数据-字段要求)
  * [POST 上传数据 HTTP响应 例子](#post-上传数据-http响应-例子)
  * [POST 查询数据 JSON格式](#post-查询数据-json格式)
    * [POST 查询数据 非聚合 例子](#查询数据-非聚合-例子)  
    * [POST 查询数据 非聚合 树形结构 例子](#查询数据-非聚合-树形结构-例子)
    * [POST 查询数据 聚合 例子](#查询数据-聚合-例子)
    * [POST 查询数据 字段要求](#post-查询数据-字段要求)
  * [POST 查询数据 HTTP响应 返回文件 例子](#post-查询数据-http响应-返回文件-例子)
    * [返回文件 HTTP响应 正常](#返回文件-http响应-正常)
    * [返回文件 HTTP响应 数据格式错误](#返回文件-http响应-数据格式错误)
    * [返回文件 HTTP响应 搜索无结果](#返回文件-http响应-搜索无结果)
  * [POST 查询数据 HTTP响应 不返回文件 例子](#post-查询数据-http响应-不返回文件-例子)
    * [不返回文件 HTTP响应 正常](#不返回文件-http响应-正常)
    * [不返回文件 HTTP响应 数据格式错误](#不返回文件-http响应-数据格式错误)
    * [不返回文件 HTTP响应 搜索无结果](#不返回文件-http响应-搜索无结果)

## 功能与注意事项

### pip安装所需模块

[返回目录](#目录)

    pip install -r requirements.txt

### 服务端Usage

[返回目录](#目录)

    Usage: main.py -v <version> [-p <port>] [-f] [-c] [-r]
    
    -v <version> : 版本，float格式，如1.1，详见docs/conf.py  
    -p <port> ：端口，可在conf.py中更改，也可以启动时自定义  
    -f ：强制更新数据至 -v 所指定的的版本，需要关闭所有其他服务端进程  
    -c ：手动清除无效的合并数据(v1为0)，需要关闭所有其他服务端进程  
    -r ：手动清除过期文件，默认清楚24小时前的csv文件，正在被打开的文件将被跳过  
    
注：服务端在处理完强制更新，清除无效数据，清除文件后才会开始接受请求  

### 版本

[返回目录](#目录)

MongoDB : v3.6.5  
Tornado ：v5.0.2  
Motor ：v1.2.2  
Python : v3.6  
其余版本详见requirements.txt  

### UUID1  

[返回目录](#目录)

由于UUID1短时间内相对随机位数较少(14bit随机seq number => 2^14 = 16384情况)，考虑到生日悖论的效应，如果短时间内多次生成UUID1(或存在多Tornado进程时)可能出现重复的情况，如文件正被打开(多Tornado进程时)会出现查询无结果的错误，如文件已经存在并未被打开将会覆盖原有文件，导致针对原有文件的查询不准确。不推荐使用UUID4。

### Tornado 与 Motor

[返回目录](#目录)

服务端所使用的Tornado框架是一个异步非阻塞网络框架，Motor是基于协程的异步非阻塞的MongoDB驱动。  
服务端的设计基于协程，推荐的使用方法是为每一个CPU核开一个进程。 

## 服务端结构

[返回目录](#目录)

![服务端结构](https://github.com/QKKQQK/mh_data/blob/feat/tree-representation/docs/app_structure.jpg)
 
## 数据库数据格式

### 原始数据 集合 文档结构

[返回目录](#目录)

    {
        "_id" : ObjectId,
        "openid" : String,
        "pid" : ObjectId,
        "name" : String,
        "flag" : Int32,
        "exttype" : Int32,
        "type" : Int32,
        "tag" : ObjectId[],
        "klist" : ObjectId[],
        "rlist" : ObjectId[],
        "extlist" : Object,
        "ugroup" : Int32,
        "uid" : ObjectId,
        "fid" : ObjectId,
        "eid" : ObjectId,
        "v1" : Double,
        "v2" : Double,
        "v3" : Object
        "cfg" : String,
        "utc_date" : Date
    }

### 原始数据 集合 字段要求  

[返回目录](#目录)

字段 | 意义 | BSON类型 | 建立索引 | 例子
---- | ---- | --- | ---- | ----
_id  | 数据的MongoDB _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
openid | 数据提交第三方 _id, uuid格式 | String | 是  | "04ca7d4a-706e-11e8-adc0-fa7ae01bbebc"  
pid | 数据的父级节点 _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
name | 事件名称 | String | 是 | "文档查看"  
flag | 数据状态标志 | Int32 | 是 | 1  
exttype | 事件所属小类别代码 | Int32 | 是 | 512  
type | 事件所属大类别代码 | Int32 | 是 | 50  
tag | 事件相关标签(备用) | Array | 是 | [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]  
klist | 知识点树路径 | Array | 是 | [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]  
rlist | 关系树路径 | Array | 是 | [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]  
extlist | 拓展路径(备用) | Object | 是  (索引例子：extlist, extlist.test_path) | {test_path : [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]}  
ugroup | 用户所属大分类(如"届")代码 | Int32 | 是 | 2016  
uid | 用户 _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
fid | 文件 _id(备用) | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
eid | 设备 _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
v1 | 数值(如"操作次数") | Double | 是 | 10.0 (注：v1不可为0或负数)
v2 | 数值(如"事件时长") | Double | 是 | 10.0 (注：v2不可为负数)
v3 | 拓展数值(备用) | Object | 是  (索引例子：v3, v3.test_val1, v3.test_val2) | {test_val1 : 10.0, test_val2 : 9999.0} (注：v3任意值不可为负数, v3初始化时会有一个对默认键值{'placeholder' : 0})
cfg | 字符串值 | String | 是 | "Y\|Y\|Y\|"  
utc_date | 原始数据创建日期时间 | Date | 是 | "2018-06-12 10:53:54.247"  

### 合并数据 集合 文档结构

[返回目录](#目录)

    {
        "_id" : ObjectId,
        "openid" : String,
        "pid" : ObjectId,
        "name" : String,
        "flag" : Int32,
        "exttype" : Int32,
        "type" : Int32,
        "tag" : ObjectId[],
        "klist" : ObjectId[],
        "rlist" : ObjectId[],
        "extlist" : Object,
        "ugroup" : Int32,
        "uid" : ObjectId,
        "fid" : ObjectId,
        "eid" : ObjectId,
        "v1" : Double,
        "v1_norm" : Double,
        "v2" : Double,
        "v2_norm" : Double,
        "v3" : Object,
        "v3_norm" : Object,
        "cfg" : String,
        "utc_date" : Date,
        "version" : Double
    }

### 合并数据 集合 字段要求  

[返回目录](#目录)

字段 | 意义 | BSON类型 | 建立索引 | 例子
---- | ---- | --- | ---- | ----
_id  | 数据的MongoDB _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
openid | 数据提交第三方 _id, uuid格式 | String | 是  | "04ca7d4a-706e-11e8-adc0-fa7ae01bbebc"  
pid | 数据的父级节点 _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
name | 事件名称 | String | 是 | "文档查看"  
flag | 数据状态标志 | Int32 | 是 | 1  
exttype | 事件所属小类别代码 | Int32 | 是 | 512  
type | 事件所属大类别代码 | Int32 | 是 | 50  
tag | 事件相关标签(备用) | Array | 是 | [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]  
klist | 知识点树路径 | Array | 是 | [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]  
rlist | 关系树路径 | Array | 是 | [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]  
extlist | 拓展路径(备用) | Object | 是  (索引例子：extlist, extlist.test_path) | {test_path : [ObjectId("5b3a62680000000000000000"), ObjectId("5b3a62680000000000000001")]}  
ugroup | 用户所属大分类(如"届")代码 | Int32 | 是 | 2016  
uid | 用户 _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
fid | 文件 _id(备用) | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
eid | 设备 _id | ObjectId | 是 | ObjectId("5b3a62680000000000000000")  
v1 | 数值(如"操作次数") | Double | 是 | 10.0 (注：v1不可为0或负数)
v1_norm | v1归一值(如"操作次数") | Double | 是 | 0.896
v2 | 数值(如"事件时长") | Double | 是 | 10.0 (注：v2不可为负数)
v2_norm | v2归一值(如"事件时长") | Double | 是 | 0.764
v3 | 拓展数值(备用) | Object | 是  (例子：v3, v3.test_val1, v3.test_val2) | {test_val1 : 10.0, test_val2 : 9999.0, placeholder : 0}
v3_norm | v3各attr归一值(备用) | Object | 是 (索引例子：v3_norm, v3_norm.test_val1, v3_norm.test_val2) | {test_val1 : 0.243, test_val2 : 1.034, placeholder : 0}
cfg | 字符串值 | String | 是 | "Y\|Y\|Y\|"  
utc_date | 原始数据创建日期时间，分钟级 | Date | 是 | "2018-06-12 10:53:00" 
version | 数据创建时的服务端版本号 | Double | 是 | 1.1

## API  

### POST 上传数据 JSON格式
#### 路径: /data

#### POST 上传数据 例子

[返回目录](#目录)

    {
        "data" : [{
            "_id" : {"$oid" : "5a0ab7dad5cb310b9830ef27"},  
            "openid" : "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc",  
            "pid"  : {"$oid" : "5a0ab7dad5cb310b9830ef27"},  
            "name" : "密码重置",
            "flag" : 1,
            "exttype" : 512,  
            "type" : 50,  
            "tag": [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}],  
            "klist" : [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}],  
            "rlist" : [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}], 
            "extlist" : { "path_1" : [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}] },  
            "ugroup" : 2015,  
            "uid" : {"$oid" : "5a0ab7dad5cb310b9830ef27"},  
            "fid" : {"$oid" : "5a0ab7dad5cb310b9830ef27"},  
            "eid" : {"$oid" : "5a0ab7dad5cb310b9830ef27"},  
            "v1" : 10.0,  
            "v2" : 15.0,  
            "v3" : {  "val_1" : 123.456 },  
            "cfg" : "Y|Y|Y|",  
            "utc_date" : {"$date" : 1530010000000}
        }, {
            ...
        }, {
            ...
        }]
    }

### POST 上传数据 字段要求  

[返回目录](#目录)

字段 | 意义 | 必需 | 类型 |  要求 | 默认值 | 例子  
---- | ---- | ---- | ---- | ---- | ----- | ----- 
_id  | 数据的MongoDB _id | 是 | Object | 24位16进制数字字符串，不可重复 | 无 | {"$oid" : "5a0ab7dad5cb310b9830ef27"}  
openid | 数据提交第三方 _id | 是 | String | 使用UUID1 | 无 | "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc"  
pid  | 数据的父级节点 _id | 否 | Object | 24位16进制数字字符串 | {"$oid" : "000000000000000000000000"} | {"$oid" : "5a0ab7dad5cb310b9830ef27"}  
name | 事件名称 | 否 | String | 无 | "" | "密码重置"   
flag | 数据有效性 | 否 | Number | 1为有效，0为无效(被删除，不会显示在搜索结果里但保留在数据库中) | 1 | 1  
exttype | 事件所属小类别代码 | 是 | Number | 无 | 无 | 512  
type | 事件所属大类别代码 | 是 | Number | 无 | 无 |50  
tag | 事件相关标签(备用) | 否 | Object[] | 24位16进制数字字符串 | [] | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]  
klist | 知识点树路径 | 否 | Object[] | 24位16进制数字字符串 | [] | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]  
rlist | 关系树路径 | 否 | Object[] | 24位16进制数字字符串 | [] | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]  
extlist | 拓展路径(备用) | 否 | Object | 24位16进制数字字符串 | {} | {"path_1" : [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]}  
ugroup | 用户所属大分类(如"届")代码 | 否 | Number | 无 | 0 | 2015  
uid | 用户 _id | 否 | Object | 24位16进制数字字符串 | {"$oid" : "000000000000000000000000"} | {"$oid" : "5a0ab7dad5cb310b9830ef27"}  
fid | 文件 _id(备用) | 否 | Object | 24位16进制数字字符串 | {"$oid" : "000000000000000000000000"} | {"$oid" : "5a0ab7dad5cb310b9830ef27"}  
eid | 设备 _id | 是 | Object | 24位16进制数字字符串 | 无 | {"$oid" : "5a0ab7dad5cb310b9830ef27"}  
v1 | 数值(如"操作次数") | 是 | Number | v1必须大于0 | 无 | 10.0  
v2 | 数值(如"事件时长") | 否 | Number | 无 | 0 | 15.0  
v3 | 拓展数值(备用) | 否 | Object | 仅用于存储Number类，不可用于储存Object类 | {} | {"val_1" : 123.456}  
cfg | 字符串值 | 否 | String | 无 | "" | "Y\|Y\|Y\|"  
utc_date | 数据时间戳 | 是 | Object | 时间戳(毫秒)，无需考虑时区 | 无 | {"$date" : "1530010000000"}  

### POST 上传数据 HTTP响应 例子

[返回目录](#目录)

    {
        "code": 2, // 0为正常，1为POST请求格式出错，2为一部分或全部数据出错
        "data": {
            "inserted_ids": [ // 成功插入的数据的_id
                {
                    "$oid": "000000000000000000000001"
                }
            ],
            "updated_ids": [ // 成功更新的数据的_id
                {
                    "$oid": "000000000000000000000002"
                },
                {
                    "$oid": "000000000000000000000003"
                }
            ],
            "err_ids_with_msgs": [ // 出错的数据的_id和错误信息
                {
                    "_id" : {
                        "$oid": "000000000000000000000004"
                    },
                    "err_msg" : "对象化失败，请检查数据格式"
                }
            ]
        },
        "count": {
            "success": 3, // 成功插入或更新的数据数量
            "fail": 1, // 插入或更新失败的数据数量
            "n_insert": 1, // 成功插入的数据数量
            "n_overwrite": 2 // 成功更新的数据数量
        }
    }

### POST 查询数据 JSON格式
#### 路径: /data

#### 查询数据 非聚合 例子

[返回目录](#目录)

    {
        "metadata" : {
            "file" : false
        }, 
        "data" : {
            "openid" : "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc",
            "extlist" : {
                "test_path" : [{
                    "$oid" : "000000000000000000000001"
                }]
            },
            "v3" : {
                "test_1" : [10, 20, 300],
                "test_2" : [20, 500]
            },
            "v3_upper" : {
                "test_1" : [-1, 40, 400],
                "test_2" : [20, -2]
            },
            "rlist" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "pid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "uid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "fid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "eid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "name" : ["重置密码", "迟到"],
            "tag" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "klist" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "cfg" : ["Y|Y|N", "任意字符串"],
            "ugroup" : [2010, 2012, 2013, 2016],
            "ugroup_upper" : [-1, 2012, 2014, -2],
            "exttype" : [100, 200, 250, 500],
            "exttype_upper" : [-1, 200, 300, -2],
            "type" : [10, 20, 25, 50],
            "type_upper" : [-1, 20, 30, -2],
            "v1" : [0.1, 100, 2000, 99999],
            "v1_upper" : [-1, 100, 3000, -2],
            "v2" : [0, 100, 2000, 99999],
            "v2_upper" : [-1, 100, 2000, 99999],
            "date" : [{ // 2018/07/24/00:00:00 GMT+08:00
                "$date" : 1532361600000
            }, { // 2018/07/30/00:00:00 GMT+08:00
                "$date" : 1532880000000
            }], 
            "date_upper" : [{ // 2018/07/24/23:59:59 GMT+08:00
                "$date" : 1532447999000
            }, { // 2018/07/30/23:59:59 GMT+08:00
                "$date" : 1532966399000
            }],
            "sort_order_by" : ["ugroup", "type", "utc_date"],
            "sort_asc" : [1, -1, 1]
        }
    }

#### 查询数据 非聚合 树形结构 例子

[返回目录](#目录)

    {
        "metadata" : {
            "file" : true,
            "tree" : true,
            "tree_attr_proj" : ["v1", "v2", "v2_norm", "v3.test_1", "v3_norm.test_1"],
            "tree_group_type" : ["max", "avg", "sum"],
            "path" : "rlist",
            "show_raw_data" : true
        }, 
        "data" : {
            "openid" : "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc",
            "extlist" : {
                "test_path" : [{
                    "$oid" : "000000000000000000000001"
                }]
            },
            "v3" : {
                "test_1" : [10, 20, 300],
                "test_2" : [20, 500]
            },
            "v3_upper" : {
                "test_1" : [-1, 40, 400],
                "test_2" : [20, -2]
            },
            "rlist" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "pid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "uid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "fid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "eid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "name" : ["重置密码", "迟到"],
            "tag" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "klist" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "cfg" : ["Y|Y|N", "任意字符串"],
            "ugroup" : [2010, 2012, 2013, 2016],
            "ugroup_upper" : [-1, 2012, 2014, -2],
            "exttype" : [100, 200, 250, 500],
            "exttype_upper" : [-1, 200, 300, -2],
            "type" : [10, 20, 25, 50],
            "type_upper" : [-1, 20, 30, -2],
            "v1" : [0.1, 100, 2000, 99999],
            "v1_upper" : [-1, 100, 3000, -2],
            "v2" : [0, 100, 2000, 99999],
            "v2_upper" : [-1, 100, 2000, 99999],
            "date" : [{ // 2018/07/24/00:00:00 GMT+08:00
                "$date" : 1532361600000
            }, { // 2018/07/30/00:00:00 GMT+08:00
                "$date" : 1532880000000
            }], 
            "date_upper" : [{ // 2018/07/24/23:59:59 GMT+08:00
                "$date" : 1532447999000
            }, { // 2018/07/30/23:59:59 GMT+08:00
                "$date" : 1532966399000
            }],
            "sort_order_by" : ["ugroup", "type", "utc_date"],
            "sort_asc" : [1, -1, 1]
        }
    }

#### 查询数据 聚合 例子

[返回目录](#目录)

    {
        "metadata" : {
            "file" : false
        }, 
        "data" : {  
            "openid" : "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc",
            "extlist" : {
                "test_path" : [{
                    "$oid" : "000000000000000000000001"
                }]
            },
            "v3" : {
                "test_1" : [10, 20, 300],
                "test_2" : [20, 500]
            },
            "v3_upper" : {
                "test_1" : [-1, 40, 400],
                "test_2" : [20, -2]
            },
            "rlist" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "pid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "uid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "fid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "eid" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "name" : ["重置密码", "迟到"],
            "tag" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "klist" : [{
                "$oid" : "000000000000000000000001"
            }, {
                "$oid" : "000000000000000000000002"
            }],
            "cfg" : ["Y|Y|N", "任意字符串"],
            "ugroup" : [2010, 2012, 2013, 2016],
            "ugroup_upper" : [-1, 2012, 2014, -2],
            "exttype" : [100, 200, 250, 500],
            "exttype_upper" : [-1, 200, 300, -2],
            "type" : [10, 20, 25, 50],
            "type_upper" : [-1, 20, 30, -2],
            "v1" : [0.1, 100, 2000, 99999],
            "v1_upper" : [-1, 100, 3000, -2],
            "v2" : [0, 100, 2000, 99999],
            "v2_upper" : [-1, 100, 2000, 99999],
            "utc_date" : [{ // 2018/07/24/00:00:00 GMT+08:00
                "$date" : 1532361600000
            }, { // 2018/07/30/00:00:00 GMT+08:00
                "$date" : 1532880000000
            }], 
            "utc_date_upper" : [{ // 2018/07/24/23:59:59 GMT+08:00
                "$date" : 1532447999000
            }, { // 2018/07/30/23:59:59 GMT+08:00
                "$date" : 1532966399000
            }],
            "sort_order_by" : ["ugroup", "type", "utc_date"],
            "sort_asc" : [1, -1, 1],
            "aggr_group_by" : ["uid", "type"],
            "aggr_attr_proj" : ["v2", "v1_norm", "v3.test_1", "v3_norm.test_1"],
            "aggr_attr_group_type" : ["sum", "avg"]
        }
    }
    

### POST 查询数据 字段要求 

[返回目录](#目录)

字段 | 意义 | 必需 | 类型 | 要求 | 匹配类型 | 例子  
---- | ---- | ---- | ---- | ---- | ----- | -----
metadata. | 查询方式，返回结果方式参数 | 是 |
file | 是否以文件形式返回(返回文件名，不包含文件扩展名) | 是 | Boolean | 需要返回树结构时file必须为true |  | true 
tree | 是否以树形结构文件返回(只能返回文件) | 否(file参数为true时为必需) | Boolean | 树形结构不适用于聚合查询 |  | true 
tree_attr_proj | 树形结构显示的数值字段(v1, v2, v3.attr, v1_norm, v2_norm, v3_norm.attr) | 否(tree参数存在时为必需) | String[] | 无 |  | ["v1", "v2", "v2_norm", "v3.test_1", "v3_norm.test_1"]
tree_group_type | 树形结构聚合类型("max", "min", "sum", "avg") | 否(tree参数为true时为必需) | String[] | 无 |  | ["max", "avg", "sum"]
path | 树形结构路径字段名称("rlist", "klist", "extlist.attr") | 否(tree参数为true时为必需) | String | 无 |  | "rlist"
show_raw_data | 树形结构是否返回合并数据 | 否(tree参数为true时为必需) | Boolean | 无 |  | true 
data. | 查询数据条件参数 | 是 | 
openid | 数据提交第三方 _id | 是 | String | 使用UUID1 | 匹配 | "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc"  
extlist | 匹配拓展路径(备用) | 否 | Object | 24位16进制数字字符串 | 匹配 | {"path_1" : [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]}，相应匹配条件：extlist['path_1']的值(Array类型)中存在{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
v3 | 拓展数值匹配下限(备用) | 否 | Object | 仅用于匹配Number类 | 匹配，范围匹配 | {"test_1" : [10, 20], "test_2" ： [20, 500]}， 配合v3_upper使用，v3的各键值Array中每一位(下限)对应v3_upper相应键值的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)
v3_upper | 拓展数值匹配上限(备用) | 否(v3参数存在时为必需) | Object | 仅用于匹配Number类 | 匹配，范围匹配 | {"test_1" : [-1, 40], "test_2" : [20, -2]}，配合v3的相应匹配条件：(v3['test_1'] <= 10 或 (20 <= v3['test_1'] <= 40))并且(v3['test_2'] = 20 或 v3['test_2'] >= 500)  
rlist | 匹配关系树路径 | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：rlist的值(Array类型)中存在{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
pid | 匹配数据的父级节点 _id | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：pid的值(ObjectId类型)为{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
uid | 匹配用户 _id | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：uid的值(ObjectId类型)为{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
fid | 匹配文件 _id(备用) | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：fid的值(ObjectId类型)为{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
eid | 匹配设备 _id | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：eid的值(ObjectId类型)为{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
name | 匹配事件名称 | 否 | String[] | 无 | 匹配 | ["重置密码", "迟到"]，相应匹配条件：name的值(String类型)为"重置密码"或"迟到"
tag | 匹配事件相关标签(备用) | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：tag的值(Array类型)中存在{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
klist | 匹配知识点树路径 | 否 | Object[] | 24位16进制数字字符串 | 匹配 | [{"$oid" : "5a0ab7dad5cb310b9830ef26"}, {"$oid" : "5a0ab7dad5cb310b9830ef27"}]，相应匹配条件：klist的值(Array类型)中存在{"$oid" : "5a0ab7dad5cb310b9830ef26"}或{"$oid" : "5a0ab7dad5cb310b9830ef27"}
cfg | 匹配字符串值 | 否 | String[] | 无 | 匹配 | ["Y\|Y\|Y\|", "ACDCB"]，相应匹配条件：cfg的值(String类型)为"Y\|Y\|Y\|"或"ACDCB"  
ugroup | 用户所属大分类(如“届”)代码匹配下限 | 否 | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [2010, 2012, 2013, 2016]， 配合ugroup_upper使用，ugroup的Array中每一位(下限)对应ugroup_upper的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)
ugroup_upper | 用户所属大分类(如“届”)代码匹配上限 | 否(ugroup参数存在时为必需) | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [-1, 2012, 2014, -2]，配合ugroup的相应匹配条件：ugroup <= 2010 或 ugroup = 2012 或 2013 <= ugroup <= 2014 或 ugroup >= 2016
exttype | 事件所属小类别代码匹配下限 | 否 | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [100, 200, 250, 500]， 配合exttype_upper使用，exttype的Array中每一位(下限)对应exttype_upper的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)
exttype_upper | 事件所属小类别代码匹配上限 | 否(exttype参数存在时为必需) | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [-1, 200, 300, -2]，配合exttype的相应匹配条件：exttype <= 100 或 exttype = 200 或 250 <= exttype <= 300 或 exttype >= 500
type | 事件所属大类别代码匹配下限 | 否 | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [10, 20, 25, 50]， 配合type_upper使用，type的Array中每一位(下限)对应type_upper的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)
type_upper | 事件所属大类别代码匹配上限 | 否(type参数存在时为必需) | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [-1, 20, 30, -2]，配合type的相应匹配条件：type <= 10 或 type = 20 或 25 <= type <= 30 或 type >= 50
v1 | 数值(如"操作次数")匹配下限 | 否 | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [0.1, 100, 2000, 99999]， 配合v1_upper使用，v1的Array中每一位(下限)对应v1_upper的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)
v1_upper | 数值(如"操作次数")匹配上限 | 否(v1参数存在时为必需) | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [-1, 100, 3000, -2]，配合v1的相应匹配条件：v1 <= 0.1 或 v1 = 100 或 2000 <= v1 <= 3000 或 v1 >= 99999
v2 | 数值(如"操作次数")匹配下限 | 否 | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [0, 100, 2000, 99999]， 配合v2_upper使用，v2的Array中每一位(下限)对应v2_upper的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)
v2_upper | 数值(如"操作次数")匹配上限 | 否(v2参数存在时为必需) | Number[] | 仅用于匹配Number类 | 匹配，范围匹配 | [-1, 100, 3000, -2]，配合v2的相应匹配条件：v2 <= 0 或 v2 = 100 或 2000 <= v2 <= 3000 或 v2 >= 99999  
utc_date | 数值(如"操作次数")匹配下限 | 否 | Object[] | 毫秒时间戳 | 匹配，范围匹配 | [{"$date" : 1532361600000}, {"$date" : 1532880000000}]， 等同于[2018/07/24/00:00:00 GMT+08:00, 2018/07/30/00:00:00 GMT+08:00], 配合utc_date_upper使用，utc_date的Array中每一位(下限)对应utc_date_upper的Array位置(上限)，组成一个范围，当一个范围下限等于上限时将变成匹配，下限小于上限时将变成范围匹配，下限大于上限时，上限为-1则为范围匹配(<=下限)，上限为-2时则为范围匹配(>=下限)  
utc_date_upper | 数值(如"操作次数")匹配上限 | 否(utc_date参数存在时为必需) | Object[] | 毫秒时间戳 | 匹配，范围匹配 | [{"$date" : 1532447999000}, -2]，配合utc_date的相应匹配条件：((2018/07/24/00:00:00 GMT+08:00) <= utc_date <= (2018/07/24/23:59:59 GMT+08:00))或 (utc_date >= (2018/07/30/00:00:00 GMT+08:00))  
sort_order_by | 排序所根据的字段 | 否(不推荐排序) | String[] | 无 | | ["ugroup", "type", "utc_date"]，以ugroup排序，同一ugroup再以type排序，同一type以时间排序  
sort_asc | 排序所根据的字段的升序降序 | 否(sort_order_by参数存在时为必需) | Number[] | 1为升序，-1为降序 | | [1, -1, 1]
aggr_group_by | 聚合查询时聚合的字段 | 否(aggr_attr_proj和aggr_attr_group_type参数存在时为必需) | String[] | 无 | | ["uid", "type"]，先以uid聚合，同一uid的数据再以type聚合  
aggr_attr_proj | 聚合查询时需要聚合的数值字段(v1, v2, v3.attr, v1_norm, v2_norm, v3_norm.attr) | 否(aggr_group_by和aggr_attr_group_type参数存在时为必需) | String[] | 无 | | ["v2", "v1_norm", "v3.test_1", "v3_norm.test_1"]，其余数值字段不返回结果  
aggr_attr_group_type | 聚合查询操作("max", "min", "sum", "avg") | 否(aggr_group_by和aggr_attr_proj参数存在时为必需) | String[] | 无 | | ["sum", "avg"]  

### POST 查询数据 HTTP响应 返回文件 例子

注：demo文件夹中有示例csv文件

#### 返回文件 HTTP响应 正常

[返回目录](#目录)

    {
        "code": 0,
        "data": { // 返回文件名，再通过GET /files/813f4686-90a3-11e8-b091-f40f2429b7c7.csv 获取数据
            "uuid": "813f4686-90a3-11e8-b091-f40f2429b7c7"
        }
    }
  
    
#### 返回文件 HTTP响应 数据格式错误  

##### 对应异常：JSON格式不正确，搜索条件的类型出错，如pid需要Object[]类型实际传输数据为Object类型，搜索条件无法转换成正确的BSON格式，如ObjectId不是24个hex字符

[返回目录](#目录)

    {
        "code": 1,
        "err_msg": "数据格式错误"
    }    
    
#### 返回文件 HTTP响应 搜索无结果

[返回目录](#目录)

    {
        "code": 5,
        "err_msg": "搜索无结果"
    }

### POST 查询数据 HTTP响应 不返回文件 例子

#### 不返回文件 HTTP响应 正常

[返回目录](#目录)

    {
        "code": 0,
        "data": [
            {
                "_id": {
                    "$oid": "000000000000000000000001"
                },
                "extlist": {},
                "exttype": 123,
                "fid": {
                    "$oid": "000000000000000000000001"
                },
                "flag": 1,
                "klist": [
                    {
                        "$oid": "000000000000000000000001"
                    },
                    {
                        "$oid": "000000000000000000000002"
                    }
                ],
                "name": "测试",
                "openid": "f857e9f6-6e26-11e8-adc0-fa7ae01bbebc",
                "pid": {
                    "$oid": "000000000000000000000002"
                },
                "rlist": [
                    {
                        "$oid": "000000000000000000000001"
                    },
                    {
                        "$oid": "000000000000000000000002"
                    }
                ],
                "tag": [],
                "type": 123,
                "ugroup": 2011,
                "uid": {
                    "$oid": "000000000000000000000001"
                },
                "cfg": "测试字符串",
                "eid": {
                    "$oid": "000000000000000000000001"
                },
                "utc_date": {
                    "$date": 1530009960000
                },
                "v1": 20,
                "v2": 600,
                "v3": {
                    "placeholder": 0,
                    "test1": 200,
                    "test2": 300
                },
                "version": 1,
                "v1_norm": 0.5252185346943057,
                "v2_norm": 1.1215241260759554,
                "v3_norm": {
                    "placeholder": 0,
                    "test1": 0.928912943312656,
                    "test2": 1
                }
            }, {
                ...
            }, {
                ...
            }
        ],
        "count": {
            "n_record": 3
        }
    }
    
#### 不返回文件 HTTP响应 数据格式错误  

##### 对应异常：JSON格式不正确，搜索条件的类型出错，如pid需要Object[]类型实际传输数据为Object类型，搜索条件无法转换成正确的BSON格式，如ObjectId不是24个hex字符

[返回目录](#目录)

    {
        "code": 1,
        "err_msg": "数据格式错误"
    }    
    
#### 不返回文件 HTTP响应 搜索无结果

[返回目录](#目录)

    {
        "code": 0,
        "data": [],
        "count": {
            "n_record": 0
        }
    }

[返回目录](#目录)
