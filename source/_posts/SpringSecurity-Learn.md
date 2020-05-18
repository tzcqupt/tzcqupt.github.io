---
layout: post
title: SpringSecurity学习笔记
Author: tzcqupt
tags: [SpringSecurity]
categories: [SpringSecurity]
comments: true
toc: true
date: 2020-05-08 00:00:00
---

# SpringSecurity入门

## 无用户登录

### 依赖

~~~xml
 
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
    <relativePath/> 
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
 </dependencies>
~~~

### controller

~~~java
@RestController
public class IndexController {
    @GetMapping("hello")
    public String hello(){
        return "hello Spring Security!";
    }
    @GetMapping({"index","/"})
    public String index(){
        return "这是登录成功的index页面";
    }
    @GetMapping("failIndex")
    public String failIndex(){
        return "这是登录失败的index页面";
    }
}
~~~



启动项目会发现`Using generated security password: 30abfb1f-36e1-446a-a79b-f70024f552ab`，这就是 Spring Security 为默认用户 user 生成的临时密码，是一个 UUID 字符串。

访问hello接口需要登录，输入user和上面的密码才能访问。

spring Security中的源码:`private String password = UUID.randomUUID().toString();`

## 用户配置

### 配置文件

~~~properties
spring.security.user.name=tang
spring.security.user.password=123
~~~

~~~java
//spring security 源码
@ConfigurationProperties(prefix = "spring.security")
public class SecurityProperties {}
~~~

### 配置类

#### PasswordEncoder接口

~~~java
public interface PasswordEncoder {
    //加密
	String encode(CharSequence rawPassword);
    //密码校对
	boolean matches(CharSequence rawPassword, String encodedPassword);
    //是否对密码再次加密
	default boolean upgradeEncoding(String encodedPassword) {
		return false;
	}
}
~~~

#### 代码配置

1. 注册`passwordEncoder`到容器中
2. 在内存中定义用户

~~~java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    PasswordEncoder passwordEncoder() {
        //暂时先不加密密码
        return NoOpPasswordEncoder.getInstance();
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("tang")
                .password("123").roles("admin");
    }
}
~~~

> 配置多个用户，使用`and`相连

## 自定义表单登录页

~~~java
@Override
public void configure(WebSecurity web) throws Exception {
    //配置静态资源过滤
    web.ignoring().antMatchers("/js/**", "/css/**","/images/**");
}
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login.html")
            .permitAll()
            .and()
        	// 暂时先关闭csrf
            .csrf().disable();
}
~~~

1. `and`表示结束当前配置，继续下一个配置。
2. `permitAll`表示登录相关的页面/接口不要被拦截。

3. 复制静态页面到`resource`下的`static`目录。

# SpringSecurity中的表单登录

## 传统页面交互

### 配置登录接口地址

#### 服务端配置

~~~java
.and()
.formLogin()
.loginPage("/login.html")
.loginProcessingUrl("/doLogin")
.permitAll()
.and()
~~~

#### 登录页面修改

~~~html
<form action="/doLogin" method="post">
<!--省略-->
</form>
~~~

### 登录参数修改配置

~~~java
.and()
.formLogin()
.loginPage("/login.html")
.loginProcessingUrl("/doLogin")
.usernameParameter("name")
.passwordParameter("passwd")
.permitAll()
.and()
~~~

前端页面同步修改

### 登录回调

#### 登录成功回调

- `defaultSuccessUrl`

如果在浏览器中输入了其他地址，例如 `http://localhost:8080/hello`，结果因为没有登录，又重定向到登录页面，此时登录成功后，就不会来到 `/index` ，而是来到 `/hello` 页面。

使用重载方法时，如果手动设置第二个参数为 true，则 `defaultSuccessUrl `的效果和 `successForwardUrl` 一致。

- `successForwardUrl`

  `successForwardUrl` 表示不管你是从哪里来的，登录后一律跳转到 `successForwardUrl` 指定的地址。

#### 登录失败回调

- `failureForwardUrl`
- `failureUrl`

#### 相关配置

~~~java
.and()
.formLogin()
.loginPage("/login.html")
.loginProcessingUrl("/doLogin")
.usernameParameter("name")
.passwordParameter("passwd")
.defaultSuccessUrl("/index")
.failureUrl("failure")
.permitAll()
~~~

