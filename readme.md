#  J2EE快速开发框架

## 项目简介

- 使用Maven对项目进行模块化管理，提高项目的易开发性、扩展性。
- 实现了通用的系统管理模块功能，包含：用户、角色、权限、菜单、字典管理。
- 实现了基于AOP解耦的日志模块。
- 实现了通用的异常处理和响应模型、错误码标准规范。（非restful）
- 实现了基于JWT和Redis的Token认证。
- 实现了支持根据参数动态构建复杂SQL语句的构建器，基于freemarker。
- 颜值还可以的后台UI界面。

app意思是应用程序Application，开发时替换成你自己的项目名

欢迎进群交流

### 点击加入QQ群[631511782](https://jq.qq.com/?_wv=1027&k=47ErLEy)



## 模块说明
- app-admin         后台管理界面模块
- app-api           开放给移动端或其他终端的接口模块
- app-auth          基于Redis的身份认证模块
- app-base          基础模块，包含底层DAO、Service等的封装
- app-file          简单的文件服务模块，使用FTP服务器上传和下载
- app-generator     代码生成器
- app-log           通用日志模块
- app-system        通用的系统管理模块
- app-utils         一些通用的工具类

## 技术选型
- 核心框架：Spring Framework 4.3.6
- 安全框架：Apache Shiro 1.3.2
- 持久层框架：Hibernate 5.2.6.Final
- 数据库连接池：Alibaba Druid 1.0.29
- Token生成和管理：JWT、Redis
- 日志管理：SLF4J、Log4j2
- 数据库：MySQL
- 后台前端框架：Jquery EasyUI 1.5.X 和 JQuery EasyUI 1.5.x of Insdep Theme 

## 使用说明

### 1. 启动说明

    * 后台帐号密码：admin  123456

    * 项目依赖Redis服务，请先安装Redis客户端。
    
    * 项目有2个war包模块，请使用不同的端口运行启动。
        app-admin：是后台管理界面
        app-api：是api模块，实现了基于jwt和redis的token认证。一般应用于前后端分离的项目，如Android、IOS等客户终端调用的接口都来源于此模块，使用token进行身份认证。
        
    * 数据库：
        运行前请先创建数据库，数据库名：app-xxx（xxx表示不同环境）
        项目使用了Hibernate注解映射，会自动生成表结构
        项目启动后，导入SQL文件（app-admin模块sql文件夹下app-dev.sql）进行导入数据。
        
    * 环境配置/打包：
    	 app-admin 和 app-api 模块下都包含4套环境配置，不同的环境请自行修改里面的参数。
    	 说明：
    	 local - 本地环境
    	 dev - 开发环境
    	 test - 测试环境
    	 pro - 生成环境
    	 
    	 使用maven打包时，可以选择不同的环境配置文件
    	 
    
### 2. 基于Redis的身份认证模块使用说明
依赖app-auth模块，也可将该模块可以单独打成jar包再引用

使用Redis存储Token，在需要集成身份认证的项目的spring-mvc.xml文件中配置：

