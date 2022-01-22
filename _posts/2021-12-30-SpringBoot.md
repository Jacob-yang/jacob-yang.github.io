# SpringBoot做非空验证

## 1.导入spring-boot-starter-validation包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
    <version>2.0.7.RELEASE</version>
    <scope>compile</scope>
</dependency>
```

## 2.创建全局异常

>目前测试的异常抛出类型为MethodArgumentNotValidException 异常，但是看其余资料说有另外两种异常就一并捕获了。

```java
import com.djfy.djhik.controller.entity.ResponseEntity;
import org.springframework.validation.BindException;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import java.util.List;
import java.util.Set;

/**
 * @ClassName: GlobalExceptionResolver
 * @Description: TODO
 * @Author: JacobYang
 * @Date: 2021/11/26 09:27
 * @Version: 1.0
 */
@ControllerAdvice
public class GlobalExceptionResolver {

    // @RestControllerAdvice

    /*  数据校验处理  这里返参可以自定义成系统统一的返参*/
    @ExceptionHandler({ MethodArgumentNotValidException.class})
    @ResponseBody
    public ResponseEntity validatorExceptionHandler(MethodArgumentNotValidException e) {
        StringBuilder errorMessage = new StringBuilder();
        if (e instanceof MethodArgumentNotValidException) {
            BindingResult bindingResult = ((MethodArgumentNotValidException) e).getBindingResult();
            List<FieldError> errors = bindingResult.getFieldErrors();
            errors.forEach(error ->
                    errorMessage.append(error.getDefaultMessage()).append(";")
            );
        }
        return ResponseEntity.buildError(String.valueOf(errorMessage));
    }

    @ExceptionHandler({ BindException.class})
    @ResponseBody
    public ResponseEntity validatorExceptionHandler(BindException e) {
        StringBuilder errorMessage = new StringBuilder();
        if (e instanceof BindException) {
            BindingResult bindingResult = ((BindException) e).getBindingResult();
            List<FieldError> errors = bindingResult.getFieldErrors();
            errors.forEach(error ->
                    errorMessage.append(error.getDefaultMessage()).append(";")
            );
        }
        return ResponseEntity.buildError(String.valueOf(errorMessage));
    }

    @ExceptionHandler({ ConstraintViolationException.class})
    @ResponseBody
    public ResponseEntity validatorExceptionHandler(ConstraintViolationException e) {
        return ResponseEntity.buildError((msgConvertor((e).getConstraintViolations())));
    }


    /**
     * 校验消息转换拼接
     *
     * @param bindingResult
     * @return
     */
    public static StringBuilder msgConvertor(BindingResult bindingResult) {
        List<FieldError> fieldErrors = bindingResult.getFieldErrors();
        StringBuilder sb = new StringBuilder();
        fieldErrors.forEach(fieldError -> sb.append(fieldError.getDefaultMessage()).append(","));

        return sb.deleteCharAt(sb.length() - 1);
    }

    private String msgConvertor(Set<ConstraintViolation<?>> constraintViolations) {
        StringBuilder sb = new StringBuilder();
        constraintViolations.forEach(violation -> sb.append(violation.getMessage()).append(","));

        return sb.deleteCharAt(sb.length() - 1).toString().toLowerCase();
    }

}
```

## 3.添加验证注解

>在要验证的controller 参数前面添加 `@Valid`注解,如果要验证的对象里面包含List或其余的javaBean
>继续在上面添加valid注解，才能够对javaBean中属性进行验证，valid 是具有传递性的。

- controller添加注解

```java
public Object addHouseNode(@RequestBody @Valid AddHouseNodesVo vo)
```

- AddHouseNodesVo里面的list类型添加注解

```java
@Setter
@Getter
@ApiModel("添加")
public class AddHouseNodesVo {
    @Valid
    @NotNull(message = "小区编号不能为空")
    @ApiModelProperty(value = "小区编号",required =true,example ="233")
    private String rqIndexCode;
    @Valid
    @NotNull(message = "节点不能为空")
    private ArrayList<NodesVo> nodes;
}
```

- 对List泛型的一个类具体属性进行验证

```java
@Setter
@Getter
public class NodesVo {

    @ApiModelProperty(value = "添加节点类型1-幢，2-单元",required = true,example ="2")
    @NotNull(message = "节点类型不能为空")
    private Integer nodeType;

    @ApiModelProperty(value = "添加节点名称",required = true,example ="一单元")
    @NotNull(message = "节点名称不能为空")
    private String nodeName;

    @ApiModelProperty(value = "父节点id",required = true,example ="2")
    @NotNull(message = "父节点id不能为空")
    private String parentId;

    @ApiModelProperty(value = "父节点类型 1-幢，0-小区",required = true,example ="1")
    @NotNull(message = "父节点类型不能为空")
    private Integer parentType;
}
```

## 4.常用注解

@NotNull 注解元素必须是非空

@Null 注解元素必须是空

@Digits 验证数字构成是否合法

@Future 验证是否在当前系统时间之后

@Past 验证是否在当前系统时间之前

@Max 验证值是否小于等于最大指定整数值

@Min 验证值是否大于等于最小指定整数值

@Pattern 验证字符串是否匹配指定的正则表达式

@Size 验证元素大小是否在指定范围内

@DecimalMax 验证值是否小于等于最大指定小数值

@DecimalMin 验证值是否大于等于最小指定小数值

@AssertTrue 被注释的元素必须为true

@AssertFalse 被注释的元素必须为false

@Email 被注释的元素必须是电子邮箱地址

@Length 被注释的字符串的大小必须在指定的范围内

@NotEmpty 被注释的字符串的必须非空

@Range 被注释的元素必须在合适的范围内

## 5.扩展@Valid和@Validated 区别

Spring Validation验证框架对参数的验证机制提供了@Validated（Spring’s JSR-303规范，是标准JSR-303的一个变种），javax提供了@Valid（标准JSR-303规范），配合BindingResult可以直接提供参数验证结果。

在检验Controller的入参是否符合规范时，使用@Validated或者@Valid在基本验证功能上没有太多区别。但是在分组、注解地方、嵌套验证等功能上两个有所不同：

### 1. 分组：

```
@Validated：提供了一个分组功能，可以在入参验证时，根据不同的分组采用不同的验证机制

@Valid：作为标准JSR-303规范，还没有吸收分组的功能。
```

### 2.注解位置

- @Validated：可以用在类、方法和方法参数上。
- @Valid：可以用在方法、构造函数、方法参数和成员属性（字段）上

### 3.嵌套验证：

- 嵌套验证就是类嵌套类的验证，比如我要在集合上加一个@notnull的注解，要求该集合中的每一个对象都被验证，如果只用@Validated是不会验证的。我们要用@Validated配合@Valid来进行验证,或者使用@Valid来验证



原文地址：https://blog.csdn.net/dyy0213/article/details/107459668