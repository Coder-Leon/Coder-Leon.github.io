---
layout: post
title: 'Shiro技术学习之Shiro的认证过程'
subtitle: 'Shiro认证过程'
date: 2018-05-20
categories: 后端技术
tags: 后端技术
---

最近帮导师做一个项目，需要用到权限控制，又重新学习了一下Shiro，虽然Spring Security也可以做权限控制，但是没接触过具体不太清楚，以后有时间还是接触一下为好吧。知乎上看到一个热评，Shiro与Spring Security的比较，不明觉厉。  

-  Shiro is much easier to use, implement and most importantly understand than Spring.

- The only reason Spring Security is much more well-known is because of the brand name “Spring” which is famous for simplicity, but ironically many find installing Spring Security little difficult
Spring Security, however has a better community support.

- Apache Shiro has an additional module over Spring Security of handling Cryptography

## Shiro的认证流程图
用ProcessOn画的，学生党，图片都存在七牛云的免费的对象存储空间了。。。
![](http://oyzvmt76c.bkt.clouddn.com/shiro1.png)

Shiro的认证流程大致可以分为如图的5个过程，然后呢？
  
  Talk is cheap, show me code!
  
  直接上源码吧。
  
  ``` java
  public class ShiroAuthentication {
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();
    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("Leon", "1001");
    }

    @Test
    public void testAuthentication(){
        // 1、创建SecurityManager
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);

        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("Leon", "1001");

        // 2、主体提交认证
        subject.login(token);

        System.out.println("isAuthenticated: " + subject.isAuthenticated());

        subject.logout();
        System.out.println("isAuthenticated: " + subject.isAuthenticated());
    }
}
  ```
  上面的Demo里其实已经完成了前两个步骤，创建SecurityManager和Subject提交认证。
  接下来我们从源码中查看Subject类的login方法片段。
  
``` java
public void login(AuthenticationToken token) throws AuthenticationException {
        this.clearRunAsIdentitiesInternal();
        Subject subject = this.ç(this, token);

```
  
  可以看到，这里其实本质上就是第三步提到的SecurityManager认证。
  
  继续查看SecurityManager的login方法片段。
  
``` java
public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info;
        try {
            info = this.authenticate(token);
        } catch (AuthenticationException var7) {
            AuthenticationException ae = var7;
```
  
  其中authenticate方法是AuthenticatingSecurityManager类中的，继续查看其源码。
  
``` java
    public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        return this.authenticator.authenticate(token);
    }
```
  
  查看其再次调用的authenticate方法，为Authenticator抽象类中的方法，源码如下：
  
``` java
public final AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
        if (token == null) {
            throw new IllegalArgumentException("Method argument (authentication token) cannot be null.");
        } else {
            log.trace("Authentication attempt received for token [{}]", token);

            AuthenticationInfo info;
            try {
                info = this.doAuthenticate(token);
                if (info == null) {
                    String msg = "No account information found for authentication token [" + token + "] by this " + "Authenticator instance.  Please check that it is configured correctly.";
                    throw new AuthenticationException(msg);
                }
```
  
  到此为止，其实就完成了第四个步骤，Authenticator认证。我们继续查看doAuthenticate方法的源码。
  
``` java
    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        this.assertRealmsConfigured();
        Collection<Realm> realms = this.getRealms();
        return realms.size() == 1 ? this.doSingleRealmAuthentication((Realm)realms.iterator().next(), authenticationToken) : this.doMultiRealmAuthentication(realms, authenticationToken);
    }
```
  
  其doAuthenticate方法，为ModularRealmAuthenticator类中,即前面提到的第五步Realm认证。
  
### 完整的源码可以去我的github上clone,链接在此：
  <https://github.com/Coder-Leon/shiro_demo>

