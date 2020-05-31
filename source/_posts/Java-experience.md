---
layout: post
title: Java 自己开发经验总结
Author: tzcqupt
tags: [ 经验总结]
categories: [经验总结]
comments: true
toc: true
date: 2020-05-30 00:00:00
---

# MyBatis

## #和$的用法

1. 当传递的参数作为sql中的变量是,使用#

2. 当传递的参数作为sql的特殊字段时,使用$接收.如接收该参数作为列名,排序字段asc/desc

   > 如传递的参数有order和sort 其中order表示排序依据的字段,sort传递的时asc/desc表示是升序还是降序
   >
   > ```java
   > //mapper接口
   > List<SysMenu> findMenuByPage(QueryDTO queryDTO);
   > // 实体类
   > @Data
   > public class QueryDTO {
   >   private Integer offset;
   >   private Integer limit;
   >   private String order;
   >   /** 排序的字段 */
   >   private String sort;
   >   /** 搜索 */
   >   private String search;
   > }
   > ```
   >
   > ```xml
   > <select id="findMenuByPage" resultType="com.tang.car.entity.SysMenu" parameterType="com.tang.car.dto.QueryDTO">
   >         select m.*,p.name as parentName from sys_menu m
   >         ,sys_menu p
   >         <trim prefix="where" prefixOverrides="and">
   >             <if test="search!=null and search!='' ">
   >                 and m.name like concat('%',#{search},'%')
   >             </if>
   >             and m.menu_id=p.menu_id
   >         </trim>
   >         <if test="sort!=null and sort!='' ">
   >             order by #{sort}
   >         </if>
   >     </select>
   > ```

## 配置分页插件

### 依赖

```xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
<!--pagerhelper 的 springboot 配置方式-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>

<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-autoconfigure</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.1.2</version>
</dependency>
```

### 使用

```java
public DataGridResult findByPage(QueryDTO queryDTO) {
    // 使用分页插件
    PageHelper.offsetPage(queryDTO.getOffset(), queryDTO.getLimit());
    CarExample example = new CarExample();
    String sort = queryDTO.getSort();
    if (!StringUtils.isEmpty(sort)) {
        example.setOrderByClause("id " + queryDTO.getOrder());
    }
    List<Car> cars = carMapper.selectByExample(example);
    PageInfo<Car> info = new PageInfo<>(cars);
    long total = info.getTotal();
    return new DataGridResult(total, cars);
}
```

# SpringMVC

## 数据验证部分

### 配置文件

~~~xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.9.Final</version>
</dependency>
~~~

### @Valid

#### 实体类注解验证

~~~java
 @Data
public class Transaction {
	// 产品编号
	@NotNull // 不能为空
	private Long productId;

	// 用户编号
	@NotNull // 不能为空
	private Long userId;

	// 交易日期
	@Future // 只能是将来的日期
	@DateTimeFormat(pattern = "yyyy-MM-dd") // 日期格式化转换
	@NotNull // 不能为空
	private Date date;

	// 价格
	@NotNull // 不能为空
	@DecimalMin(value = "0.1") // 最小值0.1元
	private Double price;

	// 数量
	@Min(1) // 最小值为1
	@Max(100) // 最大值
	@NotNull // 不能为空
	private Integer quantity;

	// 交易金额
	@NotNull // 不能为空
	@DecimalMax("500000.00") // 最大金额为5万元
	@DecimalMin("1.00") // 最小交易金额1元
	private Double amount;

	// 邮件
	@Pattern(// 正则式
			regexp = "^([a-zA-Z0-9]*[-_]?[a-zA-Z0-9]+)*@"
					+ "([a-zA-Z0-9]*[-_]?[a-zA-Z0-9]+)+[\\.][A-Za-z]{2,3}([\\.][A-Za-z]{2})?$",
			// 自定义消息提示
			message = "不符合邮件格式")
    //javax.validation.constraints.@Email(message = "邮箱格式错误")
	private String email;

