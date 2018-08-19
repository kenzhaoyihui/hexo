title: SSM 配置文件整合
author: Kenzhaoyihui
tags:
  - JAVA
categories:
  - JAVA
date: 2018-08-02 20:46:00
---
#### Spring + SpringMVC + MyBatis

* /WEB-INF/web.xml

```
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">


  <!-- 1. Start the Spring container-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
  </context-param>

  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>


  <!-- 2. springmvc dispatcherservlet handler, handle the request-->
  <servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:spring-mvc-servlet.xml</param-value>
    </init-param>

    <load-on-startup>1</load-on-startup>
  </servlet>


<!-- 3. UTF-8 fillter-->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>

    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>

    <init-param>
      <param-name>forceRequestEncoding</param-name>
      <param-value>true</param-value>
    </init-param>

    <init-param>
      <param-name>forceResponseEncoding</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>

  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>


 <!-- 4. Use Rest URI, tranfer the POST request to PUT or DELETE request, need the hiddenHttpMethodFilter-->
  <filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

  <filter>
    <filter-name>HttpPutFormContentFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>HttpPutFormContentFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

</web-app>


```


* Spring配置文件 resources/applicationContext.xml


db.properties

``` 
jdbc.jdbcUrl = jdbc:mariadb://127.0.0.1:3306/ssm_crud
jdbc.driverClass = org.mariadb.jdbc.Driver
jdbc.user = root
jdbc.password = 123456

```

applicationContext.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">


<!--Spring  configuration file, mianly about the transaction-->

<!--扫描除了controller包以外的包， 防止和springmvc里扫描重复， 防止springmvc ioc容器被多次扫描-->
    <context:component-scan base-package="com.yzhao">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    
    <!-- 加载数据库配置，为之后的数据源做准备-->
   <context:property-placeholder location="classpath:dbconfig.properties"/>
   
   <!--+++++++++++++++++++++++++++++++++++++++++++++++++++++++DataSource+++++++++++++++++++++++++++++++++++++++++-->
    <!--DataSource 注册数据库资源，这里用的c3p0， 当然也可以用其他的，如druid-->
    <bean id="pooledDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}"/>
        <property name="driverClass" value="${jdbc.driverClass}"/>
        <property name="user" value="${jdbc.user}"/>
        <property name="password" value="123456"/>
    </bean>
    
    
   <!-- Transaction 声明式事物处理-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <!-- manage the dataSource-->
        <property name="dataSource" ref="pooledDataSource"/>
    </bean>

    <!-- Can use the based annotation transaction or based the XML transaction(Advice to use the XML) 配置切面和切入点-->
    <aop:config>
        <!-- PointCut-->
        <aop:pointcut id="txPoint" expression="execution(* com.yzhao.crud.service..*(..))"/>

        <!-- transaction advisor-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
    </aop:config>


    <!-- Config the transaction enhancement(improvement), how the transaction to cut 创建通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- All the method is the transaction methods-->
            <tx:method name="*"/>

            <!-- methods of startwith the get-->
            <tx:method name="get*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    
    
  
   <!--+++++++++++++++++++++++++++++++++++++++++++++++++++About MyBatis++++++++++++++++++++++++++++++++++++++++-->
    <!-- Configuration about mybatis 整合mybatis-->

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--导入mybatis核心配置文件 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="dataSource" ref="pooledDataSource"/>

        <!-- mybatis mapper file， 导入映射文件-->
        <property name="mapperLocations" value="classpath:mapper/*.xml"/>
    </bean>

    <!-- Config the mybatis mapperScannerConfig, add the mybatis interface implements to spring IOC-->
    <!--Spring为mapper接口创建代理对象 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- Scanner all the dao interface implements , then add to the spring IOC-->
        <property name="basePackage" value="com.yzhao.crud.dao"/>
    </bean>


    <!-- Configuration the sqlSession-->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory" />
        <constructor-arg name="executorType" value="BATCH"/>
    </bean>


```


* resources/mybatis-config.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
<!--mybatis设置，如开启驼峰规则，懒加载等 -->
    <settings>

        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <setting name="lazyLoadingEnabled" value="true"/>
    </settings>


    <typeAliases>
        <package name="com.yzhao.crud.bean"/>
    </typeAliases>
    
    <!--分页插件 -->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <property name="reasonable" value="true"/>
        </plugin>
    </plugins>
    


    <!--mapper file path mapper映射文件路径-->
    <!--<mappers>-->
        <!--<mapper resource="mappers/OrderMapper.xml"/>-->
        <!--<mapper resource="mappers/UserMapper.xml"/>-->
    <!--</mappers>-->

```

* resources/[dispatcherServletName]-servlet.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- SpringMVC configuration file-->
	<!--扫描controller包，处理映射url和method -->
    <context:component-scan base-package="com.yzhao" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- ViewResolver 视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- Standand Configuration-->
    <!-- static resources 静态资源访问-->
    <mvc:default-servlet-handler/>

    <!--springmvc  the higher function, like: JSR303, ajax, mapping, "RequestMappingHandleMapping, ExceptionHandleExceptionResolver, RequestMappingHandlerAdapter-->
    <mvc:annotation-driven/>

