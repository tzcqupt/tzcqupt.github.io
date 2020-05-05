---
layout: post
title: Shiro入门学习笔记
Author: tzcqupt
tags: [Shiro]
categories: [SpringSecurity]
comments: true
toc: true
date: 2020-05-03 00:00:00
---

# 概念

Shiro架构图

![](https://gitee.com/tzcqupt/blog-image/raw/master/img/shiro架构.jpg)



## subject

主体，可以是用户也可以是程序，主体要访问系统，系统需要对主体进行认证、授权。

## securityManager

安全管理器，主体进行认证和授权都 是通过securityManager进行。

## authenticator

认证器，主体进行认证最终通过authenticator进行的。

## authorizer

授权器，主体进行授权最终通过authorizer进行的。

## sessionManager

web应用中一般是用web容器对session进行管理，shiro也提供一套session管理的方式。

## SessionDao

通过SessionDao管理session数据，针对个性化的session数据存储需要使用sessionDao。

## cache Manager

缓存管理器，主要对session和授权数据进行缓存，比如将授权数据通过cacheManager进行缓存管理，和ehcache整合对缓存数据进行管理。

## realm

域，领域，相当于数据源，**通过realm存取认证、授权相关数据。**

## cryptography

**密码管理，提供了一套加密/解密的组件，方便开发。比如提供常用的散列、加/解密等功能。**

# 使用Shiro认证

![](https://gitee.com/tzcqupt/blog-image/raw/master/img/Shiro认证流程.png)

## 依赖

~~~xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
</dependency>
<!--其他依赖-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jcl</artifactId>
    <version>1.7.25</version>
</dependency>
~~~

## Shiro认证流程



### 通过配置文件创建工厂

#### 配置文件shiro-first.ini

~~~ini
#[对用户信息进行配置]
[users]
#用户账号和密码
tang=123
tzcqupt=1234
~~~

#### 测试代码

~~~java
@Slf4j
public class ShiroTest01 {
    /**
     * 测试从配置文件加载用户信息
     * Shiro管理认证的登录和退出
     * <p>1. 通过ini配置文件创建securityManager工厂</p>
     * <p>2. 创建SecurityManager</p>
     * <p>3. 将securityManager设置当前的运行环境中</p>
     * <p>4. 从SecurityUtils里边创建一个subject</p>
     * <p>5. 在认证提交前准备token（令牌）</p>
     * <p>6. 执行认证提交</p>
     * <p>7. 验证是否认证通过</p>
     */
    @Test
    public void testLoginAndLogout() {
        Factory<SecurityManager> factory = new IniSecurityManagerFactory(
                "classpath:shiro-first.ini");
        String username = "tang";
        String password = "123";
        this.authenticate(factory, username, password);
    }

    private void authenticate(Factory<SecurityManager> factory, String username, String password) {
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
        Subject subject = SecurityUtils.getSubject();
        // 这里的账号和密码 将来是由用户输入进去
        UsernamePasswordToken token = new UsernamePasswordToken(username,
                password);
        log.info(token.getUsername() + "准备认证登录了");
        try {
            subject.login(token);
        } catch (AuthenticationException e) {
            log.error(token.getUsername() + "认证失败了,出现了异常" + e.getMessage());
        }
        boolean isAuthenticated = subject.isAuthenticated();
        log.info(token.getUsername() + "是否认证通过：" + isAuthenticated);
        log.info(token.getUsername() + "准备退出登录了");

        subject.logout();
        // 是否认证通过
        isAuthenticated = subject.isAuthenticated();

        log.info(token.getUsername() + "是否认证通过：" + isAuthenticated);
    }

}

~~~

### 自定义Relam

认证流程是认证器去找realm去查询我们相对应的数据**。而默认的realm是直接去与配置文件来比对的，一般地，**我们在开发中都是让realm去数据库中比对。

#### 普通的Relam


##### Relam实体

~~~java
public class CustomRealm extends AuthorizingRealm {
    /**
     * 设置realm的名称
     * @param name
     */
    @Override
    public void setName(String name) {
        super.setName("customRealm");
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // token是用户输入的
        // 第一步从token中取出身份信息
        String userCode = (String) token.getPrincipal();

        // 第二步：根据用户输入的userCode从数据库查询
        // ....

        // 如果查询不到返回null
        //数据库中用户账号是zhangsansan
        /*if(!userCode.equals("zhangsansan")){//
            return null;
        }*/
        
        // 模拟从数据库查询到密码
        String password = "123";
        // 如果查询到返回认证信息AuthenticationInfo
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(
                userCode, password, this.getName());
        return simpleAuthenticationInfo;
    }
}
~~~
##### 配置文件shiro-realm.ini

~~~ini
[main]
#自定义Realm
customRealm=com.tang.shiro.realm.CustomRealm
#将realm设置到securityManager，相当于Spring注入
securityManager.realms=$customRealm
~~~

##### 测试代码

~~~java
 @Test
    public void testCustomRealm() {
        Factory<SecurityManager> factory = new IniSecurityManagerFactory(
                "classpath:shiro-realm.ini");
        this.authenticate(factory, "tang", "123");
    }
~~~

#### 自定义realm支持md5

密码加密算法：MD5 在程序中对原始密码+盐进行散列，将散列值存储到数据库中，并且将盐页存储到数据库中。

##### 散列算法测试

~~~java
@Slf4j
public class Md5Test {
    @Test
    public void testMd5(){
        //原始密码
        String source = "123";
        //盐
        String salt="java";
        //散列次数
        int hashIterations = 2;
        Md5Hash md5Hash = new Md5Hash(source, salt, hashIterations);
        String password_md5 =  md5Hash.toString();
        log.info(password_md5);
        //7574cfad48966ca3a0a076a46741d306
        SimpleHash simpleHash = new SimpleHash("md5", source, salt, hashIterations);
        log.info(simpleHash.toString());
    }
}
~~~

##### Relam实体

~~~java
public class CustomRealmMd5 extends AuthorizingRealm {
    /**
     * 设置realm的名称
     * @param name
     */
    @Override
    public void setName(String name) {
        super.setName("customRealmMd5");
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // token是用户输入的
        // 第一步从token中取出身份信息
        String userCode = (String) token.getPrincipal();

        // 第二步：根据用户输入的userCode从数据库查询
        // ....
        // 如果查询不到返回null
        //数据库中用户账号是zhangsansan
        /*if(!userCode.equals("zhangsansan")){//
            return null;
        }*/

        // 模拟从数据库查询到密码,散列值
        String password = "7574cfad48966ca3a0a076a46741d306";
        // 从数据库获取salt
        String salt = "java";

        // 如果查询到返回认证信息AuthenticationInfo
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(
                userCode, password, ByteSource.Util.bytes(salt), this.getName());

        return simpleAuthenticationInfo;
    }
}

~~~

##### 配置文件shiro-realm-md5.ini

~~~ini
[main]
#定义凭证匹配器
credentialsMatcher=org.apache.shiro.authc.credential.HashedCredentialsMatcher
#散列算法
credentialsMatcher.hashAlgorithmName=md5
#散列次数
credentialsMatcher.hashIterations=2
#自定义Realm
customRealm=com.tang.shiro.realm.CustomRealmMd5
customRealm.credentialsMatcher=$credentialsMatcher
#将realm设置到securityManager，相当于Spring注入
securityManager.realms=$customRealm
~~~

##### 测试代码

~~~java
 @Test
    public void testCustomRealmMd5() {
        Factory<SecurityManager> factory = new IniSecurityManagerFactory(
                "classpath:shiro-realm-md5.ini");
        String username = "tang";
        String password = "123";
        this.authenticate(factory, username, password);
    }
~~~

# 使用Shiro授权

## 依赖

~~~xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
</dependency>
<!--其他依赖-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jcl</artifactId>
    <version>1.7.25</version>
</dependency>
~~~

## Shiro授权流程

![](https://gitee.com/tzcqupt/blog-image/raw/master/img/shiro授权流程.png)

### 配置文件shiro-permission.ini

~~~ini
#用户
[users]
#用户tang的密码是123，此用户具有role1和role2两个角色
tang = 123,role1,role2
wang = 123,role2

#权限
[roles]
#角色role1对资源user拥有create、update权限
role1 = user:create,user:update
#角色role2对资源user拥有create、delete权限
role2 = user:create,user:delete
#角色role3对资源user拥有create权限
role3 = user:create

#权限标识符号规则：资源:操作:实例(中间使用半角:分隔)
#表示对用户资源的01实例进行create操作。
#user:create:01
#表示对用户资源进行create操作，相当于user:create:*，对所有用户资源实例进行create操作。
#user:create
#表示对用户资源实例01进行所有操作。
#user:*:01
~~~

### 测试代码

~~~java
	/**
     * 授权测试
     */
    @Test
    public void testAuthorization() {
        Factory<SecurityManager> factory = new IniSecurityManagerFactory(
                "classpath:shiro-permission.ini");
        String username = "tang";
        String password = "123";
        Subject subject = this.authenticate(factory, username, password);
        this.authorization1(subject);
    }	
	/**
     * 封装授权验证
     * 编程式认证
     * @param subject
     */
    private void authorization(Subject subject) {
        // 基于角色的授权
        // hasRole传入角色标识
        boolean hasRole = subject.hasRole("role1");
        log.info("单个角色role1判断" + hasRole);
        // hasAllRoles是否拥有多个角色
        boolean hasAllRoles = subject.hasAllRoles(Arrays.asList("role1",
                "role2", "role3"));
        log.info("多个角色role1,role2,role3判断" + hasAllRoles);

        // 使用check方法进行授权，如果授权不通过会抛出异常
        try {
            subject.checkRole("role3");
        } catch (UnauthorizedException e) {
            log.error("没有role3的权限");
        }


        // 基于资源的授权
        // isPermitted传入权限标识符
        boolean isPermitted = subject.isPermitted("user:create:1");
        log.info("单个权限判断" + isPermitted);

        boolean isPermittedAll = subject.isPermittedAll("user:create:1",
                "user:delete");
        log.info("多个权限判断" + isPermittedAll);
        // 使用check方法进行授权，如果授权不通过会抛出异常
        try {
            subject.checkPermission("items:create:1");
        } catch (UnauthorizedException e) {
            log.error("没有对资源1的商品创建操作");
        }
    }
~~~

> 自定义Relam同上

# 与Spring整合-认证

## 简单整合

### 依赖

shiro版本：1.4.0

spring版本：4.3.24.RELEASE

打包成war包，Idea中tomcat运行命令：`clean tomcat7:run`

~~~xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
</dependency>
<!--servlet-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
</dependency>
</dependencies>
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <path>/</path>
                    <port>8081</port>
                    <uriEncoding>UTF-8</uriEncoding>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
~~~

### web.xml

~~~xml
 <welcome-file-list>
    <welcome-file>list.jsp</welcome-file>
</welcome-file-list>
    <!-- 加载Spring的配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- 配置springmvc -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--
    1. 配置  Shiro 的 shiroFilter.
    2. DelegatingFilterProxy 实际上是 Filter 的一个代理对象. 默认情况下, Spring 会到 IOC 容器中查找和
    <filter-name> 对应的 filter bean. 也可以通过 targetBeanName 的初始化参数来配置 filter bean 的 id.
    -->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
~~~

### spring-mvc.xml

~~~xml
<!--springMVC的配置文件，和web.xml一样放在WEB-INF目录下，名字为spring-sevlet-->
<context:component-scan base-package="com.tang.shiro"/>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"/>
    <property name="suffix" value=".jsp"/>
</bean>
<!--开启注解配置-->
<mvc:annotation-driven/>
<mvc:default-servlet-handler/>

~~~

### applicationContext.xml

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--1. 配置 SecurityManager! -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
       <property name="authenticator" ref="authenticator"/>
        <property name="realm" ref="customRealm"/>

    </bean>
    <!--配置自定义的，不加密的realm-->
    <bean id="customRealm" class="com.tang.shiro.realms.CustomRealm"/>
   
    <!--
    4. 配置 LifecycleBeanPostProcessor. 可以自定的来调用配置在 Spring IOC 容器中 shiro bean 的生命周期方法. 
    -->
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <!--  
    5. 启用 IOC 容器中使用 shiro 的注解. 但必须在配置了 LifecycleBeanPostProcessor 之后才可以使用. 
    -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor"/>
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager"/>
    </bean>
    <!--  
    6. 配置 ShiroFilter. 
    6.1 id 必须和 web.xml 文件中配置的 DelegatingFilterProxy 的 <filter-name> 一致.
                      若不一致, 则会抛出: NoSuchBeanDefinitionException. 因为 Shiro 会来 IOC 容器中查找和 <filter-name> 名字对应的 filter bean.
    -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="/login.jsp"/>
        <property name="successUrl" value="/list.jsp"/>
        <!--认证失败-->
        <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
        <!--  
        	配置哪些页面需要受保护. 
        	以及访问这些页面需要的权限. 
        	1). anon 可以被匿名访问
        	2). authc 必须认证(即登录)后才可能访问的页面. 
        	3). logout 登出.
        	4). roles 角色过滤器
        -->

        <property name="filterChainDefinitions">
            <value>
                /login.jsp = anon
                /fail.jsp = anon
                /unauthorized.jsp = anon
                /shiro/login = anon
                /shiro/logout = logout
                /** = authc
            </value>
        </property>
    </bean>
</beans>


~~~

### 自定义的Relam

~~~java
public class CustomRealm extends AuthorizingRealm {
    /**
     * 能登录成功的账号
     */
    private final String USER_NAME="tang";
    /**
     * 登录成功的账号密码
     */
    private final String PASSWORD="123";
    /**
     * 设置realm的名称
     * @param name
     */
    @Override
    public void setName(String name) {
        super.setName("customRealm");
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    /**
     * 用户认证逻辑
     * 只要用户名/密码为tang/123就认证通过
     * @param token
     * @return 认证信息
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // token是用户输入的
        // 第一步从token中取出身份信息
        String userCode = (String) token.getPrincipal();
        // 第二步：根据用户输入的userCode从数据库查询
        // ....

        // 如果查询不到返回null
        //数据库中用户账号是tang
        if(!USER_NAME.equals(userCode)){
            return null;
        }

        // 模拟从数据库查询到密码
        // 如果查询到返回认证信息AuthenticationInfo
        return new SimpleAuthenticationInfo(
                userCode, PASSWORD, this.getName());
    }
}

~~~

### 登录控制器Controller

~~~java
@Controller
@RequestMapping("shiro")
public class LoginController {

    /**
     * 登录逻辑，认证
     * @param username 用户名
     * @param password 密码
     * @return 具体页面
     */
    @PostMapping("login")
    public ModelAndView login(@RequestParam("username") String username,
                              @RequestParam("password") String password) {
        Subject currentUser = SecurityUtils.getSubject();
        ModelAndView mv = new ModelAndView();
        if (!currentUser.isAuthenticated()) {
            // 把用户名和密码封装为 UsernamePasswordToken 对象
            UsernamePasswordToken token = new UsernamePasswordToken(username, password);
            // rememberme
            token.setRememberMe(true);
            try {
                System.out.println("1. " + token.hashCode());
                // 执行登录.
                currentUser.login(token);
            } catch (AuthenticationException | AuthorizationException ae) {
                // 认证/授权失败
                System.out.println("登录失败: " + ae.getMessage());
                mv.setViewName("unauthorized");
                return mv;
            }
            //认证成功
            mv.setViewName("list");
            return mv;
        }
        mv.setViewName("list");
        return mv;
    }
}

