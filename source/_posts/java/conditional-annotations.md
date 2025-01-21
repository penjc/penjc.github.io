---
title: 条件注解
comment: true   # true/false对应开启/关闭本文章评论
date: 2024-02-09
tags:
  - JAVA
categories:
  - [JAVA]
---

## **条件注解（Conditional Annotations）**

条件注解（Conditional Annotations）是 Spring Framework 中的一种机制，允许根据特定的条件动态地加载 Bean 或配置。通过使用条件注解，开发者可以灵活地控制某些组件是否被加载到 Spring 容器中。

---

## **常见的条件注解**

### 1. **@Conditional**
`@Conditional` 是一个通用的条件注解，可以结合自定义的 `Condition` 实现类来定义复杂的条件逻辑。

#### 使用方法：
```java
@Configuration
public class MyConfig {

    @Bean
    @Conditional(MyCondition.class)
    public MyService myService() {
        return new MyService();
    }
}
```

#### 自定义条件类：
```java
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 自定义条件逻辑
        String property = context.getEnvironment().getProperty("my.custom.property");
        return "enabled".equalsIgnoreCase(property);
    }
}
```

#### 说明：
- **`Condition` 接口**：定义了条件逻辑，通过 `matches` 方法决定是否加载对应的 Bean。
- **条件逻辑来源**：可以基于环境变量、系统属性、类路径中是否存在特定类等条件。

---

### 2. **@ConditionalOnProperty**
`@ConditionalOnProperty` 是 Spring Boot 提供的条件注解，专门用于基于配置属性值的条件判断。

#### 示例：
```java
@Bean
@ConditionalOnProperty(name = "feature.toggle", havingValue = "true", matchIfMissing = false)
public MyFeatureService myFeatureService() {
    return new MyFeatureService();
}
```

#### 参数说明：
- **`name`**：指定需要检查的属性名称。
- **`havingValue`**：指定属性值需要等于什么时加载 Bean。
- **`matchIfMissing`**：当属性缺失时，是否匹配（默认为 `false`）。

#### 示例用途：
如果在配置文件中有如下内容：
```properties
feature.toggle=true
```
则 `myFeatureService` 会被加载；否则不会被加载。

---

### 3. **@ConditionalOnClass**
`@ConditionalOnClass` 根据类路径中是否存在特定类来决定是否加载 Bean。

#### 示例：
```java
@Bean
@ConditionalOnClass(name = "com.fasterxml.jackson.databind.ObjectMapper")
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

#### 用途：
- 如果当前类路径中存在 `ObjectMapper`（例如 Jackson 库），则加载该 Bean。
- 常用于判断依赖是否可用。

---

### 4. **@ConditionalOnMissingClass**
`@ConditionalOnMissingClass` 是 `@ConditionalOnClass` 的反向逻辑。

#### 示例：
```java
@Bean
@ConditionalOnMissingClass(value = "com.fasterxml.jackson.databind.ObjectMapper")
public FallbackService fallbackService() {
    return new FallbackService();
}
```

#### 用途：
- 如果类路径中缺少 `ObjectMapper`，则加载 `FallbackService`。

---

### 5. **@ConditionalOnBean**
`@ConditionalOnBean` 用于判断 Spring 容器中是否存在特定类型的 Bean。

#### 示例：
```java
@Bean
@ConditionalOnBean(name = "myDependency")
public MyService myService() {
    return new MyService();
}
```

#### 用途：
- 如果 Spring 容器中已经加载了名为 `myDependency` 的 Bean，则加载 `myService`。

---

### 6. **@ConditionalOnMissingBean**
`@ConditionalOnMissingBean` 与 `@ConditionalOnBean` 的逻辑相反。

#### 示例：
```java
@Bean
@ConditionalOnMissingBean(MyService.class)
public MyService defaultMyService() {
    return new MyService();
}
```

#### 用途：
- 如果 Spring 容器中尚未加载 `MyService` 类型的 Bean，则加载 `defaultMyService`。

---

### 7. **@ConditionalOnExpression**
`@ConditionalOnExpression` 基于 Spring 表达式语言（SpEL）进行条件判断。

#### 示例：
```java
@Bean
@ConditionalOnExpression("${feature.toggle} == true")
public MyFeatureService myFeatureService() {
    return new MyFeatureService();
}
```

#### 用途：
- 允许通过更复杂的逻辑控制组件加载。

---

### **条件注解的工作原理**

1. Spring 在扫描 Bean 时，检查是否有条件注解。
2. 根据条件注解的类型，Spring 动态判断是否满足条件：
    - 例如检查属性值（`@ConditionalOnProperty`）。
    - 检查类路径中的依赖（`@ConditionalOnClass`）。
    - 检查容器中是否存在某个 Bean（`@ConditionalOnBean`）。
3. 如果条件成立，则加载该 Bean；否则跳过加载。

---

### **条件注解的应用场景**

- **模块化功能开关**：
    - 通过配置文件启用或禁用某些功能模块。
    - 使用 `@ConditionalOnProperty` 控制功能是否加载。

- **依赖判断**：
    - 仅在某些第三方库存在时加载相关功能。
    - 使用 `@ConditionalOnClass` 实现。

- **防止 Bean 冲突**：
    - 确保某个 Bean 只会在未定义时加载。
    - 使用 `@ConditionalOnMissingBean`。

---

## **总结**
条件注解提供了一种灵活的机制，允许在运行时根据环境和配置动态加载组件。它们是 Spring 和 Spring Boot 项目中实现模块化、配置驱动开发的重要工具。根据具体需求选择合适的条件注解，可以显著提升项目的可配置性和扩展性。