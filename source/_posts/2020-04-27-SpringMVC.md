layout: post
title: SpringMVC学习笔记
Author: tzcqupt
tags:
  - Java
  - SpringMVC
comments: true
toc: true
date: 2020-04-27 00:00:00
---
# 入门案例执行过程

1. 服务器启动,应用被加载。读取到 web.xml 中的配置创建 spring 容器并且初始化容器中的对象。

2. 浏览器发送请求,被 DispatcherServlet 捕获,该 Servlet 并不处理请求,而是把请求转发出去。转发的路径是根据请求 URL,匹配@RequestMapping 中的内容。

3. 匹配到了后,执行对应方法。该方法有一个返回值。

4. 根据方法的返回值,借助 InternalResourceViewResolver 找到对应的结果视图。

5. 渲染结果视图,响应浏览器。

   ![SpringMVC执行流程](..\img\SpringMVC执行流程.PNG)
   
# 常用配置

## web.xml

~~~xml
	    <!-- 配置 spring mvc 的核心控制器 -->
	    <servlet>
	        <servlet-name>dispatcherServlet</servlet-name>
	        <servlet-class>
	            org.springframework.web.servlet.DispatcherServlet
	        </servlet-class>
	        <!-- 配置初始化参数,用于读取 SpringMVC 的配置文件 -->
	        <init-param>
	            <param-name>contextConfigLocation</param-name>
	            <param-value>classpath:springmvc.xml</param-value>
	        </init-param>
	        <!-- 配置 servlet 的对象的创建时间点：应用加载时创建。
	             取值只能是非 0 正整数,表示启动顺序 -->
	        <load-on-startup>1</load-on-startup>
	    </servlet>
	    <servlet-mapping>
	        <servlet-name>dispatcherServlet</servlet-name>
	        <url-pattern>/</url-pattern>
	    </servlet-mapping>
	    <!-- 配置过滤器,解决中文乱码的问题 -->
	    <filter>
	        <filter-name>characterEncodingFilter</filter-name>
	        <filter-class>org.springframework.web.filter.CharacterEncodingFilter
	        </filter-class>
	        <!-- 指定字符集 -->
	        <init-param>
	            <param-name>encoding</param-name>
	            <param-value>UTF-8</param-value>
	        </init-param>
	        <init-param>
	            <param-name>forceEncoding</param-name>
	            <param-value>true</param-value>
	        </init-param>
	    </filter>
	    <filter-mapping>
	        <filter-name>characterEncodingFilter</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	    <filter>
	        <filter-name>httpFilter</filter-name>
	        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
	    </filter>
	    <filter-mapping>
	        <filter-name>httpFilter</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
~~~

## springmvc.xml

  ~~~xml
    <!-- 配置创建 spring 容器要扫描的包 -->
    <context:component-scan base-package="com.tang.ssm.springmvc"/>
    <!-- 配置视图解析器 -->
    <bean id="viewResolver"
          class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!-- 配置spring开启注解mvc的支持 -->
    <mvc:annotation-driven/>
    <!--配置过滤器不过滤静态资源文件-->
    <!-- location 表示路径,mapping 表示文件,**表示该目录下的文件以及子目录的文件 -->
    <mvc:resources location="/css/" mapping="/css/**"/>
    <mvc:resources location="/images/" mapping="/images/**"/>
    <mvc:resources location="/js/" mapping="/javascript/**"/>
    <!--配置类型转换器工厂-->
    <bean id="converterService"
          class="org.springframework.context.support.ConversionServiceFactoryBean">
        <!-- 给工厂注入一个新的类型转换器 -->
        <property name="converters">
            <array>
                <!-- 配置自定义类型转换器 -->
                <bean class="com.tang.ssm.springmvc.utils.StringToDateConverter"/>
            </array>
        </property>
    </bean>
    <!-- 引用自定义类型转换器 -->
    <mvc:annotation-driven conversion-service="converterService"/>
  ~~~
# 获取控制器参数

## 无注解获取参数

参数名称 和 HTTP 请求的参数名称保持一致。

~~~java
public Map<String, Object> noAnnotation(
Integer intVal, Long longVal, String str) {}
//http://localhost:8080/my/no/annotation?intVal=10&longVal=200
~~~

