---
title: 过滤器和拦截器
comment: true   # true/false对应开启/关闭本文章评论
date: 2024-02-04
tags:
  - JAVA
categories:
  - [JAVA]
---

# Java 过滤器与拦截器详解

在 Java Web 开发中，**过滤器**和**拦截器**是两种常用的机制，分别用于对 HTTP 请求和响应进行处理。它们虽然有些相似，但用途和实现方式存在一些关键区别。

---

# 📘 过滤器（Filter）

在 Java 项目中，使用过滤器（Filter）的流程主要包括以下步骤：

---

## **1. 创建过滤器类**
过滤器需要实现 `javax.servlet.Filter` 接口，并重写其中的三个方法：
- `init(FilterConfig filterConfig)`：用于初始化过滤器。
- `doFilter(ServletRequest request, ServletResponse response, FilterChain chain)`：核心逻辑处理方法。
- `destroy()`：用于销毁过滤器实例。

### 示例代码
```java
import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter("/*") // 匹配所有路径
public class MyFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("过滤器初始化");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("请求到达过滤器 - 前置处理");

        // 继续调用下一个过滤器或目标资源（如 Servlet）
        chain.doFilter(request, response);

        System.out.println("响应经过过滤器 - 后置处理");
    }

    @Override
    public void destroy() {
        System.out.println("过滤器销毁");
    }
}
```

---

## **2. 注册过滤器**

### **方法一：使用注解**
自 Servlet 3.0 开始，支持使用 `@WebFilter` 注解注册过滤器。注解的参数包括：
- `urlPatterns`：过滤器匹配的 URL 模式。
- `filterName`：过滤器名称（可选）。
- `dispatcherTypes`：指定过滤器适用的 Dispatcher 类型（如 `REQUEST` 或 `FORWARD`）。

```java
@WebFilter(filterName = "MyFilter", urlPatterns = "/*")
public class MyFilter implements Filter {
    // 实现方法同上
}
```

这种方式简单直观，适用于轻量级配置。

---

### **方法二：通过 `FilterRegistrationBean` 注册（Spring 项目中）**

在 Spring 项目中，过滤器通常通过配置类注册，使用 `FilterRegistrationBean`：

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<MyFilter> registerMyFilter() {
        FilterRegistrationBean<MyFilter> registration = new FilterRegistrationBean<>();
        
        // 设置过滤器
        registration.setFilter(new MyFilter());
        
        // 设置过滤路径
        registration.addUrlPatterns("/*");
        
        // 设置过滤器优先级（数字越小优先级越高）
        registration.setOrder(1);

        return registration;
    }
}
```

使用 `FilterRegistrationBean` 可以灵活配置过滤器，例如动态启用/禁用过滤器、注入依赖等。

---

### **方法三：配置在 `web.xml` 文件中**

在传统的 Servlet 容器中，可以通过 `web.xml` 配置过滤器：

```xml
<filter>
    <filter-name>MyFilter</filter-name>
    <filter-class>com.example.MyFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>MyFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

这种方式适用于老版本的 Servlet 容器。

---

## **3. 使用过滤器处理请求和响应**

在过滤器中，可以对请求和响应进行以下操作：

1. **修改请求内容**：
    - 可以读取和修改请求参数，例如添加或移除参数。

2. **设置响应内容**：
    - 在响应返回给客户端之前，可以添加自定义头部、修改内容等。

### 示例：添加自定义头部
```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    HttpServletResponse httpResponse = (HttpServletResponse) response;

    // 添加自定义响应头
    httpResponse.setHeader("X-Custom-Header", "MyHeaderValue");

    chain.doFilter(request, response);
}
```

3. **身份验证**：
    - 检查用户是否登录，未登录时直接返回错误响应。

### 示例：身份验证过滤器
```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    HttpServletRequest httpRequest = (HttpServletRequest) request;
    HttpServletResponse httpResponse = (HttpServletResponse) response;

    String authToken = httpRequest.getHeader("Authorization");

    if (authToken == null || !authToken.equals("valid-token")) {
        httpResponse.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized");
        return;
    }

    chain.doFilter(request, response);
}
```

---

## **4. 控制过滤器执行顺序**

如果有多个过滤器，可以通过优先级（`order`）或 `web.xml` 配置的顺序来控制执行顺序。

