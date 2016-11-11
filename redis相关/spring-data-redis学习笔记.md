### 前言
---
Spring Data Redis是spring-data的一个部分，用于redis和spring整合，使得redis更易于配置和使用。封装了常用的一些操作，使得用户不在关注底层操作实现，更加专注于逻辑实现。

#### 安装和依赖

pom.xml

``` xml
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.7.3</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
      <version>1.7.2.RELEASE</version>
    </dependency>
```

#### 配置

新建一个文件命名为：redis.properties

```xml
# Redis settings

redis.host=192.168.1.115
redis.port=6379
redis.pass=
redis.maxIdle=300
redis.testOnBorrow=true
```

新建spring配置文件命名为(命名空间为IDEA自动添加 快捷键`Alt+Enter`)：appContext-cache.xml

添加redis.properties值的引用以及添加包自动扫描，appContext-cache.xml添加如下内容：
```xml
    <context:property-placeholder location="classpath:redis.properties" />
    <context:component-scan base-package="com.shundai.cache"/>
```

配置redis参数(实际生成环境有较多参数，只是其中两个)，appContext-cache.xml添加如下内容：

```xml
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>
```

配置jedisConnectionFactory以及redisTemlate，appContext-cache.xml添加如下内容：

```xml
    <bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:hostName="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" p:poolConfig-ref="poolConfig">
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory"   ref="connectionFactory" />
    </bean>
```

最后看一下总的appContext-cache.xml配置文件内容：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:redis.properties" />

    <context:component-scan base-package="com.shundai.cache"/>

    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="${redis.maxIdle}" />
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
    </bean>

    <bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
          p:hostName="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" p:poolConfig-ref="poolConfig">
    </bean>

    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory"   ref="connectionFactory" />
    </bean>
</beans>
```

**自此，简单的redis连接以及参数配置完成**


