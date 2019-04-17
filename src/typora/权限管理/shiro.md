# Shiro

## 简介

### Shiro 的组成

![image-20181206211700207](/Users/dyh/Library/Application Support/typora-user-images/image-20181206211700207.png)



**Authentication**：身份认证 / 登录，验证用户是不是拥有相应的身份；

**Authorization**：授权，即权限验证，验证某个已认证的用户是否拥有某个权限；即判断用户是否能做事情，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限；

**Session Manager**：会话管理，即用户登录后就是一次会话，在没有退出之前，它的所有信息都在会话中；会话可以是普通 JavaSE 环境的，也可以是如 Web 环境的；

**Cryptography**：加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储；

**Web Support**：Web 支持，可以非常容易的集成到 Web 环境；

**Caching**：缓存，比如用户登录后，其用户信息、拥有的角色 / 权限不必每次去查，这样可以提高效率；

**Concurrency**：shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去；

**Testing**：提供测试支持；

**Run As**：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问；

**Remember Me**：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。



### Shiro 的工作流程

![image-20181206213650461](/Users/dyh/Library/Application Support/typora-user-images/image-20181206213650461.png)

**Subject**：主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 `Subject`；所有 `Subject` 都绑定到 `SecurityManager`，与 `Subject` 的所有交互都会委托给`SecurityManager`；

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 `Subject`；可以看出它是 `Shiro` 的核心，它负责与后边介绍的其他组件进行交互，类似于 SpringMVC 的 `DispatcherServlet `

**Realm**：域，Shiro 从从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源

实际执行上应用程序和`Subject`交互，`Subject` 委托给` SecurityManager`， 而`SecurityManager`一般会委托先给 `Authenticator` 先进行身份认证，而` Authenticator` 会委托给 `AuthenticationStrategy `进行多 `Realm` 身份验证，默认` ModularRealmAuthenticator `会调用 `AuthenticationStrategy` 进行多 `Realm` 身份验证，

### Shiro 的架构

![image-20181206214246561](/Users/dyh/Library/Application Support/typora-user-images/image-20181206214246561.png)

**Subject**：主体，可以看到主体可以是任何可以与应用交互的 “用户”；

**SecurityManager**：相当于 SpringMVC 中的` DispatcherServlet`;

**Authenticator**：认证器，负责主体认证的，这是一个扩展点，如果用户觉得 Shiro 默认的不好，可以自定义实现；其需要认证策略（`AuthenticationStrategy`），即什么情况下算用户认证通过了；

**Authrizer**：授权器，或者访问控制器，用来决定主体是否有权限进行相应的操作；即控制着用户能访问应用中的哪些功能；

**Realm**：可以有 1 个或多个 Realm，可以认为是安全实体数据源，即用于获取安全实体的；可以是 JDBC 实现，也可以是 LDAP 实现，或者内存实现等等；由用户提供；注意：Shiro 不知道你的用户 / 权限存储在哪及以何种格式存储；所以我们一般在应用中都需要实现自己的 Realm；

**SessionManager**：会话管理，不仅仅是 web 环境中的会话，非 web 环境中也可以使用会话，而 		`SessionManager` 就是专门用来管理会话的

**CacheManager**：缓存控制器，来管理如用户、角色、权限等的缓存的；因为这些数据基本上很少去改变，放到缓存中后可以提高访问的性能

**Cryptography**：密码模块，Shiro 提高了一些常见的加密组件用于如密码加密 / 解密的。



## 身份验证

**principals**：身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个 principals，但只有一个 Primary principals，一般是用户名 / 密码 / 手机号。

**credentials**：证明 / 凭证，即只有主体知道的安全值，如密码 / 数字证书等。

最常见的 principals 和 credentials 组合就是用户名 / 密码了。接下来先进行一个基本的身份认证。

![image-20181207112927359](/Users/dyh/Library/Application Support/typora-user-images/image-20181207112927359.png)



## 授权

### 名次解释

![image-20181207154801388](/Users/dyh/Library/Application Support/typora-user-images/image-20181207154801388.png)	

**主体**
主体，即访问应用的用户，在 Shiro 中使用 Subject 代表该用户。用户只有授权后才允许访问相应的资源。

**资源**
在应用中用户可以访问的任何东西，比如访问 JSP 页面、查看/编辑某些数据、访问某个业务方法、打印文本等等都是资源。用户只要授权后才能访问。

**权限**
安全策略中的原子授权单位，通过权限我们可以表示在应用中用户有没有操作某个资源的权力。即权限表示在应用中用户能不能访问某个资源，如： 访问用户列表页面
查看/新增/修改/删除用户数据（即很多时候都是 CRUD（增查改删）式权限控制）
打印文档等等。。。

