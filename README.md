# 海外钱台接口文档

**版本: 2016-11-17**

* [0. 请求方式](#0-请求方式)
	* [0.0. 名词解释](#00-名词解释)
	* [0.1. 非 OAuth 2.0 接口](#01-非-oauth-20-接口)
	* [0.2. OAuth 2.0 接口](#02-oauth-20-接口)
* [1. 子商户注册/查询接口](#1-子商户注册查询接口)
	* [1.1. /mch/v1/signup 注册接口](#11-mchv1signup-注册接口)
	* [1.2. /mch/v1/uploadcert 补件](#12-mchv1uploadcert-补件)
	* [1.3. /mch/v1/query 查询用户信息](#13-mchv1query-查询用户信息)
	* [1.4. /tool/v1/mcc mcc 列表接口](#14-toolv1mcc-mcc-列表接口)
* [2. 支付接口](#2-支付接口)
	* [2.1. /trade/v1/payment 支付接口](#21-tradev1payment-支付接口)
	* [2.2. /trade/v1/close 关闭订单](#22-tradev1close-关闭订单)
	* [2.3. /trade/v1/refund 退款](#23-tradev1refund-退款)
	* [2.4. /trade/v1/reversal 撤销/冲正](#24-tradev1reversal-撤销冲正)
	* [2.5. /trade/v1/query 查询订单信息](#25-tradev1query-查询订单信息)
	* [2.6. 预下单结果通知.](#26-预下单结果通知)
	* [2.7. 支付宝线上支付重定向参数](#27-支付宝线上支付重定向参数)
* [3. OAuth 授权接口](#3-oauth-授权接口)
	* [3.1. /oauth/v2/authorize 用户授权](#31-oauthv2authorize-用户授权)
	* [3.2. /oauth/v2/access_token 获取/刷新 access_token](#32-oauthv2access_token-获取刷新-access_token)
* [4. OAuth 查询接口](#4-oauth-查询接口)
	* [4.1. /user/v1/baseinfo 获取用户信息](#41-userv1baseinfo-获取用户信息)
	* [4.2. /user/v1/tradelist 获取订单信息](#42-userv1tradelist-获取订单信息)
* [5. 清算查询接口](#5-清算查询接口)
	* [5.1. /settlement/v1/query](#51-settlementv1query)
* [6. 附录](#5-附录)
	* [6.1 respcd 列表](#61-respcd-列表)

---

## 0. 请求方式

### 0.0. 名词解释

- appcode: 开发者唯一标示.
- appkey: 开发者密钥, 调用除 SDK 接口外的其它接口时使用, 包括退款, 冲正及 OAuth 获取 access_token 等.

### 0.1. 非 OAuth 2.0 接口

1. 根据各接口文档, 使用 GET 或 POST 方式请求.
2. 对**注册**, **补件**等需要上传文件的接口, Content-Type 需设置为 `multipart/form-data`.
3. 如无特殊说明, 所有请求均应加上以下 HTTP 头:
    - X-QF-APPCODE: 分配给开发者的 `appcode`.
    - X-QF-SIGN: 数据签名.
4. 数据签名的计算方式: 将所有请求参数 (文件除外) 按照字典序排序, 并拼装成 `a=1&b=2` 的格式, 再在最后拼接上事先分配的 `appkey`, 并计算 MD5 得到签名. 如对于请求数据:

    ```
    {
        "username": "some_user",
        "mobile": "14401234567",
        "email": "email@example.com",
        "idnumber": "110000201608220123",
        "name": "测试企业",
        "country": "中国",
        "city": "成都",
        "address": "XX区XX路XX号",
        "shopname": "XX商店",
        "bankuser": "王",
        "bankaccount": "1234567890",
        "bankcountry": "中国",
        "bankcity": "成都",
        "bankname": "中国建设银行",
        "bankcode": "123",
        "mcc_id": "6125485337528623306",
        "channel_type": "1",
        "licenseactive_date": "2016-07-10",
        "licensenumber": "12345678",
        "bankaddr": "银行详细地址",
        "bankswiftcode": "001"
    }
    ```

    按照字典序拼好后为

    ```
    address=XX区XX路XX号&bankaccount=1234567890&bankaddr=银行详细地址&bankcity=成都&bankcode=123&bankcountry=中国&bankname=中国建设银行&bankswiftcode=001&bankuser=王&channel_type=1&city=成都&country=中国&email=email@example.com&idnumber=110000201608220123&licenseactive_date=2016-07-10&licensenumber=12345678&mcc_id=6125485337528623306&mobile=14401234567&name=测试企业&shopname=XX商店&username=some_user
    ```

    假设 `appkey` 为 789012, 则待签名字符串为

    ```
    address=XX区XX路XX号&bankaccount=1234567890&bankaddr=银行详细地址&bankcity=成都&bankcode=123&bankcountry=中国&bankname=中国建设银行&bankswiftcode=001&bankuser=王&channel_type=1&city=成都&country=中国&email=email@example.com&idnumber=110000201608220123&licenseactive_date=2016-07-10&licensenumber=12345678&mcc_id=6125485337528623306&mobile=14401234567&name=测试企业&shopname=XX商店&username=some_user789012
    ```

    计算出的签名为 `E019253E135863C335333C983DF05359`.

### 0.2. OAuth 2.0 接口

1. 按照标准 OAuth 流程, 将用户重定向到 `/oauth/v2/authorize` 接口, 并需带上 `response_type`, `client_id`, `scope` 和 `redirect_uri` 等参数. 目前 `client_id` 与 `appcode` 保持一致.
2. 用户确认授权后, 会将用户再重定向到 `redirect_uri`, 并附带 `code` 参数.
3. 使用 `code` 调用 `/oauth/v2/access_token` 接口获取 `access_token`.

---

## 1. 子商户注册/查询接口

### 1.1. /mch/v1/signup 注册接口

注册子商户接口, 调用成功后会返回子商户 id `mchid`, 通过审核后可以使用子商户进行收款.

- POST

    | 参数名             | 描述               | 是否必填 | 备注                                                                     | 示例                     |
    |--------------------|--------------------|----------|--------------------------------------------------------------------------|--------------------------|
    | username           | 用户名             | 必填     | 格式必须为: 1) 邮箱+数字后缀; 2) 11 位手机号 二者之一, 否则无法通过审核. | some_user@example.com001 |
    | idnumber           | 身份证号           | 必填     |                                                                          | 100000201608220123       |
    | name               | 企业名称           | 必填     |                                                                          | XX商店                   |
    | country            | 国家               | 必填     |                                                                          | 中国                     |
    | city               | 城市               | 必填     |                                                                          | 北京                     |
    | address            | 详细地址           | 必填     |                                                                          | XX区XX路XX号             |
    | shopname           | 店铺名称/收据名称  | 必填     |                                                                          | XX商店                   |
    | bankaccount        | 银行卡号           | 必填     |                                                                          | 1234567890               |
    | bankuser           | 开户人姓名         | 必填     |                                                                          | 王某某                   |
    | bankcountry        | 开户行所在国家     | 必填     |                                                                          | 中国                     |
    | bankcity           | 开户行所在市       | 必填     |                                                                          | 北京                     |
    | bankaddr           | 开户行详细地址     | 必填     |                                                                          | XX区XX路支行             |
    | bankname           | 开户行名称         | 必填     |                                                                          | 中国建设银行             |
    | bankcode           | 开户行编号         | 必填     |                                                                          | 123                      |
    | bankswiftcode      | 联行号             |          |                                                                          | 456                      |
    | mcc_id             | 商户类别 id        | 必填     | 通过调用 mcc 列表接口获取                                                | 6125485337528623306      |
    | legalperson        | 法人姓名           | 必填     |                                                                          | 王某某                   |
    | channel_type       | 渠道 ID            | 必填     |                                                                          |                          |
    | email              | 邮箱               | 必填     |                                                                          | mail@example.com         |
    | mobile             | 手机号             | 必填     |                                                                          | 14412345678              |
    | licensenumber      | 证书编号           | 必填     |                                                                          | 1234567890               |
    | licenseactive_date | 公司申报生效日期   | 必填     |                                                                          | 2016-08-01               |
    | idcardfront        | 法人身份证正面照片 | 必填     | 文件                                                                     |                          |
    | licensephoto       | 公司商业登记证照片 | 必填     | 文件                                                                     |                          |
    | goodsphoto         | 经营场所内景照片   | 必填     | 文件                                                                     |                          |
    | shopphoto          | 经营场所外景照片   | 必填     | 文件                                                                     |                          |
    | credit_front       | 公司注册证书照片   |          | 文件                                                                     |                          |
    | shoper             | 业务员在店铺照片   |          | 文件                                                                     |                          |
    | other_1            | 其他               |          | 文件                                                                     |                          |
    | other_2            | 其他               |          | 文件                                                                     |                          |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "mchid": "BvDtmKJA5mx7GpN0"  // 子商户 id
    }
    ```

### 1.2. /mch/v1/uploadcert 补件

补充上传注册接口中未传的文件.

- POST

    | 参数名       | 描述                    | 是否必填 | 备注 |
    |--------------|-------------------------|----------|------|
    | mchid        | signup 接口返回的 mchid | 必填     |      |
    | idcardfront  | 法人身份证正面照片      |          | 文件 |
    | licensephoto | 公司商业登记证照片      |          | 文件 |
    | goodsphoto   | 经营场所内景照片        |          | 文件 |
    | shopphoto    | 经营场所外景照片        |          | 文件 |
    | credit_front | 公司注册证书照片        |          | 文件 |
    | shoper       | 业务员在店铺照片        |          | 文件 |
    | other_1      | 其他                    |          | 文件 |
    | other_2      | 其他                    |          | 文件 |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": ""
    }
    ```

### 1.3. /mch/v1/query 查询用户信息

- GET

    | 参数名 | 描述                    | 是否必填 | 备注         |
    |--------|-------------------------|----------|--------------|
    | mchid  | signup 接口返回的 mchid | 必填     | 子商户 mchid |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "data": {
            "username": "",       // 用户名
            "name": "",           // 企业名称
            "email": "",          // 邮箱
            "mobile": "",         // 手机号
            "idnumber": "",       // 身份证号
            "legalperson": "",    // 法人姓名
            "shopname": "",       // 店铺名
            "country": "",        // 国家
            "city": "",           // 城市
            "address": "",        // 地址
            "businessaddr": "",   // 营业地址
            "bankaccount": "",    // 银行卡号
            "bankuser": "",       // 开户人
            "bankcountry": "",    // 开户行国家
            "bankcity": "",       // 开户行城市
            "bankaddr": "",       // 开户行地址
            "bankname": "",       // 银行名称
            "bankswiftcode": "",  // 联行号
            "state": 1            // 用户状态
        }
    }
    ```

### 1.4. /tool/v1/mcc mcc 列表接口

该接口可以直接 HTTP GET 访问, 无需附加参数及签名.

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "data": {
            "mcc_list": [{
                "mcc_id": 6128181289566687125,
                "mcc_name": "XXX"
            },
            ...
            ]
        }
    }
    ```

---

## 2. 支付接口

### 2.1. /trade/v1/payment 支付接口

- POST

    | 参数名       | 描述                                                              | 是否必填 | 备注          | 示例                |
    |--------------|-------------------------------------------------------------------|----------|---------------|---------------------|
    | mchid        | signup 接口返回的 mchid                                           | 必填     | 子商户 mchid  | BvDtmKJA5mx7GpN0    |
    | pay_type     | 交易类型, 必填                                                    | 必填     |               | 800208              |
    | out_trade_no | 外部订单号, 必填                                                  | 必填     |               | 1470020842103       |
    | txdtm        | 外部订单生成时间, 必填                                            | 必填     |               | 2016-08-01 11:07:22 |
    | txamt        | 交易金额, 必填, 单位为分, 币种为商户绑定的币种                    | 必填     |               | 10                  |
    | product_name | 商品名称                                                          |          |               |                     |
    | valid_time   | 订单有效期, 单位为秒                                              |          | 最少为 300 秒 |                     |
    | udid         | 设备唯一标示                                                      |          |               |                     |
    | openid       |                                                                   |          |               |                     |
    | sub_openid   |                                                                   |          |               |                     |
    | auth_code    | 反扫消费者二维码得到的 auth_code                                  |          |               |                     |
    | lnglat       | 经纬度, 格式为 "12.34 56.78", 精度在前, 维度在后, 以一个空格分隔. |          |               |                     |

    `pay_type` 有如下选项:

    | pay_type | 交易类型         |
    |----------|------------------|
    | 800108   | 支付宝扫码       |
    | 800151   | 支付宝线上预下单 |
    | 800201   | 微信预下单       |
    | 800207   | 微信 H5 下单     |
    | 800208   | 微信扫码         |

    **注意事项**:
    1. 支付宝撤销订单即使是已完成的订单也会撤销**并退款**.
    2. 微信关闭订单只会关闭未完成的订单.
    3. 取决于支付方案的不同, 某些交易类型可能无法使用.

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "Success",
        "pay_type": "800101",
        "syssn": "201607280901020011216135",  // 流水号
        "sysdtm": "2016-07-28 14:13:10",
        "out_trade_no": "1469686389649",
        "txdtm":"2016-07-28 14:13:09",
        "txamt": "1",
        "txcurrcd": "HKD",
        "qrcode": "https://qr.alipay.com/xxxx"  // 预下单时返回的二维码地址
    }
    ```

### 2.2. /trade/v1/close 关闭订单

订单未支付成功时, 关闭订单; 订单支付成功后无法关闭. **目前仅支持微信支付**.

- POST

    | 参数名       | 描述                          | 是否必填                     | 备注         | 示例                     |
    |--------------|-------------------------------|------------------------------|--------------|--------------------------|
    | mchid        | signup 接口返回的 mchid       | 必填                         | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | syssn        | 创建订单时返回的 syssn.       | 和 `out_trade_no` 二选一必填 |              | 201607280901020011216135 |
    | out_trade_no | 创建订单时使用的 out_trade_no | 和 `syssn` 二选一必填        |              | 1470020842103            |
    | txdtm        | 外部订单生成时间, 必填        | 必填                         |              | 2016-08-01 11:07:22      |
    | txamt        | 订单金额                      | 必填                         |              | 10                       |
    | udid         | 设备唯一标示                  |                              |              |                          |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "",
        "sysdtm": "2016-07-28 11:01:39",
        "orig_syssn": "201607280901020011216108", // 原订单号
        "syssn": "201607280901020011216110"  // 关闭订单操作的订单号
    }
    ```

### 2.3. /trade/v1/refund 退款

- POST

    | 参数名       | 描述                     | 是否必填 | 备注         | 示例                     |
    |--------------|--------------------------|----------|--------------|--------------------------|
    | mchid        | signup 接口返回的 mchid  | 必填     | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | syssn        | 创建订单时返回的 syssn   | 必填     |              | 201607280901020011216135 |
    | out_trade_no | 外部订单号, 必填         | 必填     |              | 1470020842103            |
    | txdtm        | 外部订单生成时间, 必填   | 必填     |              | 2016-08-01 11:07:22      |
    | txamt        | 交易金额, 必填, 单位为分 | 必填     |              | 10                       |
    | udid         | 设备唯一标示             |          |              |

- Response

    ``` javascript
    {
        "orig_syssn": "201607280901020011216135",
        "respmsg": "",
        "txdtm": "2016-07-28 14:13:50",
        "txamt": "1",
        "out_trade_no": "1469686430937",
        "sysdtm": "2016-07-28 14:13:51",
        "syssn": "201607280901020011216137",
    }
    ```

### 2.4. /trade/v1/reversal 撤销/冲正

支付失败时关闭订单. 支付成功时退款并关闭订单.

- POST

    | 参数名       | 描述                     | 是否必填 | 备注         | 示例                     |
    |--------------|--------------------------|----------|--------------|--------------------------|
    | mchid        | signup 接口返回的 mchid  | 必填     | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | syssn        | 创建订单时返回的 syssn   | 必填     |              | 201607280901020011216135 |
    | out_trade_no | 外部订单号, 必填         | 必填     |              | 1470020842103            |
    | txdtm        | 外部订单生成时间, 必填   | 必填     |              | 2016-08-01 11:07:22      |
    | txamt        | 交易金额, 必填, 单位为分 | 必填     |              | 10                       |
    | udid         | 设备唯一标示             |          |              |

- Response

    ``` javascript
    {
        "orig_syssn": "201607280901020011216135",
        "respmsg": "",
        "txdtm": "2016-07-28 14:13:50",
        "txamt": "1",
        "out_trade_no": "1469686430937",
        "sysdtm": "2016-07-28 14:13:51",
        "syssn": "201607280901020011216137",
    }
    ```

### 2.5. /trade/v1/query 查询订单信息

- POST

    | 参数名       | 描述                    | 是否必填 | 备注                                              | 示例                                              |
    |--------------|-------------------------|----------|---------------------------------------------------|---------------------------------------------------|
    | mchid        | signup 接口返回的 mchid | 必填     | 子商户 mchid                                      | BvDtmKJA5mx7GpN0                                  |
    | syssn        | 订单号                  |          | 支持批量查询, 使用 "," 分割多个 syssn             | 201607280901020011216135,201607280901020011216136 |
    | out_trade_no | 外部订单号              |          | 支持批量查询, 使用 "," 分割                       | 1470020842103,1470020842104                       |
    | pay_type     | 支付类型                |          | 支持批量查询, 使用 "," 分割                       | 800201,800208                                     |
    | respcd       | 交易状态码              |          |                                                   | 0000                                              |
    | start_time   | 起始时间                |          | 默认会从本月 1 日 0 点开始.                       | 2016-08-01 11:00:00                               |
    | end_time     | 结束时间                |          | 结束时间和起始时间必须在一个月内, 即不能跨月查询. | 2016-08-21 11:00:00                               |
    | page         | 查询页码                |          |                                                   | 1                                                 |
    | page_size    | 每页返回数量            |          | 默认为 10                                         | 20                                                |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "respmsg": "", 
        "page": 1,
        "page_size": 10,
        "data": [{
            "pay_type": "800201",
            "sysdtm": "2016-07-26 17:02:01",
            "order_type": "payment",
            "txcurrcd": "HKD", 
            "txdtm": "2016-07-26 17:02:01",
            "txamt": "1",
            "out_trade_no": "1469523721486",
            "syssn": "201607260901020011216001",
            "cancel": "2",
            "respcd": "1142",
            "errmsg": ""
        }]
    }
    ```

### 2.6. 预下单结果通知.

对预下单订单, 当得到支付结果后, 会将结果以 HTTP POST 的形式通知给开发者预先设置的回调地址, 并在 HTTP Header 中设置签名 `X-QF-SIGN`. 其中包括以下数据:

| 参数名       | 描述                    | 备注 | 示例                                              |
|--------------|-------------------------|------|---------------------------------------------------|
| mchid        | signup 接口返回的 mchid |      | BvDtmKJA5mx7GpN0                                  |
| syssn        | 订单号                  |      | 201607280901020011216135,201607280901020011216136 |
| out_trade_no | 外部订单号              |      | 1470020842103,1470020842104                       |
| pay_type     | 支付类型                |      | 800201,800208                                     |
| txamt        | 交易金额                |      | 1                                                 |
| txdtm        | 交易时间                |      | 2016-08-21 11:00:00                               |
| respcd       | 交易状态码              |      | 0000                                              |
| respmsg      | 交易结果文字描述        |      |                                                   |

### 2.7. 支付宝线上支付重定向参数

对支付宝线上支付, 支付完成后会重定向到商户设定的 return_url, 并以 GET 方式附带上以下参数:

| 参数名       | 描述             | 备注 | 示例                     |
|--------------|------------------|------|--------------------------|
| mchid        | 子商户 mchid     |      | BvDtmKJA5mx7GpN0         |
| syssn        | 订单号           |      | 201607280901020011216135 |
| out_trade_no | 外部订单号       |      | 1470020842103            |
| pay_type     | 支付类型         |      | 800151                   |
| txamt        | 交易金额         |      | 1                        |
| txdtm        | 交易时间         |      | 2016-08-21 11:00:00      |
| respcd       | 交易状态码       |      | 0000                     |
| respmsg      | 交易结果文字描述 |      |                          |

---

## 3. OAuth 授权接口

### 3.1. /oauth/v2/authorize 用户授权

商户引导已登录钱方用户重定向到该页面, 并用 GET 方式附加上以下参数. 用户在页面上点击同意授权后, 会带着 `code` 参数重定向到 `redirect_uri`.

一个完整的调用 url 应该类似这样: `http://127.0.0.1:9527/oauth/v2/authorize?response_type=code&client_id=123456&scope=user_tradelist,user_baseinfo&redirect_uri=http%3A%2F%2Fbaidu.com`

- GET

    | 参数名        | 描述                                                                    | 是否必填 | 备注                 |
    |---------------|-------------------------------------------------------------------------|----------|----------------------|
    | response_type | 固定为 "code"                                                           | 必填     |                      |
    | client_id     | 分配给商户的 appcode                                                    | 必填     |                      |
    | scope         | 需要申请的权限, 可选的有 user_tradelist user_baseinfo, 多选时用逗号分隔 |          | 默认为 user_baseinfo |
    | redirect_uri  | 用户同意授权后跳转到的 url.                                             |          |                      |

### 3.2. /oauth/v2/access_token 获取/刷新 access_token

商户获取到 code 后, 使用 code 首次获取 access_token. 或是使用 refresh_token 获取 access_token.

- POST

    | 参数名        | 描述                                                                                          | 是否必填                               | 备注 |
    |---------------|-----------------------------------------------------------------------------------------------|----------------------------------------|------|
    | grant_type    | 首次使用 code 时为 "authorization_code", 后续使用 refresh_token 重新获取时为 "refresh_token". | 必填                                   |      |
    | client_id     | 分配给商户的 appcode                                                                          | 必填                                   |      |
    | client_secret | 分配给商户的 appkey                                                                           | 必填                                   |      |
    | code          | authorize 接口重定向时附带的 code                                                             | 必填                                   |      |
    | refresh_token | 刷新 access_token 使用的 refresh_token.                                                       | `grant_type` 为 `refresh_token` 时必填 |      |

- Response

    ``` javascript
    {
        "userid": "8JRvY",
        "token_type": "Bearer",
        "access_token": "93a4bb59-5d65-451f-903e-67c01264803e",
        "refresh_token": "7248224a-aa2a-47ca-a056-b9a1b363d4d7"
        "expires_in": 7200,
    }
    ```

---

## 4. OAuth 查询接口

### 4.1. /user/v1/baseinfo 获取用户信息

使用 access_token 获取用户信息

- GET

    | 参数名       | 描述         | 是否必填 | 备注 |
    |--------------|--------------|----------|------|
    | access_token | access_token | 必填     |      |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "data": {
            "nickname": "\u94b1\u53f0\u6d4b\u8bd5",
            "userid": "rVY8nPPzMYLJV"
        },
        "resperr": "",
        "respmsg": ""
    }
    ```

### 4.2. /user/v1/tradelist 获取订单信息

使用 access_token 获取订单信息, 只能获取 3 个月内的交易.

- GET

    | 参数名       | 描述         | 是否必填 | 备注 |
    |--------------|--------------|----------|------|
    | access_token | access_token | 必填     |      |
    | start_time   | 起始时间     | 必填     |      |
    | end_time     | 结束时间     | 必填     |      |
    | page         | 页码         | 必填     |      |
    | page_size    | 每页数量     | 必填     |      |

- Response

    ``` javascript
    {
        "respcd": "0000",
        "data": {
            "page": 1,
            "pagesize": 10,
            "tradelist": [{
                "pay_type": "800208",
                "sysdtm": "2016-07-20 14:47:50",
                "txcurrcd": "HKD",
                "txdtm": "2016-07-20 14:47:49",
                "txamt": "1",
                "syssn": "201607200901020011215925",
                "customer_id": ""
            }]
        },
        "resperr": "",
        "respmsg": ""
    }
    ```

---

## 5. 清算查询接口

### 5.1 /settlement/v1/query

- GET

    | 参数名      | 描述                              | 是否必填   | 备注         | 示例                     |
    |-------------|-----------------------------------|------------|--------------|--------------------------|
    | mchid       | signup 接口返回的 mchid           | 必填       | 子商户 mchid | BvDtmKJA5mx7GpN0         |
    | date_start  | 查询清算的起始时间                | 必填       |              | 20160101                 |
    | date_end    | 查询清算的结束时间                | 必填       |              | 20160130                 |
    | paymethod   | 支付方式                          | 非必填     |              | 2(支付宝), 3(微信)       |

- Response

    ``` javascript
    {
        "resperr": "",
        "respcd": "0000",
        "respmsg": "",
        "data": [
            {
                "chnl_mch_id": "1315135101",            //通道分配的商户号
                "refund_amt": 0,                        //交易退款总金额
                "date_settlement": "2016-02-29",        //清算时间
                "paymethod": "微信",                    //支付方式
                "date_end": "2016-02-29",               //交易起始时间
                "date_start": "2016-02-23",             //交易结束时间
                "currency": "HKD",                      //币种
                "pay_amt": 433280000,                   //交易支付总金额
                "chnl_poundage_amt": 4309600,           //通道手续费
                "pay_net_amt": 433280000,               //交易支付净额
                "settlement_amt": 428970400             //清算总金额
            }
        ]
    }

    ```

---

## 6. 附录

### 6.1 respcd 列表

| respcd | 错误原因                         |
|--------|----------------------------------|
| 1100   | 系统维护                         |
| 1101   | 需要主动冲正                     |
| 1102   | 重复请求                         |
| 1103   | 报文格式错误                     |
| 1104   | 报文参数错误                     |
| 1105   | 终端未激活                       |
| 1106   | 终端不匹配                       |
| 1107   | 终端被封禁                       |
| 1108   | MAC 校验失败                     |
| 1109   | 加解密错误                       |
| 1110   | 客户端充值, 流水号错误           |
| 1111   | 外部服务不可用                   |
| 1112   | 内部服务不可用                   |
| 1113   | 用户不存在                       |
| 1114   | 用户被封禁                       |
| 1115   | 用户受限                         |
| 1116   | 用户密码错误                     |
| 1117   | 用户不在线                       |
| 1118   | 风控禁止交易                     |
| 1119   | 交易类型受限                     |
| 1120   | 交易时间受限                     |
| 1121   | 交易卡类型受限                   |
| 1122   | 交易币种受限                     |
| 1123   | 交易额度受限                     |
| 1124   | 无效交易                         |
| 1125   | 已退货                           |
| 1126   | 原交易信息不匹配                 |
| 1127   | 数据库错误                       |
| 1128   | 文件系统错误                     |
| 1129   | 已上传凭证                       |
| 1130   | 交易不在允许日期                 |
| 1131   | 渠道错误                         |
| 1132   | 客户端版本信息错误               |
| 1133   | 用户渠道信息错误                 |
| 1134   | 撤销交易刷卡与消费时不是同一张卡 |
| 1135   | 用户配置错误                     |
| 1136   | 交易不存在                       |
| 1137   | 联系方式不存在                   |
| 1138   | 用户更新密钥错                   |
| 1139   | 卡号或者卡磁错误                 |
| 1140   | 账户未审核通过                   |
| 1141   | 计算通道MAC错误                  |
| 1142   | 订单已关闭                       |
| 1143   | 交易不存在                       |
| 1144   | 请求处理失败(协议)               |
| 1145   | 订单状态等待支付                 |
| 1146   | 订单处理业务错误                 |
| 1141   | 通道加密磁道错误                 |
| 1147   | 微信刷卡失败                     |
| 2001   | 机构不存在                       |
| 2002   | 商户绑定失败                     |
| 2003   | 签到失败                         |
| 2004   | 消费者余额不足                   |
| 2005   | 消费者二维码过期                 |
| 2006   | 消费者二维码非法                 |
| 2007   | 消费者关闭了这次交易             |
| 2008   | 传递给通道的参数错误             |
| 2009   | 连接通道失败                     |
| 2010   | 和通道交互的未知错误             |
| 2011   | 交易流水号重复                   |
| 2012   | 用户的通道证书配置错误           |
| 2999   | 通道处理中                       |
| 1151   | 原预授权信息不匹配               |
| 1152   | 预授权完成不在允许日期           |
| 1153   | 预授权完成金额错误               |
| 1154   | 内部错误                         |
| 1155   | 不允许撤销的交易                 |
| 1161   | 交易结果未知，须查询             |
| 1170   | channeld不能提供服务             |
| 1180   | 路由重置, 需重新路由             |
| 1181   | 订单过期                         |
| 1201   | 余额不足                         |
| 1202   | 付款码错误                       |
| 1203   | 账户错误                         |
| 1204   | 银行错误                         |

---

<!--
vim: foldlevel=2
-->
