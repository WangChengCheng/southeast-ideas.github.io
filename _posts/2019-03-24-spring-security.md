---
layout: post
title:  spring-boot-security-start
date:   2019-03-24 17:41:00 +0800
categories: 框架
tag: Project SpringSecurity
author: Michael Wang
source_id: 190324174100
---

* content
{:toc}

使用spring-boot-starter-security对Spring Boot项目增加接口验证。

>添加如下依赖，再次启动项目直接访问接口时，会出现身份验证接口。
```xml
<dependencies>
  ...
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
  ...
</dependencies>
```
>可以通过如下配置关闭Spring Security
```properties
security.basic.enabled=false
```

一般每个系统都有自己的用户体系，所以我们需要定义自己的认证逻辑以及登录界面。
首先，总体配置需要继承`WebSecurityConfigurerAdapter`。
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.csrf.CookieCsrfTokenRepository;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserService userService;
    @Autowired
    private AuthenticationSuccessHandler myAuthenticationSuccessHandler;
    @Autowired
    private AuthenticationFailureHandler myAuthenticationFailureHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
            //登录未授权时调用的接口
            .loginPage(SecurityConstants.DEFAULT_UNAUTHENTICATION_URL)
            //自定义登录时前台调用的接口（仅仅是定义api形式，具体逻辑自己无需实现）
            //需要两个参数{"username": "", "password": ""}，header的'content-type'设置为'application/x-www-form-urlencoded'
            .loginProcessingUrl(SecurityConstants.DEFAULT_LOGIN_PROCESSING_URL)
            //登录成功后TODO
            .successHandler(myAuthenticationSuccessHandler)
            //登录失败后TODO
            .failureHandler(myAuthenticationFailureHandler)
            .and()
            .authorizeRequests()
            //对哪些接口不进行验证，白名单
            .antMatchers(SecurityConstants.DEFAULT_LOGIN_PROCESSING_URL).permitAll()
            .anyRequest()
            .authenticated()
            .and()
            //关闭csrf验证
            .csrf().disable();
    }
    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {
        //登录时密码的加密方式。前台传明文密码，spring security自动加密后与数据库中的加密密码进行比对
        //即数据库中存储的用户密码需要经过BCryptPasswordEncoder加密（也可以通过实现PasswordEncoder接口自定义加密规则）
        auth.userDetailsService(userService).passwordEncoder(new BCryptPasswordEncoder());
    }

}
```

其中，`WebSecurityConfig`涉及的`SecurityConstants`,`myAuthenticationSuccessHandler`,
`myAuthenticationFailureHandler`,代码如下。

```java
public interface SecurityConstants {
    //登录调用接口，逻辑由web security实现，仅需自定义api的uri，如"/user/login"
    String DEFAULT_LOGIN_PROCESSING_URL = "/user/login";

    //当请求需要身份验证时，调用的接口
    String DEFAULT_UNAUTHENTICATION_URL = "/user/authentication/require";
}
```

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component("myAuthenticationSuccessHandler")
public class MyAuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {
    private final Logger LOG = LoggerFactory.getLogger(MyAuthenticationSuccessHandler.class);

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        LOG.info("Login success");
        LOG.info("authentication={}", authentication);
        response.setStatus(HttpStatus.OK.value());
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(authentication));
    }
}
```

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component("myAuthenticationFailureHandler")
public class MyAuthenticationFailureHandler extends SimpleUrlAuthenticationFailureHandler {
    private final Logger LOG = LoggerFactory.getLogger(com.southeastideas.xxx.conf.MyAuthenticationFailureHandler.class);

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        LOG.warn("login failure");
        response.setStatus(HttpStatus.UNAUTHORIZED.value());
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write(objectMapper.writeValueAsString(new BaseResponse(exception.getMessage())));
    }
}
```

```java
import com.southeastideas.xxx.service.UserService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.security.web.DefaultRedirectStrategy;
import org.springframework.security.web.RedirectStrategy;
import org.springframework.security.web.savedrequest.HttpSessionRequestCache;
import org.springframework.security.web.savedrequest.RequestCache;
import org.springframework.security.web.savedrequest.SavedRequest;
import org.springframework.ui.ModelMap;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.*;
import com.southeastideas.xxx.model.User;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@RestController
@RequestMapping("/user")
public class UserController {
    private static Logger LOG = LoggerFactory.getLogger(UserController.class);

    @Autowired
    private UserService userService;

    // 原请求信息的缓存及恢复
    private RequestCache requestCache = new HttpSessionRequestCache();

    // 用于重定向
    private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy();

    /**
     * 当需要身份认证的时候，跳转过来
     * @param request
     * @param response
     * @return
     */
    @RequestMapping("/authentication/require")
    @ResponseStatus(code = HttpStatus.UNAUTHORIZED)
    public String requireAuthentication(HttpServletRequest request, HttpServletResponse response) throws IOException {
        SavedRequest savedRequest = requestCache.getRequest(request, response);
        if (savedRequest != null) {
            String targetUrl = savedRequest.getRedirectUrl();
            LOG.info("引发跳转的请求是:" + targetUrl);
            //举例
            if (StringUtils.endsWithIgnoreCase(targetUrl, ".html")) {
                redirectStrategy.sendRedirect(request, response, "/login.html");
            }
        }
        return "访问的服务需要身份认证，请引导用户到登录页";
    }
}
```

上面是总体配置，若实现自定义认证逻辑，需要实现`org.springframework.security.core.userdetails.UserDetailsService`接口。
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import tk.mybatis.mapper.weekend.Weekend;
import tk.mybatis.mapper.weekend.WeekendCriteria;
import com.southeastideas.xxx.mapper.UserMapper;
import com.southeastideas.xxx.model.User;

@Service
public class UserService implements UserDetailsService {

    private static final Logger LOG = LoggerFactory.getLogger(UserService.class);

    @Autowired
    private UserMapper userMapper;
    /**
    * 从数据库中根据username查找到该用户
    * @param username
    * @return 
    */
    public User findByUserName(String username) {
        Weekend<User> weekend = Weekend.of(User.class);
        WeekendCriteria<User, Object> criteria = weekend.weekendCriteria();
        criteria.andEqualTo(User::getUserName, username);
        return userMapper.selectOneByExample(weekend);
    }


    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LOG.info("username: {}", username);
        User user = findByUserName(username);
        org.springframework.security.core.userdetails.User userDetail =
                new org.springframework.security.core.userdetails.User(
                        username,
                        user.getUserPwd(),//数据库中用户的加密密码，之后将与前台传递并经过security加密的密码进行对比
                        AuthorityUtils.commaSeparatedStringToAuthorityList("admin")
                );
        return userDetail;
    }
}

```

## 参考
[SpringBoot + Spring Security 基本使用及个性化登录配置详解](https://www.jb51.net/article/140429.htm)
