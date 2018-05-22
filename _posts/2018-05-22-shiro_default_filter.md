---
layout: post
title: 'Shiro技术学习之Shiro的默认拦截器'
subtitle: 'Shiro默认拦截器'
date: 2018-05-22
categories: 后端技术
tags: 后端技术
---

Shiro内置了很多默认的拦截器，比如身份验证、授权等相关的。默认拦截器可以参考org.apache.shiro.web.filter.mgt.DefaultFilter中的枚举拦截器：  
``` java
public enum DefaultFilter {  
    anon(AnonymousFilter.class),  
    authc(FormAuthenticationFilter.class),  
    authcBasic(BasicHttpAuthenticationFilter.class),  
    logout(LogoutFilter.class),  
    noSessionCreation(NoSessionCreationFilter.class),  
    perms(PermissionsAuthorizationFilter.class),  
    port(PortFilter.class),  
    rest(HttpMethodPermissionFilter.class),  
    roles(RolesAuthorizationFilter.class),  
    ssl(SslFilter.class),  
    user(UserFilter.class);  
}   
```

<style>
  table{
    width: 100%;
    font-size:90%;
  }
  table th,td{
    text-align:center;
  }
</style>


| 默认拦截器名 | 拦截器类 | 说明（括号里的表示默认值）  |
| --- | --- | --- |
| 身份验证相关 |   |   |
| authc | org.apache.shiro.<br>web.filter.authc.<br>FormAuthenticationFilter | 基于表单的拦截器；如“/**=authc”，如果没有登录会跳到相应的登录页面登录；主要属性：usernameParam：表单提交的用户名参数名（ username）；  passwordParam：表单提交的密码参数名（password）； rememberMeParam：表单提交的密码参数名（rememberMe）；  loginUrl：登录页面地址（/login.jsp）；successUrl：登录成功后的默认重定向地址； failureKeyAttribute：登录失败后错误信息存储key（shiroLoginFailure）； |
| authcBasic | org.apache.shiro.<br>web.filter.authc.<br>BasicHttpAuthenticationFilter | Basic HTTP身份验证拦截器，主要属性： applicationName：弹出登录框显示的信息（application）； |
| logout | org.apache.shiro.<br>web.filter.authc.<br>LogoutFilter | 退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/）;示例“/logout=logout” |
| user | org.apache.shiro.<br>web.filter.authc.<br>UserFilter| 用户拦截器，用户已经身份验证/记住我登录的都可；示例“/**=user” |
| anon | org.apache.shiro.<br>web.filter.authc.<br>AnonymousFilter | 匿名拦截器，即不需要登录即可访问；一般用于静态资源过滤；示例“/static/**=anon” |
| 授权相关 |  |  |
| roles | org.apache.shiro.<br>web.filter.authz.<br>RolesAuthorizationFilter | 角色授权拦截器，验证用户是否拥有所有角色；主要属性： loginUrl：登录页面地址（/login.jsp）；unauthorizedUrl：未授权后重定向的地址；示例“/admin/**=roles[admin]” |
| perms | org.apache.shiro.<br>web.filter.authz.<br>PermissionsAuthorizationFilter | 权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例“/user/**=perms["user:create"]” |
| port | org.apache.shiro.<br>web.filter.authz.<br>PortFilter | 端口拦截器，主要属性：port（80）：可以通过的端口；示例“/test= port[80]”，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样 |
| rest | org.apache.shiro.<br>web.filter.authz.<br>HttpMethodPermissionFilter | rest风格拦截器，自动根据请求方法构建权限字符串（GET=read, POST=create,PUT=update,DELETE=delete,HEAD=read,TRACE=read,OPTIONS=read, MKCOL=create）构建权限字符串；示例“/users=rest[user]”，会自动拼出“user:read,user:create,user:update,user:delete”权限字符串进行权限匹配（所有都得匹配，isPermittedAll）； |
| ssl | org.apache.shiro.<br>web.filter.authz.<br>SslFilter | SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口（443）；其他和port拦截器一样； |


这些默认的拦截器会自动注册，可以直接在ini配置文件中通过“拦截器名.属性”设置其属性：
``` java
    perms.unauthorizedUrl=/unauthorized  
```
另外如果某个拦截器不想使用了可以直接通过如下配置直接禁用：
``` java
    perms.enabled=false  
```