~~~

### 访问

运行项目，使用tang/123登录项目，可以访问list.jsp

## 自定义Relam

### applicationContext.xml

~~~xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="authenticator" ref="authenticator"/>
    <!--<property name="realm" ref="customRealm"/>-->
    <property name="realms">
        <list>
            <ref bean="jdbcRealm"/>
            <ref bean="secondRealm"/>
        </list>
    </property>
</bean>
<bean id="jdbcRealm" class="com.tang.shiro.realms.RealmMd5">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <property name="hashAlgorithmName" value="MD5"/>
            <property name="hashIterations" value="1024"/>
        </bean>
    </property>
</bean>

<bean id="secondRealm" class="com.tang.shiro.realms.RealmSha1">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <property name="hashAlgorithmName" value="SHA1"/>
            <property name="hashIterations" value="1024"/>
        </bean>
    </property>
</bean>
~~~

### Md5测试

~~~java
@Test
public void testMd5() {
    String hashAlgorithmName = "MD5";
    Object credentials = "123456";
    Object salt = ByteSource.Util.bytes("admin");
    int hashIterations = 1024;
    Object result = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
    System.out.println(result);
}
~~~

### SHA1测试

~~~java
@Test
public void testSha1() {
    String hashAlgorithmName = "SHA1";
    Object credentials = "123456";
    Object salt = ByteSource.Util.bytes("admin");
    int hashIterations = 1024;
    Object result = new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
    System.out.println(result);
}
~~~