## 使用@RequestParam获取参数

确定前后端参数名称的映射关系。

~~~java
public Map<String, Object> requestParam(
@RequestParam("int_val") Integer intVal,
@RequestParam("long_val") Long longVal,
@RequestParam("str_val", required = false) String strVal) { }
//http://localhost:8080/my/annotation?int_val=1&long_val=2&str_val=str
~~~

## 传递数组

需要传递数组参数时，每个参数的数组元素只需要通过逗号分隔即可。

~~~java
public Map<String, Object> requestArray(
int [] intArr, Long []longArr, String[] strArr) {}
//http://localhost:8080/my/requestArray?intArr=1,2,3&longArr=4,5,6&strArr=str1,str2,str3

~~~

## 传递JSON

前端需要将数据转为json  `JSON.stringify(data)` 后端采用@RequestBody

~~~js
var params = {
    userName : userName,
    note : note
}; 
$.post({
    url : "./insert",
    // 此处需要告知传递参数类型为 JSON，不能缺少
    contentType : "application/json",
    // 将 JSON 转化为字符串传递
    data : JSON.stringify(params),
    // 成功后的方法
    success : function(result) {
    if (result == null || result.id == null) {
    	alert("插入失败");
    return;
	}
	alert("插入成功");
};
~~~

~~~java
/**
* 打开请求页面
* @return 字符串，指向页面
 */
 @GetMapping("/add")
 public String add() {
	return "/user/add";
 }
 /**
* 新增用户
* @param user 通过@RequestBody 注解得到 JSON 参数
* @return 回填 id 后的用户信息
 */
 @PostMapping("/insert")
 @ResponseBody
 public User insert(@RequestBody User user) {
    userService.insertUser(user);
    return user;
 } 
~~~

## 通过URL传递参数

~~~java
@GetMapping("/{id}")
// 响应为 JSON 数据集
@ResponseBody
// @PathVariable 通过名称获取参数
public User get(@PathVariable("id") Long id) {
 return userService.getUser(id);
} 
//localhost:8080/user/1
~~~

## 获取格式化参数

日期参数格式化可以在配置文件中配置，`spring.mvc.date-format=yyyy-MM-dd `也可以在方法上加注解。

~~~html
<table>
    <tr>
        <td>日期（yyyy-MM-dd）</td>
        <td>
        	<input type="text" name="date" value="2017-08-08" />
        </td>
    </tr>
    <tr>
    	<td>金额（#,###.##）</td>
    	<td>
    		<input type="text" name="number" value="1,234,567.89" />
    	</td>
    </tr>
    <tr>
        <td colspan="2" align="right">
        	<input type="submit" value="提交"/>
        </td>
    </tr>
</table> 
~~~

~~~java
// 获取提交参数
@PostMapping("/format/commit")
@ResponseBody
public Map<String, Object> format(
@DateTimeFormat(iso=ISO.DATE) Date date,
@NumberFormat(pattern = "#,###.##") Double number) {
     Map<String, Object> dataMap = new HashMap<>();
     dataMap.put("date", date);
     dataMap.put("number", number);
     return dataMap;
} 
~~~





# 常用注解

Rest风格的注解，推荐使用，直接限定了提交方式
- @GetMapping
- @DeleteMapping
- @PostMapping
- @PutMapping

## @RequestParam

1. 作用：把请求中的指定名称的参数传递给控制器中的形参赋值
2. 属性
      1. value：请求参数中的名称
      2. required：请求参数中是否必须提供此参数,默认值是true,必须提供
3. 代码如下
	~~~java
	@RequestMapping(path="/hello") 
	public String sayHello(@RequestParam(value="username",required=false)String name) {
	    	//前台传递过来的参数名时username==>赋值给name变量
		  System.out.println("aaaa"); 
		  System.out.println(name); 
		  return "success"; 
		} 
	~~~

## @RequestBody
1. 作用：将前台传递的JSON数据封装为POJO，也可以封装为集合

      >1. 前台传递JSON数据需要转换为字符串 JSON.stringify()
      >2. 传递的数据要和POJO实体的参数名保持一致，便于封装

2. 属性  
      required：是否必须有请求体,默认值是true

3. 代码如下

  ~~~java
   @PostMapping(path="/addUser") 
   public ModelAndView addUser(@RequestBody User user) { 
       ModelAndView mv = new ModelAndView(} ; 
        //...
     return mv; 
   } 
  @DeleteMapping(path="/deleteUsers") 
   public ModelAndView deleteUsers(@RequestBody List<Long> idList) { 
         ModelAndView mv = new ModelAndView(} ; 
        //...
     	return mv; 
   } 
   @PostMapping(path="/addRoless") 
   public ModelAndView addRoles(@RequestBody List<Role> roleList) { 
         ModelAndView mv = new ModelAndView(} ; 
        //...
     	return mv; 
   } 
  ~~~

## @PathVariable

1. 作用：拥有绑定url中的占位符的。例如：url中有/delete/{id},{id}就是占位符
2. 属性  
    value：指定url中的占位符名称
3. Restful风格的URL
      1. 请求路径一样,可以根据不同的请求方式去执行后台的不同方法
      2. restful风格的URL优点
          1. 结构清晰
          2. 符合标准
          3. 易于理解
          4. 扩展方便
4. 代码
    ~~~java
        <a href="user/hello/1">入门案例</a>
        @RequestMapping(path="/hello/{id}") 
        public String sayHello(@PathVariable(value="id") String id) { 
            System.out.println(id); 
            return "success"; 
        } 
    ~~~
    
## ＠RequestAttribute

从HTTP的request对象中取出请求属性，范围只是在一次请求中存在。类似**@RequestParam**的使用

## @SessionAttributes

1. 作用：作用在类上，Spring MVC 执行完控制器的逻辑后， 将数据模型中对应的属性名称或者属性类型保存到 HTTP 的 Session 对象中。

2. 属性

   - value：指定存入属性的名称

   - types: 类型 两者取或关系

3. 代码如下

   ~~~java
   
       @Controller 
       @RequestMapping(path="/user")
        // 把数据存入到session域对象中  
       @SessionAttributes(value= {"username","password","age"},types= {String.class,Integer.class}) 
       public class HelloController { 
           /*** 向session中存入值 * @return */ 
           @RequestMapping(path="/save") 
           public String save(Model model) { 
               System.out.println("向session域中保存数据"); 
               model.addAttribute("username", "root"); 
               model.addAttribute("password", "123"); 
               model.addAttribute("age", 20); 
               return "success"; 
               }
           /*** 从session中获取值 * @return */ 
           @RequestMapping(path="/find") 
           public String find(ModelMap modelMap) { 
               String username = (String) modelMap.get("username"); 
               String password = (String) modelMap.get("password"); 
               Integer age = (Integer) modelMap.get("age"); 
               System.out.println(username + " : "+password +" : "+age); 
               return "success"; 
           }
           /*** 清除值 * @return */ 
           @RequestMapping(path="/delete") 
           public String delete(SessionStatus status) {
           status.setComplete(); 
           return "success"; } 
       }
   ~~~

## ＠SessionAttribute

从session中获取值，作用在方法参数上

~~~java
@RequestMapping("/sessionAttribute")
	public ModelAndView sessionAttr(@SessionAttribute("id") Long id) {
		ModelAndView mv = new ModelAndView();
		Role role = roleService.getRole(id);
		mv.addObject("role", role);
		mv.setView(new MappingJackson2JsonView());
		return mv;
	}
~~~

## @RequestHeader

1. 作用：获取指定请求头的值
2. 属性  
     value：请求头的名称
3. 代码如下
    ~~~java
        @RequestMapping(path="/hello") 
        public String sayHello(@RequestHeader(value="Accept") String header) {
            System.out.println(header); 
            return "success"; } 
    ~~~

## @CookieValue
1. 作用：用于获取指定cookie的名称的值
2. 属性  
     value：cookie的名称
3. 代码
    ~~~java
    @RequestMapping(path="/hello") 
    public String sayHello(@CookieValue(value="JSESSIONID") String cookieValue) {
        System.out.println(cookieValue); 
        return "success"; 
    } 
    ~~~
    
## 为控制器添加通知

> 内容为SpringBoot相关

### @ControllerAdvice

作用于类，标识全局性的控制器的拦截器.

定义一个控制器的通知类，允许定义一些关于增强控制器的各类通知和限定增强哪些控制器功能等。

### @InitBinder

定义控制器参数绑定规则，如转换规则、格式化等，它会在参数转换之前执行。

### @ExceptionHandler

注册一个控制器异常，使用当控制器发生注册异常 时， 就会跳转到该方法上。

做通用异常处理时经常用到。

可以在一个@ControllerAdvice标记的类中的某个方法上使用，可以配合**@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)**使用,来响应错误码页面。

~~~java
@ControllerAdvice(
// 指定拦截包的控制器
basePackages = { "com.tang.springboot.controller.*" }, 
// 限定被标注为@Controller 或者@RestController 的类才被拦截
annotations = {Controller.class, RestController.class})
public class VoControllerAdvice { 
     // 异常处理，可以定义异常类型进行拦截处理
     @ExceptionHandler(value = NotFoundException.class)
     // 以 JSON 表达方式响应
     @ResponseBody 
     // 定义为服务器错误状态码
     @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
     public Map<String, Object> exception(HttpServletRequest request, 
        NotFoundException ex) { 
            Map<String, Object> msgMap = new HashMap<>(); 
            // 获取异常信息
            msgMap.put("code", ex.getCode()); 
            msgMap.put("message", ex.getCustomMsg()); 
            return msgMap; 
 		} 
}
//正常的controller中的方法

@GetMapping(value="/user/exp/{id}", 
// 产生 JSON 数据集
produces = MediaType.APPLICATION_JSON_UTF8_VALUE) 
// 响应成功
@ResponseStatus(HttpStatus.OK)
@ResponseBody 
public UserVo getUserForExp(@PathVariable("id") Long id) { 
     User user = userService.getUser(id); 
     // 如果找不到用户，则抛出异常，进入控制器通知
     if (user == null) {
    throw new NotFoundException(1L, "找不到用户【" + id +"】信息");
     }
     UserVo userVo = changeToVo(user); 
     return userVo; 
}
~~~

### @ModelAttribute

可以在控制器方法执行之前，对数据模型进行操作。

1. 作用
   1. 出现在方法上：表示当前方法会在控制器方法执行前先执行。
   2. 出现在参数上：获取指定的数据给参数赋值。

2. 应用场景  
      当提交表单数据不是完整的实体数据时,保证没有提交的字段使用数据库原来的数据。
3. 具体的代码
	~~~java
   	/**
      	 * 在进入控制器方法前运行，先从数据库中查询角色，然后以键role保存角色对象到数据模型
      	 * @param id 角色编号
      	 * @return 角色
           */
   @ModelAttribute("role")
   public Role initRole(@RequestParam(value="id", required = false) Long id) {
       //判断id是否为空
       if (id == null || id < 1 ) {
           return null;
       }
       Role role = roleService.getRole(id);
       return role;
   }
   
   /**
      	 * @ModelAttribute注解从数据模型中取出数据，
      	 * @param role 从数据模型中取出的角色对象
      	 * @return 角色对象
      	 */
   @RequestMapping(value="getRoleFromModelAttribute")
   @ResponseBody
   public Role getRoleFromModelAttribute(@ModelAttribute("role") Role role) {
       return role;
   }
   ~~~

# 数据验证

利用JSR 303注解

## 简单格式校验

~~~java
package com.tang.ssm.springmvc.pojo;

import lombok.Data;
import org.springframework.format.annotation.DateTimeFormat;

import javax.validation.constraints.*;
import java.util.Date;
/**
 * 用来测试校验的实体类
 * @author tzcqupt
 */
@Data
public class Transaction {
	// 产品编号
	@NotNull // 不能为空
	private Long productId;
	//@Null 被标识的必须为null
	// 用户编号
	@NotNull // 不能为空
	private Long userId;

	// 交易日期
	@Future // 只能是将来的日期
	@NotNull // 不能为空
    @DateTimeFormat(pattern = "yyyy-MM-dd") // 日期格式化转换
    //@NumberFormat(pattern="#,###.##")
	private Date date;

	// 价格
	@NotNull // 不能为空
	@DecimalMin(value = "0.1") // 最小值0.1元
    //@DecimalMax(value)  //最大值 小数
	private Double price;

	// 数量
	@Min(1) // 最小值为1 
	@Max(100) // 最大值 整数
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
	private String email;

	// 备注
	@Size(min = 0, max = 256) // 0到255个字符
	private String note;

}

~~~

## 业务校验

~~~java
public class TransactionValidator implements Validator {
	@Override
	public boolean supports(Class<?> clazz) {
		//判断验证是否为Transaction，如果是则进行判断[修改为：验证]
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

## 绑定到控制器中

~~~java
@InitBinder
public void initBinder(DataBinder binder) {
    // 数据绑定器加入验证器
    binder.setValidator(new TransactionValidator());
}
//@Valid启动验证器
@RequestMapping("/validator")
public ModelAndView validator(@Valid Transaction trans, Errors errors) {
    // 是否存在错误
    if (errors.hasErrors()) {
        // 获取错误信息
        List<FieldError> errorList = errors.getFieldErrors();
        for (FieldError error : errorList) {
            // 打印字段错误信息
            log.error("fied :" + error.getField() + "\t" + "msg:" + error.getDefaultMessage());
        }
    }
    ModelAndView mv = new ModelAndView();
    mv.setViewName("index");
    return mv;
}
~~~



# 响应数据和结果视图

## 返回值分类
### 字符串

会直接跳转到物理页面
  ~~~java
@RequestMapping(value="/hello") 
public String sayHello() { 
    System.out.println("Hello SpringMVC!!"); 
    // 跳转到XX页面 return "success"; 
}  
  ~~~
### void 

由于springmvc框架传递的参数默认带了Servlet原始API,可以将其作为控制器方法中的参数
  ~~~java
   @RequestMapping("/testReturnVoid")
    public void testReturnVoid(HttpServletRequest request,
        HttpServletResponse response) 
          throws Exception {
      }
  ~~~
  
1. 使用 request 转向页面  
      `request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request, response);`
2. 也可以通过 response 页面重定向  
      `response.sendRedirect("testRetrunString") `
3. 也可以通过 response 指定响应结果，例如响应 json 数据：
 ~~~java
response.setCharacterEncoding("utf-8");
response.setContentType("application/json;charset=utf-8");
response.getWriter().write("json 串");
 ~~~
 
### ModelAndView

本质和SessionAttributes类似,都是往域中存取值
~~~java
   @RequestMapping("/testReturnModelAndView")
   public ModelAndView testReturnModelAndView() {
         ModelAndView mv = new ModelAndView();
         mv.addObject("username", "张三");
         mv.setViewName("success");
           return mv;
      }
      // jsp中取值
    ${requestScope.username}
~~~
>返回ModelAndView类型时,浏览器跳转只能是请求转发

## 转发和重定向  
### 转发 forward  
* `@Controller`中的String类型返回值的控制器方法,默认是请求转发
* 代码 `return "forward:/WEB-INF/pages/success.jsp";`
 >使用了forward,后面跟的就是物理视图

### 重定向Redirect

#### 普通重定向

 ~~~java
@RequestMapping("/addRole")
public ModelAndView addRole(ModelAndView mv,String roleName,String note) {
    // 插入角色,绑定数据模型，重定向到显示角色信息
    Role role = new Role;
    role.setRoleName(roleName);
    role.setNote(note); 
    //插入角色后，会回填角色编号 
    roleService.insertRole(role);
    mv.addObject(”roleName”, roleName); 
    mv.addObject(”note”,note);
    mv.addObject("id”,role.getid());            
    return "redirect:./showRoleInfo.do"; 
} 
 ~~~
>重定向到jsp页面,不能将页面放在WEB-INF目录中

#### 重定向传递 Java 对象

渲染视图时，是从Session里面获取值，修改对象的值后，直接将该对象的属性和值放入到session中，

避免重定向时二次查询。提高效率。

![重定向传递对象流程图](..\img\SpringMVC重定向传递对象流程图.PNG)

~~~java
// 显示用户
// 参数 user 直接从数据模型 RedirectAttributes 对象中取出
@RequestMapping("/showUser") 
public String showUser(User user, Model model) { 
     System.out.println(user.getId()); 
     return "data/user"; 
} 
// 使用字符串指定跳转
@RequestMapping("/redirect1") 
public String redirect1(String userName, String note, RedirectAttributes ra) { 
     User user = new User(); 
     user.setNote(note); 
     user.setUserName(userName); 
     userService.insertUser(user); 
     // 保存需要传递给重定向的对象
     ra.addFlashAttribute("user", user);
     return "redirect:/user/showUser"; 
} 
// 使用模型和视图指定跳转
@RequestMapping("/redirect2") 
public ModelAndView redirect2(String userName, String note, RedirectAttributes ra) { 
     User user = new User(); 
     user.setNote(note); 
     user.setUserName(userName); 
     userService.insertUser(user); 
     // 保存需要传递给重定向的对象
     ra.addFlashAttribute("user", user);
     ModelAndView mv = new ModelAndView(); 
     mv.setViewName("redirect:/user/showUser"); 
     return mv; 
}
~~~



## ResponseBody响应json数据  

@ResponseBody 注解实现将 controller 方法返回对象转换为 json 响应给客户端
>**前提 ：有jackson的包>2.7.0**,Spring 4.3需要2.8.7
~~~java
   @RequestMapping("/testResponseJson")
   //@RequestBody 获取请求体内容,key=value形式
    public @ResponseBody Account testResponseJson(@RequestBody Account account) {
            System.out.println("异步请求："+account);
            return account; 
      }
~~~
![SpringMVC@ResponseBody 注解转换为 JSON 流程图](..\img\SpringMVC@ResponseBody 注解转换为 JSON 流程图.PNG)

## 文件上传

### 前提
1. form 表单的 `enctype `取值必须是：`multipart/form-data`
    (默认值是:`application/x-www-form-urlencoded`)  
    `enctype`:是表单请求正文的类型
2. 提交方式时POST
3. 提供一个文件选择域`<input type="file" />`

### 举例

#### 前端

~~~html
   <form action="/fileUpload" method="post" enctype="multipart/form-data">
        名称：<input type="text" name="picname"/><br/>
        图片：<input type="file" name="uploadFile"/><br/>
    	<input type="submit" value="上传"/>
   </form>
~~~

#### 后端
 ~~~java
@RequestMapping("/fileUpload")
public String testFileUpload(String picname,MultipartFile 
                             uploadFile,HttpServletRequest request) throws Exception{
    // 一系列判断文件名和路径是否存在等问题
    //定义文件名
    String fileName = "";
    //1.获取原始文件名
    String uploadFileName = uploadFile.getOriginalFilename();
    //2.截取文件扩展名
    String extendName = 
        uploadFileName.substring(uploadFileName.lastIndexOf(".")+1, 
                                 uploadFileName.length());
    //3.把文件加上随机数，防止文件重复
    String uuid = UUID.randomUUID().toString().replace("-", "").toUpperCase();
    //4.判断是否输入了文件名
    if(!StringUtils.isEmpty(picname)) {
        fileName = uuid+"_"+picname+"."+extendName; }else {
        fileName = uuid+"_"+uploadFileName; }
    System.out.println(fileName);
    //2.获取文件路径
    ServletContext context = request.getServletContext();
    String basePath = context.getRealPath("/uploads");
    //3.解决同一文件夹中文件过多问题
    String datePath = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
    //4.判断路径是否存在
    File file = new File(basePath+"/"+datePath);
    if(!file.exists()) {
        file.mkdirs();
    }
    //5.使用 MulitpartFile 接口中方法，把上传的文件写到指定位置
    uploadFile.transferTo(new File(file,fileName));
    return "success";
}
 ~~~
>需要commons-fileupload-1.3.1
#### 配置文件解析器

**MultipartResolver**有**CommonsMultipartResolver**和**StandardServletMultipartResolver**两个实现，前者是Spring 3.1和Servlet 3.0以前用，且需要commons-fileupload配合。后者则不需要

##### Servlet3.0以下的CommonsMultipartResolver配置

~~~xml
<!-- 配置文件上传解析器 --> 
<!-- id 的值是固定的-->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置上传文件的最大尺寸为 5MB --> 
    <property name="maxUploadSize"> 
        <value>5242880</value>
    </property>
</bean>
~~~
##### Servlet3.0以上的**StandardServletMultipartResolver**配置

~~~java
@Bean(name = "multipartResolver") 
public MultipartResolver 工nitMultipartResolver() { 
    return new StandardServletMultipartResolver();
}
//用注解配置wen.xml时配置相关参数
public class MyWebAppInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {
	//Spring IoC容器配置
	@Override
	protected Class<?>[] getRootConfigClasses() {
		//可以返回Spring的Java配置文件数组
		return new Class<?>[] {};
	}

	//DispatcherServlet的URI映射关系配置
	@Override
	protected Class<?>[] getServletConfigClasses() {
		//可以返回Spring的Java配置文件数组
		return new Class<?>[] { WebConfig.class };
	}

	//DispatchServlet[修改为：DispatcherServlet]拦截请求匹配
	@Override
	protected String[] getServletMappings() {
		return new String[] { "*.do" };
	}
	
	/**
	 * @param dynamic Servlet动态加载配置
	 */
	@Override
	protected void customizeRegistration(Dynamic dynamic) {
		//文件上传路径
		String filepath = "e:/mvc/uploads";
		//5MB
		Long singleMax = (long) (5*Math.pow(2, 20));
		//10MB
		Long totalMax = (long) (10*Math.pow(2, 20));
		//配置MultipartResolver，限制请求，单个文件5MB，总共文件10MB
		dynamic.setMultipartConfig(new MultipartConfigElement(filepath, singleMax, totalMax, 0));
	}

}
~~~

## 拦截器

1. 自定义普通类实现`HandlerInterceptor`接口
2. 配置拦截器
    ~~~xml
    <!-- 配置拦截器 --> 
   <mvc:interceptors> 
       <mvc:interceptor> 
           <mvc:mapping path="/**"/>
            <bean id="handlerInterceptorDemo1"
            class="com.tang.ssm.springmvc.interceptor.HandlerInterceptorDemo1"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
   ~~~
3. 细节
    1. `preHandle`只要配置了都会调用
        * 按拦截器定义**顺序**调用
        * 只要配置了都会调用
        * 如果程序员决定该拦截器对请求进行拦截处理后还要调用其他的拦截器,或者是业务处理器去进行处理,则返回 true;  
        如果程序员决定不需要再调用其他的组件去处理请求,则返回 false。
    2. `postHandle`
        * 按拦截器定义**逆序**调用
        * 在拦截器链内**所有**拦截器返成功调用
        * 在业务处理器处理完请求后,但是 DispatcherServlet 向客户端返回响应前被调用,在该方法中对用户请求 request 进行处理。
    3. `preHandle`
        * 按拦截器定义**逆序**调用
        * 只有 `preHandle`返回 `true`才调用   
        * 可以在该方法中进行一些资源清理的操作。
    4. 2个拦截器调用顺序
		- preHandle都返回`true`的情况
            ~~~java
            【Demo1的 preHandle 拦截器拦截了】
            【Demo2的 preHandle 拦截器拦截了】
            【拦截器中的方法执行了】
            【Demo2的 postHandle 方法执行了】
            【Demo1的 postHandle 方法执行了】
            【Demo2的 afterCompletion 方法执行了】
            【Demo1的 afterCompletion 方法执行了】
            ~~~
        - Demo2的preHandle返回`false`的情况
            ~~~java
            【Demo1的 preHandle 拦截器拦截了】
            【Demo2的 preHandle 拦截器拦截了】
            【Demo1的 afterCompletion 方法执行了】
            ~~~

# 转换器

## 配置

~~~xml
<!--配置类型转换器工厂-->
<bean id="converterService"
      class="org.springframework.context.support.ConversionServiceFactoryBean">
    <!-- 给工厂注入一个新的类型转换器 -->
    <property name="converters">
        <array>
            <!-- 配置自定义类型转换器 -->
            <bean class="com.tang.ssm.springmvc.utils.StringToDateConverter"/>
        </array>
    </property>
</bean>
<!-- 引用自定义类型转换器 -->
<mvc:annotation-driven conversion-service="converterService"/>
~~~

## 格式注解转换

~~~java
@DateTimeFormat(pattern = "yyyy-MM-dd")// 日期格式化转换
private Date Date
@DateTimeFormat(iso＝ ISO.DATE) 
private Date date
@NumberFormat(pattern="#,###.##") 
private Double amount
~~~