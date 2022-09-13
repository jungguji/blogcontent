---
layout: posts
title: >-
  Neither BindingResult nor plain target object for bean name command available
  as request attribute
date: 2022-09-09 16:24:22
tags:
  - Spring
  - JSP
---

## 발생 경위

기존 Spring 4.3.3을 사용하던 프로젝트를 5.3.22으로 업그레이드 이후 발생하였다.

### 에러로그 전체

```java
java.lang.IllegalStateException: Neither BindingResult nor plain target object for bean name 'command' available as request attribute
	at org.springframework.web.servlet.support.BindStatus.<init>(BindStatus.java:153)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.getBindStatus(AbstractDataBoundFormElementTag.java:178)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.getPropertyPath(AbstractDataBoundFormElementTag.java:199)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.getName(AbstractDataBoundFormElementTag.java:164)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.writeDefaultAttributes(AbstractDataBoundFormElementTag.java:123)
	at org.springframework.web.servlet.tags.form.AbstractHtmlElementTag.writeDefaultAttributes(AbstractHtmlElementTag.java:460)
	at org.springframework.web.servlet.tags.form.SelectTag.writeTagContent(SelectTag.java:405)
	at org.springframework.web.servlet.tags.form.AbstractFormTag.doStartTagInternal(AbstractFormTag.java:87)
	at org.springframework.web.servlet.tags.RequestContextAwareTag.doStartTag(RequestContextAwareTag.java:83)
	at org.apache.jsp.WEB_002dINF.xxxx.xxxx.xxxx_jsp._jspService(new_jsp.java:263)
	at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:476)
	at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:386)
	at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:330)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.core.ApplicationDispatcher.invoke(ApplicationDispatcher.java:728)
	at org.apache.catalina.core.ApplicationDispatcher.doInclude(ApplicationDispatcher.java:591)
	at org.apache.catalina.core.ApplicationDispatcher.include(ApplicationDispatcher.java:527)
	at org.apache.jasper.runtime.JspRuntimeLibrary.include(JspRuntimeLibrary.java:868)
	at org.apache.jasper.runtime.PageContextImpl.doInclude(PageContextImpl.java:679)
	at org.apache.jasper.runtime.PageContextImpl.include(PageContextImpl.java:673)
	at org.apache.tiles.request.jsp.JspRequest.doInclude(JspRequest.java:123)
	at org.apache.tiles.request.AbstractViewRequest.dispatch(AbstractViewRequest.java:47)
	at org.apache.tiles.request.render.DispatchRenderer.render(DispatchRenderer.java:45)
	at org.apache.tiles.request.render.ChainedDelegateRenderer.render(ChainedDelegateRenderer.java:68)
	at org.apache.tiles.impl.BasicTilesContainer.render(BasicTilesContainer.java:259)
	at org.apache.tiles.template.InsertAttributeModel.renderAttribute(InsertAttributeModel.java:188)
	at org.apache.tiles.template.InsertAttributeModel.execute(InsertAttributeModel.java:132)
	at org.apache.tiles.jsp.taglib.InsertAttributeTag.doTag(InsertAttributeTag.java:299)
	at org.apache.jsp.WEB_002dINF.xxxx.xxxx.xxxx_jsp._jspx_meth_t_005finsertAttribute_005f4(layout_jsp.java:10881)
	at org.apache.jsp.WEB_002dINF.xxxx.xxxx.xxxx_jsp._jspService(layout_jsp.java:1242)
	at org.apache.jasper.runtime.HttpJspBase.service(HttpJspBase.java:70)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.jasper.servlet.JspServletWrapper.service(JspServletWrapper.java:476)
	at org.apache.jasper.servlet.JspServlet.serviceJspFile(JspServlet.java:386)
	at org.apache.jasper.servlet.JspServlet.service(JspServlet.java:330)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.core.ApplicationDispatcher.invoke(ApplicationDispatcher.java:728)
	at org.apache.catalina.core.ApplicationDispatcher.processRequest(ApplicationDispatcher.java:470)
	at org.apache.catalina.core.ApplicationDispatcher.doForward(ApplicationDispatcher.java:395)
	at org.apache.catalina.core.ApplicationDispatcher.forward(ApplicationDispatcher.java:316)
	at org.apache.tiles.request.servlet.ServletRequest.forward(ServletRequest.java:265)
	at org.apache.tiles.request.servlet.ServletRequest.doForward(ServletRequest.java:228)
	at org.apache.tiles.request.AbstractClientRequest.dispatch(AbstractClientRequest.java:57)
	at org.apache.tiles.request.render.DispatchRenderer.render(DispatchRenderer.java:45)
	at org.apache.tiles.impl.BasicTilesContainer.render(BasicTilesContainer.java:259)
	at org.apache.tiles.impl.BasicTilesContainer.render(BasicTilesContainer.java:397)
	at org.apache.tiles.impl.BasicTilesContainer.render(BasicTilesContainer.java:238)
	at org.apache.tiles.impl.BasicTilesContainer.render(BasicTilesContainer.java:221)
	at org.apache.tiles.renderer.DefinitionRenderer.render(DefinitionRenderer.java:59)
	at org.springframework.web.servlet.view.tiles3.TilesView.renderMergedOutputModel(TilesView.java:147)
	at org.springframework.web.servlet.view.AbstractView.render(AbstractView.java:316)
	at org.springframework.web.servlet.DispatcherServlet.render(DispatcherServlet.java:1404)
	at org.springframework.web.servlet.DispatcherServlet.processDispatchResult(DispatcherServlet.java:1148)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1087)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:963)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:898)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:634)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:741)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:231)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.filters.ExpiresFilter.doFilter(ExpiresFilter.java:1228)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter.doFilterInternal(OpenEntityManagerInViewFilter.java:186)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:94)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:330)
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityInterceptor.java:118)
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityInterceptor.java:84)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:113)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:103)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthenticationFilter.java:113)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.authentication.rememberme.RememberMeAuthenticationFilter.doFilter(RememberMeAuthenticationFilter.java:146)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContextHolderAwareRequestFilter.java:154)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.java:45)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:199)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:199)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:110)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:57)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:87)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(WebAsyncManagerIntegrationFilter.java:50)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:342)
	at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:192)
	at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:160)
	at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:354)
	at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:267)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117)
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:193)
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:166)
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:199)
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:96)
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:543)
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:139)
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:81)
	at org.apache.catalina.valves.AbstractAccessLogValve.invoke(AbstractAccessLogValve.java:678)
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:87)
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343)
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:609)
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65)
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:810)
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1623)
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
	at java.base/java.lang.Thread.run(Thread.java:829)
```

