# [-[Session]-]

[-[Session]-]是用户与[-[Web]-]应用维持连接的一种手段。当用户操作应用期间，无论什么时候都有唯一的[-[Session]-]保存用户的相关信息。对于[-[Shiro]-]来说，[-[Session]-]部分是其的一大亮点，因为[-[Shiro]-]实现并且维护了自己的[-[Session]-]环境，不需要兼容其他的容器。换句话就是说[-[Shiro]-]实现了自己的[-[Session]-]并且维护了这个[-[Session]-]的生命周期。

要了解[-[Session]-]的生命周期就需要先了解[-[Subject]-]的创建。

![Subject创建](/img/ShiroSubjectCreate.png)

创建[-[Subject]-]有两个方面，一个是请求时经过[-[AbstractShiroFilter]-]时调用的`CreateSubject`方法，另一个是调用`Subject.login`方法进行登录操作。虽然这有两种方式都能创建`Subject`对象，但是还是所不同的。当你访问一个[-[Web]-]应用，首先请求会调用`AbstractShiroFilter.CreateSubject`这个方法，等登录时才会手动调用`Subject.login`方法，当登录之后本次会话的所有请求的`Subject`的创建都由`AbstractShiroFilter`来负责。

![Session创建](/img/ShiroSessionCreate.png)

从上面的图可以我们知道在创建[-[Subject]-]需要获取[-[Session]-]以及[-[Principals]-]。当在未鉴权时发起请求，此时[-[Session]-]和[-[Pricipals]-]的相关信息都为空所以创建的[-[Subject]-]就不存在任何有用的东西，我们可以称此时的[-[Subject]-]是**匿名Subject**。

当我们通过登录这个操作在后台调用`Subject.login`方法创建[-[Subject]-]，此方法会调用`Realm.doGetAuthenticationInfo`来获取认证信息，并且将该信息放入到上下文中用来创建`Subejct`;

<details>
<summary>源码</summary>

````java
protected Subject createSubject(AuthenticationToken token, AuthenticationInfo info, Subject existing) {
    SubjectContext context = createSubjectContext();
    context.setAuthenticated(true);
    context.setAuthenticationToken(token);
    context.setAuthenticationInfo(info);
    context.setSecurityManager(this);
    if (existing != null) {
        context.setSubject(existing);
    }
    return createSubject(context);
}
````
</details>

此时由于创建的[-[Subject]-]对象带有认证信息，所以在`SaveToSesion`方法中就满足了当出现该此请求获取不到[-[Session]-]并且[-[Subject]-]带有认证信息的条件，此时就需要通过`SessionFactory.createSession`方法来创建[-[Session]-]。

鉴权之后，在本次会话中用户的每一次请求可以通过[-[SessionID]-]来获取[-[Session]-]并创建对应的[-[Subject]-]，此时[-[Subject]-]对象中包含用户的鉴权信息。

!> 需要注意的是，上面的解释是对于Web应用底层的[-[SessionManager]-]为`DefaultWebSessionManager`的流程，此时[-[Shiro]-]自己维护会话，直接废弃了[-[Servlet]-]容器的会话管理。如果相关交由[-[Servlet]-]容器管理，则底层`SessionManager`具体实现应为`ServletContainerSessionManager`，并且之后相关流程也不同。

具体情况参考[Session Problem](https://programmer.help/blogs/session-problem-after-spring-mvc-integrates-shiro.html)这篇文章。

总结一下，[-[Session]-]的创建最终调用的是`SessionFactory.createSession`方法，所有我们只需要继承重写相关方法就可以自定义[-[Session]-]。

## [-[Session]-]维护

[-[Session]-]创建的问题已经解决了，那么来看看[-[Shiro]-]是如何维护[-[Session]-]的生命周期的。[-[Session]-]生命周期中最主要有两方面：一、请求时更新会话的最新访问时间。二、检测并且删除过期的[-[Session]-]。

对于会话的更新，就又不得不提到`AbstractShiroFilter.doFitlerInternal`中调用的`updateSessionLastAccessTime`方法。流程如下

![Session生命周期](/img/ShiroSessionLifeCycle.png)

当每次请求到来之后，[-[Shiro]-]会调用`SessionManager`的`touch`方法。该方法包含了对[-[Session]-]生命周期维护的具体实现。首先，[-[Shiro]-]会通过`lookupRequiredSession`方法来开启定时任务，该任务来剔除应用中过期的[-[Session]-]。之后通过`validate`方法来校验当前访问的会话状态，对于过期、停止状态的[-[Session]-]的具体实现交由对应的[-[SessionManager]-]处理。之后回到`AbstractNativeSessionManager`中调用[-[Session]-]的`touch`方法，并且继续`onChange`逻辑。

<details>
<summary>ExecutorServiceSessionValidationScheduler（定时任务的实现）</summary>

````java
public void enableSessionValidation() {
    if (this.interval > 0l) {
        this.service = Executors.newSingleThreadScheduledExecutor(new ThreadFactory() {  
            private final AtomicInteger count = new AtomicInteger(1);

            public Thread newThread(Runnable r) {  
                Thread thread = new Thread(r);  
                thread.setDaemon(true);  
                thread.setName(threadNamePrefix + count.getAndIncrement());
                return thread;  
            }  
        });                  
        this.service.scheduleAtFixedRate(this, interval, interval, TimeUnit.MILLISECONDS);
    }
    this.enabled = true;
}

public void run() {
    if (log.isDebugEnabled()) {
        log.debug("Executing session validation...");
    }
    Thread.currentThread().setUncaughtExceptionHandler((t, e) -> {
        log.error("Error while validating the session, the thread will be stopped and session validation disabled", e);
        this.disableSessionValidation();
    });
    long startTime = System.currentTimeMillis();
    try {
        this.sessionManager.validateSessions();
    } catch (RuntimeException e) {
        log.error("Error while validating the session", e);
        //we don't stop the thread
    }
    long stopTime = System.currentTimeMillis();
    if (log.isDebugEnabled()) {
        log.debug("Session validation completed successfully in " + (stopTime - startTime) + " milliseconds.");
    }
}
````
</details>