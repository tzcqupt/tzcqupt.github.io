---
layout: post
title: Spring入门学习笔记
Author: tzcqupt
tags: [Java, Spring]
categories: [Spring]
comments: true
toc: true
date: 2020-04-27 00:00:00
---
# Spring注解系列

## 用于创建对象  

相当于`<bean id="" class="">`

### @Component   

- 作用： 是标明哪个类被扫描进入 Spring IoC 容器。
- 属性：   
    value：指定 bean 的 id。如果不指定 value 属性，默认 bean 的 id是当前类的类名,首字母小写。
    ~~~java
    @Component("user")
    public class User { 
        @Value("1")
        private Long id; 
        @Value("user_name_1")
        private String userName; 
        @Value("note_1")
        private String note;           
            }        
    ~~~

### @Controller @Service @Repository  

作用和属性一样,不过一般分别用于表现层,业务层,持久层的注解

## 用于注入数据    

相当于：`<property name="" ref=""><property name="" value="">`
### @Autowired  

作用：
        自动按照类型（by type）注入。当使用注解注入属性时,set 方法可以省略。它只能注入其他 bean 类型。 也能作用在方法上。 
        当有多个类型匹配时,使用要注入的对象变量名称作为 bean 的 id,在 spring 容器查找,找到了也可以注入成功。找不到就报错。

### @Primary  

标注在类上，告诉 Spring IoC 容器，当发现有多个同样类型的 Bean 时，请优先使用该类进行注入。

### @Qualifier  

- 作用：
    - 在自动按照类型注入的基础之上,再按照 bean 的 id 注入。  
    - 它在给**字段**注入时不能独立使用,必须和`@Autowire`一起使用；
    - 但是给方法参数注入时,可以独立使用(如代码中有多个数据源)。  
- 属性：  
     value：指定 bean 的 id。

### @Resource

- 作用：  
     直接按照 bean 的 id 注入。它也只能注入其他 bean 类型。
- 属性：  
     name：指定 bean 的 id。

### @Value

- 作用：  
     注入基本数据类型和 String 类型数据的。可以读取properties里的字段，配合@PropertySource使用。
- 属性：  
     value：用于指定值
	~~~java
	  @PropertySource("classpath:jdbc.properties")
	  public class JdbcConfigBySpring {

		 @Value("${jdbc.url}")
		 String url;
	  }          
	~~~

## 用于改变作用范围的 

相当于：`<bean id="" class="" scope="">`

### @Scope 

- 作用：  指定 bean 的作用范围。
- 属性：  
    value：指定范围的值。  
    取值：`singleton prototype request session`

## 和生命周期相关      
~~~xml
<bean id="" class="" init-method="" destroy-method="" />
~~~

### @PostConstruct  

用于指定初始化方法。

### @PreDestroy  

用于指定销毁方法。

## 其他注解

### @Configuration

- 作用：  
    用于指定当前类是一个 spring 配置类,当创建容器时会从该类上加载注解。获取容器时需要使用
    AnnotationApplicationContext(有@Configuration 注解的类.class)。
- 属性：  
    value:用于指定配置类的字节码

### @ComponentScan  

- 作用  
    用于指定 spring 在初始化容器时要扫描的包。作用和在 spring 的 xml 配置文件中的
    `<context:component-scan base-package="com.tang"/>`是一样的。
- 属性  
    basePackages：用于指定要扫描的包。和该注解中的 value 属性作用一样。

### @Bean

- 作用    
    该注解只能写在方法上,表明使用此方法创建一个对象,并且放入 spring 容器，不填写名称就默认将方法名称作为Bean名称保存到Ioc容器中。
- 属性   
    name：给当前`@Bean`注解方法创建的对象指定一个名称(即 bean 的 id）。
- 细节 
    使用注解配置方法时,如果方法有参数,spring会去容器中查找有没有可用的bean对象,查找的方式和`Autowired`注解一致。 

### @PropertySource

- 作用  
    用于加载.properties 文件中的配置。例如我们配置数据源时,可以把连接数据库的信息写到properties 配置文件中,就可以使用此注解指定 properties 配置文件的位置。
- 属性  
    - `value[]`：用于指定 `properties`文件位置。如果是在类路径下,需要写上 `classpath:`
    - `ignoreResourceNotFound`: boolean 值，默认为 `false`

### @Import

- 作用  
    用于导入其他配置类,在引入其他配置类时,可以不用再写@Configuration 注解。当然,写上也没问题。

- 属性  
    value[]：用于指定其他配置类的字节码。

- 代码

    ~~~java
    @Configuration
    @ComponentScan(basePackages = "com.tang.spring") 
    @Import({ JdbcConfig.class})
    public class SpringConfiguration { }
        @Configuration
        @PropertySource("classpath:jdbc.properties")
        public class JdbcConfig{
        }
    ~~~

### @Profile

常用来用来定义不同环境的数据库。

#### 标识方法

- @Profile("dev") 
- xml中部分数据标识`<beans profile="test"></beans> `
- 整个xml的环境标识
    ~~~xml
    <beans  xmlns="http://www.springframework.org/schema/beans"  profile="test"> 
    ~~~

#### 启动Profile方式

##### 测试代码上加注解

~~~java
@ActiveProfiles("test")
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {SpringConfig.class})
public class SpringProfileTest {
    
}
~~~

