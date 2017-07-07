---
layout: post
title: Spring RestFul API统一异常处理
date: 2017-05-07
tags: [JAVA, Spring]
---

## 概要
> 基于Spring MVC的保险类产品投保接口项目 [wy-insurance-interface](http://git.winbaoxian.com:8888/wy-serverside/wy-insurance-interface)，提供了个险、车险投保统一接口。接口通过HTTP请求并返回Json来完成通信。业务处理过程中，会遇到许多需要处理的异常信息，现在项目中通过在Controller层try catch来封装异常信息，代码耦合度高，维护工作量大。因此，加入Spring全局异常处理来统一处理维护异常信息。

目前，车险模块统一返回格式如下：
```json
{
  "resultInfo": {
    "code": 400,
    "partnerCode": "wyxx",
    "insureComCode": "CI0003",
    "transId": "17dc52a6d79515b9811205a122ef060004",
    "operationCode": 106,
    "serialNo": "2017041120533863111973665250",
    "msg": "未查询到续保信息",
    "msgDetail": ""
  }
}
```

* 成功：code=200

    如果业务处理成功，设置JSON对象的resultInfo中code为200，同时会返回其他实际业务对象，carInfo，carBizInfo，carForceInfo，carOwner，channelInfo...

* 失败：code=400
    如果业务处理失败，设置code为400，并在JSON对象msg中放入错误信息，msgDetail中放详细错误信息。


异常统一处理后加入 [HTTP状态码](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)，来标示具体业务以外的状态，例如：

* 200
    * 服务正常

* 500
    * 服务器内部错误

* 404
    * NOT FOUND

* 400
    * 系统异常

车险接口统一参数 示例1：
```json
{
    "status": 400,
    "code": 400,
    "message": "参数校验不合法",
    "messageDetail": "证件类型不合法「1:身份证；2:军官证；3:港澳通行证；4:护照；5:异常身份证；6:出生证明；7:工商登记证件；9:其他」;手机号不合法「1开头的11位数字」;"
}
```

示例2：
```json
{
    "status": 500,
    "code": 500,
    "message": "Internal Server Error",
    "messageDetail": "请检查请求地址是否正确「/carInsure/autoQuote1」 "
}
```

示例3：
```json
{
    "status": 200,
    "code": 200,
    "message": "成功",
    "messageDetail": "",
    "throwable": null,
    "data": {
        "resultInfo": {
            "code": 200,
            "errorCodes": [],
            "partnerCode": "wyxx",
            "insureComCode": "CI0003",
            "transId": "a108c4c749fd45e6b94c189391cdca7120",
            "operationCode": 110,
            "serialNo": "2017070718234263734223665250",
            "msg": "成功"
        },
        "baseInfo": {
            "isShowCheckCode": 0
        },
        "carInfo": {},
        "carInfoList": {
            "total": 0
        },
        "carBizInfo": {
            "bizFlag": 1,
            "startDate": 1501516800000,
            "bizRealPremium": 1709.28,
            "bizPremium": 1709.28,
            "riskList": [
                {
                    "riskCode": "A",
                    "riskName": "车辆损失保险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "434720.00",
                            "value": "434720.00"
                        }
                    ]
                },
                {
                    "riskCode": "B",
                    "riskName": "第三者责任保险",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 1364.98,
                    "riskRealPremium": 1364.98,
                    "defaultAmount": 500000,
                    "amount": 500000,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "5万",
                            "value": "50000"
                        },
                        {
                            "key": "10万",
                            "value": "100000"
                        },
                        {
                            "key": "15万",
                            "value": "150000"
                        },
                        {
                            "key": "20万",
                            "value": "200000"
                        },
                        {
                            "key": "30万",
                            "value": "300000"
                        },
                        {
                            "key": "50万",
                            "value": "500000"
                        },
                        {
                            "key": "100万",
                            "value": "1000000"
                        },
                        {
                            "key": "150万",
                            "value": "1500000"
                        },
                        {
                            "key": "200万",
                            "value": "2000000"
                        },
                        {
                            "key": "250万",
                            "value": "2500000"
                        },
                        {
                            "key": "300万",
                            "value": "3000000"
                        },
                        {
                            "key": "350万",
                            "value": "3500000"
                        },
                        {
                            "key": "400万",
                            "value": "4000000"
                        },
                        {
                            "key": "450万",
                            "value": "4500000"
                        },
                        {
                            "key": "500万",
                            "value": "5000000"
                        },
                        {
                            "key": "550万",
                            "value": "5500000"
                        },
                        {
                            "key": "600万",
                            "value": "6000000"
                        },
                        {
                            "key": "650万",
                            "value": "6500000"
                        },
                        {
                            "key": "700万",
                            "value": "7000000"
                        },
                        {
                            "key": "750万",
                            "value": "7500000"
                        },
                        {
                            "key": "800万",
                            "value": "8000000"
                        },
                        {
                            "key": "850万",
                            "value": "8500000"
                        },
                        {
                            "key": "900万",
                            "value": "9000000"
                        },
                        {
                            "key": "950万",
                            "value": "9500000"
                        },
                        {
                            "key": "1000万",
                            "value": "10000000"
                        }
                    ]
                },
                {
                    "riskCode": "D3",
                    "riskName": "车上人员责任保险（司机）",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 34.31,
                    "riskRealPremium": 34.31,
                    "defaultAmount": 10000,
                    "amount": 10000,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "1万",
                            "value": "10000"
                        },
                        {
                            "key": "2万",
                            "value": "20000"
                        },
                        {
                            "key": "3万",
                            "value": "30000"
                        },
                        {
                            "key": "4万",
                            "value": "40000"
                        },
                        {
                            "key": "5万",
                            "value": "50000"
                        },
                        {
                            "key": "10万",
                            "value": "100000"
                        }
                    ]
                },
                {
                    "riskCode": "D4",
                    "riskName": "车上人员责任保险（乘客）",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 87.04,
                    "riskRealPremium": 87.04,
                    "defaultAmount": 10000,
                    "amount": 10000,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "1万",
                            "value": "10000"
                        },
                        {
                            "key": "2万",
                            "value": "20000"
                        },
                        {
                            "key": "3万",
                            "value": "30000"
                        },
                        {
                            "key": "4万",
                            "value": "40000"
                        },
                        {
                            "key": "5万",
                            "value": "50000"
                        },
                        {
                            "key": "10万",
                            "value": "100000"
                        }
                    ]
                },
                {
                    "riskCode": "G1",
                    "riskName": "全车盗抢险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "434720.00",
                            "value": "434720.00"
                        }
                    ]
                },
                {
                    "riskCode": "F",
                    "riskName": "玻璃单独破碎险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "进口玻璃",
                            "value": "2"
                        },
                        {
                            "key": "国产玻璃",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "L",
                    "riskName": "车身划痕损失险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "2000",
                            "value": "2000"
                        },
                        {
                            "key": "5000",
                            "value": "5000"
                        },
                        {
                            "key": "1万",
                            "value": "10000"
                        },
                        {
                            "key": "2万",
                            "value": "20000"
                        }
                    ]
                },
                {
                    "riskCode": "Z",
                    "riskName": "自燃损失险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "434720.00",
                            "value": "434720.00"
                        }
                    ]
                },
                {
                    "riskCode": "A4",
                    "riskName": "指定修理厂险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "X1",
                    "riskName": "发动机涉水险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "434720.00",
                            "value": "434720.00"
                        }
                    ]
                },
                {
                    "riskCode": "A6",
                    "riskName": "车辆损失保险无法找到第三方特约险",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M",
                    "riskName": "不计免赔险(不计免赔险合并)",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 222.95,
                    "riskRealPremium": 222.95,
                    "defaultAmount": 1,
                    "amount": 1,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_A",
                    "riskName": "不计免赔险(车损)",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_B",
                    "riskName": "不计免赔险（三者险）",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 1,
                    "amount": 1,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_G1",
                    "riskName": "不计免赔险（机动车盗抢险）",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_D3",
                    "riskName": "不计免赔险（车上人员责任险（司机））",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 1,
                    "amount": 1,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_D4",
                    "riskName": "不计免赔险（车上人员责任险（乘客））",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 1,
                    "amount": 1,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_L",
                    "riskName": "不计免赔险（车身划痕损失险）",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_Z",
                    "riskName": "不计免赔险（自燃险）",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "M_X1",
                    "riskName": "不计免赔险（发动机涉水险）",
                    "riskFlag": 0,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 0,
                    "amount": 0,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                },
                {
                    "riskCode": "BZ",
                    "riskName": "是否投保交强险",
                    "riskFlag": 1,
                    "riskDesc": "",
                    "riskPremium": 0,
                    "riskRealPremium": 0,
                    "defaultAmount": 1,
                    "amount": 1,
                    "unitAmount": null,
                    "quantity": null,
                    "amountList": [
                        {
                            "key": "不投保",
                            "value": "0"
                        },
                        {
                            "key": "投保",
                            "value": "1"
                        }
                    ]
                }
            ],
            "mFlag": 1,
            "repeatFlag": 0,
            "appoint": "1、车险保单查询制度特别约定：投保次日起，您可以通过本公司网页（www.95590.cn），客户服务电话（95590），营业点核实保单及理赔信息，若对查询结果有异议，请致电本公司客户服务电话;",
            "quotationNoBI": "277829"
        },
        "carForceInfo": {
            "forceFlag": 1,
            "startDate": 1501516800000,
            "forceRealPremium": 950,
            "taxRealPremium": 1500,
            "appoint": "1、车险保单查询制度特别约定:投保次日起，您可以通过本公司网页（www.95590.cn），客户服务电话（95590），营业点核实保单及理赔信息，若对查询结果有异议，请致电本公司客户服务电话;",
            "quotationNoCI": "",
            "repeatFlag": 0
        },
        "insuredInfo": {},
        "applicantInfo": {},
        "carOwner": {
            "idType": 1
        },
        "addresseeInfo": {},
        "carOrderInfo": {
            "totalPremium": 4159.28
        },
        "payInfo": {},
        "channelInfo": {}
    }
}
```

