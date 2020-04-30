---
layout: post
title: SpringBoot学习笔记
date: 2020-04-27
Author: tzcqupt
tags: [Java, SpringBoot]
comments: true
toc: true
---

# SpringBoot测试整合

## 单元测试

### 依赖

~~~xml
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-test</artifactId>
     <scope>test</scope>
</dependency>
~~~

### 使用

~~~java
@RunWith(SpringRunner.class) //底层用junit  SpringJUnit4ClassRunner
@SpringBootTest(classes={XdclassApplication.class})//启动整个springboot工程
public class SpringBootTests { }
~~~

## 测试进阶-MockMvc

~~~java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
@Slf4j
public class MockTest {
    @Autowired
    protected MockMvc mockMvc;
    @Test
    public void testGetUser() throws Exception {
        MvcResult result = mockMvc.perform(
                MockMvcRequestBuilders.get("/user/1")
                        .contentType(MediaType.APPLICATION_JSON_UTF8))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
        log.info(result.getResponse().toString());
    }
}
~~~

# 启动方式

ide启动/Jar包方式/war包方式

## Jar包方式

`pom.xml`新增如下配置。执行`mvn install`后，会在`target`目录下生成jar包，使用`java -jar springboot-xxx.jar` 即可运行

~~~xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <!-- spring-boot:run 中文乱码解决 -->
                    <jvmArguments>-Dfile.encoding=UTF-8</jvmArguments>
                </configuration>
            </plugin>
        </plugins>
    </build>
~~~

## War包方式

1. 修改pom.xml里的打包方式

2. 修改启动类，使之继承SpringBootServletInitializer

   ~~~java
   @SpringBootApplication
   public class MyApplicationView extends SpringBootServletInitializer {
   
       @Override
       protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
           return application.sources(MyApplicationView.class);
       }
   
       public static void main(String[] args) throws Exception {
           SpringApplication.run(MyApplicationView.class, args);
       }
   
   }
   ~~~

3. 部署在tomcat中，启动tomcat，访问项目即可。

# 拦截器

## 过滤器Filter

简单理解 人--->检票员（filter）---> 景点

### SpringBoot启动默认加载的Filter

requestContextFilter、hiddenHttpMethodFilter、formContentFilter、characterEncodingFilter

### Filter优先级

```java
Ordered.HIGHEST_PRECEDENCE;
Ordered.LOWEST_PRECEDENCE;
int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;
//低位值意味着更高的优先级
//自定义Filter，避免和默认的Filter优先级一样，不然会冲突
```

### 自定义Filter

1. 使用`Servlet3.0`的注解进行配置
2. 启动类里面增加` @ServletComponentScan`，进行扫描
3. 新建一个Filter类，`implements Filter`，并实现对应的接口
4. `@WebFilter `标记一个类为filter，被Spring进行扫描注册到容器 
    	urlPatterns：拦截规则，支持正则
5. 控制chain.doFilter的方法的调用，来实现是否通过放行  
   	不放行，web应用`resp.sendRedirect("/index.html");`
    场景：权限控制、用户登录(非前端后端分离场景)等

### 参考代码

~~~java
@WebFilter(urlPatterns = "/api/*", filterName = "loginFilter")
public class LoginFilter  implements Filter{	
	 /**
	  * 容器加载的时候调用
	  */
	  @Override
      public void init(FilterConfig filterConfig) throws ServletException {
          System.out.println("init loginFilter");
      }
	  
	  /**
	   * 请求被拦截的时候进行调用
	   */
      @Override
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    	  System.out.println("doFilter loginFilter");
    	  
    	  HttpServletRequest req = (HttpServletRequest) servletRequest;
          HttpServletResponse resp = (HttpServletResponse) servletResponse;
          String username = req.getParameter("username");
          
          if ("xdclass".equals(username)) {
        	  filterChain.doFilter(servletRequest,servletResponse);
          } else {
        	  resp.sendRedirect("/index.html");
        	  return;
          }    

      }

      /**
       * 容器被销毁的时候被调用
       */
      @Override
      public void destroy() {
    	  System.out.println("destroy loginFilter");
      }
~~~

## 监听器Listener

常用的监听器 `ServletContextListener、HttpSessionListener、ServletRequestListener`

~~~java
@WebListener
public class RequestListener implements ServletRequestListener {

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        System.out.println("======requestDestroyed========");
    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        System.out.println("======requestInitialized========");

    }
~~~

## 拦截器Interceptor

### 自定义拦截器

~~~java
@Slf4j
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, Object handler)
            throws Exception {
        log.info("处理器前方法");
        // 返回true，不会拦截后续的处理
        // 返回false 会拦截后面的一起处理
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response, Object handler,
                           ModelAndView modelAndView) throws Exception {
        log.info("处理器后方法");
    }
    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        log.info("处理器完成方法");
    }
}
~~~

### 配置拦截器