##### JVM参数配置

JAVA_OPTS="-Dspring.profiles.active=test"

##### web.xml中配置

~~~xml
<!--使用 Web 环境参数-->
<context-param> 
    <param-name>spring.profiles.active</param-name>
    <param-value>test</param-value> 
</context-param>
<!--使用 SpringMVC 的 DispatcherServlet 环境参数-->
<init-param> 
    <param-name>spring.profiles.active</param-name>
    <param-value>test</param-value>  
</init-param>
~~~

## 通过注解获取容器

~~~java
ApplicationContext ac = 
        new AnnotationConfigApplicationContext(SpringConfiguration.class);
~~~

# AOP
## Spring中的术语解释

### Joinpoint(连接点)  

所谓连接点是指那些被拦截到的点。在 spring 中,这些点指的是**方法**,因为 spring 只支持方法类型的连接点。

### Pointcut(切入点)  

所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。

### Advice(通知/增强)    

所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。  
通知的类型：前置通知、后置通知、异常通知、最终通知、环绕通知。

### Introduction(引介)  

引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field。

### Target(目标对象)  

代理的目标对象。

### Weaving(织入)  

是指把增强应用到目标对象来创建新的代理对象的过程。  
spring 采用**动态代理**织入，而 AspectJ 采用编译期织入和类装载期织入。

### Proxy(代理)  

一个类被 AOP 织入增强后，就产生一个结果代理类。

### Aspect(切面)  

是切入点和通知（引介）的结合。

## xml配置实例

~~~xml
<!-- 准备工作:导入aop(约束)命名空间 1.配置目标对象 2.配置通知对象 3.配置将通知织入目标对象 -->
<context:component-scan base-package="com.tang.ssm.spring"/>
<!-- 准备工作:导入aop(约束)命名空间 1.配置目标对象 2.配置通知对象 3.配置将通知织入目标对象 -->
<bean name="roleServiceXmlImpl" class="com.tang.ssm.spring.service.impl.RoleServiceXmlImpl"/>
<bean name="myAdvice" class="com.tang.ssm.spring.aop.XmlRoleAspect"/>
<!-- 开启使用注解完成织入 -->
<!--<aop:aspectj-autoproxy/>-->
<!-- XML配置aop -->
<aop:config>
    <!--配置切入点//service下面的所有子包 id 随便起名字-->
    <aop:pointcut expression="execution(* com.tang.ssm.spring.service.impl.RoleServiceXmlImpl.*(..))"
                  id="pc"/>
    <aop:aspect ref="myAdvice">
        <!--指定名为before方法作为前置通知 pointcut-ref为pointcut里面的id-->
        <aop:before method="before"
                    pointcut="execution(* com.tang.ssm.spring.service.impl.RoleServiceXmlImpl.*(..)) and args(obj,sort)"/>
        <!--后置通知,方法出现异常不会调用-->
        <aop:after-returning method="afterWithoutException"
                             pointcut-ref="pc"/>
        <!--环绕通知,目标执行方法前后都会调用-->
        <aop:around method="around" pointcut-ref="pc"/>
        <!--异常拦截通知-->
        <aop:after-throwing method="afterException"
                            pointcut-ref="pc"/>
        <!--后置通知,出现异常也会调用-->
        <aop:after method="after" pointcut-ref="pc"/>
        <!-- 引入增强 -->
        <aop:declare-parents
                types-matching="com.tang.ssm.spring.service.impl.RoleServiceXmlImpl+"
                implement-interface="com.tang.ssm.spring.service.RoleVerifier"
                default-impl="com.tang.ssm.spring.service.impl.RoleVerifierImpl"/>
    </aop:aspect>
</aop:config>
~~~

## 注解配置

~~~java
@EnableAspectJAutoProxy
//主类上开启AOP通知注解
@Component
@Aspect
public class AnnoRoleAspect {
    /**
     * 引入,检测参数是否为空
     */
    @DeclareParents(value = "com.tang.ssm.spring.service.impl.RoleServiceImpl+", defaultImpl = RoleVerifierImpl.class)
    public RoleVerifier roleVerifier;
    @Autowired
    private Log log;

