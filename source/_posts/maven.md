---
title: maven
mathjax: true
categories: 
tags: 
---
# Maven

<!--more-->

## 注意事项

### 1.刷新maven依赖后自动重置java Complier和Language Level

解决方案：

在pom.xml中添加以下代码

```xml
<properties>
    <maven.complier.source>1.8</maven.complier.source>
    <maven.complier.target>1.8</maven.complier.target>
</properties>
```

### 2.maven静态资源过滤问题

​	即target中无法实时生成java目录下的xml，properties文件

​	解决方式：

​	在pom.xml中加入以下build

```xml
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