```
<!-- 加载配置文件数据 -->
    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath*:config/redis.properties</value>
                <value>classpath*:config/token.properties</value>
            </list>
        </property>
        <property name="ignoreUnresolvablePlaceholders" value="true"/>
    </bean>

<!--授权认证配置 begin-->

    <!--通过Key获得对应用户的bean-->
    <bean id="userRepository" class="io.zhijian.app.authorization.resolver.UserRepository"/>
    <mvc:annotation-driven>
        <mvc:argument-resolvers>
            <!--配置注入登录用户的解析器-->
            <bean id="currentUserMethodArgumentResolver"
                  class="io.zhijian.authorization.resolvers.CurrentUserMethodArgumentResolver">
                <!--需要解析的用户类-->
                <property name="userModelClass" value="io.zhijian.system.model.response.UserResponse"/>
                <!--查询用户的bean-->
                <property name="userModelRepository" ref="userRepository"/>
            </bean>
        </mvc:argument-resolvers>
    </mvc:annotation-driven>

    <mvc:interceptors>
        <!--身份验证的拦截器-->
        <bean id="authorizationInterceptor" class="io.zhijian.authorization.interceptor.AuthorizationInterceptor">
            <!--验证信息存储的Http头-->
            <property name="httpHeaderName" value="Authorization"/>
            <!--验证信息通用前缀，例如Bearer-->
            <property name="httpHeaderPrefix" value=""/>
            <!--验证失败时的错误信息-->
            <property name="unauthorizedErrorMessage" value="invalid&#x0020;token"/>
            <!--管理验证信息的bean-->
            <property name="manager" ref="tokenManager"/>
            <!--token验证器-->
            <property name="validator" ref="tokenValidator"/>
        </bean>
    </mvc:interceptors>

    <!--管理验证信息的bean-->
    <bean id="tokenValidator" class="io.zhijian.app.authorization.jwt.JwtTokenValidator"></bean>

    <!--Redis配置-->
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!--最大连接数-->
        <property name="maxTotal" value="${redis.maxTotal}" />
        <!--最大空闲连接数-->
        <property name="maxIdle" value="${redis.maxIdle}" />
        <!--初始化连接数-->
        <property name="minIdle" value="${redis.minIdle}"/>
        <!--最大等待时间-->
        <property name="maxWaitMillis" value="${redis.maxWaitMillis}" />
        <!--对拿到的connection进行validateObject校验-->
        <property name="testOnBorrow" value="${redis.testOnBorrow}" />
        <!--在进行returnObject对返回的connection进行validateObject校验-->
        <property name="testOnReturn" value="${redis.testOnReturn}" />
        <!--定时对线程池中空闲的链接进行validateObject校验-->
        <property name="testWhileIdle" value="true" />
    </bean>

    <!--Redis连接池-->
    <bean id="jedisPool" class="redis.clients.jedis.JedisPool">
        <constructor-arg index="0" ref="jedisPoolConfig"/>
        <constructor-arg index="1" value="${redis.host}"/>
        <constructor-arg index="2" value="${redis.port}" type="int"/>
        <constructor-arg index="3" value="${redis.timeout}" type="int"/>
        <constructor-arg index="4" value="${redis.password}"/>
    </bean>

    <!--管理验证信息的bean-->
    <bean id="tokenManager" class="io.zhijian.authorization.manager.impl.RedisTokenManager">
        <!--Redis客户端连接池配置-->
        <property name="jedisPool" ref="jedisPool"/>
        <!--Token失效时间-->
        <property name="tokenExpireSeconds" value="${token.expire.seconds}"/>
        <!--一个用户是否可以绑定多个Token-->
        <property name="singleTokenWithUser" value="${single.token.with.user}"/>
        <!--在每次有效操作后刷新过期时间-->
        <property name="flushExpireAfterOperation" value="${flush.expire.after.operation}"/>
    </bean>

    <!--授权认证配置 end-->
    
```

添加配置文件redis.properties和token.properties
```
#redis配置

redis.host = 192.168.1.200
redis.port = 6379
redis.timeout = 100000
redis.password = xxxxxx

redis.minIdle=100
redis.maxIdle=300
redis.maxTotal=600
redis.maxWaitMillis=1000
redis.testOnBorrow=true
redis.testOnReturn=true

```

```
#Token配置

#1分钟 = 60000	毫秒(ms)
#1小时 = 3600000	毫秒(ms)
#1天 = 86400000	毫秒(ms)
#7天 = 604800000	毫秒(ms)

#base64加密字符串
jwt.base64.secret = MDk4ZjZiY2Q0NjIxZDM3M2NhZGU0ZTgzMjYyN2I0ZjY=
jwt.refresh.base64.secret = MHNkamZsYWpkZmppMjM0MDk4amtkc2E4dTIwMzg0bGFmYXNkZjQ1NndlcnUwOTIzdWxh

#jwt token过期时间，毫秒
jwt.expires.second = 60000

#refresh token过期时间
jwt.refresh.expires.second = 604800000

#######################################################

#是否在每次有效操作后刷新token在redis中的过期时间，是=true，否=false
flush.expire.after.operation = true

#是否一个用户单对应唯一一个Token，是=true，否=false
#说明：如果是同一帐号多端登录设置为false
single.token.with.user = false

#redis token过期时间，秒
token.expire.seconds = 3600
```

配置具体参考app-api模块中，spring-mvc.xml的配置