### 注销登录

默认的注销登录接口为`/logout`

~~~java
.and()
.logout()
 //二选一
.logoutUrl("/logout")
.logoutRequestMatcher(new AntPathRequestMatcher("/logout","POST"))
.logoutSuccessUrl("/index")
.deleteCookies()
.clearAuthentication(true)
.invalidateHttpSession(true)
.permitAll()
.and()
~~~

1. `.logoutUrl("/logout")`修改默认的注销url
2. `logoutRequestMatcher`修改注销URL和请求方式。
3. `logoutSuccessUrl` 表示注销成功后要跳转的页面。
4. `deleteCookies` 用来清除 Cookie。
5. `clearAuthentication` 和`invalidateHttpSession` 分别表示清除认证信息和使 HttpSession 失效，默认可以不用配置，默认就会清除。

## JSON交互数据

有状态：服务端需要记录每次会话的客户端信息。

无状态：服务端不保存任何客户端请求者信息，客户端的每次请求必须自备自描述信息，通过这些信息失败客户端身份。

### 无状态登录流程

1. 客户端发送账户名/密码到服务端进行认证
2. 认证铜鼓哦吼，服务端将用户信息加密并且编码成一个token，返回给客户端
3. 以后客户端每次发送请求，都需要携带认证的 token
4. 服务端对客户端发送来的 token 进行解密，判断是否有效，并且获取用户登录信息

### 登录成功

~~~java
.successHandler((req, resp, authentication) -> {
    Object principal = authentication.getPrincipal();
    resp.setContentType("application/json;charset=utf-8");
    PrintWriter out = resp.getWriter();
    out.write(new ObjectMapper().writeValueAsString(principal));
    out.flush();
    out.close();
})
~~~

3个参数：

- HttpServletRequest

- HttpServletResponse

  利用 HttpServletRequest 我们可以做服务端跳转，利用 HttpServletResponse 我们可以做客户端跳转，也能返回JSON数据。

- Authentication

  保存了 刚刚登录成功的用户信息。

### 登录失败

~~~java
.failureHandler((req, resp, e) -> {
    resp.setContentType("application/json;charset=utf-8");
    PrintWriter out = resp.getWriter();
    out.write(e.getMessage());
    out.flush();
    out.close();
})
~~~

### 未认证处理

~~~java
.csrf().disable().exceptionHandling()
.authenticationEntryPoint((req, resp, authException) -> {
            resp.setContentType("application/json;charset=utf-8");
            PrintWriter out = resp.getWriter();
            out.write("尚未登录，请先登录");
            out.flush();
            out.close();
        }
);
~~~

### 注销登录

~~~java
.and()
.logout()
.logoutUrl("/logout")
.logoutSuccessHandler((req, resp, authentication) -> {
    resp.setContentType("application/json;charset=utf-8");
    PrintWriter out = resp.getWriter();
    out.write("注销成功");
    out.flush();
    out.close();
})
.permitAll()
.and()
~~~

## 自动登录

~~~java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .and()
            .rememberMe()
        	// 配置remember-me 令牌生成的key，避免spring security使用UUID生成
        	// 服务端重启/浏览器打开关闭后，还能登录。
        	.key("tang")
            .and()
            .csrf().disable();
}
~~~

> 添加`.rememberMe()`即可

# 授权配置

## 简单授权配置

### 用户配置

~~~java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
            .withUser("tang")
            .password("123").roles("admin")
            .and()
            .withUser("tzcqupt")
            .password("123")
            .roles("user");
}
~~~



### 服务端配置

~~~java
http.authorizeRequests()
        .antMatchers("/admin/**").hasRole("admin")
        .antMatchers("/user/**").hasRole("user")
        .anyRequest().authenticated()
        .and()
~~~

### 角色继承

让admin自动拥有user的角色

~~~java
@Bean
RoleHierarchy roleHierarchy() {
    RoleHierarchyImpl hierarchy = new RoleHierarchyImpl();
    hierarchy.setHierarchy("ROLE_admin > ROLE_user");
    return hierarchy;
}
~~~

> 需要手动给角色加上`ROLE_`前缀