```

* resources/mapper/*-mapper.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yzhao.crud.dao.EmployeeMapper">
  <resultMap id="BaseResultMap" type="com.yzhao.crud.bean.Employee">
    <id column="emp_id" jdbcType="INTEGER" property="empId" />
    <result column="emp_name" jdbcType="VARCHAR" property="empName" />
    <result column="gender" jdbcType="CHAR" property="gender" />
    <result column="email" jdbcType="VARCHAR" property="email" />
    <result column="d_id" jdbcType="INTEGER" property="dId" />
  </resultMap>

  <resultMap id="WithDeptResultMap" type="com.yzhao.crud.bean.Employee">
    <id column="emp_id" jdbcType="INTEGER" property="empId" />
    <result column="emp_name" jdbcType="VARCHAR" property="empName" />
    <result column="gender" jdbcType="CHAR" property="gender" />
    <result column="email" jdbcType="VARCHAR" property="email" />
    <result column="d_id" jdbcType="INTEGER" property="dId" />

    <association property="department" javaType="com.yzhao.crud.bean.Department">
      <id column="dept_id" property="deptId" />
      <result column="dept_name" property="deptName"/>
    </association>
  </resultMap>
  <sql id="Example_Where_Clause">
    <where>
      <foreach collection="oredCriteria" item="criteria" separator="or">
        <if test="criteria.valid">
          <trim prefix="(" prefixOverrides="and" suffix=")">
            <foreach collection="criteria.criteria" item="criterion">
              <choose>
                <when test="criterion.noValue">
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue">
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue">
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue">
                  and ${criterion.condition}
                  <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
  </sql>
  <sql id="Update_By_Example_Where_Clause">
    <where>
      <foreach collection="example.oredCriteria" item="criteria" separator="or">
        <if test="criteria.valid">
          <trim prefix="(" prefixOverrides="and" suffix=")">
            <foreach collection="criteria.criteria" item="criterion">
              <choose>
                <when test="criterion.noValue">
                  and ${criterion.condition}
                </when>
                <when test="criterion.singleValue">
                  and ${criterion.condition} #{criterion.value}
                </when>
                <when test="criterion.betweenValue">
                  and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                </when>
                <when test="criterion.listValue">
                  and ${criterion.condition}
                  <foreach close=")" collection="criterion.value" item="listItem" open="(" separator=",">
                    #{listItem}
                  </foreach>
                </when>
              </choose>
            </foreach>
          </trim>
        </if>
      </foreach>
    </where>
  </sql>
  <sql id="Base_Column_List">
    emp_id, emp_name, gender, email, d_id
  </sql>
  <sql id="WithDept_Column_List">
    e.emp_id, e.emp_name, e.gender, e.email, e.d_id, d.dept_id, d.dept_name
  </sql>
  <select id="selectByExample" parameterType="com.yzhao.crud.bean.EmployeeExample" resultMap="BaseResultMap">
    select
    <if test="distinct">
      distinct
    </if>
    <include refid="Base_Column_List" />
    from tbl_emp
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null">
      order by ${orderByClause}
    </if>
  </select>


  <!--add the Department information with the select-->
  <select id="selectByExampleWithDept" parameterType="com.yzhao.crud.bean.EmployeeExample" resultMap="WithDeptResultMap">
    select
    <if test="distinct">
      distinct
    </if>
    <include refid="WithDept_Column_List" />
    from tbl_emp e
    left join tbl_dept d on e.`d_id`= d.`dept_id`
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null">
      order by ${orderByClause}
    </if>
  </select>
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from tbl_emp
    where emp_id = #{empId,jdbcType=INTEGER}
  </select>
  <select id="selectByPrimaryKeyWithDept" parameterType="java.lang.Integer" resultMap="WithDeptResultMap">
    select
    <include refid="WithDept_Column_List" />
    from tbl_emp e
    left join tbl_dept d on e.`d_id`=d.`dept_id`
    where emp_id = #{empId,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from tbl_emp
    where emp_id = #{empId,jdbcType=INTEGER}
  </delete>
  <delete id="deleteByExample" parameterType="com.yzhao.crud.bean.EmployeeExample">
    delete from tbl_emp
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
  </delete>
  <insert id="insert" parameterType="com.yzhao.crud.bean.Employee">
    insert into tbl_emp (emp_id, emp_name, gender, 
      email, d_id)
    values (#{empId,jdbcType=INTEGER}, #{empName,jdbcType=VARCHAR}, #{gender,jdbcType=CHAR}, 
      #{email,jdbcType=VARCHAR}, #{dId,jdbcType=INTEGER})
  </insert>
  <insert id="insertSelective" parameterType="com.yzhao.crud.bean.Employee">
    insert into tbl_emp
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="empId != null">
        emp_id,
      </if>
      <if test="empName != null">
        emp_name,
      </if>
      <if test="gender != null">
        gender,
      </if>
      <if test="email != null">
        email,
      </if>
      <if test="dId != null">
        d_id,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="empId != null">
        #{empId,jdbcType=INTEGER},
      </if>
      <if test="empName != null">
        #{empName,jdbcType=VARCHAR},
      </if>
      <if test="gender != null">
        #{gender,jdbcType=CHAR},
      </if>
      <if test="email != null">
        #{email,jdbcType=VARCHAR},
      </if>
      <if test="dId != null">
        #{dId,jdbcType=INTEGER},
      </if>
    </trim>
  </insert>
  <select id="countByExample" parameterType="com.yzhao.crud.bean.EmployeeExample" resultType="java.lang.Long">
    select count(*) from tbl_emp
    <if test="_parameter != null">
      <include refid="Example_Where_Clause" />
    </if>
  </select>
  <update id="updateByExampleSelective" parameterType="map">
    update tbl_emp
    <set>
      <if test="record.empId != null">
        emp_id = #{record.empId,jdbcType=INTEGER},
      </if>
      <if test="record.empName != null">
        emp_name = #{record.empName,jdbcType=VARCHAR},
      </if>
      <if test="record.gender != null">
        gender = #{record.gender,jdbcType=CHAR},
      </if>
      <if test="record.email != null">
        email = #{record.email,jdbcType=VARCHAR},
      </if>
      <if test="record.dId != null">
        d_id = #{record.dId,jdbcType=INTEGER},
      </if>
    </set>
    <if test="_parameter != null">
      <include refid="Update_By_Example_Where_Clause" />
    </if>
  </update>
  <update id="updateByExample" parameterType="map">
    update tbl_emp
    set emp_id = #{record.empId,jdbcType=INTEGER},
      emp_name = #{record.empName,jdbcType=VARCHAR},
      gender = #{record.gender,jdbcType=CHAR},
      email = #{record.email,jdbcType=VARCHAR},
      d_id = #{record.dId,jdbcType=INTEGER}
    <if test="_parameter != null">
      <include refid="Update_By_Example_Where_Clause" />
    </if>
  </update>
  <update id="updateByPrimaryKeySelective" parameterType="com.yzhao.crud.bean.Employee">
    update tbl_emp
    <set>
      <if test="empName != null">
        emp_name = #{empName,jdbcType=VARCHAR},
      </if>
      <if test="gender != null">
        gender = #{gender,jdbcType=CHAR},
      </if>
      <if test="email != null">
        email = #{email,jdbcType=VARCHAR},
      </if>
      <if test="dId != null">
        d_id = #{dId,jdbcType=INTEGER},
      </if>
    </set>
    where emp_id = #{empId,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="com.yzhao.crud.bean.Employee">
    update tbl_emp
    set emp_name = #{empName,jdbcType=VARCHAR},
      gender = #{gender,jdbcType=CHAR},
      email = #{email,jdbcType=VARCHAR},
      d_id = #{dId,jdbcType=INTEGER}
    where emp_id = #{empId,jdbcType=INTEGER}
  </update>
```

