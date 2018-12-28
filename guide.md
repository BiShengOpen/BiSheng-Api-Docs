## **个人API开发者文档**

### 相关解释:

- 请求方法: GET,POST等

- 请求host: openapi.bisheng.top

- 网关名称: openapi 

- 请求路径:
注意请求路径是以:/网关名称/APIKey/资源路径 的形式,如:/openapi/4IPW5FujlbS/order/new;其中openapi为网关名称,4IPW5FujlbS为APIKey

- 产品规则：
基础币种_计价币种。如BTC/USDT，相关参数为BTC_USDT；ETH/BTC， 相关参数为ETH_BTC。以此类推

请求参数:
query |类型| 描述
---|---|---
APIKey|string| accesskey(除行情接口外所有)
Timestamp|string| 时间戳,为unixtime，1970-1-1 UTX以来的秒数。此值必须与当前时间相差不超过2分钟。
SignatureMethod|stirng|签名方法,固定为secp256k1
SignatureVersion|int|签名版本,目前固定为1
Signature|string|签名

以上是必须要带的query参数,当然还有一些业务的query参数,请根据需要带上

==注意:签名（Signature）的值需要进行base64编码==

### 签名步骤

##### 1. 按照ASCII码对请求参数进行排序[query],如果同一个query参数有多个值,需要对参数值再进行排序。按照升序
比如:原始的请求参数有:
- APIKey
- Timestamp
- SignatureMethod
- SignatureVersion
- foo
- bar
- ABC

经过排序后为:
- ABC
- APIKey
- SignatureMethod
- SignatureVersion
- Timestamp
- bar
- foo

#### 2.拼接要hash的字符串

分为四部分:

=="\n"实际上是换行符==,不是两个字符'\\'和'n'

1. 请求方法(全大写)
```
POST\n
```
2. 请求host

```
openapi.bisheng.top\n
```
3. 请求路径

```
/openapi/4IPW5FujlbS/order/new\n
```
4. 请求参数(排序后的),每个参数之间以'&'间隔,最后没有换行符

```
ABC=xxx&APIKey=xxx&SignatureMethod=xxx&SignatureVersion=xxx&Timestamp=xxx&bar=xxx&foo=xxx
```

最终的字符串:

```
POST\n
openapi.bisheng.top\n
/openapi/4IPW5FujlbS/order/new\n
ABC=xxx&APIKey=xxx&SignatureMethod=xxx&SignatureVersion=xxx&Timestamp=xxx&bar=xxx&foo=xxx
```

首先对这个字符串使用hmacsha256方法进行hash得到H，然后再调用secp256k1进行签名,签名的私钥即用户申请API得得到的APISecrect。得到签名sign后，再进行base64转码，query参数Signature的值就是这个base64转码后的结果。

最终的请求为:
https://openapi.bisheng.top/openapi/4IPW5FujlbS/order/new?ABC=xxx&APIKey=xxx&SignatureMethod=xxx&SignatureVersion=xxx&Timestamp=xxx&bar=xxx&foo=xxx&Signature=xxxxxx


## 示例

1. 请求方法:POST
2. 请求host:openapi.bisheng.top
3. 请求路径:/openapi/nShkJqyAdbcWdb6J/order/new
4. 请求参数

query |值
---|---
APIKey|nShkJqyAdbcWdb6J
Timestamp|1545041797
SignatureMethod|secp256k1
SignatureVersion|1
Signature|需要签名计算
bar|ordernum
foo|userid

先对query进行排序,拼接要hash的字符串,如下:

```
POST
openapi.bisheng.top
/openapi/nShkJqyAdbcWdb6J/order/new
APIKey=nShkJqyAdbcWdb6J&SignatureMethod=secp256k1&SignatureVersion=1&Timestamp=1545041797&bar=ordernum&foo=userid
```
HmacSHA256 hash后得到下面的结果(16进制表示):

```
d523b9f831746008de1f1b24f41971d8f3ae52d15c80b454cebb452f7cc8aaf3
```

假如privatekey为:

```
92d96bc72cd669c080340d4229989769b93ed09744260edd77b11b85f0658320
```
签名过程如下(以golang为例)：

```
    privateKey := "92d96bc72cd669c080340d4229989769b93ed09744260edd77b11b85f0658320"
	private, err := crypto.HexToECDSA(privateKey)
	if err != nil {
		log.Fatal(err)
	}
```