	// 备注
	@Size(min = 0, max = 256) // 0到255个字符
	private String note;

~~~

#### 其他业务验证

~~~java
import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

public class TransactionValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        //判断验证是否为Transaction，如果是则进行验证
        return Transaction.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Transaction trans = (Transaction) target;
        //求交易金额和价格×数量的差额
        double dis = trans.getAmount() - (trans.getPrice() * trans.getQuantity());
        //如果差额大于0.01，则认为业务错误
        if (Math.abs(dis) > 0.01) {
            //加入错误信息
            errors.rejectValue("amount", null, "交易金额和购买数量与价格不匹配");
        }
    }
}
~~~

#### 绑定器@InitBinder

~~~java
@RestController
@RequestMapping("/validate")
@Slf4j
public class ValidateController {
    
	@InitBinder
	public void initBinder(DataBinder binder) {
		// 数据绑定器加入验证器
		binder.setValidator(new TransactionValidator());
	}
}
~~~

#### 验证方法

~~~java
@RequestMapping("/validator")
	public String validator(@Valid Transaction trans, Errors errors) {
        Map<String, Object> errMap = new HashMap<>(16);
		// 获取错误列表
        List<ObjectError> oes = errors.getAllErrors();
        if (oes.size() > 0) {
            for (ObjectError oe : oes) {
                String key = null;
                String msg = null;
                // 字段错误
                if (oe instanceof FieldError) {
                    FieldError fe = (FieldError) oe;
                    key = fe.getField();// 获取错误验证字段名
                } else {
                    // 非字段错误
                    key = oe.getObjectName();// 获取验证对象名称
                }
                // 错误信息
                msg = oe.getDefaultMessage();
                errMap.put(key, msg);
            }

        } else {
            errMap.put("success", trans.toString());
        }
        return errMap;
    }
~~~

### @Validated

> **加入组别验证**

#### 实体类注解

~~~java
@Data
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class BindPojo {
    //inferface : 只是作为标记一个组别 可以在vo验证的某个字段上面加入多个组别,这样没有加入的组别就不会验证这个字段
    @NotNull(message = "can not be null", groups = {groupKey.class})
    private String key;
    @NotNull(message = "can not be null")
    private String desc;
    private Date startDate;
    private Date endDate;

    public interface groupKey {
    }
}
~~~

#### 绑定器@InitBinder

