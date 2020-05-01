layout: post
title: ElasticSearch学习笔记
Author: tzcqupt
tags:
  - Java
  - ElasticSearch
comments: true
toc: true
date: 2020-04-27 00:00:00
---
# 安装与启动

1. windows解压即可。版本5.6.8  需要jdk1.8及以上

2. 图形界面[elasticsearch-head-master](https://github.com/mobz/elasticsearch-head)js编写,需要运行在node.js上

3. 安装nodejs,在该图形界面软件里面执行`cnpm install`，并执行`cnpm install -g grunt-cli`

4. 执行`grunt server`，访问[http://localhost:9100](http://localhost:9100)

5. 在ElasticSearch里config文件夹的elasticsearch.yml文件里增加下面两行,允许跨域访问

   ~~~yml
   http.cors.enabled: true
   http.cors.allow-origin: "*"
   ~~~

6. 重新启动ES服务,在图形界面里面http://localhost:9100/连接es服务进行管理。

7. 官方推荐的图形界面`Kibana`,支持restful风格调试,Windows下安装需和ElasticSearch版本保持一致

8. `Kibana`配置,在安装目录下的config目录,修改`kibana.yml`文件，修改elasticSearch服务器的地址.

   ~~~yaml
   elasticsearch.url: "http://127.0.0.1:9200/"
   ~~~

9. 进入安装目录下的bin目录,运行`kibana.bat`即可。

10. 安装ik分词器。解压`elasticsearch-analysis-ik-5.6.8.zip`到`D:\soft\elasticsearch-5.6.8\plugins\ik-analyzer`里面,重启es即可。

11. 在kibana控制台,点击左侧菜单的`DevTools`即可进入，输入下面请求验证ik分词器效果。

    ~~~json
    POST _analyze
    {
      "analyzer": "ik_max_word",
      "text":     "我是中国人"
    }
    ~~~
   

12. win10下配置应用开机启动

    ~~~bash
    @echo off
    start  "es head plugin" "C:\Windows\System32\cmd.exe" 
    d:
    cd D:\soft\kibana-5.6.8-windows-x86\bin
    kibana.bat
    taskkill /f /im cmd.exe
    exit
    %放入这里面开机启动C:\Users\tzcqupt\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup%
    ~~~

# 相关概念

~~~xml
传统数据库==>Databases==>Tables==>Rows==>Colums
ElasticSearch==>Indices(索引库)==>Types==>Documents==>Fields
~~~

## **映射 mapping**

mapping是处理数据的方式和规则方面做一些限制,如某个字段的数据类型、默认值、分析器、是否被索引等处理es里面数据的一些使用规则设置也叫做映射。理解为定义的数据库的表结构定义。

## **桶**

1. 指满足特定条件的文档集合。
2. 当聚合开始被执行，每个文档里面的值通过计算来决定符合哪个桶的条件，如果匹配到，文档将放入相应的桶并接着开始聚合操作。
3. 桶可以被嵌套在其他桶里面

## **指标**

1. 桶能让我们划分文档到有意义的集合，但是最终我们需要的是对这些桶内的文档进行一些指标的计算。分桶是一种达到目的地的手段：它提供了一种给文档分组的方法来让我们可以计算感兴趣的指标。

2. 大多数指标是简单的数学运算（如：最小值、平均值、最大值、汇总），这些是通过文档的值来计算的。

## **桶和指标的组合**

　　聚合是由桶和指标组成的。聚合可能只有一个桶，可能只有一个指标，或者可能两个都有。也有可能一些桶嵌套在其他桶里面。

# 客户端操作

利用postman发送http请求给ElasticSearch，发送的请求为restful形式。GET/POST/DELETE/PUT


## 创建

![postman发送请求示例](..\img\postman发送json数据创建索引给ElasticSearch.png)

## 新增文档


![新增文档](..\img\ES测试新增文档.png)


POST http://localhost:9200/blog/airticle/1 

> artile后为**_id**的值==>某篇具体文档的id值,参见localhost:9100

## 查询

### 关键词查询

POST http://localhost:9200/blog/airticle/_search

~~~json
{
	"query":{
		"term":{
			"title":"档"
		}
	}
}
~~~

> 目前使用的是标准分词器,只能用一个汉字的查询

### query_string查询

POST http://localhost:9200/blog/airticle/_search

~~~json
{
	"query":{
		"query_string":{
			"default_field":"title",
			"query":"Java"
		}
}
}
~~~

### 标准分析器的效果

在postman工具中查看效果[http://localhost:9200/_analyze?analyzer=standard&text=Java程序员](http://localhost:9200/_analyze?analyzer=standard&text=Java程序员)

中文分析器IKAnylazer,将压缩包放入ElasticSearch软件里的plugins文件夹下

使用ik分词器，两种算法 ik_smart/ik_max_word 最少切分/最细粒度划分

`analyzer=standard`==>`analyzer=ik_smart`

# SpringBoot整合的单机配置

~~~yaml
spring: 
    data:
        elasticsearch:
          cluster-name: elasticsearch
          cluster-nodes: 127.0.0.1:9300
~~~



# 集群配置

每个客户端里面最开始不能有data文件夹（存放索引的相关数据）

elasticsearch.yml配置

~~~yml
#节点1的配置信息
#集群名称,保证唯一
cluster.name: my-elasticsearch
#节点名称,必须不一样
node.name: node-1
#必须为本机的ip地址
network.host: 127.0.0.1
#服务端口号,集群如果在同一个机器下必须不一样
http.port: 9201
#集群间通信端口号,在同一机器下必须不一样
transport.tcp.port: 9301
#设置集群自动发现机器ip集合
discovery.zen.ping.unicast.hosts: ["127.0.0.1:9301","127.0.0.1:9302","127.0.0.1:9303"]
~~~

# **Java客户端管理ES**

## pom.xml配置

~~~xml
		<dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>5.6.8</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>transport</artifactId>
            <version>5.6.8</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.24</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.21</version>
        </dependency>
		<dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.12</version>
        </dependency>
~~~

## java代码

~~~java
/*
	创建索引步骤：
        1、创建一个Settings对象，相当于一个配置信息。主要配置集群的名称
        2、创建一个客户端Client对象
        3、使用Client对象创建一个索引库
        4、关闭Client对象
	设置Mappings步骤
        1、创建Settings对象
        2、创建Client对象
        3、创建一个mapping信息,json数据,可以为字符串,可以是XcontentBuilder对象
        4、使用client对象向es服务器发送mapping信息
        5、关闭client对象
	添加文档步骤
        1、创建Settings对象
        2、创建Client对象
        3、创建一个文档对象,可以为json格式的字符串（利用jackson==>ObjectMapper的writeValueAsString方法）,可以是XcontentBuilder对象
        4、使用Client对象把文档添加到索引库中
        5、关闭Client对象
	查询
        根据id搜索
        根据term查询（关键词）
        QueryString查询方式（带分析的查询）
        步骤：
            1、利用创建的Settings对象创建Client对象
            2、创建一个查询对象，使用QueryBuilders工具类创建QueryBuilder对象，（可选）设置分页信息
            3、使用client执行查询（先设置参数）
            4、得到查询结果
            5、取查询结果的总记录数
            6、取查询结果列表
            7、关闭Client对象
    查询结果高亮显示
    	1、设置高亮显示的字段
    	2、设置高亮显示的前后缀，如<em></em>
    	3、在client对象执行查询之前,设置高亮显示的信息
    	4、遍历结果列表时可以从结果中取高亮结果
	
*/ 
private Settings settings = null;
private TransportClient client = null;
@Before
public void init() throws Exception{
 		Settings settings = Settings.builder()
            .put("cluster.name", "my-elasticsearch")
            .build();
        TransportClient client = new PreBuiltTransportClient(settings);
        client.addTransportAddress(new InetSocketTransportAddress(
            InetAddress.getByName("127.0.0.1"), 9301));
        client.addTransportAddress(new InetSocketTransportAddress(
            InetAddress.getByName("127.0.0.1"), 9302));
        client.addTransportAddress(new InetSocketTransportAddress(
            InetAddress.getByName("127.0.0.1"), 9303)); 
       }
public void method(){
    //创建索引库
    client.admin().indices().prepareCreate("java_test")
            //执行操作
            .get();
    //设置mappings
    XContentBuilder builder = XContentFactory.jsonBuilder()
            .startObject()
                .startObject("article")
                    .startObject("properties")
                        .startObject("id")
                            .field("type","long")
                            .field("store","true")
                        .endObject()
                        .startObject("title")
                            .field("type","text")
                            .field("store","true")
                            .field("index","true")
                            .field("analyzer","ik_smart")
                        .endObject()
                        .startObject("content")
                            .field("type","text")
                            .field("store","true")
                            .field("index","true")
                            .field("analyzer","ik_smart")
                        .endObject()
                    .endObject()
                .endObject()
            .endObject();
        // 使用client对象把mapping信息设置到索引库中
        client.admin().indices().
            //设置要做映射的索引
            preparePutMapping("java_test")
            //设置要映射的type
            .setType("article")
            // mapping信息,可以是XContentBuilder对象,也可以是json格式的字符串
            .setSource(builder)
            //执行操作
            .get();
    // 添加文档对象1--利用XContentBuilder对象
        XContentBuilder builder = XContentFactory.jsonBuilder()
            .startObject()
            .field("id", "1")
            .field("title", "elasticsearch学习")
            .field("content", "创建文档对象添加文档")
            .endObject();
        client.prepareIndex()
            //设置索引名称
            .setIndex("java_test")
            // 设置type
            .setType("article")
            // 设置id,可以不设置,会自动生成一个随机id
            .setId("1")
            //设置文档信息
            .setSource(builder)
            //执行操作
            .get();
    // 添加文档对象2--利用jackson将实体类对象转换为json
    	Article article = new Article();
        article.setId(2l);
        article.setTitle("jackson保存文档对象");
        article.setContent("导入jackson jar包,使用ObjectMapper的writeValueAsString方法");
        ObjectMapper objectMapper = new ObjectMapper();
        String json = objectMapper.writeValueAsString(article);
        client.prepareIndex()
            //设置索引名称
            .setIndex("java_test")
            // 设置type
            .setType("article")
            // 设置id,可以不设置,会自动生成一个随机id
            .setId("2")
            //设置文档信息
            .setSource(json, XContentType.JSON)
            //执行操作
            .get();
    // 查询--构建不同的查询对象
    // 1、通过id查询
    IdsQueryBuilder queryBuilder = QueryBuilders.idsQuery().addIds("1", "2");
    // 2、根据关键词查询
    QueryBuilder termQueryBuilder = QueryBuilders.termQuery("title", "jackson");
    // 3、通过queryString查询 可以设置默认的搜索域
    //3.1 全局查询
    QueryBuilder queryStringBuilder = QueryBuilders.queryStringQuery("无效数据");
    // 3.2 设置默认的搜索域
    QueryBuilder defaultQueryBuilder = QueryBuilders
            .queryStringQuery("jar")
            .defaultField("title");
    // 搜索结果高亮显示
    // 配置高亮
    HighlightBuilder highlightBuilder = new HighlightBuilder();
        // 高亮显示的字段
        highlightBuilder.field("title");
        highlightBuilder.preTags("<em>");
        highlightBuilder.postTags("</em>");
    //执行查询
        SearchResponse searchResponse = client.prepareSearch("java_test")
            .setTypes("article")
            //设置分页
            //起始查询的记录
            .setFrom(0)
            //每页显示的行数,默认显示10条
            .setSize(3)
            //设置高亮信息
            .highlighter(highlightBuilder)
            // 传入查询对象
            .setQuery(queryBuilder)
            .get();
        //取查询结果
        SearchHits searchHits = searchResponse.getHits();
        //取总记录数
        System.out.println("查询结果总记录数" + searchHits.getTotalHits());
        //查询结果列表
        Iterator<SearchHit> iterator = searchHits.iterator();
        while (iterator.hasNext())
        {
            SearchHit searchHit = iterator.next();
            // 打印文档对象,以json格式输出
            System.out.println(searchHit.getSourceAsString());
            Map<String, Object> document = searchHit.getSource();
            System.out.println(document.get("id"));
            System.out.println(document.get("title"));
            System.out.println(document.get("content"));
            System.out.println("****高亮结果****");
            Map<String, HighlightField> highlightFields = 		                                             searchHit.getHighlightFields();
            System.out.println(highlightFields);
            // 取出高亮显示的结果
            HighlightField field = highlightFields.get(highlightField);
            Text[] fragments = field.getFragments();
            if (fragments!=null){
                String title = fragments[0].toString();
                System.out.println(title);
            }
        }
    
}
@After
public void close(){
    client.close();
}
~~~

> 需确保pom文件里面的jar包没有和luence里面的jar包冲突

# SpringDataElasticSearch

## pom文件

~~~xml
<dependencies>
    <!-- junit单元测试 -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.0.4.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.0</version>
    </dependency>
    <!--elasticsearch-->
    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>5.6.8</version>
    </dependency>
    <dependency>
        <groupId>org.elasticsearch.client</groupId>
        <artifactId>transport</artifactId>
        <version>5.6.8</version>
    </dependency>
    <!--elasticsearch-->
    <!--jackson-->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.6</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.9.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-elasticsearch</artifactId>
        <version>3.0.5.RELEASE</version>
        <exclusions>
            <exclusion>
                <groupId>org.elasticsearch.plugin</groupId>
                <artifactId>transport-netty4-client</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
~~~

## Spring文件配置

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:elasticsearch="http://www.springframework.org/schema/data/elasticsearch"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/data/elasticsearch http://www.springframework.org/schema/data/elasticsearch/spring-elasticsearch.xsd">
    <!--SpringDataElasticSearch配置-->
    <!--配置client对象-->
    <elasticsearch:transport-client id="esClient" cluster-name="my-elasticsearch"
                                    cluster-	nodes="127.0.0.1:9301,127.0.0.1:9302,127.0.0.1:9303"/>
    <!--配置包扫描器,扫描dao接口-->
    <elasticsearch:repositories base-package="com.tang.elasticsearch.dao"/>
    <!--配置模板-->
    <bean id="template" name="elasticsearchTemplate"
          class="org.springframework.data.elasticsearch.core.ElasticsearchTemplate">
        <constructor-arg name="client" ref="esClient"/>
    </bean>
</beans>
~~~

## 实体类配置

~~~java
@Data
@Document(indexName = "spring_data_es", type = "article")
/*
@Document 作用在类，标记实体类为文档对象，一般有两个属性
    indexName：对应索引库名称
    type：对应在索引库中的类型
    shards：分片数量，默认5
    replicas：副本数量，默认1
*/
public class Article
{
    @Id
    //@Id 作用在成员变量，标记一个字段作为id主键
    @Field(type = FieldType.Long, store = true)
    /*
 @Field 作用在成员变量，标记为文档的字段，并指定字段映射属性：
        type：字段类型，取值是枚举：FieldType
     index：是否索引，布尔类型，默认是true
        store：是否存储，布尔类型，默认是false
        analyzer：分词器名称
    */
    private long id;

    @Field(type = FieldType.text, store = true, analyzer = "ik_smart")
 private String title;

    @Field(type = FieldType.text, store = true, analyzer = "ik_smart")
    private String content;
}
~~~

## 接口

~~~java
@Repository("articleDao")
public interface IArticleDao extends ElasticsearchRepository<Article, Long>{}
~~~

## 基本操作步骤

~~~java
@Autowired
private IArticleDao articleDao;

@Autowired
private ElasticsearchTemplate elasticsearchTemplate;

@Test
public void method()
{
    //创建索引,并配置映射关系
    elasticsearchTemplate.createIndex(Article.class);
    //配置映射
    this.elasticsearchTemplate.putMapping(Article.class);
    // 添加文档
    Article article = new Article();
    article.setId(2L);
    article.setTitle("SpringDataElasticSearch2");
    article.setContent("添加文档操作2");
    articleDao.save(article);
    //删除文档
    articleDao.deleteById(2L);
    // 更新文档和添加文档一致,保证两次添加的文档id一致即可
    // 查询索引库
    Iterable<Article> all = articleDao.findAll();
    //根据id查询
    Optional<Article> option = articleDao.findById(6L);
    //查询所有，并按照id降序--需要确保有无参构造函数
    Iterable<Article> articles = this.articleDao.findAll(Sort.by(Sort.Direction.DESC, "id"))
        // 自定义查询--需要在接口类根据spring框架的要求定义方法名称,无需手动去实现
        List<Article> list = articleDao.findByTitle("SpringData");
    // 分页查询
    // 设置分页条件,默认从0页开始
    Pageable pageable = PageRequest.of(0,5);
    //按照规则定义方法名称
    List<Article> list = 	 articleDao.findByTitleOrContent("SpringData","SpringData", pageable);
}
//原生方式查询
// 创建原生的查询对象
NativeSearchQuery query = new NativeSearchQueryBuilder()
    .withQuery(QueryBuilders.queryStringQuery("SpringData")
               .defaultField("title"))
    .withPageable(PageRequest.of(0,15))
    .build();
// 执行查询
List<Article> articles = elasticsearchTemplate.queryForList(query, Article.class);
~~~