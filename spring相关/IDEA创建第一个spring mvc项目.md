# 使用IDEA一步步搭建ssm项目

### 搭建spring spring mvc项目

首先使用maven创建好demo webapp项目

pom.xml文件添加如下依赖：

``` xml
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>3.2.17.RELEASE</version>
</dependency>

<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>3.2.17.RELEASE</version>
</dependency>

<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>servlet-api</artifactId>
  <version>2.5</version>
</dependency>

<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jstl</artifactId>
  <version>1.2</version>
</dependency>
```

添加好依赖以后，需要配置web.xml用于拦截所有请求到spring mvc处理

web.xml添加如下内容：

``` xml
    <servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

该servlet名为mvc-dispatcher（名称可修改），用于拦截请求（url-pattern为 / ，说明拦截所有请求），并交由Spring MVC的后台控制器来处理。这一项配置是必须的。

为了能够处理中文的post请求，再配置一个encodingFilter，以避免post请求中文出现乱码情况：

``` xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
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
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

最后web.xml内容为：

``` xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Spring Mvc Demo Application</display-name>

  <servlet>
    <servlet-name>mvc-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>mvc-dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
  
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
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
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>

</web-app>
```

由于前面配置的`servlet-name`为`mvc-dispatcher` 。所以需在`web.xml`同级目录下新建 `mvc-dispatcher-servlet.xml`（-servlet前面是在servlet里面定义的servlet名）

`mvc-dispatcher-servlet.xml`文件内容为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```


添加一个包，`com.shundai.controller`。新建一个类命名为`HelloController`添加如下方法和注解

```java
package com.shundai.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

/**
 * Created by 001 on 2016/6/30.
 */
@Controller
public class HelloController {

    @RequestMapping(value = "/",method = RequestMethod.GET)
     public String index(){
         return "index";
     }

}

```

回到mvc-dispatcher-servlet.xml，进行相关配置。首先加入component-scan标签，指明controller所在的包，并扫描其中的注解（最好不要复制，输入时按IDEA会在beans xmlns中添加相关内容）：

需要添加xml的命名空间，xml变成如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.shundai.controller"/>
</beans>
```


再进行js、image、css等静态资源访问的相关配置，这样，SpringMVC才能访问网站内的静态资源(IDEA按下`ALT+ENTER`可以自己导入命名空间)：
```xml
<mvc:default-servlet-handler/>
```

再开启springmvc注解模式，由于我们利用注解方法来进行相关定义，可以省去很多的配置：

```xml
<mvc:annotation-driven/>
```

再进行视图解析器的相关配置：

```xml
<bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/pages/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

mvc-dispatcher-servlet.xml文件如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.shundai.controller"/>

    <mvc:default-servlet-handler/>

    <mvc:annotation-driven/>

    <bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```
在`/WEB-INF/views/`新建一个index.jsp文件，内容为：
```java
<%@ page isELIgnored="false" %>
<%@ page language="java" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>

```


配置好tomca之后访问，应该正常。

再次更改`HelloController`代码如下：

```java
package com.shundai.controller;

import com.shundai.model.Member;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;

/**
 * Created by 001 on 2016/6/30.
 */
@Controller
public class HelloController {

    @RequestMapping(value = "/",method = RequestMethod.GET)
    public String index(Model model){

        model.addAttribute("name","jincarry");

        return "index";

     }

}

```

更改`/views/index.jsp`为如下内容：

```java
<%@ page isELIgnored="false" %>
<%@ page language="java" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<body>
<h2>Hello World!</h2><h3>${name}</h3>
</body>
</html>

```
浏览器访问输出：

