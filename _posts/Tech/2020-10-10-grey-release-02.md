# 灰度发布落地实战2

---

#### Java落地代码实现：



#### 第二种方案，基于服务调用服务

原理：用户请求打到 服务A，aop切面拦截请求取得request header中的 version字段，根据不同的规则，使用ribbon转发到不同的服务（根据eureka cilent 中的 metadata）



服务A --> 服务B

<img src="../../../public/img/posts/grey-service00.png">

项目结构介绍：

cloud-eureka：7900

api-passenger：8080

service-sms：8003 和 8004



这次是 api-passenger 调用 service-sms 服务，服务与服务之间的灰度发布实现

ribbon在 api-passenger中，因为进rule和出rule是一个线程，所以可以用ThreadLocal把它获取到

<img src="../../../public/img/posts/grey-service01.png" />

在 api-passenger 中定义 TestCallServiceSmsController 用于测试调用服务：

```java
/**
 * 用于服务与服务之间的灰度发布测试
 */
@RestController
@RequestMapping("/test")
public class TestCallServiceSmsController {

    @Autowired
    private RestTemplate restTemplate;

    /**
     * 该方法会调用 service-sms 中的方法
     * @return
     */
    @GetMapping("/call")
    public String testCall(){

        return restTemplate.getForObject("http://service-sms/test/sms-test",String.class);
    }

    @GetMapping("/test")
    public String testCall1(){

        return "api-passenger";
    }
}
```



定义灰度规则类：

```java
/**
 * 自定义Ribbon路由规则，用于灰度发布
 */
public class GreyRule extends AbstractLoadBalancerRule {

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }

    @Override
    public Server choose(Object o) {
        return choose(getLoadBalancer(), o);
    }

    /**
     * 重载choose() 通过 LB 取得 server
     * @param lb
     * @param key
     * @return
     */
    public Server choose(ILoadBalancer lb, Object key){
        System.out.println("灰度  rule");

        Server server = null;
        while(server == null){
            // 获取所有 可达的服务
            List<Server> reachableServers = lb.getReachableServers();

            // 获取 当前线程的参数 用户id verion=1
            Map<String,String> map = RibbonParameters.get();
            String version = "";
            if (map != null && map.containsKey("version")){
                version = map.get("version");
            }
            System.out.println("当前rule version:"+version);

            // 根据用户选服务
            for (int i = 0; i < reachableServers.size(); i++) {
                server = reachableServers.get(i);
                // 用户的version我知道了，服务的自定义meta我不知道。

                // eureka:
                //  instance:
                //    metadata-map:
                //      version: v2
                // 不能调另外 方法实现 当前 类 应该实现的功能，尽量不要乱尝试
                Map<String, String> metadata = ((DiscoveryEnabledServer) server).getInstanceInfo().getMetadata();

                String serverVersion = metadata.get("version");
                // 服务的meta也有了，用户的version也有了。
                if (version.trim().equals(serverVersion)){
                    return server;
                }

            }
        }
        // 怎么让server 取到合适的值。这个server会在循环内被赋值
        //根据业务场景，制定规则，这里可以选择用默认服务兜底
        return server;
    }
}

```

题外话：

com.netflix.loadbalancer.Server类，没有获取metadata的方法

我们可以参考Server的子类，IDEA中把鼠标点中Server进去看子类

发现DiscoveryEnabledServer中有getMetadata()方法



定义 RibbonParameters 类：

```java
/**
 * 用于 保存、获取 每个线程中的 request header
 */
@Component
public class RibbonParameters {

    private static final ThreadLocal local = new ThreadLocal();

    public static <T> T get(){
        return (T)local.get();
    }

    public static <T> void set(T t){
        local.set(t);
    }
}

```

注意：在 GreyRule中 使用了 RibbonParameters.get() ，使用ThreadLocal get()之前，必须先set() 否则会空指针

我们在aop切面中拦截请求，在GreyRule之前进行ThreadLocal set()

切面代码：