如上可以看出，权限代表了用户有没有操作某个资源的权利，即反映在某个资源上的操作允不允许，不反映谁去执行这个操作。所以后续还需要把权限赋予给用户，即定义哪个用户允许在某个资源上做什么操作（权限），Shiro 不会去做这件事情，而是由实现人员提供。

Shiro 支持粗粒度权限（如用户模块的所有权限）和细粒度权限（操作某个用户的权限，即实例级别的），后续部分介绍。

**角色**
角色代表了操作集合，可以理解为权限的集合，一般情况下我们会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便。典型的如：项目经理、技术总监、CTO、开发工程师等都是角色，不同的角色拥有一组不同的权限。

### 关键接口介绍

![image-20181207172112173](/Users/dyh/Library/Application Support/typora-user-images/image-20181207172112173.png)



**PermissionResolver**

当主体对象传入字符串时，通过该接口的`Permission resolvePermission(String var1);`方法可将字符串转换为` Permission`

**Permission**

该接口只有一个方法：`boolean implies(Permission var1);`，通过该方法可以将主体所具有的权限和主体所做的操作做比较，判断主体是否有相应操作的权限

**Authorizer**

Authorizer 的职责是进行授权（访问控制），是 Shiro API 中授权核心的入口点，其提供了相应的角色/权限判断接口以此判断主体是否具有某项操作的权限。`ModularRealmAuthorizer`即为该接口的实现类

**RolePermissionResolver** 

用于根据角色解析相应的权限集合。

**AuthorizingRealm**

如果 Realm 进行授权的话，应该继承 `AuthorizingRealm`，它继承了`CachingRealm`，使得它具备缓存的功能

**AuthorizationInfo**

获取授权信息接口

**AuthenticationInfo**

认证信息接口，该接口提供了获取凭证的方法



### 执行流程

![image-20181207134237867](/Users/dyh/Library/Application Support/typora-user-images/image-20181207134237867.png)

1. 首先调用 `Subject.isPermitted*/hasRole*`接口，其会委托给 SecurityManager，而 SecurityManager 接着会委托给 Authorizer；
2. Authorizer 是真正的授权者，如果我们调用如 isPermitted(“user:view”)，其首先会通过 PermissionResolver 把字符串转换成相应的 Permission 实例；
3. 在进行授权之前，其会调用相应的 Realm 获取 Subject 相应的角色/权限用于匹配传入的角色/权限；
4. Authorizer 会判断 Realm 的角色/权限是否和传入的匹配，如果有多个 Realm，会委托给 ModularRealmAuthorizer 进行循环判断，如果匹配如 `isPermitted*/hasRole*` 会返回 true，否则返回 false 表示授权失败。



## 编码 / 加密

### 散列算法

- 是一种不可逆的算法，一般适合存储密码之类的数据
- 一般使用散列算法都需要加盐，让它更难破解

常见的散列算法：

- MD5 加密
- SHA256 算法、SHA1算法等等、SHA512算法等等

### 加密算法

对称加密：

- AES 加密算法

非对称加密：

- RSA 加密

对称加密和非对称加密的区别：

https://blog.csdn.net/u013320868/article/details/54090295





## Shiro 与 Web 集成

### 格式

url = 拦截器 [参数]，拦截器[参数]
即如果当前请求的 url 匹配[urls] 部分的某个 url 模式，将会执行其配置的拦截器

### Shrio 自带的拦截器

***注：Shiro 默认会通过 `FilterChainManager` 加载 `DefaultFilter` 枚举类里面的拦截器***

1. anon 拦截器表示匿名访问(即不需要登录即可访问)
2. authc 拦截器表示需要身份认证通过后才能访问
3. roles[admin] 拦截器表示需要有 admin 角色授权才能访问， roles 是 
   org.apache.shiro.web.filter.auth z.RolesAuthorizationFilter 类型的
   实例，通过参数指定访问时需要的角色，如 “[admin]”，如果有多个使用 “，” 分割，且
   验证时是 hasAllRole 验证，即且的关系。
4. perms["user:create"] 拦截器表示需要有 “user:create” 权限才能访问，Perms 
   是 org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter 
   类型的实例，和 roles 类似，只是验证权限字符串



## 拦截器机制

### 拦截器介绍

![image-20181211144702680](/Users/dyh/Library/Application Support/typora-user-images/image-20181211144702680.png)

 **OncePerRequestFilter**

`OncePerRequestFilter` 保证一次请求只调用一次 `doFilterInternal`

**AdviceFilter**