### 示例：设置优先级
```java
// 优先级较高的过滤器
@Bean
public FilterRegistrationBean<FirstFilter> firstFilter() {
    FilterRegistrationBean<FirstFilter> registration = new FilterRegistrationBean<>();
    registration.setFilter(new FirstFilter());
    registration.addUrlPatterns("/*");
    registration.setOrder(1); // 优先级最高
    return registration;
}

// 优先级较低的过滤器
@Bean
public FilterRegistrationBean<SecondFilter> secondFilter() {
    FilterRegistrationBean<SecondFilter> registration = new FilterRegistrationBean<>();
    registration.setFilter(new SecondFilter());
    registration.addUrlPatterns("/*");
    registration.setOrder(2); // 优先级较低
    return registration;
}
```

---

## **5. 测试过滤器**

### 测试方法：
1. 运行应用程序。
2. 发起 HTTP 请求（例如通过浏览器、Postman、curl）。
3. 查看请求和响应，观察过滤器是否按照预期处理。

---

# 📘 拦截器（Interceptor）

**定义**：
拦截器（`Interceptor`）是一种用于横切关注点的技术，通常与 AOP（面向切面编程）配合使用。它用于对特定方法进行前置、后置处理，不同于过滤器，它更多的是操作业务逻辑。

**作用**：
拦截器主要用于：
- **业务方法的前后处理**：在方法执行之前或之后进行某些操作。
- **方法性能监控**：记录方法执行的时间等信息。
- **事务管理**：在方法执行之前或之后开启、提交或回滚事务。

**实现**：
拦截器通常通过实现 `javax.interceptor.Interceptor` 接口或继承 `HandlerInterceptor` 来实现。

在 Spring 框架中，拦截器通常通过实现 `HandlerInterceptor` 接口来创建，常见方法有：
- `preHandle()`：在请求处理之前执行。
- `postHandle()`：在请求处理之后，视图渲染之前执行。
- `afterCompletion()`：在整个请求处理完毕后执行。

**配置**：
拦截器一般通过 Spring 配置类来进行配置，或者通过注解的方式指定。

```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
        throws Exception {
        // 处理请求前
        System.out.println("Pre-Handle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
        ModelAndView modelAndView) throws Exception {
        // 处理请求后
        System.out.println("Post-Handle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
        Exception ex) throws Exception {
        // 请求完成后
        System.out.println("After Completion");
    }
}
```

---

# 📊 过滤器与拦截器的区别

| **特性**           | **过滤器（Filter）**                                     | **拦截器（Interceptor）**                                      |
|--------------------|-----------------------------------------------------|---------------------------------------------------------------|
| **定义**           | 过滤器是基于 Servlet 技术的请求处理机制，专注于请求和响应的预处理和后处理。          | 拦截器通常是面向 AOP 的技术，专注于方法调用的前后处理。              |
| **工作范围**       | 过滤器作用于 HTTP 请求和响应流的处理。                              | 拦截器作用于 Java 方法的调用，通常与业务逻辑密切相关。              |
| **适用场景**       | 请求和响应流的过滤、日志记录、身份验证、性能监控等。                          | 业务方法的拦截、事务管理、缓存管理、权限验证等。                    |
| **配置方式**       | 配置在 `web.xml` 或使用 `@WebFilter` 注解。需要手动注册到 Spring 容器 | 配置在 Spring 配置类中，或使用 `@Interceptor` 注解（在 Java EE 中）。|
| **执行顺序**       | 过滤器按配置的顺序执行，适合在请求到达 Servlet 或响应离开之前进行全局处理。          | 拦截器在方法调用前后执行，适合在业务逻辑执行之前或之后进行处理。    |
| **处理逻辑**       | 处理请求和响应的底层数据，例如修改请求或响应的内容。                          | 处理业务方法的执行，例如修改方法参数或返回值。                    |

---

## 💬 总结

- **过滤器**更多关注 HTTP 请求和响应的处理，适用于横切关注点（如日志记录、性能监控、过滤等），并且与 Servlet 生命周期紧密相关。
- **拦截器**主要用于方法调用的前后处理，更加专注于业务逻辑，适用于事务控制、权限验证等横切业务逻辑的任务。

根据你的项目需求，选择适合的技术来处理不同的横切关注点。