![](http://i.imgur.com/Y9liS8s.png)


如果需要分开写spring配置文件，比如分成`appContext-core.xml` `appContext-services.xml` `appContext-dao.xml` 方便查找以及编写。需要在web.xml里面做如下配置。为默认拦截全部的servlet添加子节点如下：
``` xml
  <servlet>
    <servlet-name>mvc-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/appContext-*.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
```

自此，spring mvc控制器视图部分到此结束

---

### 整合mybatis

新建一个spring配置文件名字为：appContext-config.xml用于存放数据库连接等配置文件

pom.xml添加所需要的依赖

```xml
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.2.8</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.18</version>
    </dependency>
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.1.1</version>
      <type>jar</type>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.2.1</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.6.11</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.0.20</version>
    </dependency>
```

添加model

新建立一个包名为com.shundai.model

创建类Member 内容为：

```java
package com.shundai.model;

/**
 * Created by 001 on 2016/6/30.
 */
public class Member {

    private int id;
    private String username;
    private String password;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```



dataSource我们选用阿里巴巴开源项目druid，不仅可以做数据库驱动 还可以检测

配置数据库连接字符串

首先新建一个databaseConfig.properties位于/WEB-INF/ 内容为：

```
jdbc.username=root
jdbc.password=whtdst
jdbc.url=jdbc:mysql://127.0.0.1:3306/address?useUnicode=true&amp;characterEncoding=UTF-8
```
`appContext-config.xml`中配置读取properties的beans
``` xml
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>/WEB-INF/databaseConfig.properties</value>
            </list>
        </property>
    </bean>
```

配置dataSource

`appContext-config.xml`中加入如下配置：

```xml
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="filters" value="stat,log4j"/>
        <property name="connectionProperties" value="druid.stat.slowSqlMillis=1000"/>
        <property name="useGlobalDataSourceStat" value="true"/>

    </bean>
```


配置mybatis

`/WEB-INF/`下创建一个文件夹命名为mybatis并且创建一个mybatis配置文件名字为mybatis-core.xml。内容为：

```xml 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- Globally enables or disables any caches configured in any mapper under this configuration -->
        <setting name="cacheEnabled" value="true"/>
        <!-- Sets the number of seconds the driver will wait for a response from the database -->
        <setting name="defaultStatementTimeout" value="3000"/>
        <!-- Enables automatic mapping from classic database column names A_COLUMN to camel case classic Java property names aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>

    <!-- Continue going here -->

</configuration>
```

配置mapper

在mybatis目录下创建目录MemberMapper 并且创建第一个mapper文件名字为member.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.shundai.dao.MemberDao">

    <cache/>

</mapper>
```

在mybatis-core.xml添加别名：内容为

``` xml
    <typeAliases>
        <typeAlias type="com.shundai.model.Member" alias="member"/>
    </typeAliases>
```

mybatis-core.xml添加Mapper文件映射：

```xml
    <mappers>
        <mapper resource="MemberMapper/member.xml"/>
    </mappers>
```

回到appContext-config.xml配置sqlsession

添加如下配置：

```xml
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="druidDataSource"/>
        <property name="configLocation" value="/WEB-INF/mybatis/mybatis-core.xml"/>
    </bean>
```

在member.xml配置resultMap

```xml
    <resultMap id="memberResult" type="member">
        <id property="id" column="member_id"/>
        <result property="username" column="member_username"/>
        <result property="password" column="member_password"/>
    </resultMap>
```

添加一个查询

```xml
    <select id="queryAll" resultMap="memberResult">
        SELECT id AS member_id,username AS member_username,password AS member_password FROM member
    </select>
```

添加DAO

建立包com.shundai.dao

添加接口名为MemberDao（注意该接口名字和Mybatis的配置文件member.xml配置文件的namespace一致）：

```java
package com.shundai.dao;

import com.shundai.model.Member;

import java.util.List;

/**
 * Created by 001 on 2016/6/30.
 */
public interface MemberDao {
    List<Member> queryAll();
}

```

新建立一个spring配置文件名为：appContext-dao.xml 内容为：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="memberDao" class="org.mybatis.spring.mapper.MapperFactoryBean">
        <property name="mapperInterface" value="com.shundai.dao.MemberDao"/>
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>
</beans>
```

新建立一个包名为com.shundai.service

建一个接口名字为MemberService 添加如下接口：

```java
package com.shundai.services;

import com.shundai.model.Member;

import java.util.List;

/**
 * Created by 001 on 2016/6/30.
 */
public interface MemberService {
    public List<Member> getAllMember();
}

```

创建一个类MemberServiceImpl实现接口MemberService：

```java
package com.shundai.services;

import com.shundai.dao.MemberDao;
import com.shundai.model.Member;

import java.util.List;

/**
 * Created by 001 on 2016/6/30.
 */
public class MemberServiceImpl implements MemberService {

    private MemberDao memberDao;

    public MemberDao getMemberDao() {
        return memberDao;
    }

    public void setMemberDao(MemberDao memberDao) {
        this.memberDao = memberDao;
    }

    public List<Member> getAllMember() {
        return memberDao.queryAll();
    }
}

```

新建配置文件appContext-service.xml

添加beans

```xml
    <bean id="memberService" class="com.shundai.services.MemberServiceImpl">
        <property name="memberDao" ref="memberDao"/>
    </bean>
```


添加事务

appContext-service.xml添加如下配置：

```xml
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="druidDataSource"/>
    </bean>
```

更改MemberServiceImpl代码为如下：

```java
package com.shundai.services;

import com.shundai.dao.MemberDao;
import com.shundai.model.Member;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

/**
 * Created by 001 on 2016/6/30.
 */
@Transactional
public class MemberServiceImpl implements MemberService {

    private MemberDao memberDao;

    public MemberDao getMemberDao() {
        return memberDao;
    }

    public void setMemberDao(MemberDao memberDao) {
        this.memberDao = memberDao;
    }

    @Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.READ_COMMITTED)
    public List<Member> getAllMember() {
        return memberDao.queryAll();
    }
}

```


将Mybatis文件夹以及子文件移动到resources下，并修改appContext-config.xml为如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>/WEB-INF/databaseConfig.properties</value>
            </list>
        </property>
    </bean>

    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="filters" value="stat,log4j"/>
        <property name="connectionProperties" value="druid.stat.slowSqlMillis=1000"/>
        <property name="useGlobalDataSourceStat" value="true"/>

    </bean>

    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="druidDataSource"/>
        <property name="configLocation" value="classpath:mybatis/mybatis-core.xml"/>
    </bean>

</beans>
```

修改mybatis-core.xml为：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <!-- Globally enables or disables any caches configured in any mapper under this configuration -->
        <setting name="cacheEnabled" value="true"/>
        <!-- Sets the number of seconds the driver will wait for a response from the database -->
        <setting name="defaultStatementTimeout" value="3000"/>
        <!-- Enables automatic mapping from classic database column names A_COLUMN to camel case classic Java property names aColumn -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- Allows JDBC support for generated keys. A compatible driver is required.
        This setting forces generated keys to be used if set to true,
         as some drivers deny compatibility but still work -->
        <setting name="useGeneratedKeys" value="true"/>
    </settings>

    <!-- Continue going here -->

    <typeAliases>
        <typeAlias type="com.shundai.model.Member" alias="member"/>

    </typeAliases>

    <mappers>
        <mapper resource="mybatis/MemberMapper/member.xml"/>
    </mappers>

</configuration>
```

更改controller为如下代码：

```java
package com.shundai.controller;

import com.shundai.model.Member;
import com.shundai.services.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import java.util.List;

/**
 * Created by 001 on 2016/6/30.
 */
@Controller
public class HelloController {

    @Autowired
    private MemberService memberService;

    public MemberService getMemberService() {
        return memberService;
    }

    public void setMemberService(MemberService memberService) {
        this.memberService = memberService;
    }

    @RequestMapping(value = "/",method = RequestMethod.GET)
    public String index(Model model){



        List<Member> list = memberService.getAllMember();

        model.addAttribute("name","jincarry");

        model.addAttribute("members",list);

        return "index";

     }

}

```

index.jsp改为如下代码：

```java
<%@ page isELIgnored="false" %>
<%@ page language="java" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<html>
<body>
<h2>Hello World!</h2><h3>${name}</h3>

<table>
    <c:forEach items="${members}" var="member">
    <tr>
        <td>${member.id}</td>
        <td>${member.username}</td>
        <td>${member.password}</td>
    </tr>
    </c:forEach>
</table>
</body>
</html>

```


配置druid监控

web.xml添加如下内容：

```xml
  <servlet>
    <servlet-name>DruidStatView</servlet-name>
    <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>DruidStatView</servlet-name>
    <url-pattern>/druid/*</url-pattern>

  </servlet-mapping>

  <filter>
    <filter-name>DruidWebStatFilter</filter-name>
    <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
    <init-param>
      <param-name>exclusions</param-name>
      <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
    </init-param>
    <init-param>
      <param-name>profileEnable</param-name>
      <param-value>true</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>DruidWebStatFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```


appContext-core.xml添加如下代码：

```xml
    <bean id="druid-stat-interceptor"
          class="com.alibaba.druid.support.spring.stat.DruidStatInterceptor">
    </bean>

    <bean id="druid-stat-pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut"
          scope="prototype">
        <property name="patterns">
            <list>
                <value>com.shundai.service.*</value>
                <value>com.shundai.dao.*</value>
                <value>com.shundai.controller.*</value>
            </list>
        </property>
    </bean>

    <aop:config>
        <aop:advisor advice-ref="druid-stat-interceptor" pointcut-ref="druid-stat-pointcut" />
    </aop:config>
```




