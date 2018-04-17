## Spring+SpringMVC+mabatis总结

###加载配置文件。
###一.配置web.xml文件。
	  
	主要配置两个类：
	1.配置DispatcherServlet(控制器）
	<servlet>
		<servlet-name>SpringMVC</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-*.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>SpringMVC</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>

	2.配置过滤器
	<filter>
		<display-name>CharacterEncodingFilter</display-name>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<description></description>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>

	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

###	二、配置mvc，mvc主要配置扫描组件，视图，和拦截器。

		<context:component-scan 
		base-package="cn.tedu.spring.controller" />
		
	<!-- 配置ViewResolver -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix"
			value="/WEB-INF/" />
		<property name="suffix"
			value=".jsp" />
	</bean>
	
	<!-- 声明拦截器组件的拦截位置 -->
	<mvc:interceptors>
		<mvc:interceptor>
			<!-- 设置被拦截器拦截处理的URL -->
			<!-- 可以设置具体URL如 /demo.do 
			也可以设置 /** 处理任何的URL
			或者 /user/** 处理 user 路径下的URL  -->
			<mvc:mapping path="/**"/>
			<!-- 排除的URL, 不被拦截器拦截的URL -->
			<mvc:exclude-mapping path="/login.do"/>
			<mvc:exclude-mapping path="/handle_login.do"/>
			<mvc:exclude-mapping path="/register.do"/>
			<mvc:exclude-mapping path="/handle_register.do"/>
			<!-- 指定拦截器 -->
			<bean class="cn.tedu.spring.web.AccessInterceptor" />
		</mvc:interceptor>
	
	</mvc:interceptors>

	**配置service** 主要是把配置文件分离，使结构清晰。
		<context:component-scan 
		base-package="cn.tedu.spring.service" />

	***配置dao（数据持久层）**需要提供db.properties文件。
	<context:component-scan 
		base-package="cn.tedu.spring.mapper" />
		
	<!-- 读取db.properties -->
	<util:properties
		id="dbConfig"
		location="classpath:db.properties" />
		
	<!-- 配置BasicDataSource -->
	<!-- 注入属性值时 -->
	<!-- name的值是BasicDataSource类中声明的属性名称 -->
	<!-- value的值是使用Spring表达式从dbConfig中获取的 -->
	<!-- Spring表达式中 -->
	<!-- dbConfig后的名称源自db.properties中使用的名称 -->
	<bean
		id="dataSource"
		class="org.apache.commons.dbcp.BasicDataSource">
		<property name="url"
			value="#{dbConfig.url}" />
		<property name="driverClassName"
			value="#{dbConfig.driver}" />
		<property name="username"
			value="#{dbConfig.username}" />
		<property name="password"
			value="#{dbConfig.password}" />
		<property name="initialSize"
			value="#{dbConfig.initSize}" />
		<property name="maxActive"
			value="#{dbConfig.maxActive}" />
	</bean>
	
	<!-- MapperScannerConfigurer -->
	<bean
		class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 配置mybatis的接口文件所在的根包 -->
		<property name="basePackage"
			value="cn.tedu.spring.mapper" />
	</bean>
	
	<!-- SqlSessionFactoryBean -->
	<bean
		class="org.mybatis.spring.SqlSessionFactoryBean">
		<!-- DataSource -->
		<!-- 取值是配置的BasicDataSource的id -->
		<property name="dataSource"
			ref="dataSource" />
		<!-- 映射文件在哪里 -->
		<property name="mapperLocations"
			value="classpath:mappers/*" />
	</bean>


###配置mybatis文件

	<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper 
	namespace="cn.tedu.spring.mapper.UserMapper">
	
	<!-- 用户注册 -->
	<!-- Integer register(User user) -->
	<!-- 当前方法需要执行INSERT操作，
	所以使用insert节点进行配置 -->
	<!-- 每个节点的id都表示对应的方法名称 -->
	<!-- parameterType表示参数类型，
	如果参数是基本类型或String可以不配置该属性 -->
	<!-- 所有增删改操作默认返回受影响的行数，
	但是，不配置返回值类型 -->
	<!-- 如果INSERT时需要得到新数据的ID，
	则需要配置useGeneratedKeys="true"，
	并配置keyProperty="id"，这个属性值是数据表中的字段名 -->
	<insert id="register"
		parameterType="cn.tedu.spring.entity.User"
		useGeneratedKeys="true"
		keyProperty="id">
		INSERT INTO t_user (
			username, password, phone
		) VALUES (
			#{username}, 
			#{password},
			#{phone}
		)
	</insert>
	
	<!-- 根据用户名查询用户信息 -->
	<!-- User findUserByUsername(String username) -->
	<!-- 所有的select节点都必须配置返回值类型 -->
	<!-- 即使返回的是基本类型或String也必须声明 -->
	<select id="findUserByUsername"
		resultType="cn.tedu.spring.entity.User">
		SELECT 
			id, username, password, phone 
		FROM 
			t_user 
		WHERE 
			username=#{username}
	</select>
	
	<!-- 查询所有用户信息 -->
	<!-- List<User> findAllUser() -->
	<!-- 如果需要查询结果是List集合，在节点上声明时，
	只需要标明List集合中的泛型类型即可 -->
	<!-- 其实，所有的select查出来的都是List -->
	<select id="findAllUser"
		resultType="cn.tedu.spring.entity.User">
		SELECT 
			id, username, password, phone 
		FROM 
			t_user
	</select>
	
	<!-- 删除用户数据 -->
	<!-- Integer delete(Integer id) -->
	<!-- 如果只有1个参数，
	且是基本类型(含包装类)或String，
	则不需要声明参数类型 -->
	<delete id="delete">
		DELETE FROM 
			t_user 
		WHERE 
			id=#{id}
	</delete>
	</mapper>




