---
layout: post
title: "Zuul网关实战02"
description: "Zuul 实战"
category: Tech
tags: [Zuul SpringCloud]
---



# Zuul网关实战02


## 网关限流 之 sentinel：

pom.xml加入：

```xml
<!-- sentinel 限流 alibaba -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-core</artifactId>
    <version>1.8.0</version>
</dependency>
```



添加初始化代码CloudZuulApplication：

```java
@SpringBootApplication
@EnableZuulProxy
public class CloudZuulApplication {

    public static void main(String[] args) {
        init();
        SpringApplication.run(CloudZuulApplication.class, args);
    }

    /**
     * 初始化 alibaba sentinel限流
     */
    public static void init(){
        // 所有限流规则的合集
        List<FlowRule> rules = new ArrayList<>();

        FlowRule rule = new FlowRule();
        // 资源名称
        rule.setResource("HelloWorld");

        // 限流的类型，这里是QPS
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);

        // 2 QPS
        rule.setCount(2);

        rules.add(rule);

        FlowRuleManager.loadRules(rules);
    }
}
```



自定义Filter：

```java
/**
 * alibaba sentinel 限流过滤器
 * zuul yml 需要打开路径： /forword1/**
 * 访问测试 http://localhost:9100/forword1/
 */
@Component
public class SentinelFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return -10;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        Entry entry = null;

        try {
            entry = SphU.entry("HelloWorld");
            // 业务逻辑
            System.out.println("正常请求");

        } catch (BlockException e) {
            System.out.println("被阻塞住了！！！");
            // 阻塞的业务逻辑
        } finally {
            if (entry != null) {
                entry.exit();
            }
        }

        return null;
    }
}
```



访问网关Zuul测试 http://localhost:9100/forword1/

多F5刷新几次，或者使用Jmeter压测看效果



项目代码地址：

https://github.com/kennedy-han/grey-notice-zuul
