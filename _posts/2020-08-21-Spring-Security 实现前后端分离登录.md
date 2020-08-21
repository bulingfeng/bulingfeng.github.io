---
title: Spring-Security 实现前后端分离登录
tags: java Spring-Framework Spring-Security
categories: Spring-Security
---

* TOC
{:toc}

# 

## 简述

使用**Spring-Security**来实现登录，但是搜到的都是通过模板引擎的方式来实现的，也就是必须通过login.html页面来登录。考虑到现在架构都是采用的是动静分离的架构，那么登录也需要使用纯Restful Api的方式来实现。

项目demo已经写好：

```
https://github.com/bulingfeng/spring-security-login.git
```

## 源码介绍

### 1、pom文件的引用

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
```

详细的引用包请查看git项目。

### 2、配置登录

```
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

	private PasswordEncoder myPasswordEncoder;

	private CustomUserService myCustomUserService;

	private ObjectMapper objectMapper;

	public SecurityConfig(CustomUserService myCustomUserService, ObjectMapper objectMapper) {
		this.myPasswordEncoder =new BCryptPasswordEncoder();
		this.myCustomUserService = myCustomUserService;
		this.objectMapper = objectMapper;
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {

		http
				.authenticationProvider(authenticationProvider())
				.httpBasic()
				//未登录时，进行json格式的提示，很喜欢这种写法，不用单独写一个又一个的类
				.authenticationEntryPoint((request,response,authException) -> {
					response.setContentType("application/json;charset=utf-8");
					response.setStatus(HttpServletResponse.SC_FORBIDDEN);
					PrintWriter out = response.getWriter();
					Map<String,Object> map = new HashMap<String,Object>();
					map.put("code",403);
					map.put("message","未登录");
					out.write(objectMapper.writeValueAsString(map));
					out.flush();
					out.close();
				})

				.and()
				.authorizeRequests()
				.anyRequest().authenticated() //必须授权才能范围

				.and()
				.formLogin() //使用自带的登录
				.permitAll()
				//登录失败，返回json
				.failureHandler((request,response,ex) -> {
					response.setContentType("application/json;charset=utf-8");
					response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
					PrintWriter out = response.getWriter();
					Map<String,Object> map = new HashMap<String,Object>();
					map.put("code",401);
					if (ex instanceof UsernameNotFoundException || ex instanceof BadCredentialsException) {
						map.put("message","用户名或密码错误");
					} else if (ex instanceof DisabledException) {
						map.put("message","账户被禁用");
					} else {
						map.put("message","登录失败!");
					}
					out.write(objectMapper.writeValueAsString(map));
					out.flush();
					out.close();
				})
				//登录成功，返回json
				.successHandler((request,response,authentication) -> {
					Map<String,Object> map = new HashMap<String,Object>();
					map.put("code",200);
					map.put("message","登录成功");
					map.put("data",authentication);
					response.setContentType("application/json;charset=utf-8");
					PrintWriter out = response.getWriter();
					out.write(objectMapper.writeValueAsString(map));
					out.flush();
					out.close();
				})
				.and()
				.exceptionHandling()
				//没有权限，返回json
				.accessDeniedHandler((request,response,ex) -> {
					response.setContentType("application/json;charset=utf-8");
					response.setStatus(HttpServletResponse.SC_FORBIDDEN);
					PrintWriter out = response.getWriter();
					Map<String,Object> map = new HashMap<String,Object>();
					map.put("code",403);
					map.put("message", "权限不足");
					out.write(objectMapper.writeValueAsString(map));
					out.flush();
					out.close();
				})
				.and()
				.logout()
				//退出成功，返回json
				.logoutSuccessHandler((request,response,authentication) -> {
					Map<String,Object> map = new HashMap<String,Object>();
					map.put("code",200);
					map.put("message","退出成功");
					map.put("data",authentication);
					response.setContentType("application/json;charset=utf-8");
					PrintWriter out = response.getWriter();
					out.write(objectMapper.writeValueAsString(map));
					out.flush();
					out.close();
				})
				.permitAll();
		//开启跨域访问
		http.cors().disable();
		//开启模拟请求，比如API POST测试工具的测试，不开启时，API POST为报403错误
		http.csrf().disable();
	}

	@Override
	public void configure(WebSecurity web) {
		//对于在header里面增加token等类似情况，放行所有OPTIONS请求。
		web.ignoring().antMatchers(HttpMethod.OPTIONS, "/**");
	}

	@Bean
	public AuthenticationProvider authenticationProvider() {
		DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
		//对默认的UserDetailsService进行覆盖
		authenticationProvider.setUserDetailsService(myCustomUserService);
		authenticationProvider.setPasswordEncoder(myPasswordEncoder);
		return authenticationProvider;
	}

}
```

### 3、查询用户信息

必须实现**UserDetailsService#loadUserByUsername**，这里只需使用用户名称查询用户信息即可，校验用户账号和密码的正确性都交给Spring-Security来实现即可。

```java
/**
 * @Author:bulingfeng
 * @Date: 2020-06-13
 */
