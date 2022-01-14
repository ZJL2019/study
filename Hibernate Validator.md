# Hibernate Validator使用介绍

1. Hibernate Validator 简介

   - Hibernate Validator可以将业务代码和校验逻辑分开
   - Hibernate Validator是Bean Validation的参考实现，提供了JSR 303规范中所有内置constraint的实现，还有附加constraint

2. Hibernate Validator的作用

   - 验证逻辑与业务逻辑质检进行了分离，降低了程序耦合度
   - 统一且规范的验证方式，无需重复编写验证代码

3. Hibernate Validator的使用

   1. 引入jar包

      ```xml
      <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.13.Final</version>
      </dependency>
      ```

   2. Java对象添加约束注解

      ```java
      @Data
      @AllArgsConstructor
      @NoArgsConstructor
      public class Account {
          private String id;
          @NotNull
          @Length(max=20)
          private String userName;
          @NotNull
          @Pattern(regexp = "[A-Z][a-z][0-9]")
          private String passWord;
          @DateTimeFormat(pattern = "yyyy-MM-dd")
          private Date cts;
          private String alias;
          @Max(10)
          @Min(1)
          private Integer level;
          private Integer vip;
      }
      ```

      ```java
      @PostMapping("/saveAccount")
          public WebResponse<Account> saveAccount(@RequestBody @Valid Account account){
              return WebResponse.buildData(account);
          }
      ```

      postman报文：

      ```json
      {
          "alias": "kalakala",
          "userName": "wokalakala"
      }
      ```

      响应报文：（校验结果）

      ```json
      {
          "timestamp": "2022-01-05T08:49:25.137+0000",
          "status": 400,
          "error": "Bad Request",
          "errors": [
              {
                  "codes": [
                      "NotNull.account.passWord",
                      "NotNull.passWord",
                      "NotNull.java.lang.String",
                      "NotNull"
                  ],
                  "arguments": [
                      {
                          "codes": [
                              "account.passWord",
                              "passWord"
                          ],
                          "arguments": null,
                          "defaultMessage": "passWord",
                          "code": "passWord"
                      }
                  ],
                  "defaultMessage": "不能为null",
                  "objectName": "account",
                  "field": "passWord",
                  "rejectedValue": null,
                  "bindingFailure": false,
                  "code": "NotNull"
              }
          ],
          "message": "Bad Request",
          "path": "/test/saveAccount"
      }
      ```

4. 封装工具类代码中校验

   1. validatorUtil工具类

      ```java
      public class ValidationUtil {
      
          private ValidationUtil() {
          }
      
          /**
           * 开启快速结束模式 failFast (true)
           */
          private static Validator failFastValidator = Validation.byProvider(HibernateValidator.class)
                  .configure()
                  .failFast(true)
                  .buildValidatorFactory()
                  .getValidator();
      
          /**
           * 全部校验
           */
          private static Validator validator = Validation.buildDefaultValidatorFactory().getValidator();
      
      
          /**
           * 注解验证参数(快速失败模式)
           *
           * @param o
           */
          public static <T> Violation fastFailValidate(T o) {
              List<Violation> list = failFastValidator.validate(o).stream()
                      .map(e -> new Violation(e.getPropertyPath().toString(), e.getInvalidValue(), e.getMessage()))
                      .collect(Collectors.toList());
              return CollectionUtils.isNotEmpty(list) ? list.get(0) : null;
          }
      
          /**
           * 注解验证参数(全部校验)
           *
           * @param o
           */
          public static <T> List<Violation> allCheckValidate(T o) {
              return validator.validate(o).stream()
                      .map(e -> new Violation(e.getPropertyPath().toString(), e.getInvalidValue(), e.getMessage()))
                      .collect(Collectors.toList());
          }
      
          @Data
          @AllArgsConstructor
          public static class Violation {
              String property;
              Object invalidValue;
              String msg;
          }
      }
      
      ```

      通过Validation.byProvider(HibernateValidator.class).configure()选择快速校验还是全部校验。

      然后通过validate，获取对应字段信息，如果校验不过，validate则会保存对应的属性信息，value信息，错误信息等。

      返回类型为<T> Set<ConstraintViolation<T>>。最终在业务代码中只需要调用校验方法，并判断其返回值即可。

      ```java
      package com.meituan.mdp.sample.controller;
      
      import java.util.ArrayList;
      import java.util.List;
      import java.util.Set;
      import javax.validation.ConstraintViolation;
      import javax.validation.Validation;
      import javax.validation.Validator;
      
      import lombok.Data;
      import org.hibernate.validator.HibernateValidator;
      public class ValidationUtil {
          /**
           * 开启快速结束模式 failFast (true)
           */
          private static Validator validator = Validation.byProvider(HibernateValidator.class).configure().failFast(false).buildValidatorFactory().getValidator();
          /**
           * 校验对象
           *
           * @param t bean
           * @param groups 校验组
           * @return ValidResult
           */
          public static <T> ValidResult validateBean(T t,Class<?>...groups) {
              ValidResult result = new ValidationUtil().new ValidResult();
              Set<ConstraintViolation<T>> violationSet = validator.validate(t,groups);
              boolean hasError = violationSet != null && violationSet.size() > 0;
              result.setHasErrors(hasError);
              if (hasError) {
                  for (ConstraintViolation<T> violation : violationSet) {
                      result.addError(violation.getPropertyPath().toString(), violation.getMessage());
                  }
              }
              return result;
          }
          /**
           * 校验bean的某一个属性
           *
           * @param obj          bean
           * @param propertyName 属性名称
           * @return ValidResult
           */
          public static <T> ValidResult validateProperty(T obj, String propertyName) {
              ValidResult result = new ValidationUtil().new ValidResult();
              Set<ConstraintViolation<T>> violationSet = validator.validateProperty(obj, propertyName);
              boolean hasError = violationSet != null && violationSet.size() > 0;
              result.setHasErrors(hasError);
              if (hasError) {
                  for (ConstraintViolation<T> violation : violationSet) {
                      result.addError(propertyName, violation.getMessage());
                  }
              }
              return result;
          }
          /**
           * 校验结果类
           */
          @Data
          public class ValidResult {
      
              /**
               * 是否有错误
               */
              private boolean hasErrors;
      
              /**
               * 错误信息
               */
              private List<ErrorMessage> errors;
      
              public ValidResult() {
                  this.errors = new ArrayList<>();
              }
              public boolean hasErrors() {
                  return hasErrors;
              }
      
              public void setHasErrors(boolean hasErrors) {
                  this.hasErrors = hasErrors;
              }
      
              /**
               * 获取所有验证信息
               * @return 集合形式
               */
              public List<ErrorMessage> getAllErrors() {
                  return errors;
              }
              /**
               * 获取所有验证信息
               * @return 字符串形式
               */
              public String getErrors(){
                  StringBuilder sb = new StringBuilder();
                  for (ErrorMessage error : errors) {
                      sb.append(error.getPropertyPath()).append(":").append(error.getMessage()).append(" ");
                  }
                  return sb.toString();
              }
      
              public void addError(String propertyName, String message) {
                  this.errors.add(new ErrorMessage(propertyName, message));
              }
          }
      
          @Data
          public class ErrorMessage {
      
              private String propertyPath;
      
              private String message;
      
              public ErrorMessage() {
              }
      
              public ErrorMessage(String propertyPath, String message) {
                  this.propertyPath = propertyPath;
                  this.message = message;
              }
          }
      
      }
      
      ```

      另一种validatorUtil工具类，其本质大同小异