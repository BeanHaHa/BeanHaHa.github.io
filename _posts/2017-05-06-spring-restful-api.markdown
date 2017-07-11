---
layout: post
title: Spring RestFul API统一异常处理
date: 2017-05-07
tags: [Spring-RestFul]
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
    "messageDetail": "手机号不合法「1开头的11位数字」;"
}
```

示例2：
```json
{
    "status": 500,
    "code": 500,
    "message": "Internal Server Error",
    "messageDetail": "请检查请求地址是否正确「/carInsure/autoQuote1」"
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
      "errorCodes": [
        
      ],
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
    "carInfo": {
      
    },
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
    "insuredInfo": {
      
    },
    "applicantInfo": {
      
    },
    "carOwner": {
      "idType": 1
    },
    "addresseeInfo": {
      
    },
    "carOrderInfo": {
      "totalPremium": 4159.28
    },
    "payInfo": {
      
    },
    "channelInfo": {
      
    }
  }
}
```

## Spring统一处理异常有三种方式
- 使用@ExceptionHandler注解
- 实现HandlerExceptionResolver接口
- 使用@ControllerAdvice注解

#### @ExceptionHandler注解方式

```java
@Controller      
public class GlobalController {               

   /**    
     * 用于处理异常的    
     * @return    
     */      
    @ExceptionHandler({MyException.class})       
    public String exception(MyException e) {       
        System.out.println(e.getMessage());       
        e.printStackTrace();       
        return "exception";       
    }       

    @RequestMapping("test")       
    public void test() {       
        throw new MyException("出错了！");       
    }                    
} 
```

该方式需要在每个类中处理异常，不适合全局异常处理。

#### HandlerExceptionResolver接口方式

```java
@Component  
public class MyExceptionResolver implements HandlerExceptionResolver {


    @Override
    public ModelAndView resolveException(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex) {

        System.out.println("This is exception handler method!");  
        return null;  
    }
} 
```


在Spring MVC中，所有用于处理在请求映射和请求处理过程中抛出的异常的类，都要实现`HandlerExceptionResolver`接口。`AbstractHandlerExceptionResolver`实现该接口和`Orderd`接口，是HandlerExceptionResolver类的实现的基类。`ResponseStatusExceptionResolver`等具体的异常处理类均在`AbstractHandlerExceptionResolver`之上，实现了具体的异常处理方式。一个基于Spring MVC的Web应用程序中，可以存在多个实现了`HandlerExceptionResolver`的异常处理类，他们的执行顺序，由其order属性决定, order值越小，越是优先执行, 在执行到第一个返回不是null的ModelAndView的Resolver时，不再执行后续的尚未执行的Resolver的异常处理方法。如下图：

![SpringMVCExceptionResolver](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2017/SpringMVCExceptionResolver.png)

车险项目就是采用继承`AbstractHandlerExceptionResolver`类实现全局异常处理：
```java
public class RestExceptionHandler extends AbstractHandlerExceptionResolver implements InitializingBean {
    @Override
    public void afterPropertiesSet() throws Exception {
        
    }

    @Override
    protected ModelAndView doResolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        return null;
    }
}
```

#### @ControllerAdvice注解方式

```java
@Slf4j
@ControllerAdvice
public class ExceptionHandlerBean  extends ResponseEntityExceptionHandler {

    /**
     * 数据找不到异常
     * @param ex
     * @param request
     * @return
     * @throws IOException
     */
    @ExceptionHandler({DataNotFoundException.class})
    public ResponseEntity<Object> handleDataNotFoundException(RuntimeException ex, WebRequest request) throws IOException {
        return getResponseEntity(ex,request,ReturnStatusCode.DataNotFoundException);
    }

    /**
     * 根据各种异常构建 ResponseEntity 实体. 服务于以上各种异常
     * @param ex
     * @param request
     * @param specificException
     * @return
     */
    private ResponseEntity<Object> getResponseEntity(RuntimeException ex, WebRequest request, ReturnStatusCode specificException) {

        ReturnTemplate returnTemplate = new ReturnTemplate();
        returnTemplate.setStatusCode(specificException);
        returnTemplate.setErrorMsg(ex.getMessage());

        return handleExceptionInternal(ex, returnTemplate,
                new HttpHeaders(), HttpStatus.OK, request);
    }

}
```

@Controlleradvice + @ExceptionHandler也可以实现全局的异常处理，继承 `ResponseEntityExceptionHandler` 类来实现全局异常捕获，并且可以返回自定义Json.


## 车险项目全局异常处理实现

使用第二种方式：继承`AbstractHandlerExceptionResolver`类，实现具体异常处理。
![RestExceptionHandler](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2017/restExceotionHandler.png)

定义RestFul接口返回对象：
```java

import org.springframework.http.HttpStatus;
import org.springframework.util.ObjectUtils;

import java.io.Serializable;

/**
 * @Description RestFul接口返回结果
 * @Author daobin<wdb@winbaoxian.com>
 * @Date 2017/7/5.
 */
