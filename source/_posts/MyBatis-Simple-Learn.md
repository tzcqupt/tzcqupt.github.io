---
layout: post
title: MyBaits入门学习笔记
date: 2020-04-27
Author: tzcqupt
tags: [MyBatis]
categories: [MyBatis]
comments: true
toc: true
---

# 基本入门
## 书写SqlMapConfig.xml

里面写好数据库配置信息,其他接口类的xml所在详细路径,需和对应的接口所在的路径一致

~~~xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!-- 配置 mybatis 的环境 -->
        <environments default="mysql">
            <!-- 配置 mysql 的环境 -->
            <environment id="mysql">
                <!-- 配置事务的类型 -->
                <transactionManager type="JDBC"/>
                <!-- 配置连接数据库的信息:用的是数据源(连接池) -->
                <dataSource type="POOLED">
                <!--其中属性 type＝”POOLED”代表采用 MyBatis 内 部提
                供的连接池方式-->
                    <property name="driver" value="com.mysql.jdbc.Driver"/>
                    <property name="url"
                              value="jdbc:mysql://localhost:3306/mysql_learn?serverTimezone=UTC&amp;useUnicode=true&amp;characterEncoding=utf-8&amp;useSSL=false"/>
                    <property name="username" value="root"/>
                    <property name="password" value="root123"/>
                </dataSource>
            </environment>
        </environments>
        <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
        <mappers>
            <mapper resource="com/tang/ssm/mybatis/mapper/RoleMapper.xml"/>
        </mappers>
    </configuration>
~~~

