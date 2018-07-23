---
layout: post
title: 微信开发
date: 2018-07-23
tags: [Wechat]
---

## 介绍

> 微信相关开发包括：公众号开发、小程序开发、微信支付。公众号又有服务号、订阅号、企业号的区别，从开发上无非是微信功能开放多少问题，技术实现区别不大。微易保箱是一个服务号，实现了C端用户订单、保单的部分功能。

## 技术栈
1. Spring这一套
2. ORM-数据库持久层框架用了Mybatis，个人比较喜欢直接写原生SQL的开发模式
3. 前端Vue2.0

## 交互模式

公众号交互模式是 用户、微信服务器、公众号服务器之间发生的信息交换、请求，如下图：

![Wechat_mutual](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2018/wechat_mutual.jpg)

发生信息交互的时候，以用户的openId作为唯一标识。同一用户在同一个公众号下openId是唯一的，如果有多个公众号，要做到唯一标识则必须把公众号都绑定到一个开放平台账号下,得到唯一一个UnionId。

## 开发ing

* 申请测试号
* 开发者配置，接管微信服务器信息

![Wechat_config](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2018/wechat_config.png)

```java
    /**
     * 微信回调
     * /box/wx/callback
     * @return
     * @throws IOException
     */
    @RequestMapping(value = "/callback")
    public void callback(HttpServletRequest request, HttpServletResponse response) {
        // 微信加密签名
        String signature = request.getParameter("signature");
        // 时间戳
        String timestamp = request.getParameter("timestamp");
        // 随机数
        String nonce = request.getParameter("nonce");
        // 随机字符串
        String echostr = request.getParameter("echostr");

        PrintWriter out = null;
        try {
            out = response.getWriter();
            // 通过检验signature对请求进行校验，若校验成功则原样返回echostr，表示接入成功，否则接入失败
            if (SignUtil.checkSignature(signature, timestamp, nonce, appToken)) {
                if(echostr != null) {
                    out.print(echostr);
                } else {
                    // 进行对应的消息/操作处理
                    String resp = btnClickService.processRequest(request);
                    out.print(resp);
                }
            }
        } catch (IOException e) {
            logger.error("处理微信回调异常：", e);
        } finally {
            out.close();
        }
    }
```

```java
    /**
     * 处理微信消息
     * @param request
     * @return
     */
    public String processRequest(HttpServletRequest request) {

        String respMessage = null;
        try {
            // 默认返回的文本消息内容
            String respContent = "你好，欢迎来到微易保险箱！\n" +
                    "我是小微，专注于解决你的保障需求，为你提供最适合、最精选保障和服务。\n\n" +
                    "请先进行绑定，绑定后可查询以本手机号投保的保单。点击<a href=\""+ weChatService.createMenuUri(bindLink) +"\">一键绑定</a>";
            Logger logger = LoggerFactory.getLogger(BtnClickService.class);
            InputStream is = request.getInputStream();
            // xml请求解析
            Map<String, String> requestMap = XMLUtil.INSTANCE.doXMLParse(is);
            // 发送方帐号（open_id）
            String fromUserName = requestMap.get("FromUserName");
            logger.info("fromUserName ----------->" + fromUserName);
            // 公众帐号
            String toUserName = requestMap.get("ToUserName");
            logger.info("toUserName ----------->" + toUserName);
            // 消息类型
            String msgType = requestMap.get("MsgType");
            // 获取文本消息内容
            String content = requestMap.get("Content");
            // 默认回复文本消息
            TextMessage textMessage = new TextMessage();
            textMessage.setToUserName(fromUserName);
            textMessage.setFromUserName(toUserName);
            textMessage.setCreateTime(System.currentTimeMillis());
            textMessage.setMsgType(MessageTypeEnum.RESP_MESSAGE_TYPE_TEXT.value());
            textMessage.setFuncFlag(0);
            textMessage.setContent(respContent);
            respMessage = XMLUtil.INSTANCE.textMessageToXml(textMessage);

            // 文本消息
            if (msgType.equals(MessageTypeEnum.REQ_MESSAGE_TYPE_TEXT.value())) {
                respMessage = "success";
                logger.info("接收到文本消息：{}, 回复：{}",content, respMessage);

            }
            // 事件推送
            else if (msgType.equals(MessageTypeEnum.REQ_MESSAGE_TYPE_EVENT.value())) {
                // 事件类型
                String eventType = requestMap.get("Event");
                // 订阅
                if (eventType.equals(MessageTypeEnum.EVENT_TYPE_SUBSCRIBE.value())) {
                    // 关注
                    logger.info("openid: {} 关注了公众号", fromUserName);
                    userService.saveClientUser(fromUserName, "");
                } else if (eventType.equals(MessageTypeEnum.EVENT_TYPE_UNSUBSCRIBE.value())) {
                    //取消关注
                    logger.info("openid: {} 取消关注", fromUserName);
                    userService.updateClientUserFollowedStatus(fromUserName, true);
                }
                else if (eventType.equals(MessageTypeEnum.EVENT_TYPE_SCAN.value())) {
                    // 已关注状态下扫码
                    String key = requestMap.get("EventKey");
                    textMessage.setContent(respContent);
                    respMessage = XMLUtil.INSTANCE.textMessageToXml(textMessage);
                }
                // 自定义菜单点击事件
                else if (eventType.equals(MessageTypeEnum.EVENT_TYPE_CLICK.value())) {
                    String eventKey = requestMap.get("EventKey");
                    // 多功能客服
                    if (eventKey.equals("serviceButton")) {
                        if (weChatService.queryServerCount()) {
                            logger.info("有客服");
                            respContent = AUTO_REPLY_MSG;
                        } else {
                            logger.info("无客服");
                            respContent = NO_SERVICE_REPLY_MSG;
                        }
                        textMessage.setMsgType(MessageTypeEnum.RESP_MESSAGE_TYPE_TEXT.value());
                        textMessage.setContent(respContent);
                        respMessage = XMLUtil.INSTANCE.textMessageToXml(textMessage);
                        logger.info("serviceButton----------->" + respMessage);

                    }
                }
            }
        } catch (Exception e) {
            logger.error("微易保箱消息处理异常：{}", e);
        }
        return respMessage;
    }
```