```java
/**
 * 拦截请求，AOP实现，获取request header
 */
@Aspect
@Component
public class RequestAspect {

    /**
     * 定义切入点
     */
    @Pointcut("execution(* com.kennedy.apipassenge.controller..*Controller*.*(..))")
    private void anyMehtod(){

    }

    /**
     * 在之前切入
     * 此时IDEA中左侧栏能看到被拦截的方法
     * @param joinPoint
     */
    @Before(value = "anyMehtod()")
    public void before(JoinPoint joinPoint){
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String version = request.getHeader("version");

        Map<String,String> map = new HashMap<>();
        map.put("version",version);

        RibbonParameters.set(map);  //写入ThreadLocal
    }
}

```



定义Ribbon配置类

```java
/**
 * 自定义Ribbon配置，用于启动类
 */
public class GrayRibbonConfiguration {

    @Bean
    public IRule ribbonRule(){
        return new GreyRule();
    }
}
```



启动类：

```java
@SpringBootApplication
@RibbonClient(name = "service-sms" , configuration = GrayRibbonConfiguration.class)
public class ApiPassengeApplication {

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(ApiPassengeApplication.class, args);
    }

}
```



service-sms 项目 yml 配置不用变，参见上篇文章



测试：

使用postman调用GET请求：http://localhost:8080/test/call

request header中设置 version = v1



结果：

v1用户访问 service-sms：8003

v2用户访问 service-sms：8004

实现了服务与服务之间灰度发布



---

#### 还可以使用 io.jmnarloch 实现

pom.xml引入：

```xml
<dependency>
    <groupId>io.jmnarloch</groupId>
    <artifactId>ribbon-discovery-filter-spring-cloud-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

项目中的 GreyRule、GreyRibbonConfiguration、RibbonParameters 可以删掉

只需要修改切面类：

```
/**
 * 拦截请求，AOP实现，获取request header
 */
@Aspect
@Component
public class RequestAspect {

    /**
     * 定义切入点
     */
    @Pointcut("execution(* com.kennedy.apipassenge.controller..*Controller*.*(..))")
    private void anyMehtod(){

    }

    /**
     * 在之前切入
     * 此时IDEA中左侧栏能看到被拦截的方法
     * @param joinPoint
     */
    @Before(value = "anyMehtod()")
    public void before(JoinPoint joinPoint){
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        String version = request.getHeader("version");

//        Map<String,String> map = new HashMap<>();
//        map.put("version",version);
//
//        RibbonParameters.set(map);  //写入ThreadLocal

        //灰度规则 匹配的地方 查db, redis
        if (version.trim().equals("v1")) {
            RibbonFilterContextHolder.getCurrentContext().add("version", "v1");
        } else if (version.trim().equals("v2")){
            RibbonFilterContextHolder.getCurrentContext().add("version", "v2");
        }
    }
}
```

部署运行，进行同样的测试成功

使用 RibbonFilterContextHolder 实现，内部也是使用了ThreadLocal实现，贴上源码：

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package io.jmnarloch.spring.cloud.ribbon.support;

import io.jmnarloch.spring.cloud.ribbon.api.RibbonFilterContext;

public class RibbonFilterContextHolder {
    private static final ThreadLocal<RibbonFilterContext> contextHolder = new InheritableThreadLocal<RibbonFilterContext>() {
        protected RibbonFilterContext initialValue() {
            return new DefaultRibbonFilterContext();
        }
    };

    public RibbonFilterContextHolder() {
    }

    public static RibbonFilterContext getCurrentContext() {
        return (RibbonFilterContext)contextHolder.get();
    }

    public static void clearCurrentContext() {
        contextHolder.remove();
    }
}

```



#### 总结：

AOP切面截取request header 传入ThreadLocal

自定义Ribbon规则GreyRule中，从ThreadLocal拿到request header，根据不同规则，路由到不同服务，也是使用自定义eureka metadata

手写ThreadLocal 和使用 io.jmnarloch 原理是一样的



项目代码地址：

https://github.com/kennedy-han/grey-notice-zuul