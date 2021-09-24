---
layout: post
title: "Zuul网关实战01"
description: "Zuul 实战"
category: Tech
tags: [Zuul SpringCloud]
---



# Zuul网关实战01


## 老项目向微服务改造中面临的问题：

### 老客户端URL不变，新的服务没有对应的接口

如：旧客户端访问 http://localhost:9100/service-sms/sms-test31

而新项目（微服务）中没有 /sms-test31

所以需要将 /sms-test31转换为 新服务有的URL映射，这里举例子映射为 /sms-test3

映射关系：/sms-test31 ---> /sms-test3



解决方案有很多种：

1. Filter使用灰度发布的方式改URL，做好老URL和新URL对应关系
2. Nginx地址映射（写固定配置文件，或者配合Node.js做映射）
3. Zuul网关中自定义Filter

本文围绕Zuul实现
#### 1. 路由到具体地址
zuul yml中配置：
```yml
# 路由到具体地址
zuul:
  routes:
    # HostFilter.java 使用的配置
    xxxx: /zuul-api-driver/**
```

匹配 /zuul-api-driver/** 下面的地址



HostFilter.java代码：

```java
/**
 * 老项目改造微服务01
 * 网关转发
 * 老客户端请求URL不变，通过网关转发到新地址
 * 如：
 * 老项目客户端请求：/zuul-api-driver
 * 转发到 http://localhost:8003/test/sms-test3
 * yml 需要打开配置
 * zuul:
 *      xxxx: /zuul-api-driver/**
 */
@Component
public class HostFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return FilterConstants.ROUTE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        // 生产环境小技巧：DB或redis中 存储过滤器开关
        // 项目起初用户量小，所以要降低使用的门槛
        // 随着用户量提升，要增加一些限制，比如 黑名单，硬件检测等过滤器
        // 一开始将配置写入DB或redis，根据情况打开后续过滤器，省去了改代码重新部署

        //也可以进行过滤器顺序，如果前面的过滤器逻辑没走通，想阻止后面的过滤器工作时
        // 前面的过滤器 RequestContext.getCurrentContext().set("key");
        // 后面的过滤器判断 RequestContext.getCurrentContext().get("key"); 如果有，return false不执行过滤器

        return true;
    }

    @SneakyThrows
    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();

        //获取 请求的url
        String remoteAddr = request.getRequestURI();

        // 此处的匹配规则，可以读取DB存入redis，通过后台系统写入DB配置规则进行转发，好处是方便升级后端接口
        // 或者本地缓存一系列的匹配规则，如用 Map

        //和 老地址 做匹配
        if(remoteAddr.contains("/zuul-api-driver")){
            // remoteAddr => newAddr

            currentContext.setRouteHost(new URI("http://localhost:8003/test/sms-test3").toURL());
        }
        return null;
    }
}

```



#### 2. 转发到Eureka的服务
RibbonFilter代码：

```java
/**
 * 老项目改造微服务02
 * 网关转发
 * 老客户端请求：http://localhost:9100/service-sms/sms-test31
 * 转发到Eureka的服务，使用Ribbon
 * 到新地址：http://localhost:9100/service-sms/sms-test3
 * 需要启动 service-sms项目，zuul yml不需要修改配置
 */
@Component
public class RibbonFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.ROUTE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();
        HttpServletRequest request = currentContext.getRequest();

        // 获取 请求url
        String remoteAddr = request.getRequestURI();

        // 和 老地址 做匹配
        if (remoteAddr.contains("/sms-test31")){
            // rmoteAddr => newAddr

            currentContext.set(FilterConstants.SERVICE_ID_KEY,"service-sms");
            currentContext.set(FilterConstants.REQUEST_URI_KEY,"/test/sms-test3");
        }

        return null;
    }
}
```



#### 3. 动态路由

可以将URL匹配规则，自定义到DB，redis或yml中（上面1和2也能实现）

application-my.yml代码：

```yaml
# 自定义路由配置，用于老项目升级，旧地址映射到新地址
# 测试类在 MyController
kennedy:
  address: addr-test1
```



自定义MyController：

```
/**
 * 从 yml中读取配置地址
 */
@RestController
public class MyController {

    @Autowired
    private MyYml myYml;

    @GetMapping("/myController")
    public String myForward(){

        //这里可以根据 yml 中读取到的服务配置信息，进行转发
        return "my controller " + myYml.getAddress();
    }
}
```



## 后端服务停止了，再通过网关访问时会报错，返回友好信息

MyFallback代码：

```java
/**
 * 服务容错，返回友好JSON，避免返回500错误
 * 正常可以访问 http://localhost:9100/service-sms/test/sms-test2
 * 把服务 service-sms 停掉之后，再访问即可触发此 Fallback
 */
@Component
public class MyFallback implements FallbackProvider {
    @Override
    public String getRoute() {
        return "*";     //匹配所有地址
    }

    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse(){

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }

            @Override
            public InputStream getBody() throws IOException {
                // 通用的返回值 结构一样。{code:11,message:"",data:""}，方便前端处理
                // 降级的代码
                // {code:-1,message:"",data:"{orderStatus:-1(未知)}"}
                String code = "{code:-100,message:\"\",data:\"{orderStatus:-1(未知)}";
                return new ByteArrayInputStream(code.getBytes());
            }

            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.BAD_REQUEST;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return HttpStatus.BAD_REQUEST.value();
            }

            @Override
            public String getStatusText() throws IOException {
                return "message";
            }

            @Override
            public void close() {

            }
        };
    }
}

```



## 查看Zuul中的 filters 和 routes

使用actuator实现

pom.xml引入：

```xml
<!-- zuul监控 filter 和 routes -->
<dependency>
    <groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```



Zuul yml配置：

```yaml
# 打开actuator 监控，过滤器和路由
# http://localhost:9100/actuator/filters
# http://localhost:9100/actuator/routes
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      ##默认是never
      show-details: ALWAYS
      enabled: true
    routes:
      enabled: true
```

 http://localhost:9100/actuator/filters 查看所有过滤器

 http://localhost:9100/actuator/routes 查看所有路由



## 网关限流

通常，网关的QPS要小于或接近于后方服务整体的QPS

也取决于网关自身的业务，除了转发请求之外，如果也会向其他系统去鉴权，这类的QPS也需要在产品上线之前做好预估

令牌桶算法

<img src="../../../public/img/posts/token bucket.png" />

使用方去取令牌，可以自定义策略：拿不到就走、或者拿不到就等

基于Grava实现LimitFilter代码：

```java
/**
 * 网关限流 Filter
 * 使用 Grava实现，令牌桶算法
 * 建议配合 Jmeter测试
 */
@Component
public class LimitFilter extends ZuulFilter {

    // 2 qps(1秒  2个 请求 Query Per Second 每秒查询量)
    private static final RateLimiter RATE_LIMITER = RateLimiter.create(2);

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return -10;     //限流过滤器执行优先级高
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();
        if (RATE_LIMITER.tryAcquire()){
            System.out.println("passed");
            return null;
        } else {
            // 被流控的逻辑
            System.out.println("被限流了");
            currentContext.setSendZuulResponse(false);
            currentContext.setResponseStatusCode(HttpStatus.TOO_MANY_REQUESTS.value());
        }
        return null;
    }
}
```





项目代码地址：

https://github.com/kennedy-han/grey-notice-zuul
