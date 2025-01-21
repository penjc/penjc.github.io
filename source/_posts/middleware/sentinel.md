---
title: Sentinel 限流与熔断
comment: true   # true/false对应开启/关闭本文章评论
date: 2024-12-27
tags:
  - Sentinel
  - 中间件
categories:
  - [中间件, Sentinel]
---

## **1. 引入 Sentinel**

在 Maven 项目中引入 Sentinel 的依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
    <version>1.8.6</version> <!-- 请根据需求选择具体版本 -->
</dependency>
```

如果使用 Spring Boot 或 Spring Cloud，可以添加以下依赖（Spring Cloud Alibaba 提供的支持）：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>2021.0.1.0</version> <!-- 请根据 Spring 版本选择对应依赖 -->
</dependency>
```

---

## **2. Sentinel 基础使用**

以下是 Sentinel 在基础 Java 项目中的代码示例：

### **(1) 基本限流示例**

在代码中保护某段逻辑：
```java
import com.alibaba.csp.sentinel.Entry;
import com.alibaba.csp.sentinel.SphU;
import com.alibaba.csp.sentinel.slots.block.BlockException;

public class SentinelBasicExample {
    public static void main(String[] args) {
        while (true) {
            try (Entry entry = SphU.entry("HelloWorldResource")) {
                // 被保护的逻辑
                System.out.println("Hello, Sentinel!");
            } catch (BlockException ex) {
                // 当触发限流或熔断时执行的逻辑
                System.out.println("Blocked by Sentinel!");
            }
        }
    }
}
```

### **(2) 配置限流规则**

配置资源的限流规则：
```java
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import com.alibaba.csp.sentinel.slots.block.RuleConstant;

import java.util.Collections;

public class SentinelRuleExample {
    public static void main(String[] args) {
        // 配置规则
        FlowRule rule = new FlowRule();
        rule.setResource("HelloWorldResource"); // 资源名需与代码中的一致
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS); // 基于 QPS 的限流
        rule.setCount(5); // 每秒最多允许 5 次访问

        // 加载规则
        FlowRuleManager.loadRules(Collections.singletonList(rule));

        // 调用资源
        while (true) {
            try (Entry entry = SphU.entry("HelloWorldResource")) {
                System.out.println("Request passed!");
            } catch (BlockException ex) {
                System.out.println("Blocked by Sentinel!");
            }
        }
    }
}
```

**运行逻辑：**
- 每秒最多允许 5 次请求通过。
- 超过 5 次时，触发限流逻辑。

---

## **3. 集成到 Spring Boot 项目**

如果你使用 Spring Boot，Sentinel 可以无缝集成。

### **(1) 添加依赖**
在 `pom.xml` 文件中引入 Spring Cloud Alibaba 的 Sentinel 依赖：
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    <version>2021.0.1.0</version>
</dependency>
```

### **(2) 配置文件**
在 `application.properties` 或 `application.yml` 中添加 Sentinel 配置：
```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8080  # Sentinel 控制台地址
        port: 8719                 # 应用与 Sentinel 控制台通信的端口
```

### **(3) 代码示例**

使用注解对某方法进行限流保护：
```java
import com.alibaba.csp.sentinel.annotation.SentinelResource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class SentinelDemoController {

    @GetMapping("/hello")
    @SentinelResource(value = "helloResource", blockHandler = "handleBlocked")
    public String hello() {
        return "Hello, Sentinel!";
    }

    // 限流时的处理方法
    public String handleBlocked(BlockException ex) {
        return "Request was blocked!";
    }
}
```

### **(4) 启动 Sentinel 控制台**
1. 下载 [Sentinel 控制台](https://github.com/alibaba/Sentinel/releases) 的 JAR 文件。
2. 使用以下命令启动：
   ```bash
   java -jar sentinel-dashboard-1.8.6.jar
   ```
3. 默认访问地址为 [http://localhost:8080](http://localhost:8080)，登录用户名和密码均为 `sentinel`。

在控制台可以动态配置规则。

---

## **4. 高级场景示例**

### **(1) 熔断降级**

熔断规则根据失败率或响应时间触发：
```java
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRule;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRuleManager;
import com.alibaba.csp.sentinel.slots.block.RuleConstant;

import java.util.Collections;

public class DegradeExample {
    public static void main(String[] args) {
        // 配置熔断规则
        DegradeRule rule = new DegradeRule();
        rule.setResource("TestDegradeResource");
        rule.setGrade(RuleConstant.DEGRADE_GRADE_RT); // 基于响应时间的熔断
        rule.setCount(50); // RT 超过 50 毫秒触发熔断
        rule.setTimeWindow(10); // 熔断持续时间为 10 秒

        DegradeRuleManager.loadRules(Collections.singletonList(rule));

        // 调用资源
        while (true) {
            try (Entry entry = SphU.entry("TestDegradeResource")) {
                Thread.sleep(100); // 模拟耗时
                System.out.println("Request passed!");
            } catch (BlockException ex) {
                System.out.println("Blocked by Sentinel!");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### **(2) 热点参数限流**

对请求中的特定参数限流：
```java
import com.alibaba.csp.sentinel.Entry;
import com.alibaba.csp.sentinel.SphU;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowItem;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowRuleManager;

import java.util.Collections;

public class ParamFlowExample {
    public static void main(String[] args) {
        // 配置热点参数限流规则
        ParamFlowRule rule = new ParamFlowRule("TestHotspot")
                .setParamIdx(0) // 限制第一个参数
                .setCount(5); // 每秒最多允许 5 次相同参数值

        ParamFlowRuleManager.loadRules(Collections.singletonList(rule));

        // 调用资源
        for (int i = 0; i < 10; i++) {
            int param = i % 2; // 模拟请求参数
            try (Entry entry = SphU.entry("TestHotspot", () -> new Object[] {param})) {
                System.out.println("Passed with param: " + param);
            } catch (BlockException ex) {
                System.out.println("Blocked with param: " + param);
            }
        }
    }
}
```

---

## **5. 总结**

1. Sentinel 是一个功能强大的流控与熔断工具，适用于各种分布式场景。
2. 提供了丰富的 API、注解以及控制台支持，便于快速集成和动态调整。
3. 实际应用中可以结合 Spring Cloud、微服务架构进行全局的服务保护。
