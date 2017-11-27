---
title: web.xml详解
permalink: web-xml
date: 2016-10-24 10:10:45
categories:
- 后端
tags:
- java
- javaee
---
web.xml文件是用来配置:欢迎页、servlet、filter、listener等的. 当你的web项目工程没用到这些时,你可以不用web.xml文件来配置你的web工程。
如果项目中有多项标签,其加载顺序依次是:context-param >> listener >> filter >> servlet(同类多个节点出现顺序依次加载)
<!--more -->
1. ​web.xml先读取context-param和listener这两种节点；
2. 然后容器创建一个ServletContext(上下文)，应用于整个项目；
3. 容器会将读取到的context-param转化为键值对并存入servletContext；
4. 根据listener创建监听；
5. 容器会读取，根据指定的类路径来实例化过滤器；
6. 此时项目初始化完成；
7. 在发起第一次请求是，servlet节点才会被加载实例化。

## 参数设置
### context-param
context-param节点是web.xml中用于配置应用于整个web项目的​上下文。包括两个子节点，其中param-name 设定上下文的参数名称。必须是唯一名称；param-value 设定的参数名称的值。

读取节点的方法如下：
```xml
${initParam.参数名}
```
Servlet中String paramValue=getServletContext().getInitParameter("参数名")​

web.xml中配置spring必须使用listener节点，但context-param节点可有可无，如果缺省则默认contextConfigLocation路径为“/WEB-INF/applicationContext.xml”；如果有多个xml文件，使用”,“分隔

### listener
```xml
<listener>
    <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
</listener>
```
为web应用程序定义监听器，监听器用来监听各种事件，比如：application和session事件，所有的监听器按照相同的方式定义，功能取决去它们各自实现的接口，常用的Web事件接口有如下几个：
`ServletContextListener`：用于监听Web应用的启动和关闭；
`ServletContextAttributeListener`：用于监听ServletContext范围（application）内属性的改变；
`ServletRequestListener`：用于监听用户的请求；
`ServletRequestAttributeListener`：用于监听ServletRequest范围（request）内属性的改变；
`HttpSessionListener`：用于监听用户session的开始和结束；
`HttpSessionAttributeListener`：用于监听HttpSession范围（session）内属性的改变。

配置Listener只要向Web应用注册Listener实现类即可，无序配置参数之类的东西，因为Listener获取的是Web应用ServletContext（application）的配置参数。为Web应用配置Listener的两种方式：
1. 使用@WebListener修饰Listener实现类即可。
2. 在web.xml文档中使用进行配置。

### servlet
servlet即配置所需用的servlet，用于处理及响应客户的请求。容器的Context对象对请求路径(URL)做出处理，去掉请求URL的上下文路径后，按路径映射规则和Servlet映射路径（）做匹配，如果匹配成功，则调用这个Servlet处理请求。
#### 为Servlet命名：
```xml
<servlet>
  <servlet-name>servlet</servlet-name>
  <servlet-class>org.whatisjava.TestServlet</servlet-class>
  <load-on-startup>1</load-on-startup>
</servlet>
```

#### 为Servlet定制URL
```xml
<servlet-mapping>
<servlet-name>servlet</servlet-name>
<url-pattern>*.do</url-pattern>
</servlet-mapping>
```
#### Load-on-startup
Load-on-startup 元素在web应用启动的时候指定了servlet被加载的顺序，它的值必须是一个整数。如果它的值是一个负整数或是这个元素不存在，那么容器会在该servlet被调用的时候，加载这个servlet 。如果值是正整数或零，容器在配置的时候就加载并初始化这个servlet，容器必须保证值小的先被加载。如果值相等，容器可以自动选择先加载谁。
当值为0或者大于0时，表示容器在应用启动时就加载这个servlet；
当是一个负数时或者没有指定时，则指示容器在该servlet被选择时才加载。
正数的值越小，启动该servlet的优先级越高。

### filter
设置过滤器:如编码过滤器,过滤所有资源
```xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
### session
设置会话(Session)过期时间,其中时间以分钟为单位,加入设置60分超时:
```xml
<session-config>
    <session-timeout>60</session-timeout>
</session-config>
```
### welcom-file-list
```xml
<welcome-file-list>
  <welcome-file>index.jsp</welcome-file>
  <welcome-file>index.html</welcome-file>
</welcome-file-list>
```
>PS:指定了两个欢迎页面,显示时按顺序从第一个找起，如果第一个存在，就显示第一个，后面的不起作用。如果第一个不存在，就找第二个，以此类推。如果都没有就404.

#### 关于欢迎页面：
访问一个网站时，默认看到的第一个页面就叫欢迎页，一般情况下是由首页来充当欢迎页的。一般情况下，我们会在web.xml中指定欢迎页。但 web.xml并不是一个Web的必要文件，没有web.xml，网站仍然是可以正常工作的。只不过网站的功能复杂起来后，web.xml的确有非常大用处，所以，默认创建的动态web工程在WEB-INF文件夹下面都有一个web.xml文件。

### error-page
```xml
<!-- 错误码 -->
<error-page>
    <error-code>404</error-code>
    <location>/error404.jsp</location>
</error-page>
<!-- 错误类型 -->
<error-page>
    <exception-type>java.lang.Exception<exception-type>
    <location>/exception.jsp<location>
</error-page>
```