### RealmMd5

~~~java
/**
 * 自定义Realm MD5加密
 * 用户名:密码为
 * user/admin:123456才能登录
 * @author tzcqupt
 * @date 2020-05-04
 */
public class RealmMd5 extends AuthorizingRealm {

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken token) throws AuthenticationException {
        System.out.println("[FirstRealm] doGetAuthenticationInfo");
        //1. 把 AuthenticationToken 转换为 UsernamePasswordToken
        UsernamePasswordToken upToken = (UsernamePasswordToken) token;
        //2. 从 UsernamePasswordToken 中来获取 username
        String username = upToken.getUsername();
        //3. 调用数据库的方法, 从数据库中查询 username 对应的用户记录
        System.out.println("从数据库中获取 username: " + username + " 所对应的用户信息.");
        //4. 若用户不存在, 则可以抛出 UnknownAccountException 异常
        if ("unknown".equals(username)) {
            throw new UnknownAccountException("用户不存在!");
        }
        //5. 根据用户信息的情况, 决定是否需要抛出其他的 AuthenticationException 异常.
        if ("monster".equals(username)) {
            throw new LockedAccountException("用户被锁定");
        }
        //6. 根据用户的情况, 来构建 AuthenticationInfo 对象并返回. 通常使用的实现类为: SimpleAuthenticationInfo
        //以下信息是从数据库中获取的.
        //1). principal: 认证的实体信息. 可以是 username, 也可以是数据表对应的用户的实体类对象.
        //2). credentials: 密码.
        Object credentials = null;
        if ("admin".equals(username)) {
            credentials = "038bdaf98f2037b31f1e75b5b4c9b26e";
        } else if ("user".equals(username)) {
            credentials = "098d2c478e9c11555ce2823231e02ec1";
        }

        //3). realmName: 当前 realm 对象的 name. 调用父类的 getName() 方法即可
        String realmName = getName();
        //4). 盐值.
        ByteSource credentialsSalt = ByteSource.Util.bytes(username);
        return new SimpleAuthenticationInfo(username, credentials, credentialsSalt, realmName);
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(
            PrincipalCollection principals) {
        return null;
    }
}
~~~