得到private,然后调用:

```
	sign, err := crypto.Sign(hashed, private)
	if err != nil {
		log.Fatal(err)
	}

	base64Sign := base64.StdEncoding.EncodeToString(sign)
```
对hash后的结果进行签名,最后调用base64.StdEncoding.EncodeToString得到最终Signature的值:

```
vCUtoKJwYHDeFQP6f8Wa7A7LLt3D0iMJPNupv1IKS1F%2BZzHD%2Fs9XwnG3nZaRA7WnrsH4646uUDa5VJrgiTiBXAE%3D
```
所以最终请求为:

```
https://openapi.bisheng.top/openapi/nShkJqyAdbcWdb6J/order/new?APIKey=nShkJqyAdbcWdb6J&SignatureMethod=secp256k1&SignatureVersion=1&Timestamp=1545041797&bar=ordernum&foo=userid&Signature=vCUtoKJwYHDeFQP6f8Wa7A7LLt3D0iMJPNupv1IKS1F%2BZzHD%2Fs9XwnG3nZaRA7WnrsH4646uUDa5VJrgiTiBXAE%3D
```

##### 二、请求接口说明

* [0 :说明](#0)
* [1.1:查询用户持仓](#1.1)
* [1.2:查询用户订单记录](#1.2)
* [1.3:挂单](#1.3)
* [1.4:撤单](#1.4)
* [1.5:查询成交回报](#1.5)
* [1.6:获取最大的成交编号](#1.6)
* [1.7:查询成交记录(根据成交编号查询)](#1.7)
* [1.8:查询成交记录(根据订单编号)](#1.8)
* [1.9:查询订单详情](#1.9)
* [2.0:查询所有待成交和部分成交的订单](#2.0)
* [2.1:查询行情](#2.1)
* [2.2:批量撤单](#2.2)
* [2.3:批量查询订单详情](#2.3)
* [2.4:批量下单](#2.4)
 
<h2 id="0">0-说明</h2>

**接口说明**
- 采用https格式restful形式接口；

**主机地址**
- https://openapi.bisheng.top


**安全措施**
- 通过签名验证


<h2 id="1.1">1.1-查询用户持仓</h2>

- 用户的持仓情况

**请求方式：**
- GET

**请求路径：**
- /account/balance

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|prdNames |string |是|产品名称，bsh：查询bsh持仓;bsh_eth_btc：查询bsh、eth、btc持仓,最多5个（产品名称大小写不敏感）|


**返回示例**
状态码：200

``` 
{
    "status": "ok",
    "ts": "20180709150823411",
    "data": [
        {
            "prdName": "ETH",
            "qty": "1",
            "qtyRa": "1"
        },
        {
            "prdName": "BSH",
            "qty": "280.66",
            "qtyRa": "0"
        }
    ]
}
```


 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|prdName |string|产品名|
|qty |string|持仓数量|
|qtyRa |string |锁仓数量|
---

<h2 id="1.2">1.2-查询用户订单记录</h2>

- 用户的下单历史记录

**请求方式：**
- GET

**请求路径：**
- /orders/list

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|type |string|是|订单类型，A/所有，B/买单，S/卖单，D/撤单|
|mkt |string |是|all/所有市场，BSH_ETH/指定市场（大小写不敏感）|
|pageNum |int |是|页(从1开始）|
|pageSize |int |是|页大小（最大不超过100）|
|startTime |int64|是|起始时间，unix时间，精确到秒，10位|
|endTime |int64|是|结束时间，unix时间，精确到秒，10位|


**请求示例：** 
```
http://testapi.yibeix.com/orders/list?userToken=9122c06895cc00ac1d7419621ad2646a1e044549ffce60f5243219f00580c42e&pageSize=10&pageNum=1&startTime=1433209363&endTime=1533715329&mkt=BSH_ETH&type=A
```

**返回示例**
状态码：200

``` 
{
    "status": "ok",
    "ts": 20180808160219137,
    "data": [
        {
            "ordNum": 2018080811624411136,
            "prdName": "BSH",
            "oppoPrdName": "ETH",
            "ordQty": "1",
            "ordOppoQty": "1",
            "ordPrice": "1",
            "ordExeQty": "0",
            "ordTime": 20180808133633533,
            "side": "B",
            "ordSts": "W"
        },
        {
            "ordNum": 2018080884963627008,
            "prdName": "BSH",
            "oppoPrdName": "ETH",
            "ordQty": "12",
            "ordOppoQty": "0.732",
            "ordPrice": "0.061",
            "ordExeQty": "0",
            "ordTime": 20180808133515533,
            "side": "B",
            "ordSts": "W"
        },
           ...
        ]
    }
}
```
 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|订单编号|
|prdName |string|产品名称|
|oppoPrdName |string |对价产品名称|
|ordQty |string|产品数量|
|ordOppoQty |string|对价产品数量|
|ordPrice |string|订单价格|
|ordExeQty |string |已经成交数量|
|ordTime |int64|订单申请时间,unixnano数|
|side |string|B/买，S/卖|
|ordSts |string|订单状态|
---
<h2 id="1.3">1.3-挂单</h2>

- 挂单

**请求方式：**
- POST
- application/json; charset=UTF-8

**请求路径：**
- /orders/create

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|side |string|是|订单类型，B/买单，S/卖单|
|mkt |string |是|eg. BSH_ETH/指定市场|
|price |string |是|对价,以计价货币(BTC、ETH、USDT、WIT等)为计价单位的价格，最多支持8位小数，多于8位直接舍去|
|qty |string |是|交易产品数量,1表示一个BTC或ETH,最多支持八位小数|
|type|string|是|订单类型, M: 市价，L: 限价|

**请求示例**
```
{
"side":"S",
"mkt":"BSH_ETH",
"price":"1",
"qty":"2",
"type":"L",
}
```

**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": "20180709150823411",
    "data": {
        "ordNum":201803302978333700
    }
} 
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|订单编号|
---
<h2 id="1.4">1.4-撤单</h2>

- 撤单

  
**请求方式：**
- POST
- application/json; charset=UTF-8

**请求路径：** 
- /orders/cancel

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|ordNum |int64|是|要撤销的订单编号|

**请求示例：** 
```
{
"ordNum":2018080893220354048,
}
```
**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": "20180709150823411",
    "data": {
        "delOrdNum": 201803310910146560
    }
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|delOrdNum |int64|申报撤单的订单编号|
---

<h2 id="1.5">1.5-查询成交回报</h2>

- 成交回报

**请求方式：**
- GET

**请求路径：**
- /records/list

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|mkt |string|是|市场名称，e.g. BSH_ETH|
|pageNum |int |是|页(从1开始）|
|pageSize |int |是|页大小（最大不超过100）|
|startTime |int64|是|起始时间，unix时间，精确到秒，10位|
|endTime |int64|是|结束时间，unix时间，精确到秒，10位|


**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808170456963,
    "data": [
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803154302533,
            "trdPrice": "1",
            "side": "S",
            "trdQty": "5",
            "trdNum": 18080100000002993,
            "fee": "0.005",
            "trdAmt": "5",
            "selfDeal": 0,
            "ordNum": 2018080354507950080
        },
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803154204533,
            "trdPrice": "1",
            "side": "S",
            "trdQty": "10",
            "trdNum": 18080100000002992,
            "fee": "0.01",
            "trdAmt": "10",
            "selfDeal": 1,
            "ordNum": 2018080354507950080
        },
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803153725533,
            "trdPrice": "1",
            "side": "B",
            "trdQty": "0.5",
            "trdNum": 18080100000002991,
            "fee": "0.0005",
            "trdAmt": "0.5",
            "selfDeal": 1,
            "ordNum": 2018080383923841024
        },
       ...
        ]
    }
}
```
 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|mkt |string|市场名称|
|trdTime |int64|撮合时间，Unixnano数|
|trdPrice |stirng|成交价格|
|trdAmt |string|成交额|
|side |string|成交类型，B/买单成交，S/卖单成交|
|trdQty |string|成交数量|
|fee |string|成交手续费|
|trdNum |int64|成交编号|
|ordNum |int64|订单编号|
|selfDeal |int64| 是否自成交，1为自成交，0为他成交|
---
<h2 id="1.6">1.6-获取最大的成交编号</h2>

- 获取当前时间成交回报的最大编号
- 成交回报最大编号只是针对币币交易业务

**请求方式：**
- GET

**请求路径：**
- /records/maxtrdnum

**请求参数：** 

|参数名|类型|说明|
|:-----|:-----|-----|
|无


**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": "20180709150823411",
    "data": {
        "maxTrdNum": 18041900000000030
    }
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|maxTrdNum |int64|当前最大成交编号|
---
<h2 id="1.7">1.7-查询成交记录(根据成交编号查询)</h2>

- 成交记录(根据起始和终止成交编号查询)
- 查询结果中成交回报的成交编号不一定连续（只返回币币交易）

**请求方式：**
- GET

**请求路径：**
- /records/trdnum

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|startNum|int64 |是|起始成交回报编号，|
|endNum |int64 |是|结束成交回报编号|

**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808171016487,
    "data": [
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803154302533,
            "trdPrice": "1",
            "side": "S",
            "trdQty": "5",
            "trdNum": 18080100000002993,
            "fee": "0.005",
            "trdAmt": "5",
            "selfDeal": 0,
            "ordNum": 2018080354507950080
        },
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803154204533,
            "trdPrice": "1",
            "side": "S",
            "trdQty": "10",
            "trdNum": 18080100000002992,
            "fee": "0.01",
            "trdAmt": "10",
            "selfDeal": 1,
            "ordNum": 2018080354507950080
        },
            ...
        ]
    }
}
```
**返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|mkt |string|市场名称|
|trdTime |int64|撮合时间，Unixnano数|
|trdPrice |stirng|成交价格|
|trdAmt |string|成交额|
|side |string|成交类型，B/买单成交，S/卖单成交|
|trdQty |string|成交数量|
|fee |string|成交手续费|
|trdNum |int64|成交编号|
|ordNum |int64|订单编号|
|selfDeal |int64| 是否自成交，1为自成交，0为他成交|
---

<h2 id="1.8">1.8-查询成交记录(根据订单编号)</h2>

- 成交记录(根据订单编号)


**请求方式：**
- GET

**请求路径：**
- /records/ordnum

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|ordNum |int64 |是|订单编号|


**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808170927869,
    "data": [
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803154204533,
            "trdPrice": "1",
            "trdQty": "10",
            "trdNum": 18080100000002992,
            "trdAmt": "10"
        },
        {
            "mkt": "BSH/ETH",
            "trdTime": 20180803154302533,
            "trdPrice": "1",
            "trdQty": "5",
            "trdNum": 18080100000002993,
            "trdAmt": "5"
        }
    ]
}
```


**返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|mkt |string|市场名称|
|trdTime |int64|撮合时间，Unixnano数|
|trdPrice |stirng|成交价格|
|trdAmt |string|成交额|
|trdQty |string|成交数量|
|trdNum |int64|成交编号|


---
<h2 id="1.9">1.9-查询订单详情</h2>

- 订单详情查询

**请求方式：**
- GET

**请求路径：**
- /orders/status


**请求参数：** 

|参数名|类型|必填|说明|
|:----- |:-----|:-----|-----|
|ordNum |int64|是|订单号|


**返回示例**
状态码：200

``` 
{
    "status": "ok",
    "ts": 20180808172311731,
    "data": {
        "ordNum": 2018080893220354048,
        "prdName": "BSH",
        "oppoPrdName": "ETH",
        "ordQty": "2",
        "ordOppoQty": "2",
        "ordPrice": "1",
        "ordExeQty": "0",
        "ordTime": 20180808161040533,
        "side": "S",
        "ordSts": "C"
    }
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|订单编号|
|prdName |string|产品名称|
|oppoPrdName |string |对价产品名称|
|ordQty |string|产品数量|
|ordOppoQty |string|对价产品数量|
|ordPrice |string|订单价格|
|ordExeQty |string |已经成交数量|
|ordTime |int64|订单申请时间,unixnano数|
|side |string|B/买，S/卖|
|ordSts |string|订单状态，"I" //初始状态；"A" //全部成交；"E" //无效订单； "P" //部分成交，"C" //已经取消；"W" //待成交|
---

<h2 id="2.0">2.0-查询所有待成交和部分成交的订单</h2>

- 查询所有待成交和部分成交的订单

**请求方式：**
- GET

**请求路径：**
- /orders/undeal/status

**请求参数：** 

|参数名|类型|必填|说明|
|:----- |:-----|:-----|-----|
|pageNum |int |是|页(从1开始）|
|pageSize |int |是|页大小（最大不超过100）|



**返回示例**
状态码：200

``` 
{
    "status": "ok",
    "ts": 20180808172524809,
    "data": [
        {
            "ordNum": 2018080886225930240,
            "prdName": "ETH",
            "oppoPrdName": "USDT",
            "ordQty": "1",
            "ordOppoQty": "1",
            "ordPrice": "1",
            "ordExeQty": "0",
            "ordTime": 20180808162544533,
            "side": "S",
            "ordSts": "W"
        },
        {
            "ordNum": 2018080823806298112,
            "prdName": "ETH",
            "oppoPrdName": "USDT",
            "ordQty": "1",
            "ordOppoQty": "1",
            "ordPrice": "1",
            "ordExeQty": "0",
            "ordTime": 20180808162529533,
            "side": "S",
            "ordSts": "W"
        },
        ...
        ]
    }
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|订单编号|
|prdName |string|产品名称|
|oppoPrdName |string |对价产品名称|
|ordQty |string|产品数量|
|ordOppoQty |string|对价产品数量|
|ordPrice |string|订单价格|
|ordExeQty |string |已经成交数量|
|ordTime |int64|订单申请时间,unixnano数|
|side |string|B/买，S/卖|
|ordSts |string|订单状态|
---


<h2 id="2.1">2.1-查询行情</h2>

- 查询行情 15档行情


**请求方式：**
- GET

**请求路径：**
- /market 

**请求参数：**

|参数名|类型|必填|说明|
|:----- |:-----|:-----|-----|
|mkt |string |是|产品名称|


**返回示例**
状态码：200

``` 
{
    "status": "ok",
    "ts": 20180803165105046,
    "data": {
        "id": 0,
        "ts": 20180803165105046,
        "close": 1,
        "open": 0.2796,
        "low": 1,
        "high": 1,
        "todayDealQty": 16.5,
        "vol": 16.5,
        "sell": [
            {
                "ordPrice": 0.0502,
                "ordQty": 0.17
            },
            {
                "ordPrice": 0.3701,
                "ordQty": 5.58
            },
            {
                "ordPrice": 0.7275,
                "ordQty": 0.46
            },
           ...
        ],
        "buy": [
            {
                "ordPrice": 0.8787,
                "ordQty": 1
            },
            {
                "ordPrice": 0.4659,
                "ordQty": 3.12
            },
            {
                "ordPrice": 0.2796,
                "ordQty": 6.2058
            },
            ...
        ]
    }
}
```
 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ts |int64|时间戳|
|close |float64|收盘价|
|open |float64|开盘价|
|low |float64|最低价|
|high |float64|最高价|
|todayDealQty |float64|今日成交量|
|vol |float64|24小时成交量|
|sell |[]float64|卖出报价 ordPrice价格 ordQty数量|
|buy |[]float64|买入报价 ordPrice价格 ordQty数量|
---
<h2 id="2.2">2.2-批量撤单</h2>

- 批量撤单

**请求方式：**
- POST
- application/json; charset=UTF-8

**请求路径：**
- /orders/batch/cancel

**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|ords|[]int64|是|订单编号，如：[2018071694976444416,2018071694976444417,2018071694976444419]

**请求示例：** 
```
{
"ords":[2018080893220354048,2018080853651200000],

}
```

**返回成功示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808164400143,
    "data": [
        {
            "ordNum": 2018080881715107840,
            "status": "ok"
        },
        {
            "ordNum": 2018080881970960384,
            "status": "ok"
        }
    ]
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|申报撤单的订单编号|

**返回有错误的示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808164716369,
    "data": [
        {
            "ordNum": 2018080803840634880,
            "status": "ok"
        },
        {
            "ordNum": 2018080805107314688,
            "status": {
                "code": 1706,
                "msg": "order is not exist"
            }
        }
    ]
}
```
 **返回参数说明** 
 
 |参数名|类型|说明|
|:-----|:-----|-----|
|status |string|状态，ok为成功申请，其他返回相应code错误码|
---

<h2 id="2.3">2.3-批量查询订单详情</h2>

- 批量查询订单详情

**请求方式：**
- POST
- application/json; charset=UTF-8

**请求路径：**
- /orders/batch/status

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|ords|[]int64|是|订单编号，如：[2018071694976444416,2018071694976444417,2018071694976444419]

**请求示例：** 
```
{
"ords":[2018080893220354048,201808085365100000],

}
```
**返回示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808165650356,
    "data": [
        {
            "ordNum": 2018080853651200000,
            "prdName": "BSH",
            "oppoPrdName": "ETH",
            "ordQty": "2",
            "ordOppoQty": "2",
            "ordPrice": "1",
            "ordExeQty": "0",
            "ordTime": 20180808162138533,
            "side": "S",
            "ordSts": "C"
        },
        {
            "ordNum": 2018080893220354048,
            "prdName": "BSH",
            "oppoPrdName": "ETH",
            "ordQty": "2",
            "ordOppoQty": "2",
            "ordPrice": "1",
            "ordExeQty": "0",
            "ordTime": 20180808161040533,
            "side": "S",
            "ordSts": "C"
        }
    ]
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|订单编号|
|prdName |string|产品名称|
|oppoPrdName |string |对价产品名称|
|ordQty |string|产品数量|
|ordOppoQty |string|对价产品数量|
|ordPrice |string|订单价格|
|ordExeQty |string |已经成交数量|
|ordTime |int64|订单申请时间,unixnano数|
|side |string|B/买，S/卖|
|ordSts |string|订单状态|
---

<h2 id="2.4">2.4-批量挂单</h2>

- 挂单

**请求方式：**
- POST
- application/json; charset=UTF-8

**请求路径：**
- /orders/batch/create


**请求参数：** 

|参数名|类型|必填|说明|
|:-----  |:-----|:-----|-----|
|side |string|是|订单类型，B/买单，S/卖单|
|mkt |string |是|eg. BSH_ETH/指定市场|
|price |string |是|对价,以计价货币(BTC、ETH、USDT、WIT等)为计价单位的价格，最多支持8位小数，多于8位直接舍去|
|qty |string |是|交易产品数量,1表示一个BTC或ETH,最多支持八位小数|

**请求示例**
```
{
	"ords":[
		{
		"side":"S",
		"mkt":"BSH_ETH",
		"price":"1",
		"qty":"2"
		},
		{
		"side":"S",
		"mkt":"BSH_ETH",
		"price":"1",
		"qty":"2"
		
		}
	]
}
```

**返回成功示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808162138236,
    "data": [
        {
            "ordNum": 2018080853382764544,
            "status": "ok"
        },
        {
            "ordNum": 2018080853651200000,
            "status": "ok"
        }
    ]
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|ordNum |int64|订单编号|

**返回有错误的示例**
状态码：200

```
{
    "status": "ok",
    "ts": 20180808162328810,
    "data": [
        {
            "ordNum": 2018080817440559104,
            "status": "ok"
        },
        {
            "ordNum": 0,
            "status": {
                "code": 1704,
                "msg": "unsupport market"
            }
        }
    ]
}
```

 **返回参数说明** 
 
|参数名|类型|说明|
|:-----|:-----|-----|
|status |string|状态，ok为成功申请，其他返回相应code错误码|
---

**错误码格式** 
```
{
    "status": "error",
    "ts": 20180808155848222,
    "error": {
        "code": 1709,
        "msg": "no permission"
    }
}
```

 **订单状态说明** 
 
|ordSts状态|说明|
|:-----|:-----|
|I|订单初始化|
|A|全部成交|
|E|错误订单|
|P|部分成交量|
|W|待成交|
|C|订单已取消|
---

**查询撤单状态说明** 
 
|ordSts状态|说明|
|:-----|:-----|
|A|撤单成功|
|E|撤单失败|
---


 **错误码说明** 
 
 | 错误码 | 说明 | 
|:-----|:-----|
|1500|服务器内部错误|
|1700|请求参数错误|
|1701|不支持的产品|
|1702|不支持的类型|
|1703|持仓不足|
|1704|不支持的市场|
|1705|数量不正确|
|1706|订单不存在|
|1707|不支持的交易类型|
|1708|时间戳错误|
|1709|账号没有权限|
|1710|订单已被取消或已完成|
|1711|查询的类型错误|
|1712|api已经被注册|
|1713|请求过于频繁|
|1714|公钥私钥生成失败|
|1715|生成策略组失败|
|1716|请求失败|
|1717|api已达最高限制|
|1718|没有产品|
|1719|查询资产错误|
---
