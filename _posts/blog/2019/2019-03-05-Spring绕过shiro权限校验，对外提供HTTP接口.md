---
title: 'Spring绕过shiro权限校验，对外提供HTTP接口'
categories:
  - Blog
tags: 
  - Spring
  - Java
---

在一些边缘分享系统或分析系统或计算系统中，涉及到一些需要需要放权的接口，也就是可以在不鉴权的情况下访问。这种情况在Spring中如何实现呢？

<!--more-->

# Spring绕过shiro权限校验，对外提供HTTP接口

> 问题：管控台用到了shiro的权限校验，但是需要通过这个web服务提供对外的HTTP接口，这个时候需要绕过shiro的权限校验，直接访问接口，怎么办？

## 配置filter

第一步：在ApplicationContext-shiro.xml中配置filterChainDefinitions，在其中添加你需要过滤的路径。

如下代码：(注意，不要添加到最后一行/**之后，因为这个表示所有的，authc表示需要经过权限校验，anon表示匿名的，即不用校验权限)


```
<!-- Shiro Filter -->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	<property name="securityManager" ref="securityManager" />
	
	<property name="loginUrl" value="/" />
	
	<property name="successUrl" value="/main/index" />
	
	<property name="unauthorizedUrl" value="/login_toLogin" />

	<property name="filterChainDefinitions">
		<value>
		/static/login/** 			= anon
		/plugins/keypad/** 			= anon
		/static/js/myjs/** 			= authc
		/static/js/** 				= anon
		/uploadFiles/uploadImgs/** 	= anon
       	/code.do 					= anon
       	/login_login	 			= anon
       	/monitor/deviceinfo         = anon
		/app_deviceinfo/isActive    = anon
		/lot_omger/omger            = anon
       	/app**/** 					= anon
       	/weixin/** 					= anon
       	/api_lotdevicemonitorinfo/** = anon
		/versionAudit/getVersionState = anon
		/**							= authc
		</value>
	</property>
</bean>
```

## 访问拦截配置放行

第二步：在ApplicationContext-mvc.xml的LoginHandlerInterceptor类调用的Const.NO_INTERCEPTOR_PATH中，添加其过滤的常量：

ApplicationContext-mvc.xml:
```
<!-- 访问拦截  -->  
  <mvc:interceptors>
	<mvc:interceptor>
		<mvc:mapping path="/**/**"/>
		<bean class="com.hct.lot.interceptor.LoginHandlerInterceptor"/>
	</mvc:interceptor>
  </mvc:interceptors>
```
LoginHandlerInterceptor:
```
if(path.matches(Const.NO_INTERCEPTOR_PATH)){
	return true;//表示不拦截
}else{
    //拦截的处理
}
```

Const.NO_INTERCEPTOR_PATH：
```
static final String NO_INTERCEPTOR_PATH = ".*/((login)|(logout)|(code)|(app)|(weixin)|(static)|(main)|(websocket)|(deviceinfo)|(omger)|(getVersionState)|(isActive)).*";	//不对匹配该值的访问路径拦截（正则）
```

## 小结

搞定关键步骤，就可以轻松的实现放行了。

不过在确定接口是否需要放行，一定要业务上的严格把关，以及对接口进行幂等、压测，避免造成服务压力。