可以由mybatis generator自动生成， idea编辑器有相关编译功能(调用的maven)，
当然也可以通过代码搞定。

*  mbg.xml, 相关配置可查看mybatis-generator 的官方网站

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">


<generatorConfiguration>
    <!--<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />-->


    <!-- Database Connection-->
    <context id="DB2Tables" targetRuntime="MyBatis3">

        <commentGenerator>
            <property name="suppressAllComments" value="true" />
        </commentGenerator>

        <jdbcConnection driverClass="org.mariadb.jdbc.Driver"
                        connectionURL="jdbc:mariadb://127.0.0.1:3306/ssm_crud"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>

        <!-- Java bean location-->
        <javaModelGenerator targetPackage="com.yzhao.crud.bean" targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>

        <!-- Sql mapper file location-->
        <sqlMapGenerator targetPackage="mapper"  targetProject="./src/main/resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!-- DAO interface location, mapper interface-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.yzhao.crud.dao"  targetProject="./src/main/java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!-- sql table generator policy-->
        <!--<table schema="DB2ADMIN" tableName="ALLTYPES" domainObjectName="Customer" >-->
            <!--<property name="useActualColumnNames" value="true"/>-->
            <!--<generatedKey column="ID" sqlStatement="DB2" identity="true" />-->
            <!--<columnOverride column="DATE_FIELD" property="startDate" />-->
            <!--<ignoreColumn column="FRED" />-->
            <!--<columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />-->
        <!--</table>-->

        <table tableName="tbl_emp" domainObjectName="Employee">

        </table>

        <table tableName="tbl_dept" domainObjectName="Department">

        </table>
    </context>
</generatorConfiguration>

```

生成代码： MBG.java
```
package com.yzhao.crud.test;

import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;

import java.io.File;
import java.util.ArrayList;
import java.util.List;

public class MBGTTest {

    public static void  main(String[] args) throws Exception{
        List<String> warnings = new ArrayList<>();
        boolean overwrite = true;
        File configFile = new File("mbg.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);

        myBatisGenerator.generate(null);

    }
}

```