    /**
     * 声明切点，简写表达式
     * execution 代表执行方法的时候会触发。
     * *: 代表任意返回类型的方法。
     * com.tang.ssm.spring.service.impl.*ServiceImpl 代指某些类，可以具体
     * (..) 任意参数
     * within():限制连接点匹配指定的包
     *
     */
    @Pointcut(value = "execution(* com.tang.ssm.spring.service.impl.RoleServiceImpl.*(..))")
    private void pointcut() {
    }

    @Before("execution(* com.tang.ssm.spring.service.impl.RoleServiceImpl.*(..))" + "&& args(obj,sort)")
    public void before(Object obj, int sort) {
        log.info("【前置通知】--参数为  " + sort);
    }

    @AfterReturning("pointcut()")
    public void afterReturning() {
        log.info("【后置通知,方法出现异常不会调用】");
    }

    @Around("pointcut()")
    public Object around(ProceedingJoinPoint proceedingJoinPoint)
        throws Throwable
    {
        log.info("【环绕通知前】");
        Object proceed = proceedingJoinPoint.proceed();
        log.info("【环绕通知后】");
        return proceed;
    }

    @AfterThrowing("pointcut()")
    public void afterException()
    {
        log.info("【异常拦截通知】");
    }

    @After("pointcut()")
    public void after()
    {
        log.info("【后置通知,出现异常也会调用】");
    }
}
~~~

# 事务

## 事务相关知识

###  数据库ACID

- 原子性： 整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像 这个事务从来没被执行过一样。
- 一致性：指一个事务可以改变封装状态（除非它是一个只读的〉。事务必须始终保 持系统处于一致的状态，不管在任何给定的时间并发事务有多少。 
- 隔离性： 它是指两个事务之间的隔离程度。
- 持久性： 在事务完成以后，该事务对数据库所做的更改便持久保存在数据库之中，并不会被回滚。

