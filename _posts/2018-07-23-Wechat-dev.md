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

微信回调
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

处理微信消息
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
* 业务展开
目前保箱业务主要围绕投保人订单展开，订单管理（A端订单 + B端订单），订单详情，电子保单申请，发票开票，以及保单续保。还有支撑计划书的微信授权获取C用户信息

## 技术点：
* 订单列表查询利用了`Google Guava`包的`ListenableFuture`并发查询A端和B端订单库，最后封装成统一格式返回前端展示
ListenableFuture会检测Future是否完成，如果完成就会自动调用回调函数，这样能减少并发程序的复杂度。

```java
// 通过MoreExecutors类的静态方法listeningDecorator方法初始化一个ListeningExecutorService的方法
// 然后使用此实例的submit方法初始化ListenableFuture对象
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newCachedThreadPool());

// 计数器，A端查询task和B端查询task完成后 继续执行主线程
final CountDownLatch latch = new CountDownLatch(2);
// 接收A/B订单 list
final List<OrderDTO> allOrder = Lists.newArrayList();


// 任务一：查询A端保单列表
ListenableFuture<List<OrderDTO>> futureTaskFromA = service.submit(new Callable<List<OrderDTO>>() {
    @Override
    public List<OrderDTO> call() throws Exception {
        List<OrderDTO> list = clientOrderManager.getToaPolicyList(mobile);
        return list;
    }
});

Futures.addCallback(futureTaskFromA, new FutureCallback<List<OrderDTO>>() {
    @Override
    public void onSuccess(List<OrderDTO> resultFromA) {
        if (!CollectionUtils.isEmpty(resultFromA)) {
            allOrder.addAll(resultFromA);
        }
        latch.countDown();
    }

    @Override
    public void onFailure(Throwable t) {
        latch.countDown();
    }
});
// 任务二：查询B端保单列表
ListenableFuture<List<OrderDTO>> futureTaskFromB = service.submit(new Callable<List<OrderDTO>>() {
    @Override
    public List<OrderDTO> call() throws Exception {
        DubboResultDTO result = customerClientOrderService.queryOrderList(mobile, null, null);
        return clientOrderManager.getTobOrderList(result);
    }
});

Futures.addCallback(futureTaskFromB, new FutureCallback<List<OrderDTO>>() {
    @Override
    public void onSuccess(List<OrderDTO> resultFromB) {
        if (!CollectionUtils.isEmpty(resultFromB)) {
            allOrder.addAll(resultFromB);
        }
        latch.countDown();
    }

    @Override
    public void onFailure(Throwable t) {
        latch.countDown();
    }
});
```

* A端订单通过PolicyService提供的Dubbo接口查询，基于[Sharding-JDBC](https://github.com/shardingjdbc/sharding-jdbc)查询100张表返回投保人订单
* 前端页面获取openId，实现类似token登陆机制
* 微信授权，用户在微信客户端中访问第三方网页，公众号可以通过微信网页授权机制，来获取用户基本信息，进而实现业务逻辑。
微信的认证授权是基于OAuth2.0机制实现的，OAuth2.0基本模式可以理解为用户在授权方进行登录，再由授权方回调请求授权方的接口并带上接口调用凭证，请求授权方再使用该凭证和秘钥信息调用授权方用户信息的相关接口
微信授权可分为静默授权`（scope=snsapi_base）`和非静默授权`（scope=snsapi_userinfo）`，区别：
静默授权可获得用户openId,并自动跳转到回调页，用户无感知
非静默授权用来获取用户的基本信息的。这种授权需要用户手动同意。无须关注，就可在授权后获取该用户的基本信息

基本步骤如下：
1. 用户同意授权，获取code
2. 通过code换取网页授权access_token
3. 刷新access_token（如果需要）
4. 拉取用户信息(需scope为 snsapi_userinfo)
5. 检验授权凭证（access_token）是否有效

```java
/**
 * 授权
 */
private static final String AUTH_URL = "https://open.weixin.qq.com/connect/oauth2/authorize?";

/**
 * 构建授权跳转URL
 * @param redirectUrl 授权后的跳转URL(我方服务器URL)
 * @param quiet 是否静默: true: 仅获取openId，false: 获取openId和个人信息(需用户手动确认)
 * @return 微信授权跳转URL
 */
public String authUrl(String redirectUrl, Boolean quiet) {
    try {
        Preconditions.checkNotNullAndEmpty(redirectUrl, "redirectUrl");
        redirectUrl = URLEncoder.encode(redirectUrl, "utf-8");
        return AUTH_URL +
                "appid=" + wechat.getAppId() +
                "&redirect_uri=" + redirectUrl +
                "&response_type=code&scope=" +
                (quiet ? AuthTypeEnum.BASE.scope() : AuthTypeEnum.USER_INFO.scope())
                + "&state=1#wechat_redirect";
    } catch (UnsupportedEncodingException e) {
        throw new WechatException(e);
    }
}
```

```java
public enum AuthTypeEnum {
    BASE("snsapi_base"),

    USER_INFO("snsapi_userinfo");

    private String scope;

    AuthTypeEnum(String scope){
        this.scope = scope;
    }

    public String scope(){
        return scope;
    }

}
```

授权开发经过了两个版本，第一个版本 在前端中间页授权，但是授权过程中会有空白页出现，后来改成了服务端授权，流程图如下：
![wechat_oauth](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2018/wechat_oauth.png)


## 微信组件化

## 微信开发小工具