使用如下：
需要身份验证的方法加上@Authorization注解即可，也可以直接在Controller类上加上该注解，这将会使该Controller中的所有方法都需要进行身份验证。
```
    @Authorization
    @RequestMapping(value = "/user/get", method = RequestMethod.GET)
    @ResponseBody
    public UserResponse get(@CurrentUser UserResponse user) {
        return userService.get(user.getUsername());
    }
```

@CurrentUser注解可以获取当前登录用户的实例，拥有@CurrentUser参数的方法，可以没有@Authorization注解，此时如果请求未登录，该参数会为null，但是如果想要使用CurrentUserMethodArgumentResolver则必须配置AuthorizationInterceptor。

当前用户的实例获取和解析，在上面的配置文件里面已经配置了解析器，但是需要自行实现UserModelRepository。

鉴权失败，会统一返回401  HTTP状态码，注意这里是http状态码，也可以在配置文件自定义鉴权失败的http状态码，默认为401（unauthorized）。

### 3. 通用日志模块
依赖app-log模块，也可将该模块可以单独打成jar包再引用

在spring-context.xml中添加配置

    <!-- 激活自动代理功能 -->
    <aop:aspectj-autoproxy proxy-target-class="true" />

    <!-- 扫描切面包路径 -->
    <bean id="logAspect" class="io.zhijian.log.aop.LogAspect">
        <property name="logPoint">
            <bean class="io.zhijian.system.service.impl.SystemLogService" />
        </property>
    </bean>
    <aop:config>
        <aop:aspect ref="logAspect">
            <aop:pointcut id="logPointCut" expression="@annotation(io.zhijian.log.annotation.Log)" />
            <aop:around pointcut-ref="logPointCut" method="save" />
        </aop:aspect>
    </aop:config>

SystemLogService实现了LogPoint类中的save方法，在该方法中实现日志的存储

具体参考app-system模块中SystemLogService.java

### 4. 复杂SQL构建器使用
我为什么做了这个东西？
因为ORM框架使用了Hibernate框架，而Hibernate对于复杂的自定义的SQL的支持力度不够，或许是我不太会使用Hibernate。
在有些时候有些业务逻辑有些复杂的查询需要连接很多张表，比如统计，并且是要根据参数来动态构建的。这个时候并没有没有任何一个实体类可以与这个结果集进行映射和匹配。以往我们都是自己在dao的Java类中使用String进行动态拼接，这样我觉得写起来很恶心，而且可读性很差，所以就有了这个构建器。
很简单的东西，基于freemarker，这里我并没有考虑查询的性能。

示例：按模块来统计日志记录的数据量，若参数中有模块名则添加条件没有则统计所有
先在项目的sql文件夹下建一个select-test.sql文件，里面是查询语句
```
<#assign data = params?eval>

select module, count(*) as count
from sys_log

<#if data.module??>
 where module =:module
</#if>

group by module order by count
```
可以看到我用了freemarker的判断语法，以及assign 语法， <#assign data = params?eval> 中的params名称是固定的。

然后在dao里面添加方法
```
/**
     * 测试SQLBuilder 的使用
     * 原理：使用freemarker模板定义要执行的sql文件，支持动态参数逻辑判断，构建SQL语句
     * @param module
     * @return
     */
    @Override
    public List<TestCount> findTest(String module) {
        try {
            Map params = new HashMap<>();
            params.put("module", module);

            String json = JSON.toJSONString(params);

            String sql = SqlBuilder.buildSql(SqlConfig.getSqlPath("select-test"), json);

            NativeQuery query = getSession().createNativeQuery(sql);
            query.setResultTransformer(new BeanTransformerAdapter<>(TestCount.class));
            setParameters(query, params);

            List<TestCount> result = query.list();

            return result;
        } catch (IOException e) {
            e.printStackTrace();
        } catch (TemplateException e) {
            e.printStackTrace();
        }
        return null;
    }
```

BeanTransformerAdapter是对Hibernate返回的结果集的映射转换器。

### 5. 代码生成器的使用
app-generator模块是个独立模块，也可将该模块可以单独打成jar包再引用
这个代码生成器比较简单，只针对该项目的结构，支持自定义其他模板，自行添加，修改代码即可

生成代码运行：io.zhijian.generator.core.generator.Run 类即可

