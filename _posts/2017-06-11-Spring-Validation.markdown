---
layout: post
title: Spring Validation 实现前置参数校验
date: 2017-06-11
tags: [Spring-Validation]
---

## 必要性
> [车险项目](http://git.winbaoxian.com:8888/wy-serverside/wy-insurance-interface)，有计划做To B业务，作为接口服务提供方，就非常有必要在项目中加入参数校验模块，限制入口参数。例如：车型查询接口-110，可对车辆信息对象`carInfo`中的车架号、车牌号、发动机号、品牌型号验参。因此，引入Spring Validation框架。

Spring 4.0支持[Bean Validation](http://beanvalidation.org/) 1.0(JSR-303)和[Bean Validation](http://beanvalidation.org/) 1.1(JSR-349)，提供@Validator注解，并且能够通过BindingResult类在Controller层的处理方法中取得错误信息。
```java
import java.util.HashMap;
import java.util.Map;

import javax.validation.Valid;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.journaldev.spring.form.model.Customer;

@Controller
public class CustomerController {

    private static final Logger logger = LoggerFactory
            .getLogger(CustomerController.class);
    
    private Map<String, Customer> customers = null;
    
    public CustomerController(){
        customers = new HashMap<String, Customer>();
    }

    @RequestMapping(value = "/cust/save", method = RequestMethod.GET)
    public String saveCustomerPage(Model model) {
        logger.info("Returning custSave.jsp page");
        model.addAttribute("customer", new Customer());
        return "custSave";
    }

    @RequestMapping(value = "/cust/save.do", method = RequestMethod.POST)
    public String saveCustomerAction(@Valid Customer customer, BindingResult bindingResult, Model model) {
        if (bindingResult.hasErrors()) {
            logger.info("Returning custSave.jsp page");
            return "custSave";
        }
        logger.info("Returning custSaveSuccess.jsp page");
        model.addAttribute("customer", customer);
        customers.put(customer.getEmail(), customer);
        return "custSaveSuccess";
    }

} 
```

### 如何校验

#### 引入Maven依赖

在pom.xml中加入maven依赖， `validation-api`， `hibernate-validator`
```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>

<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.4.0.Final</version>
</dependency>

<dependency>
    <groupId>javax.el</groupId>
    <artifactId>javax.el-api</artifactId>
    <version>2.2.4</version>
</dependency>

<dependency>
    <groupId>org.glassfish.web</groupId>
    <artifactId>javax.el</artifactId>
    <version>2.2.4</version>
</dependency>
```

#### Model上加JSR-303注解类型

JSR-303定义的校验类型：

| 注解 | 描述 |
| ------------- |:-------------:|
| *空检查* | |
| @Null      | 验证对象是否为null |
| @NotNull | 验证对象是否不为null, 无法查检长度为0的字符串    |
| @NotBlank  | 检查约束字符串是不是Null还有被trim的长度是否大于0,只对字符串,且会去掉前后空格 |
| @NotEmpty | 检查约束元素是否为NULL或者是EMPTY |
| *Booelan检查* | |
| @AssertTrue | 验证 Boolean 对象是否为 true  |
| @AssertFalse | 验证 Boolean 对象是否为 false |
| *长度检查* ||
|@Size(min=, max=) |验证对象（Array,Collection,Map,String）长度是否在给定的范围之内  |
|@Length(min=, max=) |Validates that the annotated string is between min and max included|
| *日期检查* ||
|@Past |   验证 Date 和 Calendar 对象是否在当前时间之前 |
|@Future | 验证 Date 和 Calendar 对象是否在当前时间之后 |
|@Pattern |  验证 String 对象是否符合正则表达式的规则|
| *数值检查*  | 建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为“”时无法转换为int，但可以转换为Stirng为"",Integer为null|
|@Min| 验证 Number 和 String 对象是否大等于指定的值|
|@Max| 验证 Number 和 String 对象是否小等于指定的值|
|@DecimalMax | 被标注的值必须不大于约束中指定的最大值.这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度|
|@DecimalMin |被标注的值必须不小于约束中指定的最小值. 这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度|
|@Digits |  验证 Number 和 String 的构成是否合法 |
|@Digits(integer=,fraction=) | 验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度|
|@Range(min=, max=) |检查数字是否介于min和max之间.|
|@Range(min=10000,max=50000,message="range.bean.wage")||
|@Valid | 递归的对关联对象进行校验, 如果关联对象是个集合或者数组,那么对其中的元素进行递归校验,如果是一个map,则对其中的值部分进行校验.(是否进行递归验证)|
|@CreditCardNumber| 信用卡验证|
|@Email| 验证是否是邮件地址，如果为null,不进行验证，算通过验证|
|@ScriptAssert(lang= ,script=, alias=)| |
|@URL(protocol=,host=, port=,regexp=, flags=)| |


自定义校验，车架号检验

![RestExceptionHandler](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2017/customAnnotiation.png)

```java
/**
 * @Description 车架号注解
 * @Author daobin<wdb@winbaoxian.com>
 * @Date 2017/3/24.
 */
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
@Retention(RUNTIME)
@Documented
@Constraint(validatedBy = { VinNoValidator.class })
public @interface VinNo {
    String message() default "车架号不合法「不能包含I、O、Q，第十位不能是U、Z、数字0」";

    /**「
     * @return the regular expression to match
     */
    String regexp() default "^[A-HJ-NP-PR-Z0-9]{9}[A-HJ-NP-PR-TV-Y1-9]{1}[A-HJ-NP-PR-Z0-9]{7}$";
    Class<?>[] groups() default { };

    Class<? extends Payload>[] payload() default { };

    /**
     * Defines several {@link VinNo} annotations on the same element.
     *
     * @see VinNo
     */
    @Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER })
    @Retention(RUNTIME)
    @Documented
    @interface List {
        VinNo[] value();
    }
}
```

```java
/**
 * @Description 车架号验证类
 * @Author daobin<wdb@winbaoxian.com>
 * @Date 2017/3/24.
 */
public class VinNoValidator implements ConstraintValidator<VinNo, String> {

    private String regexp;

    @Override
    public void initialize(VinNo vinNo) {
        this.regexp = vinNo.regexp();
    }

    @Override
    public boolean isValid(String s, ConstraintValidatorContext constraintValidatorContext) {
        if (s == null) {
            return true;
        }

        if (s.matches(regexp)) {
            return true;
        }
        return false;
    }
}
```

Model上的注解

```java
/**
 * 车险公司、供应商
 */
@JsonSerialize(include= JsonSerialize.Inclusion.NON_NULL)
public class VehicleInfo {

    // 城市代码 国标
    @NotNull(message = "{NotNull.cityCode}", groups = {Group110.class})
    private Long cityCode;

    // 机构代码
    private String comCode;

    // 车牌号
    @LicenseNo(groups = {Group110.class})
    private String licenseNo;

    // 上牌标记 1=已上牌，0=未上牌
    @ZeroOrOne(groups = {Group110.class})
    @NotNull(message = "{NotNull.noLicenseFlag}", groups = {Group110.class} )
    private Integer noLicenseFlag;

    // 是否新车, 1-是；0-不是
    private Integer vehicleNewFlag;

    // 发票价 置购价
    @DecimalMin(value= "0", groups = {Group110.class})
    private String vehiclePrice;

    // 购车发票开具日期
    private Date buyInvoiceDate;

    // 新车购置价含税
    @DecimalMin(value= "0", groups = {Group110.class})
    private Double vehiclePriceTax;

    // 车辆实际价值 通过起保时间、注册登记时间、新车购置价算出
    private String vehicleActualPrice;

    // 车架号
    @VinNo(groups = {GroupDefault.class, Group110.class})
    private String vehicleFrameNo;

    // 发动机号
    @EngineNo(groups = {GroupDefault.class, Group110.class})
    private String engineNo;

    // 车辆使用性质
    private String carProperty;

    // 初登日期
    private Date firstRegisterDate;

    // 是否过户车 1=是 0=否
    @ZeroOrOne(groups = {Group110.class})
    @NotNull(message = "{NotNull.transferFlag}", groups = {Group110.class} )
    private Integer transferFlag;

    // 过户日期 过户车必传
    private Date transferDate;

    // 车型名称 如：丰田GTM6480GSL多用途乘用车
    private String vehicleModelName;

    // 品牌名称
    private String vehicleBrandName;

    //  车系名称
    private String vehicleFamily;

    // 车型描述
    private String vehicleDesc;

    // 车型Id
    private String vehicleId;

    // 行业车型编码
    private String tradeVehicleId;

    // 公告号
    private String noticeNo;

    // setter getter method

}
```


applicationContext.xml文件中加入如下配置：

```xml
<!--校验器 ，LocalValidatorFactoryBean是spring提供的一个校验接口-->
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <!-- hibernate校验器 -->
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
    <!-- 指定校验使用的资源文件，在文件中配置校验错误信息，如果不指定则默认使用classpath下的ValidationMessages.properties -->
    <property name="validationMessageSource" ref="messageSource" />
</bean>
<!-- 校验错误信息配置文件 -->
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <!-- 资源文件名 -->
    <property name="basename" value="classpath:ValidationMessages"/>
    <property name="useCodeAsDefaultMessage" value="true" />
    <!-- 资源文件编码格式 -->
    <property name="fileEncodings" value="utf-8" />
</bean>
<bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor"/>
```

配置ValidationMessages.properties文件，文件名一定要是ValidationMessages，使用其他名字不能读取到，也不知是何原因

![RestExceptionHandler](https://raw.githubusercontent.com/BeanHaHa/BeanHaHa.github.io/master/assets/images/2017/ValidationMessages.png)

```txt
NotNull.insureComCode=供应商代码不能为空
Size.transId=transId长度不能超过{max}
NotEmpty.transId=transId不允许空
NotNull.transId=transId不允许NULL
NotEmpty.partnerCode=partnerCode不允许空
NotNull.partnerCode=partnerCode不允许NULL
NotEmpty.operationCode=operationCode不允许空
NotNull.operationCode=operationCode不允许空
NotNull.cityCode=城市代码cityCode不允许空
NotNull.transferFlag=transferFlag不允许空
NotNull.provCode=provCode不允许空
NotNull.noLicenseFlag=noLicenseFlag不允许空
```



## 加入@Validated，就大工搞成了

```java
/**
 * @Description 车险标准接口Controller
 * @Author daobin<wdb@winbaoxian.com>
 * @Date 2017/6/26.
 */
@RestController
@RequestMapping("carInsure")
public class StandardCarController {

    private static Logger logger = LoggerFactory.getLogger(StandardCarController.class);


    @Autowired
    protected MongoTemplate mongoTemplate;

    @Autowired
    protected InsuranceRecordService insuranceRecordService;

    @Autowired
    private InsuranceSvcBeanRoutingService routeService;


    /**
     * 车险标准入口
     *
     * @param params
     * @param partnerCode
     * @return
     */
    @RequestMapping(value = "autoQuote", method = RequestMethod.POST)
    @ResponseBody
    public RestResponse<CarMultiRespWrapper> autoQuote(@RequestParam String params, @RequestParam String partnerCode) throws Exception {

        String prodCode = CarConstants.CAR_PROD_CODE;

        CarRequestBean request = JSON.parseObject(JsonLibUtil.removeUndefinedKey(params), CarRequestBean.class);

        Integer operationCode = request.getChannelInfo().getOperationCode();
        String serialNo = SerialNoUtil.createInsuranceSerialNo(partnerCode);

        CarMultiInsureService service = InsureServiceFactory.INSTANCE.getCarMultiInsureService(prodCode, routeService.getInsuranceSvcBeanGroupId(prodCode, operationCode), serialNo);

        if (service == null) {
            throw new InsureCustomException(CarConstants.RET_MSG_INVALID_PRODCODE);
        }

        // 投保前校验
        ParameterValidateResult parameterValidateResult = service.verifyParams(request);
        if (!parameterValidateResult.isValidate()) {
            throw new InsureCustomException(parameterValidateResult.getResultMsg());
        }

        // 初始化投保上下文
        InsuranceContext insuranceContext = InsuranceContextFactory.getCarMultiPriceContext(partnerCode, serialNo, operationCode, prodCode, params, new Date());
        // 车险投保
        CarMultiRespWrapper carMultiRespWrapper = service.doInsure(request, insuranceContext);

        if (insuranceContext != null) {
            insuranceContext.getInsuranceAnalyzer().setRequestEndTime(new Date());
            this.insuranceRecordService.saveInsureDatagram(insuranceContext);
        }
        return RestFulUtil.toCarSuccessResponse(carMultiRespWrapper);
    }
}
```

```java
@Validated
public interface CarMultiInsureService {

    ParameterValidateResult verifyParams(CarRequestBean requestBean);

    @Validated({GroupDefault.class})
    CarMultiRespWrapper doInsure(@Valid CarRequestBean requestBean, InsuranceContext insuranceContext);
}
```

#### 方便处理校验不通过异常`ConstraintViolation`，写了个RestFulUtil类处理

```java
/**
 * @Description
 * @Author daobin<wdb@winbaoxian.com>
 * @Date 2017/7/5.
 */
public class RestFulUtil {
    public static <T> RestResponse<T> toCarSuccessResponse(CarMultiRespWrapper carMultiRespWrapper) {
        RestResponse.Builder builder = new RestResponse.Builder();
        builder.setStatus(HttpStatus.OK.value());
        builder.setCode(Constants.CAR_INSURE_SUCCEED.intValue());
        builder.setMessage(CarConstants.CAR_MSG_INSURE_SUCCESS);
        builder.setMessageDetail("");

        if (carMultiRespWrapper != null) {
            builder.setData(carMultiRespWrapper);
        }
        return builder.build();
    }

    public static String getConstraintViolationExMsg(ConstraintViolationException ex) {
        List<FieldError> errors = FieldError.getErrors(ex.getConstraintViolations());
        String errorMsg = "";
        for (FieldError error : errors) {
            errorMsg += error.getMessage() + ";";
        }
        return errorMsg;
    }
}
```

```java
/**
 * @Description
 * @Author daobin<wdb@winbaoxian.com>
 * @Date 2017/3/23.
 */
public class FieldError {
    private String field;
    private String code;
    private String message;

    // Getter Setter Method

    public static List<FieldError> getErrors(Set<ConstraintViolation<?>> constraintViolations) {
        List<FieldError> list = new ArrayList<>();
        for(ConstraintViolation<?> constraintViolation : constraintViolations) {
            list.add(of(constraintViolation));
        }
        return list;
    }

    private static FieldError of(ConstraintViolation<?> constraintViolation) {
        String field = StringUtils.substringAfter(constraintViolation.getPropertyPath().toString(), ".");
        return new FieldError(field, constraintViolation.getMessageTemplate(), constraintViolation.getMessage());
    }
}

```

至此，Spring Validation 搞定，增减前置参数校验 只需要在Model加减注解即可。

## 参考

[Spring参考手册 3 校验，数据绑定和类型转换](http://www.jianshu.com/p/cbaee928a2f7)

[Spring Validation Example – Spring MVC Form Validator](http://www.journaldev.com/2668/spring-validation-example-mvc-validator)

[Spring @Validated in service layer](https://stackoverflow.com/questions/19425221/spring-validated-in-service-layer)

[Spring 3 MVC and JSR303 @Valid example](https://www.mkyong.com/spring-mvc/spring-3-mvc-and-jsr303-valid-example/)

[Spring Framework Reference Documentation](http://docs.spring.io/spring/docs/4.1.2.RELEASE/spring-framework-reference/htmlsingle/#validator)