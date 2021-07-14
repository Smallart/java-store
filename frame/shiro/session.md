# [-[Session]-]

[-[Session]-]是用户与[-[Web]-]应用维持连接的一种手段。当用户操作应用期间，无论什么时候都有唯一的[-[Session]-]保存用户的相关信息。对于[-[Shiro]-]来说，[-[Session]-]部分是其的一大亮点，因为[-[Shiro]-]实现并且维护了自己的[-[Session]-]环境，不需要兼容其他的容器。

虽然[-[Shiro]-]提供了基础的[-[Session]-]，但是如果不满足我们的项目需求，我们就需要通过继承[-[Shiro]-]的相关接口来实现自己的[-[Session]-]管理。对于[-[Session]-]管理，我们着重需要关注的是创建[-[Session]-]的时机、如何创建[-[Session]-]、如何刷新[-[Session]-]以及对于过期[-[Session]-]的处理。

[-[Shiro]-]中所有有关[-[Session]-]的相关操作都是交给了[-[SessionManager]-]这个组件去完成的，关于[-[SessionMananger]-]的具体继承如下

![SessionManager继承](/img/sessionmananger-family.png)

## 创建[-[Session]-]

当客户端与应用第一次建立连接时，应用便会通过调用`subject.getSession()`来创建当前会话。其中调用地方有两处：

* 过滤器
````java
protected void saveRequestAndRedirectToLogin(ServletRequest request, ServletResponse response) throws IOException {
    saveRequest(request);
    redirectToLogin(request, response);
}
public static void saveRequest(ServletRequest request) {
    Subject subject = SecurityUtils.getSubject();
    Session session = subject.getSession();
    HttpServletRequest httpRequest = toHttp(request);
    SavedRequest savedRequest = new SavedRequest(httpRequest);
    session.setAttribute(SAVED_REQUEST_KEY, savedRequest);
}
````
* 创建[-[Subject]-]时，当前[-[Session]-]为null并且认证信息不为空时调用
```java
 protected void mergePrincipals(Subject subject) {
        PrincipalCollection currentPrincipals = null;
        ……
        if (currentPrincipals == null || currentPrincipals.isEmpty()) {
            currentPrincipals = subject.getPrincipals();
        }

        Session session = subject.getSession(false);

        if (session == null) {
            if (!isEmpty(currentPrincipals)) {
                session = subject.getSession();
                session.setAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY, currentPrincipals);
            }
        }
        ……
    }
```

跟进`subject.getSession()`方法，可以看到其底层调用的是`sessionManager.start`的方法。
```java
public Session getSession(boolean create) {
    ……
    if (this.session == null && create) {
        ……   
        SessionContext sessionContext = createSessionContext();
        Session session = this.securityManager.start(sessionContext);
        this.session = decorate(session);
    }
    return this.session;
}
// SessionSecurityManager
public Session start(SessionContext context) throws AuthorizationException {
    return this.sessionManager.start(context);
}
```

[-[Session]-]实现的相关步骤由其子类`AbstractNativeSessionManager`规定下来：