### 隔离级别
![数据库事务隔离级别](https://gitee.com/tzcqupt/blog-image/raw/master/img/数据库事务隔离级别.png)

###  事务传播行为

| 事务传播行为类型          | 说明                                                         |
| ------------------------- | ------------------------- |
| PROPAGATION_REQUIRED      | 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。 |
| PROPAGATION_SUPPORTS      | 支持当前事务，如果当前没有事务，就以非事务方式执行。         |
| PROPAGATION_MANDATORY     | 使用当前的事务，如果当前没有事务，就抛出异常。               |
| PROPAGATION_REQUIRES_NEW  | 新建事务，如果当前存在事务，把当前事务挂起。                 |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。   |
| PROPAGATION_NEVER         | 以非事务方式执行，如果当前存在事务，则抛出异常。             |
| PROPAGATION_NESTED        | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。 |

## Spring+MyBatis测试事务

### MyBatis配置文件

~~~xml
<!-- mybatis-config.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <mappers>
        <mapper resource="mybatis/RoleMapper.xml"/>
    </mappers>
</configuration>
<!-- RoleMapper.xml-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tang.ssm.spring.dao.RoleMapper">
    <insert id="insertRole" parameterType="com.tang.ssm.spring.pojo.anno.Role">
        insert into t_role (role_name, note)
        values (#{roleName}, #{note})
    </insert>
</mapper>
~~~

### Spring配置文件

~~~xml
<?xml version='1.0' encoding='UTF-8' ?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	   http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">
    <!--整合mybatis的文件，测试事务-->
    <!--启用扫描机制，并指定扫描对应的包-->
    <context:annotation-config/>
    <context:component-scan base-package="com.tang.ssm.spring.*"/>
    <!-- 数据库连接池 -->
    <import resource="applicationContext_dataSource.xml"/>
    <!-- 集成MyBatis -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--指定MyBatis配置文件-->
        <property name="configLocation" value="classpath:/mybatis/mybatis-config.xml"/>
    </bean>

    <!-- 事务管理器配置数据源事务 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 使用注解定义事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!-- 采用自动扫描方式创建mapper bean -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.tang.ssm.spring.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="annotationClass" value="org.springframework.stereotype.Repository"/>
    </bean>
</beans>
~~~
### Java代码
#### 主类
~~~java

@Configuration
@ComponentScan("com.tang.ssm.spring")
@PropertySource("classpath:datasource.properties")
@EnableAspectJAutoProxy
@EnableTransactionManagement
@Import({JdbcConfig.class})
public class SpringConfig {
}
~~~
#### 配置类
~~~java
@Configuration
public class JdbcConfig implements TransactionManagementConfigurer {
   
    /**
     * 创建一个数据源，并存入 spring 容器中
     */
    @Bean("dataSourceByAnno")
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    @Conditional({DataSourceCondition.class})
    public DataSource getDataSource() {
        ...
    }

    @Bean("jdbcTemplateByAnno")
    public JdbcTemplate getJdbcTemplate(@Autowired @Qualifier("dataSourceByAnno") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    /**
     * 注解配置事务管理器
     *
     * @return
     */
    @Bean("transactionManager")
    @Override
    public PlatformTransactionManager annotationDrivenTransactionManager() {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(getDataSource());
        return transactionManager;
    }
}
~~~
#### POJO

~~~java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Role {
    private Long id;
    private String roleName;
    private String note;
}
~~~

#### service实现类

~~~java

@Service
@Primary
public class RoleServiceImpl implements RoleService, ApplicationContextAware {
    
    @Autowired
    private RoleMapper roleMapper;
    @Autowired
    @Qualifier("log")
    private Log log;
    /**
     * 自定义容器对象
     * 解决事务传播失效问题
     */
    private ApplicationContext applicationContext = null;

    @Override
    @Transactional(propagation = Propagation.REQUIRES_NEW,
            isolation = Isolation.READ_COMMITTED)
    public int insertRole(Role role) {
        return roleMapper.insertRole(role);
    }

    /**
     * 自调用问题
     *
     * @param roleList role集合
     * @return
     */
    @Override
    @Transactional(propagation = Propagation.REQUIRED,
            isolation = Isolation.READ_COMMITTED)
    public int insertRoleListSelfCall(List<Role> roleList) {
        int count = 0;
        for (Role role : roleList) {
            try {
                //调用该类的自身方法，产生自调用问题
                this.insertRole(role);
                count++;
            } catch (Exception e) {
                log.error(e);
            }
        }
        return count;
    }

    @Override
    @Transactional(propagation = Propagation.REQUIRED,
            isolation = Isolation.READ_COMMITTED)
    public int insertRoleListNoSelfCall(List<Role> roleList) {
        int count = 0;
        //从容器中获取RoleService对象，实际是一个代理对象
        RoleService service = applicationContext.getBean(RoleService.class);
        for (Role role : roleList) {
            try {
                //调用该类方法，不会产生自调用问题
                service.insertRole(role);
                count++;
            } catch (Exception e) {
                log.error(e);
            }
        }
        return count;
    }

    /**
     * 使用生命周期的接口方法，获取IoC容器
     *
     * @param applicationContext 容器
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
}
~~~
#### 另一个service实现类

~~~java
@Service
public class RoleListServiceImpl implements RoleListService {
    @Autowired
    private RoleService roleService;
    @Autowired
    private Log log;

    @Override
    @Transactional(propagation = Propagation.REQUIRED,
            isolation = Isolation.READ_COMMITTED)
    public int insertRoleList(List<Role> roleList) {
        int count = 0;
        for (Role role : roleList) {
            try {
                count += roleService.insertRole(role);
            } catch (Exception e) {
                log.error(e);
            }
        }
        return count;
    }
}
~~~
#### mapper
~~~java
@Repository
public interface RoleMapper {
    int insertRole(Role role);
}
~~~
#### 测试方法

~~~java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:spring/applicationContext.xml"})
public class SpringTxTest {
    @Autowired
    private Log log;
    @Autowired
    private RoleListService roleListService;
    @Autowired
    private RoleService roleService;

    @Test
    public void testInsertListRole() {
        List<Role> roleList = new ArrayList<>();
        //测试通过roleListService调用roleService批量插入role
        for (int i = 1; i < 5; i++) {
            Role role = new Role();
            role.setRoleName("list-role-name-" + i);
            role.setNote("list-note-name-" + i);
            roleList.add(role);
        }
        int count = roleListService.insertRoleList(roleList);
        //搜索日志，发现创建了5个sqlSession，成功隔离了事务
        log.info("成功批量插入了" + count + "条数据到role表");
    }

    @Test
    public void testInsertListRoleSelfCall() {
        List<Role> roleList = new ArrayList<>();
        //测试通过调用自身的insertRole批量插入，还原自调用问题。
        // 注解@Transational失效
        for (int i = 1; i < 5; i++) {
            Role role = new Role();
            role.setRoleName("list-self-call-role-name-" + i);
            role.setNote("list-self-call-note-name-" + i);
            roleList.add(role);
        }
        try {
            int count = roleService.insertRoleListSelfCall(roleList);
            log.info("成功批量插入了" + count + "条数据到role表");
        } catch (Exception e) {
            //搜索日志，发现只创建了一个sqlSession。多线程下就有问题
            log.error("插入失败，产生了问题" + e);
        }
    }

    @Test
    public void testInsertListRoleNoSelfCall() {
        List<Role> roleList = new ArrayList<>();
        //测试通过调用自身的insertRole批量插入，还原自调用问题。
        // 注解@Transational失效
        for (int i = 1; i < 5; i++) {
            Role role = new Role();
            role.setRoleName("list-no-self-call-role-name-" + i);
            role.setNote("list-no-self-call-note-name-" + i);
            roleList.add(role);
        }
        try {
            int count = roleService.insertRoleListNoSelfCall(roleList);
            log.info("成功批量插入了" + count + "条数据到role表");
        } catch (Exception e) {
            //搜索日志，发现创建了5个sqlSession。
            log.error("插入失败，产生了问题" + e);
        }
    }
}
~~~

## 之前xml配置
~~~xml
    !-- 开启 spring 对注解事务的支持 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    <!-- 引入外部属性文件 -->
    <context:property-placeholder location="classpath:datasource.properties"/>
    <!--1、配置service-->
    <bean id="accountService" class="com.tang.ssm.spring.service.impl.AccountServiceImpl">
        <property name="accountDao" ref="accountDao"/>
    </bean>
    <!--2、配置dao -->
    <bean name="accountDao" class="com.tang.ssm.spring.dao.impl.AccountDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--3、配置dataSource -->
   <bean name="dataSource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}"/>
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒 -->
        <property name="connectionTimeout" value="${jdbc.connectionTimeout}"/>
        <!-- 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟 -->
        <property name="idleTimeout" value="${jdbc.idleTimeout}"/>
        <!-- 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒，参考MySQL
            wait_timeout参数（show variables like '%timeout%';） -->
        <property name="maxLifetime" value="${jdbc.maxLifetime}"/>
        <!-- 连接池中允许的最大连接数。缺省值：10；推荐的公式：((core_count * 2) + effective_spindle_count) -->
        <property name="maximumPoolSize" value="${jdbc.maximumPoolSize}"/>
        <property name="minimumIdle" value="${jdbc.minimumIdle}"/>
    </bean>
    <!-- spring中基于XML的声明式事务控制配置步骤
       1、配置事务管理器
       2、配置事务的通知
               此时我们需要导入事务的约束 tx名称空间和约束，同时也需要aop的
               使用tx:advice标签配置事务通知
                   属性：
                       id：给事务通知起一个唯一标识
                       transaction-manager：给事务通知提供一个事务管理器引用
       3、配置AOP中的通用切入点表达式
       4、建立事务通知和切入点表达式的对应关系
       5、配置事务的属性
              是在事务的通知tx:advice标签的内部

    -->
    <!--4、配置事务管理器-->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!--注入dataSource-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--5、配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 配置事务的属性
                isolation：用于指定事务的隔离级别。默认值是DEFAULT，表示使用数据库的默认隔离级别。
                propagation：用于指定事务的传播行为。默认值是REQUIRED，表示一定会有事务，增删改的选择。查询方法可以选择SUPPORTS。
                read-only：用于指定事务是否只读。只有查询方法才能设置为true。默认值是false，表示读写。
                timeout：用于指定事务的超时时间，默认值是-1，表示永不超时。如果指定了数值，以秒为单位。
                rollback-for：用于指定一个异常，当产生该异常时，事务回滚，产生其他异常时，事务不回滚。没有默认值。表示任何异常都回滚。
                no-rollback-for：用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时事务回滚。没有默认值。表示任何异常都回滚。
        -->
            <tx:method name="*" read-only="false" propagation="REQUIRED"/>
            <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
        </tx:attributes>
    </tx:advice>
    <!-- 6、配置aop-->
    <aop:config>
        <!--配置切入点表达式-->
        <aop:pointcut id="pt" expression="execution(* com.tang.ssm.spring.service.impl.*.*(..))"
        />
        <!--建立切入点表达式和事务通知的对应关系 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
    </aop:config>
~~~
## 事务编程注意事项
1. 和数据库相关的操作写一个方法，另外无关的操作，写在controller里面，以便spring释放数据库事务

	~~~java
	@RequestMapping (”/ addRole" ) 
	@ResponseBody 
	public Role addRole(Role role) { 
	//在service中完成，方便spring释放事务
		roleService.insertRole(role); 
		doSomethingFile(); 
		return role; 
		}
	~~~
2. 当一个 Controller 使用 Service 方法时，如果这个 Service 标注有＠Transactional，那么 它就会启用一个事务，而一个 Service 方法完成后，它就会释放该事务。**controller中只调用一次同样的service一次**
3. 前后有多个不同的service处理任务时，捕捉到异常后，应该抛出。方便Spring捕获异常，进行事务管理，方便回滚事务。**自行抛出异常**

    ~~~java
    @Override
        @Transactional(propagation = Propagation.REQUIRED,
                isolation = Isolation.READ_COMMITTED)
        public int doTransaction(TransactionBean trans)  {
            int resutl = 0 ; 
            try { 
            //减少库存 
            int result = prudoctService.decreaseStock( trans.getProductid, 
            trans.getQuantity()) ; 
            //如果减少库存成功则保存记录 
            if (result >0 ) { transactionService.save(trans) }; 
                }catch(Exception ex) { 
                //自行处理
                log.info(ex);
                //自行抛出异常，让 Spring 事务管理流程获取异常，进行事务管理 
                throw new RuntimeException(ex);
                 }
            }
    ~~~

# 整合Spring Jdbc

## xml配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--1.将连接池放入spring容器 -->
    <import resource="applicationContext_dataSource.xml"/>
    <!--2.将accountDao放入spring容器管理,dao里面放入jdbcTemplate -->
    <bean name="accountDao" class="com.tang.ssm.spring.dao.impl.AccountDaoXmlImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
    <!-- 配置一个数据库的操作模板：JdbcTemplate -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
~~~

## 测试代码

实体类和数据库的对应关系参考**整合SpringData JPA章节**

~~~java
@Test
public void testQueryForList()
{
    String sql = "select * from stu;";
    List<Map<String, Object>> list = template.queryForList(sql);
    Print.printList(list, 1);
}

/**
     * @author tzcqupt
     * <p>Description 查询的结果封装为List<Student>类型</p>
     * @date 2019/9/4
     **/
@Test
public void testQueryForStudentList()
{
    String sql = "select * from stu;";
    List<Student> list = template.query(sql, (resultSet, i) -> {
        String name = resultSet.getString("name");
        String address = resultSet.getString("address");
        return new Student(name, address);
    });
    Print.printList(list, 1);
}

/**
     * @author tzcqupt
     * <p>Description 查询的结果自动封装为List<Student>类型</p>
     * @date 2019/9/4
     **/
@Test
public void testQueryForBeanList()
{
    String sql = "select * from stu;";
    List<Student> list = template.query(sql, new BeanPropertyRowMapper<>(Student.class));
    Print.printList(list, 1);
}

/**
     * @author tzcqupt
     * <p>Description 查询总记录数 </p>
     * @date 2019/9/4
     **/

@Test
public void testQueryTotal()
{
    String sql = "select count(id) from stu;";
    Long total = template.queryForObject(sql, Long.class);
    System.out.println("共有" + total + "条记录");
}
}

~~~



# 整合SpringData JPA

## 相关注解解释

>所有的注解都是使用JPA的规范提供的注解，所以在导入注解包的时候，一定要导入javax.persistence下的

### @Entity

指定当前类是实体类。

### @Table 

指定实体类和表之间的对应关系。

属性：
name：指定数据库表的名称

### @Id 

指定当前字段是主键。

### @GeneratedValue 

指定主键的生成方式。

- 属性：strategy ：指定主键生成策略。

- 值:
  - GenerationType.IDENTITY由数据库自动生成（主要是自动增长型)

  - SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。

    ~~~java
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,generator="payablemoney_seq")
    @SequenceGenerator(name="payablemoney_seq", sequenceName="seq_payment")
    private Long id;
    ~~~

  - GenerationType.AUTO主键由程序控制

  - TABLE 使用一个特定的数据库表格来保存主键

    ~~~java
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator="payablemoney_gen")
    @TableGenerator(name = "pk_gen",
    table="tb_generator",
    pkColumnName="gen_name",
    valueColumnName="gen_value",
    pkColumnValue="PAYABLEMOENY_PK",
    allocationSize=1
    )
    private Long id;
    ~~~