## 其他实体类的配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tang.ssm.mybatis.mapper.RoleMapper">

	<insert id="insertRole" parameterType="role">
		<selectKey keyColumn="id" order="AFTER" keyProperty="id" resultType="long">
			select last_insert_id();
		</selectKey>
		insert into t_role(role_name, note) values(#{roleName}, #{note})
	</insert>

	<delete id="deleteRole" parameterType="long">
		delete from t_role where id= #{id}
	</delete>

	<update id="updateRole" parameterType="role">
		update t_role set role_name = #{roleName}, note = #{note} where id= #{id}
	</update>

	<select id="getRole" parameterType="long" resultType="role">
		select id,
		role_name as roleName, note from t_role where id = #{id}
	</select>

	<select id="findRoles" parameterType="string" resultType="role">
		select id, role_name as roleName, note from t_role
		where role_name like concat('%', #{roleName}, '%')
	</select>
	<select id="findAll" resultType="role">
		select id, role_name as roleName, note from t_role
	</select>
</mapper>
~~~

## 书写实体类及测试类

* 实体类建议实现序列化接口Serializable
* RoleMapper接口中定义方法
* 测试类
    ~~~java
        //1.读取配置文件
        InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.创建 SqlSessionFactory 的构建者对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        //3.使用构建者创建工厂对象 SqlSessionFactory
        SqlSessionFactory factory = builder.build(in);
        //4.使用 SqlSessionFactory 生产 SqlSession 对象
        SqlSession session = factory.openSession();
        //5.使用 SqlSession 创建 dao 接口的代理对象
        RoleMapper roleMapper = session.getMapper(RoleMapper.class);
        //6.使用代理对象执行查询所有方法
        List<Role> roles = roleMapper.findAll();
        for(Role role : roles) {
        System.out.println(role);
        }
        //测试增加删除修改方法时。需要提交事务 session.commit();
        //7.释放资源
        session.close();
        in.close();
    ~~~

# MyBatis配置
## 全部配置预览

~~~xml
   <?xml version="1.0" encoding="UTF-8" ?>
    <configuration><!--配置-->
        <properties/> <!--属性-->
        <settings/> <!--设置-->
        <typeAliases/> <!--类型命名 -->
        <typeHandlers/> <!--类型处理器-->
        <objectFactory/> <!--对象工厂-->
        <plugins/> <!--插件-->
        <environments> <!--配置环境-->
            <environment> <!--环境变量-->
                <transactionManager/> <!--事务管理器-->
                <dataSource/> <!--数据源-->
            </environment>
        </environments>
        <databaseidProvider/> <!--数据库厂商标识-->
        <mappers/> <!--映射器-->
    </configuration>
~~~
>配置顺序不能颠倒

## property子元素

方便抽取常用环境到properties文件中，如mysql
~~~xml
<properties resource="database.properties"/>
...
<environments default="mysql">
        <!-- 配置 mysql 的环境 -->
        <environment id="mysql">
            <!-- 配置事务的类型 -->
            <transactionManager type="JDBC"/>
            <!-- 配置连接数据库的信息:用的是数据源(连接池) -->
            <dataSource type="POOLED">
                <property name="driver" value="${driverClass}"/>
                <property name="url"
                          value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
~~~
## typeAliases别名
使用简写来代替全限类名，MyBatis中别名不区分大小写,可以直接配置在实体类上面
@Alias("role2")
~~~xml
 <typeAliases>
        <!--typeAlias用于配置别名。type属性指定的是实体类全限定类名。alias属性指定别名,当指定了别名就再区分大小写
        <typeAlias type="com.tang.ssm.mybatis.pojo" alias="role"></typeAlias>-->
        <!-- 用于指定要配置别名的包，当指定之后，该包下的实体类都会注册别名，将类名第一个字母小写就是别名,不再区分大小写-->
        <package name="com.tang.ssm.mybatis.pojo"/>
 </typeAliases>
~~~
## typeHandler 类型转换器  
系统提供的 typeHandler 能覆盖大部分场景的要求,但比如枚举类不行。   
### 配置typeHandler的扫描方式  

~~~xml
    <typeHandlers>
        <!--配置类型转换器-->
        <package name="com.tang.ssm.mybatis.handler"/>
    </typeHandlers>
~~~

### 自定义Handler

~~~java
@MappedTypes(SexEnum.class)
@MappedJdbcTypes(JdbcType.INTEGER)
public class SexEnumTypeHandler implements TypeHandler<SexEnum> {

	@Override
	public void setParameter(PreparedStatement ps, int i, SexEnum parameter,
			JdbcType jdbcType) throws SQLException {
		ps.setInt(i, parameter.getId());
	}

	@Override
	public SexEnum getResult(ResultSet rs, String columnName)
			throws SQLException {
		int id = rs.getInt(columnName);
		return SexEnum.getSexById(id);
	}

	@Override
   public SexEnum getResult(ResultSet rs, int columnIndex) throws SQLException {
		int id = rs.getInt(columnIndex);
		return SexEnum.getSexById(id);
	}

	@Override
	public SexEnum getResult(CallableStatement cs, int columnIndex)
			throws SQLException {
		int id = cs.getInt(columnIndex);
		return SexEnum.getSexById(id);
	}

}
~~~

### 配置自定义的handler

~~~xml
<resultMap id="userMapper" type="user">
    <result property="id" column="id" />
    <result property="userName" column="user_name" />
    <result property="password" column="password" />
    <result property="sex" column="sex"
            typeHandler="com.tang.ssm.mybatis.handler.SexEnumTypeHandler"/>
    <!--
    org.apache.ibatis.type.EnumOrdinalTypeHandler
    MyBatis 根据枚举数组下标索引的方式进行匹配的，
    也是枚举类型的默认转换类，
    它要求数据库返回一个整数作为其下标，
    它会根据下标找到对应的枚举类型

    EnumTypeHandler
    把使用的名称转化为对应的枚举，比如它会根据数据库返回的
    字符串"MALE "，进 行 Enum.valueOf(SexEnum.class，"MALE"）；
    -->
    <result property="mobile" column="mobile" />
    <result property="tel" column="tel" />
    <result property="email" column="email" />
    <result property="note" column="note" />
</resultMap>
 <select id="getUserById" resultMap="userMapper" parameterType="long">
  select id, user_name, password, sex, mobile, tel, email, note from t_user
  where id = #{id}
</select>
~~~

### 测试代码

~~~java
@Test
public void testGetUser() {
    User user = userMapper.getUserById(1L);
    log.info(user.getSex().getName());
}
~~~

## mapper引入
~~~xml
 <!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
    <mappers>
        <!--<mapper resource="com/tang/ssm/mybatis/mapper/RoleMapper.xml"/>-->
        <!--挨个引入-->
        <!--<mapper class="com.tang.ssm.mybatis.mapper.UserMapper"/>
        <mapper class="com.tang.ssm.mybatis.mapper.RoleMapper"/>-->
        <!-- package标签是用于指定dao接口所在的包,当指定了之后就不需要在写mapper以及resource或者class了 -->
        <package name="com.tang.ssm.mybatis.mapper"/>
    </mappers>
~~~
# 映射器
- cache – 该命名空间的缓存配置。
- cache-ref – 引用其它命名空间的缓存配置。
- resultMap – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
- sql – 可被其它语句引用的可重用语句块。
- insert – 映射插入语句。
- update – 映射更新语句。
- delete – 映射删除语句。
- select – 映射查询语句。
## select 查询语句
### 简单的select元素应用
~~~xml
 <select id="countUserByFirstName" parameterType="string" resultType="int">
    select count(*) total from t_user where user_name like concat(#{firstName},'%')
    </select>
~~~
- parameterType表示这条sql接受的参数类型。如MyBatis系统定义或者自定义的别名，如int、string、float。也可以是类的全限定名com.tang.ssm.mybatis.pojo.User
- resultType 表示这条sql返回的结果类型。
- #{firstName}是被传递进去的参数
### 自动映射和驼峰映射
- role_name 被roleName代替了，对应上了POJO上的属性值，mybatis会自动将结果集映射到POJO的roleName上，自动映射。
- 若数据库字段名为role_name，POJO属性名为roleName。可以开启驼峰配置，自动隐射。在setting里完成`mapUnderscoreToCamelCase=true`配置即可
~~~xml
<select id="getRole" parameterType="long" resultType="role" >
		select id,
		role_name as roleName, note from t_role where id = #{id}
	</select>
~~~
### 传递多个参数
#### 使用map接口传递参数
~~~java
//接口中方法定义
List<Role> findRolesByMap(Map<String , Object> parameterMap);
~~~
~~~xml
<select id="findRolesByMap" resultType="role" parameterType="map">
        select id,role_name as roleName,note from t_role
        where role_name like concat('%',#{roleName},'%')
        and note like concat('%',#{note},'%')
    </select>
~~~
#### 使用注解传递参数
~~~java
List<Role> findRolesByAnno(@Param("roleName") String roleName,
                               @Param("note") String note);
//xml同上
~~~
#### 使用JavaBean传递参数
~~~java
List<Role> findRolesByBean(RoleParams roleParams);
//Bean定义
@Data
public class RoleParams {
    private String roleName;
    private String note;
}
//xml同上
~~~
#### 混合使用传递参数
~~~java
List<Role> findRolesByMix(@Param("params") RoleParams roleParams,
                               @Param("page") PageParam pageParam);
//Bean定义
@Data
public class PageParam {
    private int start;
    private int limit;
}
//3.1.3全部测试代码
@Test
    public void testFindRolesWay() {
        HashMap<String, Object> parameterMap = new HashMap<>();
        parameterMap.put("roleName", "1");
        parameterMap.put("note", "1");
        List<Role> roles1 = roleMapper.findRolesByMap(parameterMap);
        log.info("测试通过map接口传递多个参数");
        roles1.forEach(System.out::println);
        List<Role> roles2 = roleMapper.findRolesByAnno("1", "1");
        log.info("测试利用注解传递多个参数");
        roles2.forEach(System.out::println);

        RoleParams roleParams = new RoleParams();
        roleParams.setRoleName("name");
        roleParams.setNote("note");
        List<Role> roles3 = roleMapper.findRolesByBean(roleParams);
        log.info("测试利用JavaBean传递多个参数");
        roles3.forEach(System.out::println);
        RoleParams roleParams2 = new RoleParams();
        roleParams2.setRoleName("name");
        roleParams2.setNote("note");
        PageParam pageParam = new PageParam();
        pageParam.setStart(0);
        pageParam.setLimit(2);
        List<Role> roles4 = roleMapper.findRolesByMix(roleParams2, pageParam);
        log.info("测试混合使用传递多个参数");
        roles4.forEach(System.out::println);
    }
~~~
~~~xml
<select id="findRolesByMix" resultType="role">
        select id,role_name as roleName,note from t_role
        where role_name like concat('%',#{params.roleName},'%')
        and note like concat('%',#{params.note},'%')
        limit #{page.start},#{page.limit}
    </select>
~~~
### 使用resultMap映射结果集

- 解决自动映射和驼峰映射的局限性

    ~~~xml
    <resultMap id="roleMapper" type="role">
            <result property="id" column="id"/>
            <result property="roleName" column="role_name"/>
            <result property="note" column="note"/>
    </resultMap>
    <select id="getRoleByIdUseResultMap" resultType="role" resultMap="roleMap">
            select id,
                   role_name, note from t_role where 1=1 and id = #{id}
    </select>
    ~~~
### 分页参数RowBounds

~~~java
List<Role> findRolesByRowBounds(@Param("roleName") String roleName,
                                    @Param("note") String note,
                                    RowBounds rowBounds);
//测试代码
    @Test
    public void testRowBounds() {
        //从第几条记录开始查，查询的记录数
        RowBounds rowBounds = new RowBounds(0, 2);
        List<Role> roles = roleMapper.findRolesByRowBounds("name", "note", rowBounds);
        log.info(roles.size());
        roles.forEach(System.out::println);
    }
~~~

~~~xml
<select id="findRolesByRowBounds" resultType="role" resultMap="roleMap">
        /*映射文件中不需要rowbounds内容，mybatis自动识别*/
        select id, role_name, note
        from t_role
        where role_name like concat('%', #{roleName}, '%')
          and note like concat('%', #{note}, '%')
    </select>
~~~
- resultMap 的id代表他的标识。type标时使用哪个类作为其映射的类，可以配置别名后，这里简写
- 子元素 result的id代表resultMap的主键 。result代表其属性。property是POJO的属性名称，column是sql的列名。
## insert 插入语句
插入后返回主键和自定义主键
~~~xml
    <!--返回主键的两种方式-->
    <insert id="insertRole" parameterType="role">
        <selectKey keyColumn="id" order="AFTER" keyProperty="id" resultType="long">
            select last_insert_id();
        </selectKey>
        insert into t_role(role_name, note) values(#{roleName}, #{note})
    </insert>
    <insert id="insertRole" parameterType="role"
            useGeneratedKeys="true" keyProperty="id" keyColumn="id">
        insert into t_role(role_name, note) values(#{roleName}, #{note})
    </insert>
    <!--自定义主键-->
    <insert id="insertRole" parameterType="role">
        <selectKey keyColumn="id" order="BEFORE" keyProperty="id" resultType="long">
            select if(max(id)=null,1,max(id)+3) from t_role
        </selectKey>
        insert into t_role(role_name, note) values(#{roleName}, #{note})
    </insert>
~~~
## update和delete
执行后都会返回一个整数
~~~xml
     <delete id="deleteRole" parameterType="long">
            delete
            from t_role
            where id = #{id}
        </delete>
    
        <update id="updateRole" parameterType="role">
            update t_role
            set role_name = #{roleName},
                note      = #{note}
            where id = #{id}
        </update>
~~~
## sql元素

抽取重复的sql语句

~~~xml
    <resultMap>...</resultMap>
    <sql id="roleCols">
            id,role_name,note
    </sql>
    <!--支持变量传递-->
    <sql id="roleColsWithArgs">
            ${alias}.id, ${alias}.role_name, ${alias}.note
    </sql>
    
    <select id="findAll" resultType="role" resultMap="roleMap">
            select <include refid="roleColsWithArgs">
            <property name="alias" value="r"/>
        </include>
            from t_role r where 1=1
    </select>
    <select id="getRoleByIdUseResultMap" resultType="role" resultMap="roleMap">
            select <include refid="roleCols"/>
            from t_role
            where 1 = 1
              and id = #{id}
    </select> 
~~~

## 参数

在sql语句中发送动态列名。用到$。详情见后方面试题目。

## 级联
### 一对多/多对一配置
~~~java
    @Data
    public class Employee {
    
        private Long id;
        private String realName;
        private SexEnum sex;
        private Date birthday;
        private String mobile;
        private String email;
        private String position;
        private String note;
        /**
         * 工牌按一对一级联
         */
        private WorkCard workCard;
        /**
         * 雇员任务，一对多级联
         */
        private List<EmployeeTask> employeeTaskList;
    }
    @Data
    public class EmployeeTask {
        private Long id;
        private Long empId;
        /**
         * 员工任务表和任务表
         * 一对一级联
         */
        private Task task;
        private String taskName;
        private String note;
    }
    //mapper接口中定义方法
    Employee getEmployeeById(Long id);
~~~
~~~xml
    <mapper namespace="com.tang.ssm.mybatis.mapper.EmployeeMapper">
        <resultMap type="com.tang.ssm.mybatis.pojo.Employee" id="employee">
            <id column="id" property="id"/>
            <result column="real_name" property="realName"/>
            <result column="sex" property="sex"
                    typeHandler="com.tang.ssm.mybatis.handler.SexEnumTypeHandler"/>
            <result column="birthday" property="birthday"/>
            <result column="mobile" property="mobile"/>
            <result column="email" property="email"/>
            <result column="position" property="position"/>
            <result column="note" property="note"/>
            <!--一对一级联配置
                雇员对工牌进行一对一级联
            如果多表查询时,主表和从表的主键列名称一致,需要将查询的结果起别名,否则只能查询出一个 
            -->
            <association property="workCard" column="id"
                         select="com.tang.ssm.mybatis.mapper.WorkCardMapper.getWorkCardByEmpId"/>
            <!--一对多级联配置
                雇员对雇员任务进行一对多级联
            -->
            <!--
            • eager，获得当前 POJO 后立即加载对应的数据 。
            • lazy ， 获得当前 POJO 后延迟加载对应的数据。
            -->
            <collection property="employeeTaskList" column="id" fetchType="lazy"
                        select="com.tang.ssm.mybatis.mapper.EmployeeTaskMapper.getEmployeeTaskByEmpId"/>
            <!--鉴别器 属性 column 代表使用哪个字段进行鉴别
            case 用于进行区分
            类似于switch...case-->
            <discriminator javaType="long" column="sex">
                <case value="1" resultMap="maleHealthFormMapper"/>
                <case value="2" resultMap="femaleHealthFormMapper"/>
            </discriminator>
        </resultMap>
        <!--女性健康表-->
        <resultMap type="com.tang.ssm.mybatis.pojo.FemaleEmployee" id="femaleHealthFormMapper"
                   extends="employee">
            <association property="femaleHealthForm" column="id"
                         select="com.tang.ssm.mybatis.mapper.FemaleHealthFormMapper.getFemaleHealthForm"/>
        </resultMap>
        <!--男性健康表-->
        <resultMap type="com.tang.ssm.mybatis.pojo.MaleEmployee" id="maleHealthFormMapper"
                   extends="employee">
            <association property="maleHealthForm" column="id"
                         select="com.tang.ssm.mybatis.mapper.MaleHealthFormMapper.getMaleHealthForm"/>
        </resultMap>
    
        <select id="getEmployeeById" parameterType="long" resultMap="employee">
            select id,
                   real_name,
                   sex,
                   birthday,
                   mobile,
                   email,
                   position,
                   note
            from t_employee
            where id = #{id}
        </select>
    <!--sqlMapConfig.xml配置延迟加载策略-->  
    <settings>
            <!--延迟加载的全局开关。开启时，所有关联对象都会延迟加载-->
            <setting name="lazyLoadingEnabled" value="true"/>
            <!--对任意延迟属性的调用会使带有延迟加载属性的对象完整加载-->
            <setting name="aggressiveLazyLoading" value="false"/>
    </settings>
~~~
### 多对多配置

~~~java
//实体类
@Data
@Alias("mmRole")
public class Role {
    private Long id;
    private String roleName;
    private String note;
    /**
     * 关联用户信息，一对多
     */
    private List<User> userList;

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

@Data
@Alias("mmUser")
public class User {
    private Long id;
    private String userName;
    private String realName;
    private SexEnum sex;
    private String mobile;
    private String email;
    private String note;
    /**
     * 对角色 一对多关联
     */
    private List<Role> roleList;

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
//接口
public interface RoleMapper {
	Role getRoleById(Long id);
	List<Role> findRoleByUserId(Long userId);
}
public interface UserMapper {
    User getUserById(Long id);
    List<User> findUserByRoleId(Long roleId);
}
~~~

~~~xml
<!--RoleMapper-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tang.ssm.mybatis.mapper.mm.RoleMapper">
    <resultMap type="com.tang.ssm.mybatis.pojo.mm.Role" id="mmRoleMapper">
        <id column="id" property="id"/>
        <result column="role_name" property="roleName"/>
        <result column="note" property="note"/>
        <collection property="userList" column="id" fetchType="lazy"
                    select="com.tang.ssm.mybatis.mapper.mm.UserMapper.findUserByRoleId"/>
    </resultMap>

    <select id="getRoleById" parameterType="long" resultMap="mmRoleMapper">
        select id, role_name, note
        from t_mm_role
        where id = #{id}
    </select>

    <select id="findRoleByUserId" parameterType="long" resultMap="mmRoleMapper">
        select r.id, r.role_name, r.note
        from t_mm_role r,
             t_mm_user_role ur
        where r.id = ur.role_id
          and ur.user_id = #{userId}
    </select>
</mapper>
<!--UserMapper-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tang.ssm.mybatis.mapper.mm.UserMapper">
    <resultMap type="com.tang.ssm.mybatis.pojo.mm.User" id="mmUserMapper">
        <id column="id" property="id"/>
        <result column="user_name" property="userName"/>
        <result column="real_name" property="realName"/>
        <result column="sex" property="sex"
                typeHandler="com.tang.ssm.mybatis.handler.SexEnumTypeHandler"/>
        <result column="mobile" property="mobile"/>
        <result column="email" property="email"/>
        <result column="note" property="note"/>
        <collection property="roleList" column="id" fetchType="lazy"
                    select="com.tang.ssm.mybatis.mapper.mm.RoleMapper.findRoleByUserId"/>
    </resultMap>
    <select id="getUserById" parameterType="long" resultMap="mmUserMapper">
        select id, user_name, real_name, sex, mobile, email, note
        from t_mm_user
        where id = #{id}
    </select>
    <select id="findUserByRoleId" parameterType="long" resultMap="mmUserMapper">
        select u.id, u.user_name, u.real_name, u.sex, u.mobile, u.email, u.note
        from t_mm_user u,
             t_mm_user_role ur
        where u.id = ur.user_id
          and ur.role_id = #{roleId}
    </select>
</mapper>
~~~
## 缓存

- MyBatis默认开启一级缓存，在SqlSession级别
- 二级缓存也是默认开启。在sqlMapConfig.xml中配置全局，在SqlSessionFactory级别
    ~~~xml
    <settings>
      <!-- 开启二级缓存的支持,该配置默认开启 --> 
        <setting name="cacheEnabled" value="true"/>
    </settings>
    ~~~
- 对应的实体类要实现Serializable接口
- 对应类的Mapper.xml文件中开启缓存支持<cache/>
- 自定义缓存配置
- 
    ~~~xml
    <cache type="com.tang.ssm.mybatis.cache.RedisCache">
        <property name="host" value="localhost">
    </cache>
    <!--在对应的select/insert等语句中修改缓存配置-->
    <select flushCache= "false" useCache= "true" />
    <insert flushCache= "false" />
    <update flushCache= "false" />
    <delete flushCache= "false" />
    <!--其他类的Mapper.xml中配置该缓存-->
    <cache-ref namespace="com.tang.ssm.mybatis.mapper.mm.RoleMapper"/>
    ~~~
# 动态sql

- if 判断语句
- choose(when,otherwise)  相当于java中的switch case语句，多分支条件判断
- trim(where,set) 辅助元素，用于处理特定的sql拼装问题，如去掉多余的and、or等
- foreach 循环语句，在in语句等列举条件常用

## if

~~~xml
        <select id="findRoles" parameterType="string" resultMap="roleMap">
            select
            <include refid="roleCols"/>
            from t_role
            where 1=1
            <if test="roleName !=null and roleName !='' ">
                and role_name like concat('%', #{roleName}, '%')
            </if>
        </select>
~~~

## choose、when、otherwise

类似 switch...case...default

~~~java
List<Role> findRolesWithParams(@Param("id")Long roleId,
                                   @Param("roleName")String roleName);
//测试
    @Test
    public void testFindRolesWithParams(){
        List<Role> roles = roleMapper.findRolesWithParams( null,"name");
//List<Role> roles = roleMapper.findRolesWithParams( null,"");
        for (Role role : roles) {
            log.info(role);
        }
    }
~~~
~~~xml
<select id="findRolesWithParams" resultMap="roleMap">
        select
        <include refid="roleCols"/>
        from t_role
        where 1=1
        <choose>
            <when test="id !=null and id !='' ">
                and id=#{id}
            </when>
            <when test="roleName !=null and roleName !='' ">
                and role_name like concat('%', #{roleName}, '%')
            </when>
            <otherwise>
                and note is not null
            </otherwise>
        </choose>
    </select>
~~~
## trim、where、set

- 当`<where></where>`里条件成立时，`where`才会组装到sql语句中。
- trim 元素意味着要去掉一些特殊的字符串，当 prefix代表的是语句的前缀，而 prefixOverrides代表的是需要去掉哪种字符串
- set 元素遇到了逗号，它会把对应的逗号去掉

    ~~~xml
    <!--去掉了where 1=1的条件-->
    <select id="findRoles" parameterType="string" resultMap="roleMap">
            select
            <include refid="roleCols"/>
            from t_role
            <trim prefix="where" prefixOverrides="and">
            <if test="roleName !=null and roleName !='' ">
                and role_name like concat('%', #{roleName}, '%')
            </if>
            </trim>
    </select>
    <!--set能去掉多余, 号 也能用trim标签实现-->
    <update id="updateRole" parameterType="role">
            update t_role
            <!--<set>
                <if test="roleName !=null and roleName !='' ">
                    role_name = #{roleName},
                </if>
                <if test="note !=null and note !='' ">
                    note = #{note},
                </if>
            </set>-->
            <trim prefix="set" suffixOverrides=",">
                    <if test="roleName !=null and roleName !='' ">
                        role_name = #{roleName},
                    </if>
                    <if test="note !=null and note !='' ">
                        note = #{note},
                    </if>
            </trim>
            where id = #{id}
        </update>
    ~~~
## foreach

~~~java
    List<Role> findRoleByIds(@Param("roleIdList") List<String> roleIdList);
    //测试类
    @Test
    public void testForeach(){
        List<Role> roleList = roleMapper.findRoleByIds(Arrays.asList("1", "2"));
        for (Role role : roleList) {
            log.info(role);
        }
    }
~~~
~~~xml
    <select id="findRoleByIds" resultType="mmRole">
            select * from t_mm_role where id in
            <foreach item="roleId" index="index" collection="roleIdList"
                     open="(" separator="," close=")">
                #{roleId}
            </foreach>
    </select>
    <!--
    collection 配置的 roleIdList 是传递进来的参数名称，它可以是一个数组、 List、 Set
    等集合。
    • item 配置的是循环中当前的元素。
    • index 配置的是当前元素在集合的位置下标 。
    • open 和 close 配置的是以什么符号将这些集合元素包装起来 。
    • separator 是各个元素的问隔符 。
    -->
~~~
## test

~~~xml
    <!--test判断字符串用toString()方法-->
    <select id="getRoleTest" resultMap="roleMap">
            select
            <include refid="roleCols"/>
            from t_role
            <if test="type=='Y'.toString()">
                where 1=1
            </if>
    </select>
~~~
## bind

~~~xml
    <!--绑定参数，解决不同数据库对字符串的拼接方式
    如 mysql 的concat()、oracle的||
    传递的参数名称还是roleName和note,但是在sql中经过bind后，
    要用pattern_roleName和pattern_note
    -->
    <select id="findRolesByAnno" resultMap="roleMap">
            <bind name="pattern_roleName" value="'%'+roleName+'%'"/>
            <bind name="pattern_note" value="'%'+note+'%'"/>
            select
            <include refid="roleCols"/>
            from t_role
            where role_name like #{pattern_roleName}
            and note like #{pattern_note}
    </select>
~~~

# 整合Spring

## 依赖

~~~xml
<spring.version>5.0.15.RELEASE</spring.version>
<mybatis.version>3.5.2</mybatis.version>
<mybatis.spring.version>2.0.3</mybatis.spring.version>
<mysql.version>5.1.47</mysql.version>
<hikariCP.version>2.7.2</hikariCP.version>
<!-- 整合mybatis框架 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>${mybatis.version}</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>${mybatis.spring.version}</version>
</dependency>
<!-- mysql驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
</dependency>
<!--数据库连接池-->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>${hikariCP.version}</version>
</dependency>
~~~

## 配置文件

SqlMapConfig.xml可以都不配置（必须要有）。具体的mapper文件单独配置。

### 与spring整合的xml

~~~xml

<!--spring整合mybatis-->
<!-- 配置 MyBatis 的 Session 工厂 -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 数据库连接池 -->
    <property name="dataSource" ref="dataSource"/>
    <!-- 扫描 po 包，使用别名 -->
    <property name="typeAliasesPackage" value="com.tang.ssm.pojo"/>
    <!-- 加载 mybatis 的全局配置文件 -->
    <property name="configLocation" value="classpath:SqlMapConfig.xml"/>
     <!-- 也可以在mybatis的全局配置文件SqlMapConfig.xml中单独配置 
    <mappers>
            <mapper resource="mybatis/mapper/RoleMapper.xml"/>
    </mappers>
    -->
    <property name="mapperLocations" value="classpath:/mybatis/mapper/*.xml"/>
</bean>
<!-- 引入外部属性文件 -->
<context:property-placeholder location="classpath:datasource.properties"/>
<!--1.将连接池放入spring容器 -->
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
<!-- 配置 Mapper 扫描器,自动扫描所有 Mapper 接口和文件-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.tang.ssm.dao"/>
     <!-- 注意使用 sqlSessionFactoryBeanName 避免出现spring 扫描组件失效问题 -->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <!-- 区分注解扫描，持久层dao可能用@Mapper注解，也可能用@Repository注解-->
    <property name="annotationClass" value="org.springframework.stereotype.Repository"/>
</bean>
<!--配置spring的事务-->
<!-- 配置事务管理器 -->
<bean id="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 配置事务的通知 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*" propagation="REQUIRED" read-only="false"/>
        <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
</tx:advice>
<!-- 配置 aop -->
<aop:config>
    <!-- 配置切入点表达式 -->
    <aop:pointcut id="pt" expression="execution(* com.tang.ssm.service.impl.*.*(..))"
                  />
    <!-- 建立通知和切入点表达式的关系 -->
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
</aop:config>
~~~

### 单独的Mapper文件

~~~xml
 <!-- SqlMapConfig.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   <!-- <mappers>
        <mapper resource="mybatis/mapper/RoleMapper.xml"/>
    </mappers> -->
</configuration>
 <!-- RoleMapper.xml -->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.tang.ssm.dao.RoleDao">

    <select id="getRole" resultType="role">
        select id,
               role_name as
                   roleName,
               note
        from t_role
        where id = #{id}
    </select>

    <delete id="deleteRole">
        delete
        from t_role
        where id = #{id}
    </delete>

    <insert id="insertRole" parameterType="role"
            useGeneratedKeys="true" keyProperty="id">
        insert into t_role (role_name, note)
        values (#{roleName}, #{note})
    </insert>

    <update id="updateRole" parameterType="role">
        update t_role
        set role_name = #{roleName},
            note      = #{note}
        where id = #{id}
    </update>
    <select id="findRoles" resultType="role">
        select id, role_name as roleName, note from t_role
        <where>
            <if test="roleName != null">
                role_name like concat('%', #{roleName}, '%')
            </if>
            <if test="note != null">
                note like concat('%', #{note}, '%')
            </if>
        </where>
    </select>
</mapper>
~~~

### 持久层

加上@Repository注解

~~~java
@Repository
public interface RoleDao {
    Role getRole(Long id);
    int deleteRole(Long id);
    int insertRole(Role role);
    int updateRole(Role role);
    List<Role> findRoles(@Param("roleName") String roleName, @Param("note") String note);
}
~~~

# 整合SpringBoot

## 普通整合

### 依赖

~~~xml
<!--SpringBoot 版本为2.1.4.Release-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--jdbc连接池 用的是HikariCP-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
~~~

### 配置文件

mybatis全局配置文件和具体的Mapper.xml同上。

> 实体类不需要特别配置。因为具体Mapper.xml会对应上

application.yaml如下

~~~yaml
server:
  port: 9999
  address: 127.0.0.1
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot_learn?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: root123
    driver-class-name: com.mysql.cj.jdbc.Driver
    #连接池配置
    hikari:
      idle-timeout: 60000
      maximum-pool-size: 30
      minimum-idle: 10
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
mybatis:
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml
  #别名配置，这样可以在mapper.xml中使用别名
  type-aliases-package: com.tang.springboot.mybatis.pojo
  configuration:
  	#控制台打印sql语句
  	log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
# springboot 开启日志
logging:
  level:
    com.tang.springboot.mybatis.mapper: debug
~~~

### 启动类

~~~java
@SpringBootApplication
//定义dao层扫描路径和扫描的注解。持久层需加上该注解。同Spring整合
@MapperScan(basePackages = "com.tang.springboot.mybatis.mapper",
        annotationClass = Repository.class)
public class MybatisCliApplication{
     public static void main(String[] args) {
        SpringApplication.run(MybatisCliApplication.class, args);
    }
}
~~~

### 小技巧

~~~java
//对于主键自增的，可以使用这种方式插入主键
@Insert("insert into t_role(role_name,note) values(#{roleName},#{note})")
@Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
int insert(Role role);
~~~



## 整合通用mapper

### 依赖

~~~xml
<!--SpringBoot 版本为2.1.4.Release-->
<!--mysql插件-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!--通用mapper会包含该引用-->
<!--jdbc连接池 用的是HikariCP-->
<!--<dependency>
       <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
     </dependency>-->
<!--通用mapper会包含该引用-->
<!--mybatis集成springBoot的启动器-->
<!--<dependency>
         <groupId>org.mybatis.spring.boot</groupId>
         <artifactId>mybatis-spring-boot-starter</artifactId>
         <version>2.0.0</version>
        </dependency>-->
<!-- 通用mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.4</version>
</dependency>
~~~

### 配置文件

~~~yaml
#server settings
server:
  port: 9999
  address: 127.0.0.1
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot_learn?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: root123
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 连接池相关配置
    hikari:
      idle-timeout: 60000
      maximum-pool-size: 30
      minimum-idle: 10
  thymeleaf:
    cache: false
# 开发阶段关闭thymeleaf的模板缓存
mybatis:
 #使用mybatis通用mapper(省略单表查询的sql),不需要映射文件
# mapper-locations: classpath:mybatis/mapper/*.xml
  #别名扫描
  type-aliases-package: com.tang.springboot.domain


~~~



### 持久层

~~~java
import org.springframework.stereotype.Repository;
import tk.mybatis.mapper.common.Mapper;
@Repository
public interface CityMapper extends Mapper<City> {
}
~~~

### 实体类

~~~java
@Data
@Table(name = "city")//数据库里的表名
public class City {
    @KeySql(useGeneratedKeys = true)//主键策略--数据库的自增
    @Id//声明为主键
    private Long id;
    @Column(name = "name")//数据库里的列名配置
    //配置非数据库表的字段@Transient
    private String name;
    private String state;
    private String country;
}
~~~

### 启动类

~~~java
import tk.mybatis.spring.annotation.MapperScan;
@SpringBootApplication
@MapperScan(value = "com.tang.springboot.mapper",
        annotationClass = Repository.class)
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
~~~

## 整合mybatis-plus

### 依赖

~~~xml
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/>
  </parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--mybatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.0.5</version>
    </dependency>

    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <!--lombok用来简化实体类-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
~~~

### 配置文件

~~~yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    # ?serverTimezone=GMT%2B8 是必须的
    url: jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useSSL=false
    username: root
    password: root123
  profiles:
    active: dev
# 配置sql输出日志
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    #全局设置主键生成策略 auto为自增。不设置为全局唯一ID配置
  #  global-config:
  #    db-config:
  #      id-type: auto
  global-config:
    db-config:
      # 表示逻辑删除的标志。默认配置 1为删除
      logic-delete-value: 1
      logic-not-delete-value: 0
~~~

### 持久层

~~~java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
@Repository
public interface UserMapper extends BaseMapper<User>
~~~

### 实体类

~~~java
@Data
public class User {
    /**
     * 设置主键自增策略。不设置默认的策略为ID_WORKER  全局唯一ID
     * //@TableId(type = IdType.AUTO)
     */
    private Long id;
    private String name;
    private Integer age;
    private String email;
    /**
     * 更好的方式来记录时间--可以在设计表的时候来实现，也可以按照下面方式
     * 1.实现元对象处理器接口MetaObjectHandler
     * 2.注解填充字段
     */

    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    //@TableField(fill = FieldFill.UPDATE)
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updateTime;
    /**
     * 乐观锁的实现方式。在表中添加version字段。更新这条数据时，如果version匹配则修改
     */
    @Version
    private Integer version;
    /**
     * 逻辑删除的实现方式。需要在配置类中注册Bean
     * 新增成功后，查询的时候也是直接查询了 WHERE deleted=0
     */
    @TableLogic
    private Integer deleted;
~~~

### 启动类&配置类

~~~java
@SpringBootApplication
public class MybatisPlusApplication {
    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusApplication.class, args);
    }
}
//配置类
@EnableTransactionManagement
@Configuration
@MapperScan(value = "com.tang.mybatisplus.mapper",annotationClass = Repository.class)
public class MybatisPlusConfig {
    /**
     * 注册乐观锁插件
     *
     * @return
     */
    @Bean
    public OptimisticLockerInterceptor optimisticLockerInterceptor() {
        return new OptimisticLockerInterceptor();
    }

    /**
     * 分页插件
     *
     * @return
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }

    /**
     * 逻辑删除标志
     *
     * @return
     */
    @Bean
    public ISqlInjector sqlInjector() {
        return new LogicSqlInjector();
    }

    /**
     * sql 执行性能分析插件
     * maxTime sql执行最大时长，超时自动停止
     * format 是否格式化，默认false
     *
     * @return
     */
    @Bean
    //设置 dev test环境开启
    @Profile({"dev", "test"})
    public PerformanceInterceptor performanceInterceptor() {
        PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
        performanceInterceptor.setMaxTime(100);
        performanceInterceptor.setFormat(true);
        return performanceInterceptor;
    }
}

~~~

### 处理器

~~~java
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ...");
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
~~~

### 测试类

#### 基本CRUD

~~~java

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
....

/**
 * mybatisPlus基本CRUD
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class MybatisPlusTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelectList() {
        System.out.println(("----- selectAll method test ------"));
        //UserMapper 中的 selectList() 方法的参数为 MP 内置的条件封装器 Wrapper
        //所以不填写就是无任何条件
        List<User> users = userMapper.selectList(null);
        users.forEach(System.out::println);
    }

    @Test
    public void testInsert() {
        User user = new User();
        user.setName("lcm2");
        user.setAge(20);
        user.setEmail("lcm2@foxmail.com");
        int result = userMapper.insert(user);
        System.out.println("插入的记录数" + result);
        System.out.println(user);
    }

    @Test
    public void testUpdate() {
        User user = new User();
        user.setId(5L);
        user.setAge(21);
        user.setName("updatetime");
        userMapper.updateById(user);
    }

    @Test
    public void testOptimisticLock() {
        User user = userMapper.selectById(1L);
        //修改数据
        user.setName("tzcqupt");
        user.setEmail("tang@tang.com");
        //执行更新
        userMapper.updateById(user);
        // 测试后version字段成功+1
    }

    @Test
    public void testOptimisticLockerFail() {
        //测试乐观锁修改失败 -修改
        User user = userMapper.selectById(1L);
        //修改数据
        user.setName("tzcqupt");
        user.setEmail("tang1@tang.com");

        //模拟另一个线程中间更新了数据
        //查询
        User user2 = userMapper.selectById(1L);
        //修改数据
        user2.setName("Helen Yao2");
        user2.setEmail("helen2@qq.com");
        userMapper.updateById(user2);
        //执行更新
        userMapper.updateById(user);
        // 第一次的修改没有成功。。version不匹配，没有修改
    }

    @Test
    public void testSelectIds() {
        List<User> users = userMapper.selectBatchIds(Arrays.asList(1, 2, 3));
        users.forEach(System.out::println);
    }

    /**
     * 简单条件查询
     */
    @Test
    public void testSelectByMap() {
        HashMap<String, Object> map = new HashMap<>();
        /**
         * map中的key要和数据库中的列名要完全对应
         */
        map.put("name", "tang");
        map.put("age", 18);
        List<User> users = userMapper.selectByMap(map);
        users.forEach(System.out::println);
    }

    /**
     * 测试分页
     */
    @Test
    public void testSelectPage() {
        Page<User> page = new Page<>(1, 3);
        userMapper.selectPage(page, null);
        page.getRecords().forEach(System.out::println);

        System.out.println(page.getCurrent());
        System.out.println(page.getPages());
        System.out.println(page.getSize());
        System.out.println(page.getTotal());
        System.out.println(page.hasNext());
        System.out.println(page.hasPrevious());
    }

    /**
     * 测试分页
     */
    @Test
    public void testSelectMapPage() {
        Page<User> page = new Page<>(1, 4);
        IPage<Map<String, Object>> mapIPage = userMapper.selectMapsPage(page, null);
        //注意：此行必须使用 mapIPage 获取记录列表，否则会有数据类型转换错误
        mapIPage.getRecords().forEach(System.out::println);

        System.out.println(page.getCurrent());
        System.out.println(page.getPages());
        System.out.println(page.getSize());
        System.out.println(page.getTotal());
        System.out.println(page.hasNext());
        System.out.println(page.hasPrevious());
    }

    @Test
    public void testDeleteById() {
        /**
         * 加入逻辑删除字段后，只是改变了deleted字段的值
         */
        int result = userMapper.deleteById(6L);
        System.out.println(result);
    }

    @Test
    public void testDeleteBatchIds() {
        int result = userMapper.deleteBatchIds(Arrays.asList(8, 9));
        System.out.println(result);
    }

    @Test
    public void testDeleteByMap() {
        HashMap<String, Object> map = new HashMap<>();
        map.put("name", "Sandy");
        map.put("age", 21);
        int result = userMapper.deleteByMap(map);
        System.out.println(result);
    }

    /**
     * 测试 性能分析插件
     * 在配置类中设置maxTime为1后报错
     */
    @Test
    public void testPerformance() {
        User user = new User();
        user.setName("我是Helen2");
        user.setEmail("helen2@sina.com");
        user.setAge(18);
        userMapper.insert(user);
    }
}
~~~

#### QueryWraper测试

~~~java

import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.baomidou.mybatisplus.core.conditions.update.UpdateWrapper;
....

/**
 * mybatisPlus Wrapper查询测试
 *
 * @author tzcqupt
 * @date 2020-03-05
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class QueryWrapperTest {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void testDelete() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper
                .isNotNull("name")
                .ge("age", 26)
                .isNull("create_time");
        int result = userMapper.delete(queryWrapper);
        System.out.println(result);
    }

    @Test
    public void testSelectOne() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.eq("name", "tang");
        User user = userMapper.selectOne(queryWrapper);
        System.out.println(user);
    }

    @Test
    public void testSelectCount() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.between("age", 20, 30);
        Integer result = userMapper.selectCount(queryWrapper);
        System.out.println(result);
    }

    @Test
    public void testSelectList() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        Map<String, Object> map = new HashMap<>();
        map.put("id", 2);
        map.put("name", "Jack");
        queryWrapper.allEq(map);
        List<User> users = userMapper.selectList(queryWrapper);
        users.forEach(System.out::println);
    }

    @Test
    public void testSelectMaps() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.notLike("name", "e")
                .likeRight("email", "t");
        List<Map<String, Object>> maps = userMapper.selectMaps(queryWrapper);
        maps.forEach(System.out::println);
    }

    /**
     * inSql、notinSql实现子查询
     */
    @Test
    public void testSelectObjs() {
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        queryWrapper.inSql("id", "select id from user where id >3");
        List<Object> list = userMapper.selectObjs(queryWrapper);
        list.forEach(System.out::println);
    }

    @Test
    public void testUpdate() {
        //修改值
        User user = new User();
        user.setAge(99);
        user.setName("Andy");

        //修改条件
        UpdateWrapper<User> userUpdateWrapper = new UpdateWrapper<>();
        userUpdateWrapper
                .like("name", "h")
                .or(i -> i.eq("name", "tang").ne("age", 20));

        int result = userMapper.update(user, userUpdateWrapper);

        System.out.println(result);
    }
}

~~~

