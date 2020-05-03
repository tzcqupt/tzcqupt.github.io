~~~java
layout: post
title: Shiro学习笔记
Author: tzcqupt
tags: [SpringSecurity,JWT]
categories: [SpringSecurity]
comments: true
toc: true
date: 2020-05-03 00:00:00
~~~

# 概念

Shiro架构图

![](D:\git\blogPagesGithub\source\img\shiro架构.jpg)



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

## 依赖

~~~xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-all</artifactId>
    <version>1.2.6</version>
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

![](D:\git\blogPagesGithub\source\img\Shiro认证流程.png)

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

