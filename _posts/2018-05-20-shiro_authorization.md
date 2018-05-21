---
layout: post
title: 'Shiro技术学习之Shiro的授权过程'
subtitle: 'Shiro授权过程'
date: 2018-05-20
categories: 后端技术
tags: 后端技术
---

其实Siro的授权过程和认证过程的步骤大致类似，和前一篇文章一样，先给流程图。

## Shiro的授权流程图

![](http://oyzvmt76c.bkt.clouddn.com/shiro_demo2.png)

Shiro的授权流程大致可以分为如图的5个过程。
  
  Talk is cheap, show me code again!
  
  直接上Demo吧。
  
  ``` java
  public class ShiroAuthorization {
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();
    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("Leon", "1001","admin","user");
    }

    @Test
    public void testAuthentication(){
        // 1、创建SecurityManager
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccountRealm);

        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("Leon", "1001");

        // 2、主体授权
        subject.login(token);

        System.out.println("isAuthenticated: " + subject.isAuthenticated());
        subject.checkRole("admin");
        subject.checkRoles("admin","user");
        System.out.println("hasRole: " + subject.hasRole("admin"));
        List<String> list = new ArrayList<String>();
        list.add("admin");
        list.add("user1");
        System.out.println("hasRoles: " + subject.hasRoles(list)[1]);
    }
}

  ```
  上面的Demo里其实已经完成了前两个步骤，创建SecurityManager和Subject提交认证。
  接下来我们从源码中查看DelegatingSubject类的checkRole方法。
  
``` java
 public void checkRole(String role) throws AuthorizationException {
        this.assertAuthzCheckPossible();
        this.securityManager.checkRole(this.getPrincipals(), role);
    }

```
  
  可以看到，这里其实就是第三步提到的SecurityManager授权。
  
  继续查看SecurityManager的checkRole方法。
  
``` java
    public void checkRole(PrincipalCollection principals, String role) throws AuthorizationException {
        this.authorizer.checkRole(principals, role);
    }
    
```
  
  其中方法是AuthorizingSecurityManager类中的，继续查看其源码，调用的是ModularRealmAuthorizer类中的checkRole方法。其实就是第四步提到的Authorizer授权。
  
``` java
   
    public void checkRole(PrincipalCollection principals, String role) throws AuthorizationException {
        this.assertRealmsConfigured();
        if (!this.hasRole(principals, role)) {
            throw new UnauthorizedException("Subject does not have role [" + role + "]");
        }
    }
```
  
  可以看到判断里有hasRole方法，我们继续查看其源码，在同一个类中：
  
``` java
public boolean hasRole(PrincipalCollection principals, String roleIdentifier) {
        this.assertRealmsConfigured();
        Iterator var3 = this.getRealms().iterator();

        Realm realm;
        do {
            if (!var3.hasNext()) {
                return false;
            }

            realm = (Realm)var3.next();
        } while(!(realm instanceof Authorizer) || !((Authorizer)realm).hasRole(principals, roleIdentifier));

        return true;
    }
```
  
  可以看到，hasRole方法使用迭代器来获取Realm，即第五步通过Realm获取角色的权限数据。
### 完整的源码可以去我的github上clone,链接在此：
  <https://github.com/Coder-Leon/shiro_demo>

