## 一、Sentinel 简介

Sentinel 是阿里巴巴开源的分布式系统流量控制组件，主要以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度来保障微服务的稳定性。

## 二、Sentinel 核心概念

- **资源**：是 Sentinel 保护的对象，可以是一段代码、一个方法、一个服务接口等。
- **规则**：围绕资源制定的控制策略，包括流量控制规则、熔断降级规则、系统保护规则等。
- **插槽链（Slot Chain）**：Sentinel 的核心骨架，不同的插槽负责不同的功能，如限流、降级、统计等，插槽按顺序执行形成链式结构。

## 三、环境配置

### 依赖引入

在 Maven 项目中，添加如下依赖：
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
</dependency>
```

## 四、流量控制规则配置

### 代码配置方式

通过 Java 代码直接配置流量控制规则，示例如下：
```java
import com.alibaba.csp.sentinel.slots.block.flow.FlowRule;
import com.alibaba.csp.sentinel.slots.block.flow.FlowRuleManager;
import java.util.Collections;

public class SentinelFlowRuleConfig implements InitializingBean {
    public static void initFlowRule() {
        // 创建流量控制规则对象
        List<FlowRule> rules = new ArrayList<>();
        FlowRule rule = new FlowRule();
        // 设置资源名称，对应要保护的服务接口或逻辑
        rule.setResource("yourResourceName");
        // 限流类型，0表示根据QPS限流，1表示根据并发线程数限流，这里设置为QPS限流
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        // 限流阈值，例如每秒请求数限制为10
        rule.setCount(10);
        // 限流应用的名称，默认是"default"
        rule.setLimitApp("default");
        // 控制行为，0表示拒绝请求并抛出异常，这里采用默认拒绝策略
        rule.setControlBehavior(RuleConstant.CONTROL_BEHAVIOR_THROW_EXCEPTION);

        // 加载规则
        FlowRuleManager.loadRules(Collections.singletonList(rule));
    }
}
```
### 常见参数说明
- **resource**：资源名称，必须与要保护的资源对应，例如某个接口方法名或 URL 等。
- **grade**：限流的类型，`RuleConstant.FLOW_GRADE_QPS`表示按每秒请求数限流，`RuleConstant.FLOW_GRADE_THREAD`表示按并发线程数限流。
- **count**：限流的阈值，即允许的最大 QPS 或最大并发线程数。
- **limitApp**：限流应用的名称，用于区分不同来源的请求，默认`default`表示不区分来源。
- **controlBehavior**：控制行为，除了`CONTROL_BEHAVIOR_THROW_EXCEPTION`（抛出异常），还可以是`CONTROL_BEHAVIOR_WARM_UP`（预热模式）、`CONTROL_BEHAVIOR_RATE_LIMITER`（匀速排队模式）等。
## 五、熔断降级规则配置

### 代码配置方式
```java
import com.alibaba.csp.sentinel.slots.block.RuleConstant;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRule;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeRuleManager;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import java.util.ArrayList;
import java.util.List;

/**
 * 降级规则配置类
 */
@Configuration
public class DegradeRuleConfig {

    @PostConstruct
    public void initDegradeRules() {
        List<DegradeRule> rules = new ArrayList<>();
        
        // 1. 针对用户登录接口配置降级规则（异常比例）
        DegradeRule loginRule = new DegradeRule();
        loginRule.setResource("userLogin"); // 资源名，需与注解中的resourceName一致
        loginRule.setGrade(RuleConstant.DEGRADE_GRADE_EXCEPTION_RATIO); // 异常比例策略
        loginRule.setCount(0.5); // 异常比例阈值为50%
        loginRule.setMinRequestAmount(10); // 最小请求数为10
        loginRule.setTimeWindow(30); // 降级时间窗口30秒
        rules.add(loginRule);
        
        // 2. 针对订单提交接口配置降级规则（慢调用）
        DegradeRule orderRule = new DegradeRule();
        orderRule.setResource("submitOrder");
        orderRule.setGrade(RuleConstant.DEGRADE_GRADE_SLOW_RT); // 慢调用策略
        orderRule.setCount(1000); // 慢调用阈值为1000ms
        orderRule.setMinRequestAmount(5); // 最小请求数为5
        orderRule.setSlowRatioThreshold(0.3); // 慢调用比例阈值为30%
        orderRule.setTimeWindow(20); // 降级时间窗口20秒
        rules.add(orderRule);
        
        // 加载规则
        DegradeRuleManager.loadRules(rules);
    }
}
```
### 使用示例（在业务方法上添加注解）

```java
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    /**
     * 用户登录接口，添加Sentinel保护和降级处理
     */
    @SentinelRiskControl(
        resourceName = "userLogin",
        riskControlUrl = "http://risk-control-service/api/v1/record",
        ruleType = "DEGRADE",
        fallbackValue = "{\"code\":503,\"message\":\"登录请求过于频繁，请稍后再试\",\"data\":null}"
    )
    @PostMapping("/login")
    public Result login(
            @RequestParam String username,
            @RequestParam String password) {
        
        // 业务逻辑实现
        if ("admin".equals(username) && "123456".equals(password)) {
            return Result.success("登录成功", "token_xxx");
        } else {
            return Result.fail("用户名或密码错误");
        }
    }
}

@PostMapping("/api/short-link/v1/create")
@SentinelResource(
        value = "create_short-link",
        blockHandler = "createShortLinkBlockHandlerMethod",
        blockHandlerClass = CustomBlockHandler.class
)
public Result<ShortLinkCreateRespDTO> createShortLink(@RequestBody ShortLinkCreateReqDTO requestParam) {
    return Results.success(shortLinkService.createShortLink(requestParam));
}
```