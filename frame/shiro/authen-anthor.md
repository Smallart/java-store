# 认证与授权

## 认证（[-[Authentication]-]）
对于某些资源需要对当前用户登录之后才能访问，[-[Shiro]-]通过过滤器链在真正调用相对应的接口之前，对当前发起请求的[-[Subject]-]判断是否鉴权。

首先通过`ShiroFilterFactoryBean`的`getFilterChainDefinitionMap`方法进行[-[URL]-]和过滤器进行映射，当请求到来时，会先到[-[AbstractShiro]-]的`doFilterInternal`方法调用自定义的过滤链。一个过滤器的逻辑如下：

![hierarchy](/img/authentication-hierarchy.png)

[-[AcessControlFilter]-]最终定义了鉴权和授权的处理接口，子类通过该接口去实现自己的逻辑。其中`isAccessAllowed`表示是否允许访问，如果不允许则走`onAccessDenied`这个方法表示不允许之后的处理。例如[-[Shiro]-]实现的[-[UserFilter]-]过滤器：

```java
protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
    if (isLoginRequest(request, response)) {
        return true;
    } else {
        Subject subject = getSubject(request, response);
        // If principal is not null, then the user is known and should be allowed access.
        return subject.getPrincipal() != null;
    }
}
protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
        saveRequestAndRedirectToLogin(request, response);
        return false;
    }
```

通过上面的[-[isAceesAllowed]-]方法使用[-[Subject]-]对象来判断当前访问者是否登录。也就是说登录前后[-[Subject]-]的状态发生了改变。

在使用[-[Shiro]-]进行登录鉴权，请求经过[-[AbstractShiroFilter]-]时调用`createSubject`方法创建[-[Subject]-]，而在[-[Controller]-]层的方法中调用[-[Subejct.login]-]，并且它也调用了`createSubject`方法。那么[-[Shiro]-]是如何验证登录密码？又是如何对[-[Subject]-]进行状态改变的？

[-[Shiro.login]-]底层调用的是`SecurityManager.login`方法，最后将相关信息交给了`ModularRealmAuthenticator.authenticate`方法最后调用我们注册在[-[SecurityManager]-]的[-[Realm]-]中的[-[getAuthenticationInfo]-]方法，最终在该方法上进行数据上的验证（这里一般对持久层进行操作）。完成验证之后调用了`createSubject`这是为什么？
```java
public Subject createSubject(SubjectContext subjectContext) {
        //包装subjectContext
        SubjectContext context = copy(subjectContext);
        //确保subjectContext包含SecurityManager
        context = ensureSecurityManager(context);
        //为subjectContext设置sesison，对于web应用来说，获取的是Http中的Sesion
        context = resolveSession(context);
        //这里从好几个地方中获取，不过对于web来说，最后还是从session中获取身份
        context = resolvePrincipals(context);
        //将上面的信息创建一个WebDelegatingSubject，调用subjectFactory中的crateSubject方法
        Subject subject = doCreateSubject(context);
        //调用subjectDao的save方法，这里将subject中身份和鉴权信息保存到session中
        save(subject);
        return subject;
    }
```
<!-- panels:start -->
<!-- div:left-panel -->
```java
protected void saveToSession(Subject subject) {
    mergePrincipals(subject);
    mergeAuthenticationState(subject);
}

public void mergePrincipals(Subject subject){
    //其整个目的就在于这句
    ……
    session.setAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY, currentPrincipals);
    ……
}

public void mergeAuthenticationState(Subject subject){
    //其整个目的在于这句
    ……
    session.setAttribute(DefaultSubjectContext.AUTHENTICATED_SESSION_KEY, Boolean.TRUE);
    ……
}
```
<!-- div:right-panel -->
虽然[-[Http]-]协议是无状态的，但是通过[-[Session]-]我们可以将鉴权的状态保存起来，这使得用户登录之后的每此请求之后都将[-[Session]-]中的相关信息包装成为[-[Subject]-]，这样就能记得用户的登录状态。
<!-- panels:end -->

## 授权（[-[Authority]-]）

所谓授权就是根据用户角色，授予该名用户能够访问的资源。不过这些资源在数据库中为文字描述。在[-[Shiro]-]中提供了两种进行权限控制的方式。
```java
//
subject.checkPermission();
//注解
@RequiresPermissions("sys:demo:view")
```
其中常用的还是注解方式，不过其注解解析器中调用的依然是`subject.chekcPermission`的方法。追踪方法可知最后会来到`AuthorizingRealm`才具体实现了该方法。该用户所包含的权限信息会包装成[-[AuthroziationInfo]-]，[-[AuthroziationInfo]-]会先从[-[Cache]-]缓存中获取，当缓存中没有则从[-[Realm]-]的`doGetAuthorizationInfo`方法中从持久层获取，最后比较是否包含相应权限。对于用户包含角色的鉴定类似。