生成代码之前需要配置config.properties文件
```

###配置生成器参数

#模板所在目录
template.path = E:\\Workspace\\J2EE\\app\\app-generator\\src\\main\\resources\\template

#代码生成输出目录
output.path = E:\\Workspace\\TestGenerator

#数据库实体类所在目录
input.path = E:\\Workspace\\J2EE\\app\\app-system\\src\\main\\java\\io\\zhijian\\system\\entity

#项目模块包根路径
base.package = io.zhijian.system

#dao生成的包路径
dao.package = io.zhijian.system.dao

#service生成的包路径
service.package = io.zhijian.system.service

#controller生成的包路径
controller.package = io.zhijian.controller.sys

#model生成的包路径、包含request、response（VO、DTO）
model.package = io.zhijian.system.model

#主键类型
pk.type = Integer

```
配置有详细的注释说明，代码也很简单，原理是根据数据实体类和模板文件生成各个模块的代码，所以生成代码之前必须要先有数据库实体类，没有做读取数据库然后生成的逻辑，那些太麻烦，有兴趣可以自己做。

然后是模板文件，在resource/template 目录下，可以自行添加和修改。

### 6. 统一异常处理、响应结果、状态码的约定
- 统一的响应结果

如下：
```
{
	"body": {
		"username": "admin"
	},
	"code": 200,
	"message": "ok",
	"now": "2017-04-06 15:42:20"
}
```

code = 状态码
message = 信息，有错误或异常时即为错误信息（客户端不应该直接显示该message，而应该自己根据code的业务标识判断，返回友好的提示给用户）
now = 服务器当前时间
body = 实际响应内容，有错误或异常时为null


- 统一异常处理

全局业务异常类：ApplicationException，包含状态码，错误信息

程序所有的异常都通过AnnotationHandlerMethodExceptionResolver 来统一拦截处理成统一的响应结果了


没有使用boolean类型的标识，boolean类型的标识会带来歧义，统一约定好的状态码或许更好。


- 状态码的约定

除了Token验证失败返回了HTTP状态码外，其他的异常、错误、逻辑判断返回的状态码都是自定义的

在base模块中的StatusCode定义了一些常用的状态码，可以直接使用。

但其他业务模块的状态码不应该定义在base模块中，也就是说base模块是不负责处理业务
各个不同的业务模块，需要自行按照StatusCode中的规格定义，如system模块中的SystemError

业务模块的状态码约定：
```

/*
        错误码格式说明（示例：202001），1为系统级错误，2为业务逻辑错误
        --------------------------------------------------------------------
        服务级错误（1为系统级错误）	服务模块代码(即业务模块标识)	具体错误代码
                2                            02	                    001
        --------------------------------------------------------------------
    */
    //2 00 001 释义：  00 = System 业务模块标识，001为具体的错误代码

约定：
00 = system模块
01 =
02 =
03 =
04 =
....

```

开发人员应该统一规范和标准，约定如何定义各模块的状态码标识



## 主要功能/界面展示
 1. 登录/主界面
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203422_ea01c37b_24573.png "在这里输入图片标题")

![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203217_f903a471_24573.png "在这里输入图片标题")
 2. 用户管理
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203237_02adaefc_24573.png "在这里输入图片标题")
 3. 角色管理
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203251_fb0ac960_24573.png "在这里输入图片标题")
 4. 权限管理
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203306_7984a3f7_24573.png "在这里输入图片标题")
 5. 菜单管理
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203319_f4a01aeb_24573.png "在这里输入图片标题")
 6. 字典管理
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203332_689beffb_24573.png "在这里输入图片标题")
 7. 日志管理
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203347_cdaeb8a5_24573.png "在这里输入图片标题")
 8. Token生成
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203625_17874873_24573.png "在这里输入图片标题")


## 感谢
@ScienJus
项目基于Redis的认证模块修改和集成了此项目
https://github.com/ScienJus/spring-authorization-manager

Jquery EasyUI Insdep主题
https://www.insdep.com

## 交流
### 点击加入QQ群[631511782](https://jq.qq.com/?_wv=1027&k=47ErLEy)
交流技术问题
![输入图片说明](http://git.oschina.net/uploads/images/2017/0410/203156_d7a3b7fd_24573.png "在这里输入图片标题")