> RealmSha1代码和RealmMd5基本一致，只需要修改`credentials`即可。

### 访问

运行项目，使用user/admin:123456登录项目，可以访问list.jsp

## 认证策略修改

默认的认证策略为`private AuthenticationStrategy authenticationStrategy = new AtLeastOneSuccessfulStrategy();` 至少一种认证通过。

### applicationContext.xml

~~~xml
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
       <property name="authenticator" ref="authenticator"/>
    </bean>
<bean id="authenticator" class="org.apache.shiro.authc.pam.ModularRealmAuthenticator">
        <property name="realms">
            <list>
                <ref bean="jdbcRealm"/>
                <ref bean="secondRealm"/>
            </list>
        </property>
        <property name="authenticationStrategy">
            <!--修改认证策略为全部成功-->
            <bean class="org.apache.shiro.authc.pam.AllSuccessfulStrategy"/>
        </property>
</bean>
~~~

### 测试

修改任意一个Realm的`credentials`进行测试。

~~~bash
# 登录失败日志
登录失败: Unable to acquire account data from realm [com.tang.shiro.realms.RealmSha1@462a5006].  The [org.apache.shiro.authc.pam.AllSuccessfulStrategy implementation requires all configured realm(s) to operate successfully for a successful authentication.
~~~

# 与Spring整合-授权

在认证的基础上修改

## JSP标签

### applicationContext.xml

~~~xml
<!-- 配置 SecurityManager! -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <property name="authenticator" ref="authenticator"/>
    <!-- 配置 授权的relam -->
    <property name="authorizer" ref="realmMd5"/>
</bean>
<bean id="realmMd5" class="com.tang.shiro.realms.RealmMd5">
    <property name="credentialsMatcher">
        <bean class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
            <property name="hashAlgorithmName" value="MD5"/>
            <property name="hashIterations" value="1024"/>
        </bean>
    </property>
</bean>

~~~

### RealmMd5

```java
public class RealmMd5 extends AuthorizingRealm {
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(
            AuthenticationToken token) throws AuthenticationException {
        //同认证逻辑
    }

    /**
     * 授权
     * @param principals
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(
            PrincipalCollection principals) {
        System.out.println("[RealmMd5] doGetAuthorizationInfo");
        //1. 从 PrincipalCollection 中来获取登录用户的信息
        Object principal = principals.getPrimaryPrincipal();

        //2. 利用登录的用户的信息来用户当前用户的角色或权限(可能需要查询数据库)
        Set<String> roles = new HashSet<>();
        roles.add("user");
        if("admin".equals(principal)){
            roles.add("admin");
        }
        //3. 创建 SimpleAuthorizationInfo, 并设置其 reles 属性.
        //4. 返回 SimpleAuthorizationInfo 对象.
        return new SimpleAuthorizationInfo(roles);
    }
}
```

> 支持一种Realm的授权。第二个realm继承`AuthenticatingRealm`即可。就不用完成授权。

### JSP标签方式

1. 引入shiro库

   `<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>`

2. jsp正文

   ~~~jsp
   Welcome: <shiro:principal/>
   
   <!-- <shiro:principal/> 输出当前用户信息，通常为登录帐号信息-->
   
   <shiro:hasRole name="admin">
       <br><br>
       <a href="${pageContext.request.contextPath}/admin.jsp">Admin Page</a>
   </shiro:hasRole>
   
   <shiro:hasRole name="user">
       <br><br>
       <a href="${pageContext.request.contextPath}/user.jsp">User Page</a>
   </shiro:hasRole>
   ~~~

## 注解方式

### LoginController

~~~java
 @GetMapping("/testShiroAnno")
    public ModelAndView testShiroAnnotation(){
        ModelAndView view = new ModelAndView();
        try {
            shiroService.testMethod();
        }catch (Exception e){
            System.out.println("权限不足，不能访问");
            System.out.println(e.getMessage());
            view.setViewName("login");
            return view;
        }
        view.setViewName("admin");
        return view;
    }
~~~

### ShiroService

~~~java
@Service
public class ShiroService {
    @RequiresRoles({"admin"})
    public void testMethod(){
        System.out.println("testMethod, time: " + LocalDate.now());
        Session session = SecurityUtils.getSubject().getSession();
        Object val = session.getAttribute("key");
        System.out.println("Service SessionVal: " + val);
    }
}
~~~

### jsp页面

~~~jsp
<a href="${pageContext.request.contextPath}/shiro/testShiroAnno">测试Shiro的权限注解</a>
~~~

### 踩坑日记

需要将`applicationContext.xml`中关于shiro权限注解的配置移到`spring-mvc.xml`中，权限注解才生效。猜测原因是Bean的生命周期。

~~~xml
	<!--
    5. 启用 IOC 容器中使用 shiro 的注解. 但必须在配置了 LifecycleBeanPostProcessor 之后才可以使用.
    -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor"/>
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager"/>
    </bean>
~~~

## 模拟从数据库中获取资源url

### applicationContext.xml

~~~xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="/login.jsp"/>
        <property name="successUrl" value="/list.jsp"/>
        <!--认证失败-->
        <property name="unauthorizedUrl" value="/unauthorized.jsp"/>
        <!--装载页面url的map （如：key=/login.jsp ,value=anno）-->
        <property name="filterChainDefinitionMap" ref="filterChainDefinitionMap"/>
    </bean>
    <!-- 配置一个 bean, 该 bean 实际上是一个 Map. 通过实例工厂方法的方式 -->
    <bean id="filterChainDefinitionMap"
          factory-bean="filterChainDefinitionMapBuilder" factory-method="buildFilterChainDefinitionMap"/>

    <bean id="filterChainDefinitionMapBuilder"
          class="com.tang.shiro.factory.FilterChainDefinitionMapBuilder"/>
~~~

### FilterChainDefinitionMapBuilder

~~~java
public class FilterChainDefinitionMapBuilder {

    public LinkedHashMap<String, String> buildFilterChainDefinitionMap(){
        LinkedHashMap<String, String> map = new LinkedHashMap<>();

        map.put("/login.jsp", "anon");
        map.put("/shiro/login", "anon");
        map.put("/shiro/logout", "logout");
        map.put("/user.jsp", "authc,roles[user]");
        map.put("/admin.jsp", "authc,roles[admin]");
        map.put("/list.jsp", "user");

        map.put("/**", "authc");

        return map;
    }
}
~~~

