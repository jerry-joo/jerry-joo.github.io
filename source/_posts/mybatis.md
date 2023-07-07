---
title: mybatis
mathjax: true
categories: mybatis
tags: 
---
## mybatis用法

<!--more-->

1.导入依赖

2.编写核心配置文件  （ressources文件夹中创建config.xml)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8&amp;serverTimezone=Asia/Shanghai"/>
                <property name="username" value="改成自己的账户"/>
                <property name="password" value="改成自己的密码"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="改成mapper层xml的路径"/>
    </mappers>
</configuration>
```

3.编写mybatis工具类(Util层)

```java
public class MybatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
	
    //获取SqlSession连接
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```



4.创建实体类

5.编写mapper（dao层）接口类

6.编写mapper（dao层）配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="接口类的路径">
    <select id="接口类方法" resultType="实体类的路径">
        select *from (数据表)
    </select>
    
</mapper>
```



7.测试 （junit包测试）

```java
public class UserDaoTest {
    @Test
    public void test(){
        SqlSession sqlSession= MybatisUtils.getSqlSession();//反射获取事务
        UserDao userDao=sqlSession.getMapper(UserDao.class);//映射事务
        
        sqlSession.commit();//提交事务处理
        sqlSession.close(); //关闭事务
    }
}
```





## 注意事项 ##

1、maven静态资源过滤问题

​	即target中无法实时生成java目录下的xml，properties文件

​	解决方式：

​	在pom.xml中加入以下build

```java
<build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
 </build>
```



2.CRUD操作时关闭session前需要提交事务