### @Column

指定实体类属性和数据库表之间的对应关系

属性：
* name：指定数据库表的列名称。
* unique：是否唯一
* nullable：是否可以为空
* inserttable：是否可以插入
* updateable：是否可以更新
* columnDefinition: 定义建表时创建此列的DDL
* secondaryTable: 从表名。如果此列不建在主表上（默认建在主表），该属性定义该列所在从表的名字搭建开发环境

## Spring整合SpringDataJpa

~~~xml

<!-- 1.dataSource 配置数据库连接池-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="jdbcUrl"
                  value="jdbc:mysql://localhost:3306/ssm_learn?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=true"/>
        <property name="user" value="root"/>
        <property name="password" value="root123"/>
    </bean>

    <!-- 2.配置entityManagerFactory -->
    <bean id="entityManagerFactory"
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="com.tang.springdata.jpa.pojo"/>
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
        </property>
        <!--JPA的供应商适配器-->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <property name="generateDdl" value="false"/>
                <property name="database" value="MYSQL"/>
                <property name="databasePlatform" value="org.hibernate.dialect.MySQLDialect"/>
                <property name="showSql" value="true"/>
            </bean>
        </property>
        <property name="jpaDialect">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect"/>
        </property>
    </bean>


    <!-- 3.事务管理器-->
    <!-- JPA事务管理器  -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <!-- 整合spring data jpa-->
    <jpa:repositories base-package="com.tang.springdata.jpa.dao"
                      transaction-manager-ref="transactionManager"
                      entity-manager-factory-ref="entityManagerFactory"/>

    <!-- 4.txAdvice-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="find*" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <!-- 5.aop-->
    <aop:config>
        <aop:pointcut id="pointcut" expression="execution(* com.tang.springdata.jpa.dao.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
    </aop:config>
    <!--开启注解扫描-->
    <context:component-scan base-package="com.tang.springdata.jpa"/>
