[TOC]

# jasig cas server
## 准备
1. 下载cas server源代码。
cas源代码托。管在github上，地址：https://github.com/Jasig/cas/releases。目前有多个releases版本，我使用的版本是4.1.3
2. 将cas server源代码导入IDE
因为cas server使用maven构建，所以导入时需要选择maven项目。因为cas server依赖的jar比较多所以等待IDE下载依赖。
3. 启动cas-server-webbapp
cas-server-webapp是SSO的登录web应用，将该应用直接部署到tomcat就可以提供单点登录服务。启动后方访问http://localhost:port/projectName就可以跳转至登录页面。

> cas server 4.1.3 要求jdk1.7及以上

## cas server模块介绍
cas server采用模块化开发，以cas-server-core为中心，不同的特性功能在不同的模块开发，在使用时可以根据自己的应用场景选择你所依赖的模块。

- cas-server-documentation
- cas-server-core
- cas-server-core-api
- cas-server-webapp
- cas-server-webapp-support
- cas-server-support-generic
- cas-server-support-jdbc
- cas-server-support-ldap
- cas-server-support-legacy
- cas-server-support-openid
- cas-server-support-radius
- cas-server-support-spnego
- cas-server-support-trusted
- cas-server-support-x509
- cas-server-support-oauth
- cas-server-support-pac4j
- cas-server-support-saml
- cas-server-integration-jboss
- cas-server-integration-memcached
- cas-server-integration-ehcache
- cas-server-integration-restlet
- cas-server-uber-webapp
- cas-server-extension-clearpass
- cas-management-webapp
- cas-server-support-rest
- cas-server-integration-hazelcast
- cas-server-integration-mongo

### cas-server-documentation
cas-server-documentation 是使用cas server的帮助文档，该文档主要从如下4方面介绍cas server

- 系统架构
主要介绍系统逻辑上的分层，系统交互处理流程，系统部署的物理架构
- 系统初始化
主要介绍各种认证方式的配置，SSO Session配置，web流程的客户化等
- 系统集成
主要介绍与CAS客户端的集成，与其他应用账号的集成
- 访问协议
主要介绍cas server支持的多张访问协议，cas,  saml, rest, OAuth等

### cas-server-core-api
规范cas server的接口定义，主要从认证、登录、监控、服务、令牌等方面
### cas-server-core
cas server的骨架核心代码，实现cas-server-core-api
### cas-server-webapp
cas server认证的web项目

## 认证流程
不画时序图了，简单说明主要接口的调用顺序
CentralAuthenticationService `->` AuthenticationManager `->` AuthenticationHandler `->` PrincipalResolver `->` AuthenticationExceptionHandler


## 开发自己的认证流程
cas server本身的功能虽然非常强大，但在现实场景中也并不一定适应。因为cas-server-webapp登录页面无验证码要素，所以我已增加验证码中场景描述如何扩展。
本次扩展分如下步骤
1. 增加一个模块用于处理我们的业务逻辑，避免与cas server目前的模块混淆。模块名称cas-server-cnepay-core
2. 模块中增加自己的认证处理Handler
3. 模块中增加获取验证码controller
4. 模块中增加自己的凭证对象
5. 模块中增加自定义的业务异常对象
6. 将该模块整合进cas-server-webapp模块中
7. 修改web页面和webflow

### 增加业务处理模块
略过