~~~java
// 如果没有业务验证，在实体类中没有验证时间，可以加在绑定器中
 	/**
     * 时间类型转换器
     */
    @InitBinder
    public void initBinder(ServletRequestDataBinder binder) {
        binder.registerCustomEditor(Date.class,
                new CustomDateEditor(
                        new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"), true));

    }
~~~

#### 验证方法

~~~java
@GetMapping("/testKey")
    public String testBind(@Validated({BindPojo.groupKey.class}) BindPojo bindPojo,
                           BindingResult result) {

        /**
         *   实体类中虽然标记了desc不能为null，但是验证的是加入了groupKey的这个组别。所以desc为null是可以提交的
         *  http://localhost:8080/SpringMVCLearn/testKey?key=key&desc=desc&startDate=2020-04-17 16:17:17&endDate=2020-04-17 16:17:17
         *  http://localhost:8080/SpringMVCLearn/testKey?key=key&startDate=2020-04-17 16:17:17&endDate=2020-04-17 16:17:17
         *  http://localhost:8080/SpringMVCLearn/testKey?desc=desc&startDate=2020-04-17 16:17:17&endDate=2020-04-17 16:17:17
         *  因为initBinder绑定器绑定了校验时间的要求，所以这样时间会校验不通过
         *  http://localhost:8080/SpringMVCLearn/testKey?key=key&desc=desc&startDate=2020-047 16:17:17&endDate=2020-04-17 16:17:17
         */

        try {
            //数据校验
            if (result.hasErrors()) {
                Map<String, String> errorMsg = new HashMap<>(16);
                for (FieldError fieldError : result.getFieldErrors()) {
                    errorMsg.put(fieldError.getObjectName() + "." + fieldError.getField(),
                            fieldError.getDefaultMessage());
                }
                return errorMsg.toString();
            } else {
                return bindPojo.toString() + "success";
            }
        } catch (Exception e) {
            return e.getMessage();
        }

    }
~~~


# Spring

## SpringAOP

### 利用自定义注解和AOP实现日志记录

#### 声明自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyLog {
    String value() default "用户操作";
}
```

#### 编写AOP切面

```java
@Aspect
@Component
@Slf4j
public class MyLogAdvice {
  @Pointcut("@annotation(com.tang.car.log.MyLog)")
  public void myPointcut() {
  }

  /** 开发通知 */
  @AfterReturning(pointcut = "myPointcut()")
  public void myAfterRet(JoinPoint joinPoint) {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    MyLog annotation = method.getAnnotation(MyLog.class);
    // 得到用户的操作
    String operation = null;
    if (Objects.nonNull(annotation)) {
      operation = annotation.value();
    }
    // 得到用户的ip
    String ip = IPUtils.getIpAddr(HttpContextUtils.getHttpServletRequest());
    // 得到参数
    Object[] args = joinPoint.getArgs();
    String jsonString = JSON.toJSONString(args);
    // 得到方法名称
    String methodName = joinPoint.getTarget().getClass().getName() + "." + method.getName();
    log.info(Instant.now() + "|" + operation + "|" + ip + "|" + methodName + "|" + jsonString);
  }
}
```

#### 使用

```java
@MyLog("菜单更新")
@PostMapping("update")
public MyCarResult updateMenu(@RequestBody SysMenu menu) {
  int result = sysMenuService.update(menu);
  if (result > 0) {
    return MyCarResult.ok("修改成功!");
  }
  return MyCarResult.error();
}
```

# 数据库

## SpringBoot集成Druid监控SQL

### 依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.8.RELEASE</version>
    <relativePath/>
</parent>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.28</version>
</dependency>
```

### 配置文件

```properties
#DataSource配置
spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=20
spring.datasource.maxWait=60000
spring.datasource.timeBetweenEvictionRunsMillis=60000
spring.datasource.minEvictableIdleTimeMillis=300000
spring.datasource.validationQuery=SELECT 1 FROM DUAL
spring.datasource.testWhileIdle=true
spring.datasource.testOnBorrow=false
spring.datasource.testOnReturn=false
spring.datasource.poolPreparedStatements=true
spring.datasource.maxPoolPreparedStatementPerConnectionSize=20
spring.datasource.filters=stat,wall
```

### 其他初始化配置

```java
@Configuration //spring的配置文件
public class DruidConfig {
    @Bean(name = "dataSource")
    //加载properties文件，以spring.datasource开头的部分构建数据源
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        DataSource dataSource = new DruidDataSource();
        return dataSource;
    }
}
// 访问方式localhost:8080/druid
// 用户名/密码都是druid
@WebServlet(name = "druidStatViewServlet", urlPatterns = "/druid/*",
        initParams = {
                @WebInitParam(name = "loginUsername", value = "druid"),
                @WebInitParam(name = "loginPassword", value = "druid"),
//      @WebInitParam(name = "allow",value = "127.0.0.1"),// IP白名单 (没有配置或者为空，则允许所有访问)
                @WebInitParam(name = "resetEnable", value = "false")
        })
public class DruidStatViewServlet extends StatViewServlet {
}

@WebFilter(filterName = "druidWebStatFilter", urlPatterns = "/*",
        initParams = {@WebInitParam(name = "exclusions", value = "*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*")})
//exclusions 不拦截资源
public class DruidWebStatFilter extends WebStatFilter {
}
```

> 因为加了@WebServlet注解,需要被SpringBoot扫描到,所以在SpringBoot启动类上需要加注解@ServletComponentScan(basePackages = "com.tang.car.config")