~~~

## 一对多查询配置

### 实体类

~~~java
@Data
@Entity
@Table(name = "cst_customer")
@EqualsAndHashCode
public class Customer
{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId;

    @Column(name = "cust_name")
    private String custName;

    @Column(name = "cust_source")
    private String custSource;

    @Column(name = "cust_industry")
    private String custIndustry;

    @Column(name = "cust_level")
    private String custLevel;

    @Column(name = "cust_address")
    private String custAddress;

    @Column(name = "cust_phone")
    private String custPhone;

    //配置客户和联系人的一对多关系
    /*@OneToMany(targetEntity=LinkMan.class)
    @JoinColumn(name="lkm_cust_id",referencedColumnName="cust_id")*/
    // 放弃外键维护配置权
    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL)
    private Set<LinkMan> linkMans = new HashSet<>();

    @Override
    public String toString() {
        return "Customer{" +
                "custId=" + custId +
                ", custName='" + custName + '\'' +
                ", custSource='" + custSource + '\'' +
                ", custIndustry='" + custIndustry + '\'' +
                ", custLevel='" + custLevel + '\'' +
                ", custAddress='" + custAddress + '\'' +
                ", custPhone='" + custPhone + '\'' +
                ", linkMans=" + linkMans +
                '}';
    }
}
//一对多中覆盖toString、equals、hashCode方法。避免循环调用导致的堆栈溢出
@Data
@Entity
@Table(name = "cst_linkman")
public class LinkMan
{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "lkm_id")
    private Long lkmId;

    @Column(name = "lkm_name")
    private String lkmName;

    @Column(name = "lkm_gender")
    private SexEnum lkmGender;

    @Column(name = "lkm_phone")
    private String lkmPhone;

    @Column(name = "lkm_mobile")
    private String lkmMobile;

    @Column(name = "lkm_email")
    private String lkmEmail;

    @Column(name = "lkm_position")
    private String lkmPosition;

    @Column(name = "lkm_memo")
    private String lkmMemo;

    //多对一关系映射：多个联系人对应客户
    @ManyToOne(targetEntity = Customer.class)
    @JoinColumn(name = "lkm_cust_id", referencedColumnName = "cust_id")
    private Customer customer;//用它的主键，对应联系人表中的外键

    /**
     * 覆盖toString方法。避免两个一直相互调用
     * @return
     */
    @Override
    public String toString() {
        return "LinkMan{" +
                "lkmId=" + lkmId +
                ", lkmName='" + lkmName + '\'' +
                ", lkmGender='" + lkmGender + '\'' +
                ", lkmPhone='" + lkmPhone + '\'' +
                ", lkmMobile='" + lkmMobile + '\'' +
                ", lkmEmail='" + lkmEmail + '\'' +
                ", lkmPosition='" + lkmPosition + '\'' +
                ", lkmMemo='" + lkmMemo + '\'' +
                '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof LinkMan)) {
            return false;
        }
        LinkMan linkMan = (LinkMan) o;
        return Objects.equals(getLkmId(), linkMan.getLkmId()) &&
                Objects.equals(getLkmName(), linkMan.getLkmName()) &&
                Objects.equals(getLkmGender(), linkMan.getLkmGender()) &&
                Objects.equals(getLkmPhone(), linkMan.getLkmPhone()) &&
                Objects.equals(getLkmMobile(), linkMan.getLkmMobile()) &&
                Objects.equals(getLkmEmail(), linkMan.getLkmEmail()) &&
                Objects.equals(getLkmPosition(), linkMan.getLkmPosition()) &&
                Objects.equals(getLkmMemo(), linkMan.getLkmMemo());

    }

    @Override
    public int hashCode() {
        return Objects.hash(getLkmId(), getLkmName(), getLkmGender(), getLkmPhone(), getLkmMobile(), getLkmEmail(), getLkmPosition(), getLkmMemo());
    }
}
~~~

