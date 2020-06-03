 ## 接口列表

|  接口  | 说明 |
|------ |----- |
|[/p/global/strategy](#strategy)| 请求全局策略|
|[/p/user/userLogin](#login) | 用户登录接口|

***
##错误码列表
|  错误码  | 说明 |
|------ |----- |
|   0   | 正确 |
|   1   | 错误 |
|   2   | 参数错误|

## 接口详情
* <span id = "strategy">全局策略</span>

    * 接口地址：/p/global/strategy

    * 返回格式：Json

    * 请求方式：Post

    * 请求示例：http://www.baidu.com/p/global/strategy

    * 接口备注：通过用户信息，请求获得全局策略。

    * 请求参数说明：

        | 名称 | 类型 | 必填 |说明|
        |----- |------| ---- |----|
        |marking |string|true|企业标识|
        |<font color=red>account | string |true|用户名|
        |<font color=red>pw | string |true|用户密码|

    * 返回参数说明：

        | 名称 | 类型 |说明|
        |----- |------|----|
        | rt | int|状态码
        |data | object|具体数据|
        |identify_method | int|登录方式|
        |send_url | string|推送地址|

    * JSON返回示例：

             {
                 "rt": 0,
                 "data": {
                     "identify_method": "3",
                     "send_url": "www.jianshu.com"
                 }
             }


---

* <span id = "login">登录接口</span>

    * 接口地址：/p/global/login

    * 返回格式：Json

    * 请求方式：Post

    * 请求示例：http://www.baidu.com/p/global/strategy

    * 接口备注：通过用户信息，请求获得全局策略。

    * 请求参数说明：

        | 名称 | 类型 | 必填 |说明|
        |----- |------| ---- |----|
        |marking |string|true|企业标识|
        |<font color=red>account | string |true|用户名|
        |<font color=red>pw | string |true|用户密码|

    * 返回参数说明：

        | 名称 | 类型 |说明|
        |----- |------|----|
        | rt | int|状态码
        |data | object|具体数据|
        |identify_method | int|登录方式|
        |send_url | string|推送地址|

    * JSON返回示例：

             {
                 "rt": 0,
                 "data": {
                     "identify_method": "3",
                     "send_url": "www.jianshu.com"
                 }
             }


---