`AdviceFilter` 提供了` AOP` 的功能，其实现和` SpringMVC` 中的 `Interceptor `思想一样

- preHandle：进行请求的预处理，然后根据返回值决定是否继续处理（true：继续过滤器链）；可以通过它实现权限控制；
- postHandle：执行完拦截器链之后正常返回后执行；
- afterCompletion：不管最后有没有异常，afterCompletion 都会执行，完成如清理资源功能。

**PathMatchingFilter**

`PathMatchingFilter` 继承了 `AdviceFilter`，提供了` url `模式过滤的功能，如果需要对指定的请求进行处理，可以扩展 `PathMatchingFilter`

**AccessControlFilter**

AccessControlFilter 继承了 PathMatchingFilter，并扩展了了两个方法

```java
public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
    return isAccessAllowed(request, response, mappedValue)
     || onAccessDenied(request, response, mappedValue);
}
```

isAccessAllowed：即是否允许访问，返回 true 表示允许；
onAccessDenied：表示访问拒绝时是否自己处理，如果返回 true 表示自己不处理且继续拦截器链执行，返回 false 表示自己已经处理了（比如重定向到另一个页面）

### 拦截器链

`Shiro `的拦截器链是通过代理 `Servlet`的`Filter`来实现拦截的，代理类为`ProxiedFilterChain`，它通过对 Servlet 容器的` FilterChain ` 进行了代理；即先走 `Shiro` 自己的 `Filter `体系，然后才会委托给 `Servlet` 容器的 `FilterChain` 进行` Servlet` 容器级别的` Filter `链执行

#### 重要接口介绍

**FilterChainResolver**

`Shiro` 内部提供了一个路径匹配的` FilterChainResolver `实现：`PathMatchingFilterChainResolver`，其根据`[urls]`中配置的 `url` 模式（默认` Ant `风格）= 拦截器链和请求的` url `是否匹配来解析得到配置的拦截器链的。

**FilterChainManager**

而`PathMatchingFilterChainResolver`内部通过 `FilterChainManager` 维护着拦截器链，比如 `DefaultFilterChainManager` 实现维护着 `url` 模式与拦截器链的关系。因此我们可以通过 `FilterChainManager` 进行动态动态增加 `url`模式与拦截器链的关系。`DefaultFilterChainManager` 会默认添加 `org.apache.shiro.web.filter.mgt.DefaultFilter` 中声明的拦截器。

```java
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



### 自定义拦截器

如果要注册自定义拦截器，`IniSecurityManagerFactory/WebIniSecurityManagerFactory` 在启动时会自动扫描 `ini` 配置文件中的` [filters]/[main]` 部分并注册这些拦截器到` DefaultFilterChainManager`；且创建相应的 `url `模式与其拦截器关系链。



## 会话管理

![image-20181211202117358](/Users/dyh/Library/Application Support/typora-user-images/image-20181211202117358.png)

AbstractSessionDAO：提供了 SessionDAO 的基础实现，如生成会话 ID（即UUID） 等；

CachingSessionDAO：提供了对开发者透明的会话缓存的功能，只需要设置相应的 CacheManager 即可；

MemorySessionDAO：直接在内存中进行会话维护；

EnterpriseCacheSessionDAO：提供了缓存功能的会话维护，默认情况下使用 MapCache 实现，内部使用 ConcurrentHashMap 保存缓存的会话；

### 会话验证

`SessionValidationScheduler`接口，定时检测会话是否过期，其有两个实现类一个是`ExecutorServiceSessionValidationScheduler`和`QuartzSessionValidationScheduler`，这两个接口的实现其实都是直接调用`AbstractValidatingSessionManager` 的` validateSessions`方法进行验证，`AbstractValidatingSessionManager `的` validateSessions` 方法进行验证；如果会话比较多，会影响性能；可以考虑如分页获取会话并进行验证。可在 ini 配置文件中配置指定使用哪种会话验证器。



## 缓存管理

Shiro 对 Cache 进行了抽象，但 Shiro 本身不实现 Cache，它整合了 EhCache。

重要接口：

```java
public interface Cache<K, V> {
    //根据Key获取缓存中的值
    public V get(K key) throws CacheException; 
    //往缓存中放入key-value，返回缓存中之前的值
    public V put(K key, V value) throws CacheException;
	...
}
```

```java
public interface CacheManager {
//根据缓存名字获取一个Cache
public <K, V> Cache<K, V> getCache(String name) throws CacheException;
}
```

```java
// 通过该接口可以注入 CacheManager
public interface CacheManagerAware { //注入CacheManager
void setCacheManager(CacheManager cacheManager);
}
```