```java
public Session start(SessionContext context) {
    Session session = createSession(context);
    applyGlobalSessionTimeout(session);
    onStart(session, context);
    notifyStart(session);
    return createExposedSession(session, context);
}
````
其中`createSession`的具体流程如下：

![创建Session](/img/progress-createSession.png)

## 刷新[-[Session]-]

[-[Shiro]-]中的[-[Session]-]定义了如下的字段：

```java
//SessionId
private transient Serializable id;
//session创建时间
private transient Date startTimestamp;
//session停止时间
private transient Date stopTimestamp;
//最后一次访问时间
private transient Date lastAccessTime;
//session的有效时长
private transient long timeout;
// sesison是否过期
private transient boolean expired;
// session的属性容器
private transient Map<Object, Object> attributes;
```

[-[Session]-]作为用户与客户端之间的一次会话，当长时间未访问时就需要清除该[-[Session]-]，此时就需要每次请求时都需要刷新该次会话包含的时间，改变相关状态。

[-[Shiro]-]每次通过`AbstractShiroFilter`的`doFilterInternal`方法来调用[-[Session]-]的`touch`方法来实现最后一次访问时间的更新。
````java
protected void doFilterInternal(ServletRequest servletRequest, ServletResponse servletResponse, final FilterChain chain)
        throws ServletException, IOException {
        ……
        subject.execute(new Callable() {
            public Object call() throws Exception {
                updateSessionLastAccessTime(request, response);
                executeChain(request, response, chain);
                return null;
            }
        });
    ……
}
protected void updateSessionLastAccessTime(ServletRequest request, ServletResponse response) {
    ……
    session.touch();
    ……
}
````

## [-[Session]-]状态监控

[-[Session]-]都有一个生存时间，如果超过这个时间内没有对[-[Sesison]-]进行访问就需要删除该[-[Session]-]。在创建[-[Session]-]的步骤间，`AbstractValidatingSessionManager`添加了对[-[Session]-]状态的监控，其中`createSession`方法中就启动了`enableSessionValidationIfNecessary()`检验类。

```java
private void enableSessionValidationIfNecessary() {
    SessionValidationScheduler scheduler = getSessionValidationScheduler();
    if (isSessionValidationSchedulerEnabled() && (scheduler == null || !scheduler.isEnabled())) {
        enableSessionValidation();
    }
}

protected synchronized void enableSessionValidation() {
    SessionValidationScheduler scheduler = getSessionValidationScheduler();
    if (scheduler == null) {
        scheduler = createSessionValidationScheduler();
        setSessionValidationScheduler(scheduler);
    }
    if (!scheduler.isEnabled()) {
        if (log.isInfoEnabled()) {
            log.info("Enabling session validation scheduler...");
        }
        scheduler.enableSessionValidation();
        afterSessionValidationEnabled();
    }
}
```
[-[Shiro]-]的`SessionValidationScheduler`方法定义了校验的接口，其默认时通过实现`enableSessionvalidation`方法定义一个定时任务来调用`SessionManager`的`validateSession`方法。

```java
// ExecutorServiceSessionValidationScheduler
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
        ……
        Thread.currentThread().setUncaughtExceptionHandler((t, e) -> {
            log.error("Error while validating the session, the thread will be stopped and session validation disabled", e);
            this.disableSessionValidation();
        });
        ……
            this.sessionManager.validateSessions();
        ……
    }
```

默认的`SessionManager`的`validateSessions`实现。

```java
public void validateSessions() {
    ……
    //失效Session个数
    int invalidCount = 0;
    //从SessionDao的getActiveSession中获取Sessions
    Collection<Session> activeSessions = getActiveSessions();

    if (activeSessions != null && !activeSessions.isEmpty()) {
        for (Session s : activeSessions) {
            try {
                SessionKey key = new DefaultSessionKey(s.getId());
                validate(s, key);
            } catch (InvalidSessionException e) {
                ……
                invalidCount++;
            }
        }
    }
    ……
}
//相关操作都会交给sessionDao去操作
protected void validate(Session session, SessionKey key) throws InvalidSessionException {
    try {
        //判断当前session是否有效
        doValidate(session);
    } catch (ExpiredSessionException ese) {
        //过期处理
        onExpiration(session, ese, key);
        throw ese;
    } catch (InvalidSessionException ise) {
        //无效处理
        onInvalidation(session, ise, key);
        throw ise;
    }
}    
```
默认情况下，定时线程是每60分钟检测一次，但是[-[Session]-]的默认生命周期为30分钟，这样看起来我们获得的[-[Session]-]可能会有问题。但是这并不需要担心，因为[-[Shiro]-]在`AbsctractNativeSessionManager`中，每次调用`getSession`时都会先调用`lookupSession`这个方法来对[-[Session]-]进行有效性检测。