### 增加认证Handler
```java
public class AcceptUsersAuthenticationHandler extends AbstractPreAndPostProcessingAuthenticationHandler {
    @Override
    protected HandlerResult doAuthentication(Credential credential) throws GeneralSecurityException, PreventedException {
        ServletRequestAttributes attr = (ServletRequestAttributes) RequestContextHolder.currentRequestAttributes();
        String code = kaptcha.getGeneratedKey(attr.getRequest());
        final UsernamePasswordCredential userPass = (UsernamePasswordCredential) credential;
        if (!code.equals(userPass.getCaptcha())) {
            throw new CaptchaException();
        }
        if (users.containsKey(userPass.getUsername())) {
            String password = users.get(userPass.getUsername());
            if (!password.equals(userPass.getPassword())) {
                throw new PasswordException("Invalid password");
            }
        } else {
            throw new AccountNotFoundException("Username is not found");
        }
        return new DefaultHandlerResult(this, new BasicCredentialMetaData(credential), principalFactory.createPrincipal(userPass.getUsername()), null);
    }

    @NotNull
    private Map<String, String> users;

    public final void setUsers(final Map<String, String> users) {
        this.users = Collections.unmodifiableMap(users);
    }
    @NotNull
    private KaptchaExtend kaptcha;

    public void setKaptcha(KaptchaExtend kaptcha) {
        this.kaptcha = kaptcha;
    }
    @Override
    public boolean supports(Credential credential) {
        return credential instanceof UsernamePasswordCredential;
    }
}
```
### 获取验证码controller
> 我使用google的开源项目kaptcha

```java
@Controller
public class RegisterKaptchaController {

    private KaptchaExtend kaptcha;

    public void setKaptcha(KaptchaExtend kaptcha) {
        this.kaptcha = kaptcha;
    }

    @RequestMapping(value = "/captcha.jpg", method = RequestMethod.GET)
    public void captcha(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        kaptcha.captcha(req, resp);
    }
}
```

### 增加自己的凭证对象
```java
public class UsernamePasswordCredential implements Credential, Serializable {

    /** Authentication attribute name for password. **/
    public static final String AUTHENTICATION_ATTRIBUTE_PASSWORD = "credential";

    /** Unique ID for serialization. */
    private static final long serialVersionUID = -700605081472810939L;

    /** Password suffix appended to username in string representation. */
    private static final String PASSWORD_SUFFIX = "+password";

    /** The username. */
    @NotNull
    @Size(min=1, message = "required.username")
    private String username;

    /** The password. */
    @NotNull
    @Size(min=1, message = "required.password")
    private String password;

    @NotNull
    @Size(min=1, message = "required.captcha")
    private String captcha;

    /** Default constructor. */
    public UsernamePasswordCredential() {}

    /**
     * Creates a new instance with the given username and password.
     *
     * @param userName Non-null user name.
     * @param password Non-null password.
     */
    public UsernamePasswordCredential(final String userName, final String password, final String captcha) {
        this.username = userName;
        this.password = password;
        this.captcha = captcha;
    }

    /**
     * {@inheritDoc}
     */
    @Override
    public String getId() {
        return this.username;
    }
    getter setter 省略......
}
```

### 增加自定义的业务异常对象
```java
public class CaptchaException extends LoginException {

    public CaptchaException() {
        super();
    }

    public CaptchaException(String msg) {
        super(msg);
    }
}
```

### 整合进cas-server-webapp
`cas-server-cnepay-core/src/main/resources/META-INF/spring`目录增加配置文件"applicationContext.xml"
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    <bean id="registerKaptchaController" class="com.cnepay.cas.server.controller.RegisterKaptchaController">
        <property name="kaptcha" ref="kaptcha" />
    </bean>
    <bean id="cnepayAuthenticationHandler"
          class="com.cnepay.cas.server.authentication.AcceptUsersAuthenticationHandler">
        <property name="users">
            <map>
                <entry key="casuser" value="Mellon"/>
            </map>
        </property>
        <property name="kaptcha" ref="kaptcha"/>
    </bean>
    <bean id="kaptcha" class="com.google.code.kaptcha.servlet.KaptchaExtend" />
</beans>
```

覆盖cas-server-core已定义的异常，注意你覆盖的异常要包含cas-server-core的原有内容。`cas-server-webapp/src/webapp/WEB-INF/cas-servlet.xml`文件增加内容
```java
<bean id="exceptionConfig" class="com.cnepay.cas.server.config.ExceptionConfig" init-method="init">
    <property name="exceptionHandler" ref="authenticationExceptionHandler" />
</bean>
```

允许cas-server-webapp访问到图片验证码。`cas-server-webapp/src/webapp/WEB-INF/web.xml`文件中增加
```xml
<servlet-mapping>
    <servlet-name>cas</servlet-name>
    <url-pattern>/captcha.jpg</url-pattern>