~~~java
//启动或者配置类实现 WebMvcConfigurer
@SpringBootApplication
public class MyApplicationView implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //注册拦截器到Spring MVC机制，然后它会返回一个拦截器注册
        InterceptorRegistration interceptorRegistration = registry.addInterceptor(new MyInterceptor());
        // 指定拦截匹配模式，限制拦截器拦截请求
        interceptorRegistration.addPathPatterns("/singleInterceptor/*");
    }
}
~~~

### 拦截规则

按照注册顺序进行拦截，先注册，先被拦截。

### 常见问题

1. 是否有加`@Configuration ` 不是启动类上配置，配置类上需要加该注解
2. 拦截路径是否有问题 **  和 * 
3. 拦截器最后路径一定要 “/**”， 如果是目录的话则是 /*/

## Filter和Intercepetor区别

1. Filter是基于函数回调 doFilter()，而Interceptor则是基于AOP思想
2. Filter在只在Servlet前后起作用，而Interceptor够深入到方法前后、异常抛出前后等
3. Filter依赖于Servlet容器即web应用中，而Interceptor不依赖于Servlet容器所以可以运行在多种环境。
4. Filter在接口调用的生命周期里，Interceptor可以被多次调用，而Filter只能在容器初始化时调用一次。

## Filter和Interceptor的执行顺序

过滤前->拦截前->action执行->拦截后->过滤后

# 整合任务

## 整合定时任务

1. 启动类加 `@EnableScheduling`开启定时任务，自动扫描

2. 定时任务类加`@Component`被容器扫描

3. 定时执行的方法加上注解 @Scheduled定期执行一次

   1. `@Scheduled(cron = "0/5 * * * * ? ")` corn表达式参考[http://cron.qqe2.com/](http://cron.qqe2.com/)

   2.  @Scheduled(fixedRate = 2000) fixedRate：固定速率，单位是毫秒。表示一个任务开始后间隔多长时间才执行下一个执行

      > 比如设置 fixedRate=5*1000。那么如果一个任务8:00:00开始执行，不管什么时候执行结束，该任务的下一次执行的时间是8:00:05

   3.  @Scheduled(fixedDelay = 1000) fixedDelay：固定延迟，单位是毫秒。表示一个任务执行结束后间隔多长时间才执行下一个执行

      >比如设置 fixedDelay=5*1000。那么如果一个任务8:00:00开始执行，8:00:01执行结束，那么该任务的下一次执行的时间是8:00:06

   4.  @Scheduled(fixedDelayString = "${eventTracking.delayFixed}") 字符串样式，可以整合到application.properties中，完成从配置文件读取

   5. initialDelay：服务启动后立即延迟指定时间执行，单位是毫秒，和其他的参数配合使用

      > @Scheduled(initialDelay = 10000,fixedDelay = 10000)

## 整合异步任务

1. 启动类加`@EnableAsync`注解开启功能，自动扫描
2. 定义异步任务类并使用`@Component`标记组件被容器扫描,异步方法加上`@Async`,也可以直接加在类上面
3. 增加Future<String> 返回结果 AsyncResult<String>("task执行完成");  
4. 如果需要拿到结果 需要判断全部的 task.isDone()

### 代码

~~~java
@Component
@Slf4j
@Async
public class AsyncTask {

    public void task1() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(1000L);
        long end = System.currentTimeMillis();
        log.info("任务1耗时= " + (end - start) + "毫秒");
    }

   //...
    public Future<String> task8() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(4000L);
        long end = System.currentTimeMillis();
        log.info("任务8耗时= " + (end - start) + "毫秒");
        return new AsyncResult<>("任务8");
    }
}
//controller
@RestController
@RequestMapping("async")
@Slf4j
public class AsyncTaskController {
    @Autowired
    private AsyncTask task;

    /**
     * 测试任务
     *
     * @return json
     * @throws InterruptedException
     */
    @GetMapping("test")
    public ResponseEntity<Long> asyncTask() throws InterruptedException {
        //http://localhost:8080/async/test
        long start = System.currentTimeMillis();
        task.task1();
        Future<String> task8 = task.task8();
         for (; ; ) {
            if (task8.isDone()) {
                break;
            }
        }
        long end = System.currentTimeMillis();
        long result = end - start;
        log.info("测试任务全部完成，耗时" + result + "毫秒");
        return ResponseEntity.ok(result);
    }
}

~~~

# 多环境配置

在classpath下新建`application-test.properties`/`application-dev.properties`或者`yml`文件

在`application.properties`中通过`spring.profiles.active=dev`来指定采用哪个环境，如果不指定，则采用当前文件中的环境配置。

# 问题记录

整合springmvc+jsp时，页面跳转时，当请求url地址栏，有两个url时，跳转失败。如`http://localhost:9999/user-add`跳转成功，而`http://localhost:9999/user/add`跳转失败，猜测和rest风格有关。

其他页面跳转失败，猜测工程里面引入了thymeleaf jar包。去掉即可。