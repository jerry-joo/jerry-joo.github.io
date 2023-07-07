---
title: springboot shiro
mathjax: true
categories: 后端
tags: 框架
---

# Shiro安全框架

<!--more-->

## 1.核心组件

- Subjuct

  操作用户

- SecurityManager

  Shiro框架的核心，用于管理其他组件，提供安全管理的各种服务

- Realms

  自定义的用户权限验证信息



## 2.认证流程

​		1.获取当前用户subject

​		2.根据用户信息执行登陆操作，用户信息用token保存

```java
subject.login(token)
```

​		3.shiro自动读取自定义Realm中认证方法

​		4.根据认证方法的结果返回前端信息

## 3.用法

### 1.依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-starter</artifactId>
    <version>1.7.1</version>
</dependency>
```



### 2.创建SecurityManager

​	新建ShiroConfig类 添加@Configuration注释

```java
@Configuration
public class Shiroconfig {
    @Bean()
    public DefaultWebSecurityManager securityManager() {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        SecurityUtils.setSecurityManager(securityManager);
        return securityManager;
    }
}
```

### 3.创建自己的Realms

​	新建Realm类   类名自定义

```java
public class AcRealm extends AuthorizingRealm {
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }
	//认证（校验密码）
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        return null;
    }
}

```

#### 	关于认证方法参数AuthenticationToken

​	AuthenticationToken接口类

```java
package org.apache.shiro.authc;

import java.io.Serializable;

public interface AuthenticationToken extends Serializable {
    Object getPrincipal();
    Object getCredentials();
}
```

​		getPrincipal()方法可以获得token中用户的名字

​		getCredentils()方法可以获得token中用户的密码



#### 认证方法

```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
    String username= (String) authenticationToken.getPrincipal();//获取用户名
    User user=userService.check(username);//连接数据库获取该用户密码
    if(user!=null){
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(username,user.getPwd(), this.getName());
        return authenticationInfo;//查到用户返回用户信息
    }
    return null;//为查到用户返回null
}
```

​	 <font color=#FF0000>注意</font>:认证方法中并未出现密码的比对。

​	原因：shiro在认证方法中创建SimpleAuthenticationInfo对象时会自动验证密码，因此明文密码比对并不会出现在Realm类中，所以认证方法并不能直接返回用户信息，必须先新建一个SimpleAuthenticationInfo，来进行密码验证。

#### 授权方法（未完成）

### 4.配置SecurityManager

在创建SecurityManager的方法中加入Realm的配置（可配置多个Realm）

```java
@Bean
public Realm R(){ //用于返回Realm对象
    return new Realm();
}

@Bean()
public DefaultWebSecurityManager securityManager() {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(R());//需要加入的配置
    SecurityUtils.setSecurityManager(securityManager);
    return securityManager;
}
```



### 5.创建测试Controller

```java
@RestController
public class AcController {
    @PostMapping("user/login")
    public Result login(@Validated @RequestBody LoginDto loginDto){
        Subject subject= SecurityUtils.getSubject(); //获取当前用户
        if(subject.isAuthenticated()){ //判断用户状态
            return Result.succ("已经登陆");
        }else{
            UsernamePasswordToken token =new UsernamePasswordToken(loginDto.getUsername(),loginDto.getPwd());
            token.setRememberMe(true);
            try{
                subject.login(token);  //执行用户登陆（调用Realm认证方法）
                return Result.succ("成功");
            }catch (UnknownAccountException e){ //认证方法返回null
                return Result.fail("账号错误");
            }catch (IncorrectCredentialsException e){ //认证密码错误时抛出
                return Result.fail("密码错误");
            }
        }
    }

    @GetMapping("user/logout")
    public Result logout(){
        Subject subject=SecurityUtils.getSubject();//获取当前用户
        subject.logout(); //注销
        return Result.succ("退出成功");
    }
}
```

## 4.跨域问题

​	Shiro会导致前端跨域问题

​	解决方法较多

​	后端解决方法：增加CorsConfig类

```java
@Configuration
public class CorsConfig {
    private CorsConfiguration buildConfig(){
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.setMaxAge(3600L);
        corsConfiguration.setAllowCredentials(true);
        return corsConfiguration;
    }

    @Bean
    public CorsFilter corsFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig());
        return new CorsFilter(source);
    }
}
```



## 5.注意事项

### 	1.循环依赖问题

​			解决方法：

​				更改依赖导入

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.6.0</version>
</dependency>
```

​				为

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.7.1</version>
</dependency>
```