@Component
@Slf4j
public class CustomUserService implements UserDetailsService {

    @Autowired
    private SysUserDao userDao;


    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SysUserPo po=userDao.selectUserByName2(username);
        return po;
    }
}
```

### 4、数据库代码

用户查询的接口

```java
/**
 * @Author:bulingfeng
 * @Date: 2020-06-13
 */
public interface SysUserDao extends BaseMapper<SysUserPo> {
    SysUserPo selectUserByName2(String userName);
}
```

Mybatis中配置查询

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.bulingfeng.login.dao.SysUserDao">
    <select id="selectUserByName2" resultType="com.bulingfeng.login.entity.SysUserPo" parameterType="java.lang.String">
        select
        id id,
        user_name userName,
        password password,
        role role
        from
        sys_user
        where user_name=#{userName}
    </select>
</mapper>
```

### 5、配置文件

```yml
server:
  port: 8080


mybatis-plus:
  type-aliases-package: com.example.entity
  mapper-locations: classpath:/mapper/*Mapper.xml

spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/spring-boot-login
    username: root
    password: mac666
    driver-class-name: com.mysql.jdbc.Driver

logging:
  level:
    root: debug
```

### 6、测试controller

```java
/**
 * @Author:bulingfeng
 * @Date: 2020-08-21
 */
@RestController
public class TestController {

    @GetMapping("/test")
    public String testApi(){
        return "success";
    }

}
```

## 测试

1、启动服务器，然后调用测试接口:

```
127.0.0.1:8080/test
```

返回消息为：

```json
{
    "code": 403,
    "message": "未登录"
}
```

2、使用账号和密码登录。（使用postman调用的方式必须为**form-data**表单的形式）

入参:username:root password:password。

返回结果:

```
{
    "code": 200,
    "data": {
        "authorities": [
            {
                "authority": "root"
            }
        ],
        "details": {
            "remoteAddress": "127.0.0.1",
            "sessionId": "240B9A8610686B534CFAC5BB7FA7D44A"
        },
        "authenticated": true,
        "principal": {
            "id": 1,
            "password": "$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG",
            "role": "ROLE_ADMIN",
            "authorities": [
                {
                    "authority": "root"
                }
            ],
            "username": "root",
            "accountNonExpired": true,
            "accountNonLocked": true,
            "credentialsNonExpired": true,
            "enabled": true
        },
        "credentials": null,
        "name": "root"
    },
    "message": "登录成功"
}
```

这里第一次调用sessionId会为null，第二次以后会正常。此问题还没有修复。

3、再次调用测试接口

```
127.0.0.1:8080/test
```

返回参数:

```
success
```

## 总结

该项目实现了登陆，并且也实现了前后端的完全分离，从而前后端没有任何耦合性，所有接口都只有Http的方式来调用。从而实现完全解耦，这种方式的优势是可以后续如果是分布式部署的话，只需要实现session共享即可完成web项目的高可用。

后续会出一个关于session共享的博文和项目来描述。

## 查阅文档

```
https://www.jianshu.com/p/650a497b3a40
https://spring.io/blog/2017/11/01/spring-security-5-0-0-rc1-released#password-storage-format
```