### 测试代码

~~~java
@Test
public void testOneToMany() {
    Customer customer = new Customer();
    customer.setCustName("TBD云集中心");
    customer.setCustLevel("VIP客户");
    customer.setCustSource("网络");
    customer.setCustIndustry("商业办公");
    customer.setCustAddress("昌平区北七家镇");
    customer.setCustPhone("010-84389340");

    LinkMan linkMan = new LinkMan();
    linkMan.setLkmName("TBD联系人");
    linkMan.setLkmGender(SexEnum.MALE);
    linkMan.setLkmMobile("13811111111");
    linkMan.setLkmPhone("010-34785348");
    linkMan.setLkmEmail("98354834@qq.com");
    linkMan.setLkmPosition("老师");
    linkMan.setLkmMemo("还行吧");

    customer.getLinkMans().add(linkMan);
    linkMan.setCustomer(customer);
    customerDao.save(customer);
    linkManDao.save(linkMan);
}
~~~

### 总结

1. 实体类的配置要和数据库对应上
2. 一对多中，*一*方这边要方便外键维护
3. 要重写**多**的一方equals、hashCode、ToString方法，去掉**一**方这边的属性

## 对多对查询配置

### 实体类

~~~java
@Data
@Entity
@Table(name = "t_mm_user")
public class SysUser {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "user_name")
    private String userName;
    @Column(name = "real_name")
    private String realName;
    private SexEnum sex;
    private String mobile;
    private String email;
    private String note;
    /**
     * 对角色 对对多关联
     */
    @ManyToMany(mappedBy = "userList")
    //放弃对中间表的维护，解决保存中的主键冲突问题
    private List<SysRole> roleList = new ArrayList<>();

    /**
     * 覆盖toString方法,避免两者互相调用，陷入循环，导致栈溢出
     *
     * @return
     */
    @Override
    public String toString() {
        return "mmUser{" +
                "id=" + id +
                ", userName='" + userName + '\'' +
                ", realName='" + realName + '\'' +
                ", sex=" + sex +
                ", mobile='" + mobile + '\'' +
                ", email='" + email + '\'' +
                ", note='" + note +
                '}';
    }
}
@Data
@Entity
@Table(name = "t_mm_role")
public class SysRole implements Serializable {
    private static final long serialVersionUID = 473635534420062409L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "role_name")
    private String roleName;
    private String note;
    /**
     * 关联用户信息，多对多
     CascadeType.ALL 开启级联更新
     */
    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "t_mm_user_role",//中间表的名称
            //中间表t_mm_user_role字段关联sys_role表的主键字段role_id
            joinColumns={@JoinColumn(name="role_id",referencedColumnName="id")},
            //中间表user_role_rel的字段关联sys_user表的主键user_id
            inverseJoinColumns={@JoinColumn(name="user_id",referencedColumnName="id")}
    )
    private List<SysUser> userList = new ArrayList<>();

    @Override
    public String toString() {
        return "mmRole{" +
                "id=" + id +
                ", roleName='" + roleName + '\'' +
                ", note='" + note + '\'' +
                ", mmUserList=" + userList +
                '}';
    }
}
~~~

### 测试代码

~~~java
@Test
public void testManyToMany() {
    SysUser sysUser = new SysUser();
    sysUser.setUserName("多对多测试用户1");
    sysUser.setEmail("tang@tang.com");
    sysUser.setMobile("13288888888");
    sysUser.setRealName("tang");
    sysUser.setNote("多对多测试note");
    sysUser.setSex(SexEnum.MALE);
    SysRole sysRole = new SysRole();
    sysRole.setRoleName("多对多测试角色1");
    sysRole.setNote("多对多测试note");
    //建立关联关系
    ArrayList<SysUser> userList = new ArrayList<>();
    userList.add(sysUser);
    ArrayList<SysRole> roleList = new ArrayList<>();
    roleList.add(sysRole);

    sysRole.setUserList(userList);
    sysUser.setRoleList(roleList);
    //保存
    sysRoleDao.save(sysRole);
    sysUserDao.save(sysUser);
}
~~~