public class RestResponse<T> implements Serializable {
    private int status;
    private int code;
    private String message;
    private String messageDetail;
    private Throwable throwable;
    private T data;

    public RestResponse(int status, int code, String message, String messageDetail, Throwable throwable, T data) {
        this.status = status;
        this.code = code;
        this.message = message;
        this.messageDetail = messageDetail;
        this.throwable = throwable;
        this.data = data;
    }

    public int getStatus() {
        return status;
    }

    public int getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

    public String getMessageDetail() {
        return messageDetail;
    }

    public Throwable getThrowable() {
        return throwable;
    }

    public T getData() {
        return data;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o instanceof RestResponse) {
            RestResponse re = (RestResponse) o;
            return getStatus() == re.getStatus() &&
                    getCode() == re.getCode() &&
                    ObjectUtils.nullSafeEquals(getMessage(), re.getMessage()) &&
                    ObjectUtils.nullSafeEquals(getMessageDetail(), re.getMessageDetail()) &&
                    ObjectUtils.nullSafeEquals(getThrowable(), re.getThrowable()) &&
                    ObjectUtils.nullSafeEquals(getData(), re.getData());
        }

        return false;
    }

    @Override
    public int hashCode() {
        // noinspection ThrowableResultOfMethodCallIgnored
        return ObjectUtils.nullSafeHashCode(new Object[] {
                getStatus(), getCode(), getMessage(), getMessageDetail(), getThrowable(), getData()
        });
    }

    public static class Builder<T> {

        private HttpStatus status;
        private int code;
        private String message;
        private String messageDetail;
        private Throwable throwable;
        private T data;

        public Builder() {}

        public Builder setStatus(int statusCode) {
            this.status = HttpStatus.valueOf(statusCode);
            return this;
        }

        public Builder setStatus(HttpStatus status) {
            this.status = status;
            return this;
        }

        public Builder setCode(int code) {
            this.code = code;
            return this;
        }

        public Builder setMessage(String message) {
            this.message = message;
            return this;
        }

        public Builder setMessageDetail(String messageDetail) {
            this.messageDetail = messageDetail;
            return this;
        }

        public Builder setThrowable(Throwable throwable) {
            this.throwable = throwable;
            return this;
        }

        public Builder setData(T data) {
            this.data = data;
            return this;
        }

        public RestResponse<T> build() {
            if (this.status == null) {
                this.status = HttpStatus.INTERNAL_SERVER_ERROR;
            }
            return new RestResponse(this.status.value(), this.code, this.message, this.messageDetail, this.throwable, this.data);
        }
    }
}

```

applicantContext.xml配置：
```xml
    <!-- 全局处理异常 START -->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="com.winbaoxian.insurance.http.converter.json.DefaultJackson2HttpMessageConverter">
                <property name="prettyPrint" value="true"/>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
    <mvc:interceptors>
        <bean id="localeChangeInterceptor" class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
            <property name="paramName" value="lang"/>
        </bean>
    </mvc:interceptors>
    <bean id="localeResolver" class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver"/>

    <!-- Unfortunately we have to repeat an instance of this class here because the mvc:message-converters element above
         does not support <ref/> elements: -->
    <bean id="jackson2HttpMessageConverter" class="com.winbaoxian.insurance.http.converter.json.DefaultJackson2HttpMessageConverter">
        <property name="prettyPrint" value="false"/>
    </bean>

    <bean class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
        <property name="order" value="1"/>
        <property name="defaultViews">
            <list>
                <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
            </list>
        </property>
    </bean>

    <bean id="restExceptionResolver" class="com.winbaoxian.insurance.handler.RestExceptionHandler">
        <property name="order" value="100"/>
        <property name="messageConverters">
            <list>
                <ref bean="jackson2HttpMessageConverter"/>
            </list>
        </property>
        <property name="errorResolver">
            <bean class="com.winbaoxian.insurance.handler.DefaultRestErrorResolver">
                <property name="localeResolver" ref="localeResolver"/>
                <property name="exceptionMappingDefinitions">
                    <map>
                        <!-- 400 (NullPointer): -->
                        <entry key="java.lang.NullPointerException" value="400, 空指针异常"/>
                        <entry key="javax.validation.ConstraintViolationException" value="400, 参数校验不合法"/>
                        <!-- 404 -->
                        <entry key="com.stormpath.blog.spring.mvc.rest.exhandler.UnknownResourceException" value="404, _exmsg"/>
                        <!-- 500 (catch all): -->
                        <entry key="Throwable" value="500, Internal Server Error"/>
                    </map>
                </property>
            </bean>
        </property>
    </bean>
    <!-- 全局处理异常 END -->
```

## 参考

[异常统一处理的三种方式与Rest接口异常的处理](https://github.com/pzxwhc/MineKnowContainer/issues/29)

[Restful服务统一异常处理机制](http://nobodyiam.com/2015/03/15/apollo-app-unified-exception-handling/)

[Spring MVC中异常处理的类体系结构](http://www.cnblogs.com/xinzhao/p/4902295.html)