* * *

## 해결 과정


### 에러로그 일부 
```java
	
  ...

java.lang.IllegalStateException: Neither BindingResult nor plain target object for bean name 'command' available as request attribute
  at org.springframework.web.servlet.support.BindStatus.<init>(BindStatus.java:153)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.getBindStatus(AbstractDataBoundFormElementTag.java:178)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.getPropertyPath(AbstractDataBoundFormElementTag.java:199)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.getName(AbstractDataBoundFormElementTag.java:164)
	at org.springframework.web.servlet.tags.form.AbstractDataBoundFormElementTag.writeDefaultAttributes(AbstractDataBoundFormElementTag.java:123)
	at org.springframework.web.servlet.tags.form.AbstractHtmlElementTag.writeDefaultAttributes(AbstractHtmlElementTag.java:460)
	at org.springframework.web.servlet.tags.form.SelectTag.writeTagContent(SelectTag.java:405)
	at org.springframework.web.servlet.tags.form.AbstractFormTag.doStartTagInternal(AbstractFormTag.java:87)
	at org.springframework.web.servlet.tags.RequestContextAwareTag.doStartTag(RequestContextAwareTag.java:83)
	at org.apache.jsp.WEB_002dINF.xxxx.xxxx.xxxx_jsp._jspService(new_jsp.java:263)

  ....

```

<br />
> Neither BindingResult nor plain target object for bean name 'command' available as request attribute

<br />

짧은 영어 실력과 구글 번역기에 도움으로 해석해보자면 BindingResult이나 'command' 이라는 이름의 bean 을 request의 속성으로 사용할 수 없어서 발생하는 에러인 것 같은데,
에러 로그를 확인 한 결과... 위에서 보이는 것 처럼, JSP의 Form Tag에서 에러가 발생하는 것으로 추측되어 4.3.3 version에서 Form Tag를 구현하는 [FormTag](https://docs.spring.io/spring-framework/docs/4.3.3.RELEASE/javadoc-api/org/springframework/web/servlet/tags/form/FormTag.html#setCommandName-java.lang.String-)를 확인해보니 commandName 관련된 메서드들이 @Deprecated 로 선언되어서 곧 없어진다고 알려주고 있었는데, 이번에 업그레이드한 5.3.22 version의 FormTag에선 commandName의 getter, setter 메서드들이 없는 것을 확인할 수 있었다.

{% asset_img "4.3.3_FormTag.png" "4.3.3 version의 CommandName 관련 method" %}

<br />

그리고 이미지를 보면 알 수 있듯이 기존의 setCommandName 메서드에서도 modelAttribute에 commandName을 저장해서 쓰는 것을 확인할 수 있는데, 5.3.22 에선 해당 메서드가 없어지면서 변수 modelAttribute에 'command'가 할당되어 있기에, 'command' 라는 bean을 속성으로 사용할 수 없다고 에러가 발생하는 것이었다.

{% asset_img "default_modelAttribute.png" "modelAttribute 변수의 기본값" %}

<br />

{% asset_img "getModelAttribute.png" "Form tag 호출 시 불리는 modelAttribute의 값" %}

<br />

이렇게 된 것이니 당연하게도 해결 방법은 modelAttribute에 우리가 지정한 속성값이 set 되도록 하면 되는데, 위에서 본 것처럼 기존의 setCommandName에서도 modelAttribute의 값을 사용하고 있었으므로, setModelAttribute로 modelAttribute 값이 저장되도록 하면 문제는 해결될 것이기에, JSP form tag에 commandName 속성을 모두 modelAttribute로 변경한 후 확인해 보면 정상적으로 동작 하는 것을 확인할 수 있을 것이다.

{% asset_img "setModelAttribute.png" "commandName 속성을 ModelAttribute로 변경 후 Form tag 호출 시 불리는 modelAttribute의 값" %}

<br />

{% asset_img "getModelAttribute_2.png" "commandName 속성을 ModelAttribute로 변경 후 Form tag 호출 시 불리는 modelAttribute의 값" %}
* * *

## 참고 사이트

- [https://web-obj.tistory.com/478](https://web-obj.tistory.com/478)
- [https://docs.spring.io/spring-framework/docs/4.3.3.RELEASE/javadoc-api/org/springframework/web/servlet/tags/form/FormTag.html#setCommandName-java.lang.String-](https://docs.spring.io/spring-framework/docs/4.3.3.RELEASE/javadoc-api/org/springframework/web/servlet/tags/form/FormTag.html#setCommandName-java.lang.String-)