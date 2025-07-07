---
title: Shiro官方文档笔记
date: 2021-06-26T00:00:00.000Z
updated: '2025-07-07 16:06:23'
categories: Java
tags:
  - Shiro
  - Java
---
# 1. 核心架构


## 1. 核心流程


![](/images/0e818ccd09cc449d1741b7182cf964c9.png)



### 1.  Subject


指需要认证的用户信息实体，subject需要通过securityManager指定Realm来查询是否存在改用户信息和给用户进行授权的操作



### 2. SecurityManager


shiro体系的核心。协调内部安全组件。如：Realm等。



### 3. Realm


通过查询特定的数据源：数据库、LDAP等。来对Subject进行认证和授权操作。



## 2. 核心架构


[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-FMppfyoU-1624680171088)([https://i.loli.net/2021/06/25/tADiGzO3lZpfrYW.png](https://i.loli.net/2021/06/25/tADiGzO3lZpfrYW.png))]



### 1. Authenticator 认证器


对Subject 中的信息进行认证。通过将Subject的信息传入Realm查询指定数据源来进行认证判断。  
当用户尝试登录时，执行认证器。



### 2. AuthenticationStrategy 认证策略器


如果配置了多个多个Realm，则通过认证策略器来决定认证成功和认证失败的情况。



### 3. Authorizer 授权器


用于对认证成功后的Subject 进行授权操作。通过Realm查询到用户的对应信息给用户授予对应的权力



### 4. SessionManager 会话管理器


Shiro 自带的会话管理器。能够创建和管理用户的Session生命周期。提供一种可靠的会话体验。默认情况下，Shiro使用现有的会话机制机制如Servlet容器。



### 5. SessionDao 会话持久器


将Session Manager中的Session对象进行CRUD操作将其存储起来。



### 6. CacheManager 缓存管理器


用于缓存Shiro中的Realm中的数据



### 7. Crypto 加密


Shiro中一个简单易用的加密包。针对于Java的加密机制，Shiro的加密Api更加简单易用。



### 8. Realm 领域


用于查询数据数据源的信息。通过将查询到用户信息返回给认证器和授权器。进行认证和授权。



Security Manager 中的默认实现的功能



+ Authentication 认证
+ Authorization 授权
+ Session Management  会话管理
+ Cache Management 缓存管理
+ [Realm](https://shiro.apache.org/realm.html) coordination  领域协调
+ Event propagation 事件传播
+ “Remember Me” Services  记住我服务
+ Subject creation Subject对象的创建
+ Logout  注销  

# 


# 2. 身份认证


Subject的构成



1. principals



通俗点来说就是用户的用户名，可以用来表示用户身份的证明。当然并不是唯一的。



2. credentials



可以用来证明该用户身份的证据。通常指的是密码或证书等。



## 1. 认证Subjects


1. 获取Subject中的principals和Credentials
2. 提交principals和credentials来进行认证操作
3. 认证成功，则允许访问。反之阻止访问和进行新的认证



实例演示：



1. **获取principals和credentials**



```html
//Example using most common scenario of username/password pair:
UsernamePasswordToken token = new UsernamePasswordToken(username, password);

//"Remember Me" built-in: 
token.setRememberMe(true);
```



UsernamePasswordToken 是用来进行Shiro进行认证的接口对象。Shiro中的认证需要认证的信息都要封装成AuthenticationToken接口的对象。UsernamePassowrdToken是其接口的实现。  
**



2. ** 提交priincipals和credentials**



```html
Subject currentUser = SecurityUtils.getSubject();

currentUser.login(token);
```



** **subject对象通过login方法将认证信息进行认证提交



3. **认证成功或失败**



**  
当认证成功后即可成功访问，如果失败Shiro会抛出异常。通过异常可以知道认证失败的原因。



```html
try {
    currentUser.login(token);
} catch ( UnknownAccountException uae ) { ...
} catch ( IncorrectCredentialsException ice ) { ...
} catch ( LockedAccountException lae ) { ...
} catch ( ExcessiveAttemptsException eae ) { ...
} ... catch your own ...
} catch ( AuthenticationException ae ) {
    //unexpected error?
}

//No problems, continue on as expected...
```



## 2. Remember Me 和 Authenticated


1. **Remember me**



+ 使用的是先前会话的Subject
+ 并且Subject是非匿名的
+ 通过调用isRemembered方法返回的true



2. **Authenticated**



+ 当前认证成功的subject
+ 调用isAuthenticated返回true



**记住和已认证两种状态不能同时发生在同一subject上。**



### 一个示例


以下是一个相当常见的场景，有助于说明为什么记住和已认证之间的区别很重要。  
假设您使用的是[Amazon.com](https://www.amazon.com/)。您已成功登录，并已在购物车中添加了几本书。但是您必须参加会议，但忘记注销。会议结束时，该回家了，您离开办公室了。  
第二天上班时，您发现自己还没有完成购买，因此回到 amazon.com。这次，亚马逊“记住”您的身份，以名字向您打招呼，并仍然为您提供一些个性化的书本推荐。对于亚马逊，`subject.isRemembered()`将返回`true`。  
但是，如果您尝试访问帐户以更新信用卡信息以购买图书，会发生什么情况？当亚马逊“记住”您(`isRemembered()` == `true`)时，它不能保证您实际上就是您(例如，某个同事正在使用您的计算机)。  
因此，在您执行敏感操作(如更新信用卡信息)之前，亚马逊会强迫您登录，以便他们保证您的身份。登录后，您的身份已通过验证，并且到亚马逊的`isAuthenticated()`现在为`true`。  
这种情况在许多类型的应用程序中经常发生，因此该功能是 Shiro 内置的，因此您可以将其用于自己的应用程序。现在，是否使用`isRemembered()`或`isAuthenticated()`来定制视图和工作流已由您决定，但是 Shiro 将保留此基本状态，以备不时之需。



## 3. 注销


当Subject调用logout方法后任何现有的Session都将失效并且任何身份都会被取消关联。



## 4. 认证流程


[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-WFk3b6E6-1624680171090)([https://i.loli.net/2021/06/25/qGAUkvJCdIhjpFw.png](https://i.loli.net/2021/06/25/qGAUkvJCdIhjpFw.png))]



**step1：**  
调用login方法提交认证信息



**step2：**  
调用security Mangager.login（token）来通过securityManager开始真正的身份认证



**step3:**  
securityManager 接收token并调用authenticator.authenticate(token)将认证信息提供给认证器进行认证。  
**step4：**



shiro默认使用的MoularRealmAuthenticator实例通过Realm进行认证。如果配置了多个Realm则使用认证策略其，来进行多次的Realm认证。来决定认证成功和失败的条件。  
**只有一个Realm的话不需要认证策略器**  
**  
**step5:**  
咨询已配置的Realm，通过使用supports方法判断其是否支持验证其身份信息，如果返回true。则调用Realm的getAuthenticationInfo方法获取认证信息。



**  
**



# 3. 授权


## 1. 授权要素


+ 权限：在程序中，表示用户可以执行什么动作或行为
+ 角色：是一个或多个权限的集合实体。
+ 用户：是一个或多个角色的集合实体。



## 2.授权Subjects


### 1. 编程授权


#### 1. 基于角色的授权


通过用户是否含有特定角色，来执行角色检查



```html
Subject currentUser = SecurityUtils.getSubject();

if (currentUser.hasRole("administrator")) {
    //show the admin button 
} else {
    //don't show the button?  Grey it out? 
}


Subject currentUser = SecurityUtils.getSubject();

//guarantee that the current user is a bank teller and 
//therefore allowed to open the account: 
currentUser.checkRole("bankTeller");
openBankAccount();
```



![](/images/d29ac67d38848b1847f0cf17371e4081.png)



![](/images/1dcb526859b11e9df87f0b0808c75575.png)



#### 2. 基于权限的授权


与基于角色类似![](/images/c964be174d332a202236fef4240bf80a.png)



![](/images/751bcc949be509350941f5d76c07cef0.png)



[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-dUuiGDuP-1624680171097)([https://i.loli.net/2021/06/25/ZKQJUnhEvT6a5ky.png](https://i.loli.net/2021/06/25/ZKQJUnhEvT6a5ky.png))]



### 2. 基于注解授权


**前提：**  
需要在程序中开启Aop支持



#### 1. RequiresAuthentication注解


调用该方法时要求用户在当前会话期间认证过



```html
@RequiresAuthentication
public void updateAccount(Account userAccount) {
    //this method will only be invoked by a
    //Subject that is guaranteed authenticated
    ...
}

//等价于
public void updateAccount(Account userAccount) {
    if (!SecurityUtils.getSubject().isAuthenticated()) {
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed authenticated here
    ...
}
```



#### 2. RequiresGuest注解


不需要身份认证即可访问的方法



```html
@RequiresGuest
public void signUp(User newUser) {
    //this method will only be invoked by a
    //Subject that is unknown/anonymous
    ...
}

//等价于
public void signUp(User newUser) {
    Subject currentUser = SecurityUtils.getSubject();
    PrincipalCollection principals = currentUser.getPrincipals();
    if (principals != null && !principals.isEmpty()) {
        //known identity - not a guest:
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to be a 'guest' here
    ...
}
```



#### 3. RequiresPermissions注解


要求用户必须有该权限才能调用



```html
@RequiresPermissions("account:create")
public void createAccount(Account account) {
    //this method will only be invoked by a Subject
    //that is permitted to create an account
    ...
}

public void createAccount(Account account) {
    Subject currentUser = SecurityUtils.getSubject();
    if (!subject.isPermitted("account:create")) {
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to be permitted here
    ...
}
```



#### 4. RequiresRoles注解


```html
@RequiresRoles("administrator")
public void deleteUser(User user) {
    //this method will only be invoked by an administrator
    ...
}

public void deleteUser(User user) {
    Subject currentUser = SecurityUtils.getSubject();
    if (!subject.hasRole("administrator")) {
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to be an 'administrator' here
    ...
}
```



#### 5. RequiresUser注解


要求用户在当前会话期间进行了会话认证或者是是哦那个Remember Me服务被记住



```html
@RequiresUser
public void updateAccount(Account account) {
    //this method will only be invoked by a 'user'
    //i.e. a Subject with a known identity
    ...
}

public void updateAccount(Account account) {
    Subject currentUser = SecurityUtils.getSubject();
    PrincipalCollection principals = currentUser.getPrincipals();
    if (principals == null || principals.isEmpty()) {
        //no identity - they're anonymous, not allowed:
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to have a known identity here
    ...
}
```



### 3. 基于jsp标签授权


### 对应目录jsp标签库


## 3. 授权流程


![](/images/1a1c1fb6386c49d35b648315ee194edc.png)



**step1:**  
subject通过调用一些角色和权限的判断方法



**step2:**  
securityManager调用上面相同的角色和权限的方法



**step3:**  
securityManager调用authorizer默认实例ModularRealmAuthorizer实列



**step4:**  
使用Realm获取用户对应的权限信息



# 4. 领域


## 1.领域认证


### 1. 领域认证流程


如果调用Realm方法支持提交的AuthenticationToken，则调用Realm的getAuthenticationInfo方法。然后通过该方法可以获取对应token在领域数据源的数据。方法执行顺序为



1. 识别token中的principal信息
2. 根据principal查找对应数据源中对应的数据
3. 确保token中的credentials与对应数据库中的数据匹配
4. 匹配返回AuthenticationInfo实例，反之则抛出AuthenticationException



**领域认证可以通过实现AuthorizingRealm抽象类来实现领域认证。**



### 2. 凭证匹配


在领域认证流程中领域必须验证Subject提交的credentials和数据库中存储的凭据是否匹配。如果匹配在身份验证成功。  
凭证匹配通过AuthenticatingRealm和子类CredentialsMatcher来实现凭据的比较



Shiro提供了CredentialsMatcher的实现。例如SimpleCredentialsMatcher和HashedCredentialsMatcher的实现类。也可以自己自定义证书匹配的规则。



#### 1. SimpleCredentialsMatcher


Shiro中所有的Realm默认使用的都是SimpleCredentialsMatcher。SimpleCredentialsMatcher对存储的凭据和AuthenticationToken中进行直接相等性检查。



#### 2. HashedCredentialsMatcher


当用户需要存储一些比较密码等一些重要的凭证时，不是直接存入数据库中而是先hash一次然后再存入数据库中。  
这样用户存储的凭证更加安全也没有人知道原始值。



## 2. 领域授权


SeurityManager将权限和角色的检查任务交给授权器。默认的授权器为ModularRealmAuthorizer。



### 1. 基于角色的授权流程


1. Subject委托SecurityManager以确定是否分配给定的角色
2. SecurityManager委托授权器
3. 授权器逐个使用所有领域，直到找到Subject指定角色。如果都没有则返回false
4. 如果在领域中成功找到则掉哦那个AuthenticationInfo.getRoles返回给定角色。并授予访问权限。



#### 2. 基于权限的授权流程


1. Subject委托SecurityManager授予或拒绝授予权限
2. SecurityManager委托给授权器
3. 授权器查找所有领域，如果查找到对应全新啊，则授予权限。反之则拒绝授予权限



# 5. 会话管理


**特性：**



1. 轻松自定义会话存储
2. 支持web
3. 可用于sso



## 1. 使用Session


Session的获取



```html
Subject currentUser = SecurityUtils.getSubject();

Session session = currentUser.getSession();
session.setAttribute( "someKey", someValue);
```



![](/images/5af956e2b2121c4421b7d33543c38eef.png)



## 2. 会话管理器


Shiro中会话管理器可以对应用中的Session进行创建、删除、验证等操作。  
Shiro中提供了默认的会话管理器DefaultSessionManager。提供了Session的验证和清楚等



## 3. 会话存储


当每次创建、更新或删除长时间未使用的Session时，未来避免Session存储空间耗尽。SessionManager将Session的创建、读取、更新、删除操作委派给  
组件SesssionDao。  
SessionDao的特点将所有你只需要实现这个接口姐可以实现任何数据的储存在任何地方。这意味着你可以将Session数据存储在内存中、关系数据库、NoSql数据库中。



Shiro中默认的Session存储在内存中。可以通过配置EHCacheSessionDao或者是实现自定以的SessionDao。



### 1. EHCache SessionDao


默认情况下未开启。如需开启需要在会话管理器中开启EHCache支持。EHCache SessionDao会将Sesssion存储在会话中。如果内存不够了，则会其他的Session存储在磁盘中。  
除了存储Session外，还以缓存身份验证和授权数据



### 2.EHCacheSessionDao 配置


默认情况下Shiro中的EhCacheManager使用的Shiro自带的ehcache.xml来设置缓存配置。



# 6. JSP标签库


## 1. 标签库配置


```html
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
```



## 2. 访客标签


不需要认证就可以生效的标签



```html
<shiro:guest>
    Hi there!  Please <a href="login.jsp">Login</a> or <a href="signup.jsp">Signup</a> today!
</shiro:guest>
```



## 3. 用户标签


当用户被认证过或者说是被记住了的话才生效的标签



```html
<shiro:user>
    Welcome back John!  Not John? Click <a href="login.jsp">here<a> to login.
</shiro:user>
```



## 3. 认证标签


用户只有被认证过才生效的标签



```html
<shiro:authenticated>
    <a href="updateAccount.jsp">Update your contact information</a>.
</shiro:authenticated>
```



## 4. 认证失败标签


用户认证失败的生效标签



```html
<shiro:notAuthenticated>
    Please <a href="login.jsp">login</a> in order to update your credit card information.
</shiro:notAuthenticated>
```



## 5.principal标签


获取用户的priincipal属性  
shiro:principa标签可以简化为principal



principal等价于subject.getPrincipal()方法



```html
Hello, <shiro:principal/>, how are you today?
//等价于
Hello, <%= SecurityUtils.getSubject().getPrincipal().toString() %>, how are you today?

  User ID: <principal type="java.lang.Integer"/>

  User ID: <%= SecurityUtils.getSubject().getPrincipals().oneByType(Integer.class).toString() %>

Hello, <shiro:principal property="firstName"/>, how are you today?
  Hello, <shiro:principal type="com.foo.User" property="firstName"/>, how are you today?
  Hello, <%= SecurityUtils.getSubject().getPrincipal().getFirstName().toString() %>, how are you today?
```



## 6. hasRole标签


当用户拥有某个权限才会生效的标签



```html
<shiro:hasRole name="administrator">
    <a href="admin.jsp">Administer the system</a>
</shiro:hasRole>
```



## 7.misssingRole标签


与hasRole标签作用相反



```html
<shiro:lacksRole name="administrator">
    Sorry, you are not allowed to administer the system.
</shiro:lacksRole>
```



## 8.hasAnyRole标签


用户只要含有任意一个角色就生效的标签



```html
<shiro:hasAnyRoles name="developer, project manager, administrator">
    You are either a developer, project manager, or administrator.
</shiro:hasAnyRoles>
```



## 9.hasPermission标签


用户拥有某项权限才生效的标签



```html
<shiro:hasPermission name="user:create">
    <a href="createUser.jsp">Create a new User</a>
</shiro:hasPermission>
```



## 10. missingsPermisssion标签


与hasPermission作用相反



```html
<shiro:lacksPermission name="user:delete">
    Sorry, you are not allowed to delete user accounts.
</shiro:lacksPermission>
```



