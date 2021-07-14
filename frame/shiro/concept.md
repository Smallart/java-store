# [-[Shiro]-]概念

## 基本架构

![ShiroBasicArchitecture](/img/ShiroBasicArchitecture.png)

在最高的概念层面，[-[Shiro]-]有三个最基本的概念：[-[Subject]-]、[-[Securitymanager]-]和[-[Realms]-]。从上面的图我们可以了解它们之间是如何作用的，接续来我们就详细的了解下这些基础概念：

* [-[Subject]-]：当为系统设计对外访问的接口时，我们一般都会考虑两点：一、当前“用户”是谁；二、它是否有该访问的权限；在[-[Shiro]-]中把当前“用户”称之为[-[Subject]-]，这是因为对于安全框架来说，当前“用户”可能是人，也可能是第三方程序，也就是说[-[Subject]-]表示操作当前系统的东西。

* [-[SecurityManager]-]：如果说[-[Subject]-]代表了当前用户的操作，那么[-[SecurityManager]-]则管理这当前所有的用户和当前应用的安全策略（每个应用中只有一个[-[SecurityManager]-]，它是单例的）。它是[-[Shiro]-]的核心并且内嵌了用户配置的所有安全组件。也就是说，一旦将相对于的安全策略配置好了，对于当前用户的操作只要调用[-[Subject]-]的接口就好了。虽然是调用[-[Subject]-]的接口，但是背后的操作都是跟[-[SecurityManager]-]交互的。

* [-[Realms]-]：对于[-[Shiro]-]来说一个最重要也是最后一个核心的概念是[-[Realm]-]。在应用和安全策略数据中，[-[Realm]-]就像是连接器链接了这两部分。如果当前用户进行鉴权操作（login）和授权操作（access control）时，[-[Shiro]-]会从[-[Realm]-]中查询相关安全策略数据，也就是说[-[Realm]-]相当于一个[-[Dao]-]层，从配置或是数据库中获取相对应的安全数据。


## 详细架构

![ShiroArchitecture](/img/ShiroArchitecture.png)

* [-[Authenticator]-]：是验证当前用户的安全组件。并且数据库中的验证信息都是[-[Authenticator]-]从[-[Realms]-]中获取和比较。

    * [-[Authentication Strategy]-]：如何项目中有多个[-[Realm]-]匹配，则更具自定义的验证策略会协调各个[-[Realm]-]，直到身份验证成功。

> 鉴权（[-[Authentication]-]）：鉴权是对当前用户进行验证的一项程序，当进行鉴权时，当前用户需要向程序提交能够证明自己的信息，并且由系统进行检验。

该步骤过程如下：

1. 收集用户的认证信息称之为主体[-[principals(userName)]-]和身份证明称之为凭证[-[credentials(password)]-]。
2. 提交这些信息
3. 如果该系统来说，这两者对的上则鉴权成功

* [-[Authorizer]-]：是决定当前用户访问权限的一个安全组件。[-[Authorizer]-]也是从持久层获得访问策略的信息（当前用户的角色或是权限信息），通过这些信息来判定当前用户是否有这么操作的权限。

* [-[SessionManager]-]：包含了如何创建和管理用户会话（[-[Session]-]）生命周期，并且[-[Shiro]-]对于Session的操作都做到了与环境无关的程度。
    * [-[SessionDao]-]：用来允许对Sesison进行持久化处理，使得设计者可以对用户会话进行操作。
    
* [-[CacheManager]-]：创建并且管理被其他安全使用[-[Cache]-]实例的生命周期，可以接入其他的缓存框架。
* [-[Cryptography ]-]: 通过加密来保证数据的安全性。[-[Shiro]-]的加密组件基于[-[Java]-]，但是更加的简化和易用，从上面详细的架构可以看到，它是独立于[-[SecurityMananger]-]的一个组件，也就是它可以独立使用，相当于常规的包。


## 缓存[-[Cache]-]

设置缓存的目的是为了提高效率，[-[Shiro]-]并没有实现真正的缓存功能，而是提供了相应的接口将其他的缓存框架整合到一起，来提供简单易用的缓存模式。其使用方式为需要的组件设置[-[Cache Manager]-]，并且设置对应的缓存名称。具体实现如下：

```java
// Session的缓存名称 默认为 shiro-activeSessionCache
OnlineWebSessionManager manager = new OnlineWebSessionManager();
// 加入缓存管理器
manager.setCacheManager(getEhCacheManager());

//对于Realm的鉴权 授权操作要设置CacheName 否则为null
UserRealm userRealm = new UserRealm();
userRealm.setAuthorizationCacheName();
userRealm.setAuthenticationCacheName();
userRealm.setCacheManager(cacheManager);
```