# 海外钱台接口文档

**版本: 2016-08-22**

* [0. 请求方式](#0-请求方式)
	* [0.1. 非 OAuth 2.0 接口](#01-非-oauth-20-接口)
	* [0.2. OAuth 2.0 接口](#02-oauth-20-接口)
* [1. 子商户注册/查询接口](#1-子商户注册查询接口)
	* [1.1. /mch/v1/signup 注册接口](#11-mchv1signup-注册接口)
	* [1.2. /mch/v1/uploadcert 补件](#12-mchv1uploadcert-补件)
	* [1.3. /mch/v1/query 查询用户信息](#13-mchv1query-查询用户信息)
* [2. 支付接口](#2-支付接口)
	* [2.1. /trade/v1/payment 支付接口](#21-tradev1payment-支付接口)
	* [2.2. /trade/v1/close 关闭订单](#22-tradev1close-关闭订单)
	* [2.3. /trade/v1/refund 退款](#23-tradev1refund-退款)
	* [2.4. /trade/v1/query 查询订单信息](#24-tradev1query-查询订单信息)
* [3. OAuth 授权接口](#3-oauth-授权接口)
	* [3.1. /oauth/v2/authorize 用户授权](#31-oauthv2authorize-用户授权)
	* [3.2. /oauth/v2/access_token 获取/刷新 access_token](#32-oauthv2access_token-获取刷新-access_token)
* [4. OAuth 查询接口](#4-oauth-查询接口)
	* [4.1. /user/v1/baseinfo 获取用户信息](#41-userv1baseinfo-获取用户信息)
	* [4.2. /user/v1/tradelist 获取订单信息](#42-userv1tradelist-获取订单信息)

---

## 0. 请求方式

### 0.1. 非 OAuth 2.0 接口

1. 根据各接口文档, 使用 GET 或 POST 方式请求. 对注册, 补件等需要上传文件的接口, Content-Type 需设置为 `multipart/form-data`.
2. 如无特殊说明, 所有请求均应加上以下 HTTP 头:
    - X-QF-APPCODE: 分配给开发者的 `appcode`.
    - X-QF-SIGN: 数据签名.
3. 数据签名的计算方式: 将所有请求参数 (文件除外) 按照字典序排序, 并拼装成 `a=1&b=2` 的格式, 再在最后拼接上事先分配的 `appkey`, 并计算 MD5 得到签名. 如对于请求数据:

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

1. 按照标准 OAuth 流程, 将用户重定向到 `/oauth/v2/authorize` 接口, 并需带上 `appid`, `scope` 和 `redirect_uri` 等参数.
2. 用户确认授权后, 会将用户再重定向到 `redirect_uri`, 并附带 `code` 参数.
3. 使用 `code` 获取 `access_token`.

---

## 1. 子商户注册/查询接口

### 1.1. /mch/v1/signup 注册接口

注册子商户接口, 调用成功后会返回子商户 id `mchid`, 通过审核后可以使用子商户进行收款.

子商户默认登录密码为身份证号后 6 位.

- POST

    | 参数名             | 描述               | 是否必填 | 备注 |
    |--------------------|--------------------|----------|------|
    | username           | 用户名             | 必填     |      |
    | idnumber           | 身份证号           | 必填     |      |
    | name               | 企业名称           | 必填     |      |
    | country            | 国家               | 必填     |      |
    | city               | 城市               | 必填     |      |
    | address            | 详细地址           | 必填     |      |
    | shopname           | 店铺名称/收据名称  | 必填     |      |
    | bankaccount        | 银行卡号           | 必填     |      |
    | bankuser           | 开户人姓名         | 必填     |      |
    | bankcountry        | 开户行所在国家     | 必填     |      |
    | bankcity           | 开户行所在市       | 必填     |      |
    | bankaddr           | 开户行详细地址     | 必填     |      |
    | bankname           | 开户行名称         | 必填     |      |
    | bankcode           | 开户行编号         | 必填     |      |
    | bankswiftcode      | 联行号             |          |      |
    | mcc_id             | 商户类别 id        | 必填     |      |
    | legalperson        | 法人姓名           |          |      |
    | channel_type       | 渠道 ID            | 必填     |      |
    | email              | 邮箱               | 必填     |      |
    | mobile             | 手机号             | 必填     |      |
    | licensenumber      | 证书编号           | 必填     |      |
    | licenseactive_date | 公司申报生效日期   | 必填     |      |
    | idcardfront        | 法人身份证正面照片 | 必填     | 文件 |
    | licensephoto       | 公司商业登记证照片 | 必填     | 文件 |
    | goodsphoto         | 经营场所内景照片   | 必填     | 文件 |
    | shopphoto          | 经营场所外景照片   | 必填     | 文件 |
    | credit_front       | 公司注册证书照片   |          | 文件 |
    | shoper             | 业务员在店铺照片   |          | 文件 |
    | other_1            | 其他               |          | 文件 |
    | other_2            | 其他               |          | 文件 |

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

    | 参数名 | 描述                    | 是否必填 | 备注 |
    |--------|-------------------------|----------|------|
    | mchid  | signup 接口返回的 mchid |          |      |

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

---

## 2. 支付接口

### 2.1. /trade/v1/payment 支付接口

- POST

    | 参数名       | 描述                                                              | 是否必填 | 备注                                         |
    |--------------|-------------------------------------------------------------------|----------|----------------------------------------------|
    | mchid        | signup 接口返回的 mchid                                           |          | 若填写则使用对应的子商户, 不填则使用主商户号 |
    | pay_type     | 交易类型, 必填                                                    | 必填     |                                              |
    | out_trade_no | 外部订单号, 必填                                                  | 必填     |                                              |
    | txdtm        | 外部订单生成时间, 必填                                            | 必填     |                                              |
    | txamt        | 交易金额, 必填                                                    | 必填     |                                              |
    | txcurrcd     | 货币, 必填                                                        | 必填     |                                              |
    | udid         | 设备唯一标示                                                      |          |                                              |
    | openid       |                                                                   |          |                                              |
    | sub_openid   |                                                                   |          |                                              |
    | auth_code    | 反扫消费者二维码得到的 `auth_code`                                |          |                                              |
    | lnglat       | 经纬度, 格式为 "12.34 56.78", 精度在前, 维度在后, 以一个空格分隔. |          |                                              |

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

- POST

    | 参数名       | 描述                          | 是否必填                     | 备注                                         |
    |--------------|-------------------------------|------------------------------|----------------------------------------------|
    | mchid        | signup 接口返回的 mchid       |                              | 若填写则使用对应的子商户, 不填则使用主商户号 |
    | syssn        | 创建订单时返回的 syssn.       | 和 `out_trade_no` 二选一必填 |                                              |
    | out_trade_no | 创建订单时使用的 out_trade_no | 和 `syssn` 二选一必填        |                                              |
    | txdtm        | 创建订单时使用的 txdtm        | 必填                         |                                              |
    | txamt        | 订单金额                      | 必填                         |                                              |
    | udid         | 设备唯一标示                  |                              |                                              |

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

    | 参数名       | 描述                          | 是否必填 | 备注                                         |
    |--------------|-------------------------------|----------|----------------------------------------------|
    | mchid        | signup 接口返回的 mchid       |          | 若填写则使用对应的子商户, 不填则使用主商户号 |
    | syssn        | 创建订单时返回的 syssn        | 必填     |                                              |
    | out_trade_no | 创建订单时使用的 out_trade_no | 必填     |                                              |
    | txdtm        | 创建订单时使用的 txdtm        | 必填     |                                              |
    | txamt        | 订单金额                      | 必填     |                                              |
    | udid         | 设备唯一标示                  |          |                                              |

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

### 2.4. /trade/v1/query 查询订单信息

- POST

    | 参数名       | 描述                                  | 是否必填 | 备注                                         |
    |--------------|---------------------------------------|----------|----------------------------------------------|
    | mchid        | signup 接口返回的 mchid               |          | 若填写则使用对应的子商户, 不填则使用主商户号 |
    | syssn        | 订单号                                |          | 支持批量查询, 使用 "," 分割多个 syssn        |
    | out_trade_no | 外部订单号, 支持批量查询              |          |                                              |
    | pay_type     | 支付类型, 支持批量查询                |          |                                              |
    | respcd       | 交易状态码                            |          |                                              |
    | start_time   | 起始时间, 默认会从本月 1 号 0 点开始. |          |                                              |
    | end_time     | 结束时间                              |          |                                              |
    | page         | 查询页码                              |          |                                              |
    | page_size    | 每页返回数量                          |          | 默认为 10                                    |

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

---

## 3. OAuth 授权接口

### 3.1. /oauth/v2/authorize 用户授权

商户引导已登录钱方用户重定向到该页面, 并用 GET 方式附加上以下参数. 用户在页面上点击同意授权后, 会带着 `code` 参数重定向到 `redirect_uri`.

一个完整的调用 url 应该类似这样: `http://127.0.0.1:9527/oauth/v2/authorize?response_type=code&client_id=123456&scope=user_tradelist,user_baseinfo&redirect_uri=http%3A%2F%2Fbaidu.com`

- GET

    | 参数名        | 描述                                                                    | 是否必填 | 备注                 |
    |---------------|-------------------------------------------------------------------------|----------|----------------------|
    | response_type | 固定为 code                                                             | 必填     |                      |
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
    | client_secret | 分配给商户的 appcode                                                                          | 必填                                   |      |
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