</servlet-mapping>
```

将我们的认证handler `AcceptUsersAuthenticationHandler`注入到`AuthenticationManager`。修改`cas-server-webapp/src/webapp/WEB-INF/deployerConfigContext.xml`文件
```xml
<bean id="authenticationManager" class="org.jasig.cas.authentication.PolicyBasedAuthenticationManager">
    <constructor-arg>
        <map>
            <entry key-ref="proxyAuthenticationHandler" value-ref="proxyPrincipalResolver"/>
            <entry key-ref="cnepayAuthenticationHandler" value-ref="proxyPrincipalResolver" />
        </map>
    </constructor-arg>
</bean>
```

### 修改webflow
cas-server-webapp模块使用spring
修改`cas-server-webapp/src/webapp/WEB-INF/login/login-webflow.xml`
1. 修改凭证对账
```xml
<var name="credential" class="com.cnepay.cas.server.credential.UsernamePasswordCredential"/>
```
2. 修改与页面绑定的元素
```xml
<view-state id="viewLoginForm" view="casLoginView" model="credential">
    <binder>
        <binding property="username" required="true"/>
        <binding property="password" required="true"/>
        <binding property="captcha" required="true"/>
    </binder>
    <on-entry>
        <set name="viewScope.commandName" value="'credential'"/>
    </on-entry>
    <transition on="submit" bind="true" validate="true" to="realSubmit"/>
</view-state>
```
3. 修改jsp页面，这里不写代码了



*至此修改的所有内容都完成了，启动服务测试吧*

# 客户端集成
cas client支持多种语言集成，java,.net,php等。客户端集成有两个步骤

- 集成cas client
- 将客户端注册到cas server

## 集成cas client
以web应用为例
1. 增加cas client 依赖
```xml
<dependency>
    <groupId>org.jasig.cas.client</groupId>
    <artifactId>cas-client-core</artifactId>
    <version>${java.cas.client.version}</version>
</dependency>
```
*cas client我使用了3.4.1版本，jdk版本为1.6*
2. 增加认证拦截过滤器
访问客户端时如果未登陆跳转至cas server
web.xml中增加
```xml
<filter>
        <filter-name>CAS Authentication Filter</filter-name>
        <filter-class>org.jasig.cas.client.authentication.AuthenticationFilter</filter-class>
        <init-param>
            <param-name>casServerLoginUrl</param-name>
            <param-value>http://cas.server.com:8081/cas/login</param-value>
        </init-param>
        <init-param>
            <param-name>serverName</param-name>
            <param-value>http://cas.client.com</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CAS Authentication Filter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```
`casServerLoginUrl` cas server登录地址
`serverName` 客户端地址
3. 增加service ticket验证过滤器
cas server 重定向回客户端携带了service ticket，客户端调用cas server接口完成ticket验证
web.xml中增加
```xml
<filter>
    <filter-name>CAS Validation Filter</filter-name>
    <filter-class>org.jasig.cas.client.validation.Cas10TicketValidationFilter</filter-class>
    <init-param>
        <param-name>casServerUrlPrefix</param-name>
        <param-value>http://cas.server.com:8081/cas</param-value>
    </init-param>
    <init-param>
        <param-name>serverName</param-name>
        <param-value>http://cas.client.com</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CAS Validation Filter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 将客户端注册到cas  server
`cas-server-webapp/src/resources/services/`目录增加文件'xx.json'。增加如下配置

```json
{
  "@class" : "org.jasig.cas.services.RegexRegisteredService",
  "serviceId" : "^http://cas.client.com\\S*",
  "name" : "sample",
  "id" : 1984,
  "description" : "sample",
  "accessStrategy" : {
    "@class" : "org.jasig.cas.services.DefaultRegisteredServiceAccessStrategy",
    "enabled" : true,
    "ssoEnabled" : true
  }
}
```

- serviceId：客户端的url需要匹配该正则表达式，如果不匹配拒绝认证
- id：每个客户端不允许重复

也可以参考 https://github.com/Jasig/java-cas-client
