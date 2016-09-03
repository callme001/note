## spring-redis整合

基于前面整合的ssm框架扩展

`pom.xml`

```xml
    <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.7.3</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.data</groupId>
      <artifactId>spring-data-redis</artifactId>
      <version>1.6.2.RELEASE</version>
    </dependency>
```

新建立一个spring配置文件命名为：appContext-cache.xml
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


    <bean id="cacheSchoolDao" class="com.shundai.cache.CacheSchoolDaoImpl">
        <property name="redisTemplate" ref="redisTemplate"/>
    </bean>

</beans>
```

建立资源配置文件命名为redis.properties：
```xml
# Redis settings
#redis.host=192.168.20.101
#redis.port=6380
#redis.pass=foobared
redis.host=192.168.1.115
redis.port=6379
redis.pass=

redis.maxIdle=300
redis.testOnBorrow=true
```


创建接口CacheSchoolDao

```java
package com.shundai.cache;

import com.shundai.model.School;

import java.util.List;

/**
 * Created by 001 on 2016/7/2.
 */
public interface CacheSchoolDao {
    List<School> queryAllSchool();
    void cacheAllSchool(List<School> schoolList);
}

```

实现类CacheSchoolDaoImpl

```java
package com.shundai.cache;

import com.shundai.model.School;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;

import java.io.*;
import java.util.List;

/**
 * Created by 001 on 2016/7/2.
 */
public class CacheSchoolDaoImpl implements CacheSchoolDao {

    @Autowired
    private RedisTemplate<Serializable, Serializable> redisTemplate;

    public RedisTemplate<Serializable, Serializable> getRedisTemplate() {
        return redisTemplate;
    }

    public void setRedisTemplate(RedisTemplate<Serializable, Serializable> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }



    public List<School> queryAllSchool() {

        return redisTemplate.execute(new RedisCallback<List<School>>() {
            public List<School> doInRedis(RedisConnection redisConnection) throws DataAccessException {
                byte[] key = redisTemplate.getStringSerializer().serialize("all_school_list");
                if (redisConnection.exists(key)) {
                    byte[] value = redisConnection.get(key);
                    ByteArrayInputStream bis = new ByteArrayInputStream (value);
                    try {
                        ObjectInputStream objectInputStream = new ObjectInputStream(bis);
                        List<School> schoolList = (List<School>)objectInputStream.readObject();

                        return schoolList;
                    } catch (IOException e) {
                        e.printStackTrace();
                    } catch (ClassNotFoundException e) {
                        e.printStackTrace();
                    }
                    return null;
                }
                return null;
            }
        });

    }

    public void cacheAllSchool(final List<School> schoolList) {
        redisTemplate.execute(new RedisCallback<Object>() {
            public Object doInRedis(RedisConnection redisConnection) throws DataAccessException {

                try {
                    ByteArrayOutputStream bos = new ByteArrayOutputStream();
                    ObjectOutputStream os = new ObjectOutputStream(bos);
                    os.writeObject(schoolList);
                    os.flush();
                    os.close();
                    byte[] b = bos.toByteArray();
                    bos.close();

                    redisConnection.set(redisTemplate.getStringSerializer().serialize("all_school_list"),b);
                } catch (IOException e) {
                    e.printStackTrace();
                }

                return null;
            }
        });
    }
}

```

创建单元测试：
```java
package com.shundai.cache;

import com.shundai.dao.SchoolDao;
import com.shundai.model.School;
import org.junit.BeforeClass;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

import static org.junit.Assert.*;

/**
 * Created by 001 on 2016/7/2.
 */
public class CacheSchoolDaoImplTest {

    private static ApplicationContext context;

    private static CacheSchoolDao cacheSchoolDao;

    @BeforeClass
    public static void init(){
        context = new ClassPathXmlApplicationContext("classpath:appContext-cache.xml");
        cacheSchoolDao = context.getBean("cacheSchoolDao",CacheSchoolDao.class);
    }
    

    @Test
    public void cacheAllSchool() throws Exception {

        List<School> schoolList = new ArrayList<School>();

        School school = new School();
        school.setId(1);
        school.setSchool_name("school_name1");
        school.setSchool_remark("school_remark1");

        School school2 = new School();
        school2.setSchool_remark("school2_remark");
        school2.setId(2);
        school2.setSchool_name("school2_name");

        schoolList.add(school);
        schoolList.add(school2);

        cacheSchoolDao.cacheAllSchool(schoolList);


        List<School> nSchoolList = cacheSchoolDao.queryAllSchool();

        Iterator<School> iterator = nSchoolList.iterator();
        while (iterator.hasNext()){
            School tmp = iterator.next();
            System.out.println("cache:" + tmp.getId() + "-----"+tmp.getSchool_name());
        }

    }

}
```


能获取如下输出表示缓存成功：

![](http://i.imgur.com/Vh1Vmv6.png)