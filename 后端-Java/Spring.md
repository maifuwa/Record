# Spring Boot
只是用于快速创建`SSM`项目的脚手架，帮助用户快速构建生产级别的`Spring`应用

版本选择：`GA` > `RC` > `M` > `SNAPSHOT`
>`GA`经过全面测试和生产就绪的稳定版本；`RC`候选发布版本。它包含所有计划的功能，但尚未经过全面测试；`M`里程碑版本。它包含一组特定的功能；`SNAPSHOT`开发版本。它可能不稳定或包含错误
## 多环境
`SpringBoot`只会读取`application.properties`或`application.yml`文件，需要先修改`application`文件，再新建`application-dev.yml`和`application-prod.yml`文件
```yml
# application.yml
spring:
  profiles:
    active: dev # 启用开发环境
    
# 开发环境配置直接写在 application-dev.yml 
server:
  port: 8080
  
# 生产环境 application-prod.yml
server:
  port: 80
```
> 除了配置文件还可以在启动jar时传入配置而且优先级更高,如`java -jar spring-boot-text.jar --server.port=80`

[maven详解](https://juejin.cn/post/7238823745828405308)
## 日志
`Spring`默认使用`Logback`作为日志实现层，`Slf4j`作为日志接口层

Log Level: `ERROR`, `WARN`, `INFO`, `DEBUG`, or `TRACE`  default `INFO`

可以通过`Spring`配置文件或在`classpath`添加`logback-spring.xml`配置文件(也可以命名为`logback.xml`、`logback-spring.groovy`...)

使用`Spring`配置文件
```yml
logging:  
  level:  
    root: warn # 设置全局日志级别  
    com.example.demo: debug # 设置指定包的日志级别  
  file:  
    path: /var/log/demo # 设置日志文件的路径  
    name: demo.log # 设置日志文件的名称
```
> Log files rotate when they reach 10 MB and, as with console output, `ERROR`-level, `WARN`-level, and `INFO`-level messages are logged by default.

使用logback-spring.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <property name="SYS_LOG_PATH" value="./document/log/system"/>
    <property name="OPE_LOG_PATH" value="./document/log/operate"/>

    <!-- 终端日志输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d %green([%thread]) %highlight(%level) %magenta(%logger{50}) - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 系统日志存放位置 -->
    <appender name="FILE-ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${SYS_LOG_PATH}/app.log</file>

        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${SYS_LOG_PATH}/%d{yyyy-MM, aux}/app-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <totalSizeCap>1GB</totalSizeCap>
            <maxHistory>100</maxHistory>
        </rollingPolicy>

        <encoder>
            <pattern>%d{yyyy-MM HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 配置操作日志存放位置 -->
    <appender name="Collect" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${OPE_LOG_PATH}/operate.log</file>

        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/%d{yyyy-MM, aux}/operate-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <totalSizeCap>1GB</totalSizeCap>
            <maxHistory>100</maxHistory>
        </rollingPolicy>

        <encoder>
            <pattern>%d{yyyy-MM HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 配置操作日志 additivity="false" 表示不将配置传递到父 logger -->
    <logger name="com.bigboss.aoplogger.logcollect" level="info" additivity="false">
        <appender-ref ref="Collect"/>
    </logger>

    <logger name="com.bigboss.aoplogger" level="info" additivity="false">
        <appender-ref ref="FILE-ROLLING"/>
        <appender-ref ref="STDOUT"/>
    </logger>

    <root level="error">
        <appender-ref ref="FILE-ROLLING"/>
        <appender-ref ref="STDOUT"/>
    </root>

</configuration>
```

> [logback日志配置项](https://logback.qos.ch/manual/configuration.html)
## Spring aop
[spring aop 中文文档](https://springdoc.cn/spring/core.html#aop)  
Spring提供了面向切面编程的丰富支持，允许通过分离应用的业务逻辑与系统级服务（例如审计（auditing）和事务（transaction）管理）进行内聚性的开发  
`spring aop`需要添加依赖

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-aop</artifactId>  
</dependency>
```
### 快速开始
操作步骤：
1. 在配置类上添加`@EnableAspectJAutoProxy`启用`@AspectJ`支持
2. 声明切面(Aspect)，创建一个类并添加上`@AspectJ`和`@Component`
3. 声明增强(Advice)和切点(Pointcut)

> Advice 与一个[切点表达式](https://springdoc.cn/spring/core.html#aop-pointcuts-designators)相关联，在切点匹配的方法执行之前(before)、之后(after)或周围(around)运行。切点表达式可以是一个内联切点，也可以是对一个 [命名切点](https://springdoc.cn/spring/core.html#aop-common-pointcuts) 的引用。    

> 任何 advice method 都可以声明一个 `org.aspectj.lang.JoinPoint` 类型的参数作为其第一个参数。请注意，around advice 方法需要声明一个 `ProceedingJoinPoint` 类型的第一个参数，它是 `JoinPoint` 的一个子类。

```java
@Slf4j  
@Aspect  
@Component  
public class DemoAop {  
  
    @Before("execution(* com.example.demo.DemoController.*(..))")  
    public void beforeLog(JoinPoint joinPoint) {  
        log.info("正在执行方法：{}", joinPoint.getSignature().getName());  
        log.info("joinPoint.String:{}, args:{},this:{} ,target:{}, kind:{}, sourceLocation:{}, staticPart:{}", joinPoint.toString(), joinPoint.getArgs(), joinPoint.getThis(), joinPoint.getTarget(), joinPoint.getKind(), joinPoint.getSourceLocation(), joinPoint.getStaticPart());  
    }  
  
    @Around("@annotation(com.example.demo.ALog)")  
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {  
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();  
        Method method = signature.getMethod();  
        ALog aLog = method.getAnnotation(ALog.class);  
        if (aLog != null) {  
            log.info("ALog message:{},MethodArgs：{}", aLog.message(), joinPoint.getArgs());  
        }  
        return joinPoint.proceed();  
    }  
}

@Retention(RetentionPolicy.RUNTIME)  
@Target(ElementType.METHOD)  
public @interface ALog {  
  
    String message();  
}
```
`execution`执行表达式的格式`execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)`  
依次是：方法修饰符、返回值类型、包名+类名.方法名(参数列表)、可能抛出的异常类型；中间两个之外为可选项  
例子：`"execution(* com.example.demo.DemoController.*(String, *))"`

> 注意：在Spring AOP中，切面本身不能成为其他切面的advice的目标。类上的 `@Aspect` 注解将其标记为一个切面，因此，将其排除在自动代理之外。

看别人代码才知道切点(Pointcut)声明作用
```java
@Pointcut("execution(public * com.macro.mall.controller.*.*(..))||execution(public * com.macro.mall.*.controller.*.*(..))")  
public void webLog() {  
}  
  
@Before("webLog()")  
public void doBefore(JoinPoint joinPoint) throws Throwable {  
}
```

### 动态代理
spring aop主要有两种实现方式：
1. JDK动态代理(默认，**需要实现接口**)
2. CGLIB代理

> JDK 动态代理是基于 Java 反射机制实现的。它通过 `Proxy` 类来创建代理对象，并通过 `InvocationHandler` 接口来定义代理对象的行为。 

> CGLIB 动态代理是基于 ASM 字节码框架实现的。它通过生成目标类的子类来创建代理对象，并通过重写目标类的方法来定义代理对象的行为。

## Spring cache
[spring Cache Abstraction](https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/integration.html#cache)  
[中文文档](https://springdoc.cn/spring/integration.html#cache)  
从3.1版本开始，`Spring`框架提供了对现有`Spring`应用透明地添加缓存的支持。缓存抽象的核心是对`Java`方法进行缓存，从而根据缓存中的可用信息减少执行次数。

> 这种方法只适用于那些无论调用多少次都能保证相同输入返回相同输出的方法

`spring`也提供了一些缓存抽象实现: `ConcurrentMap`、`Gemfire`、`Ehcached`、`etc`.

> 缓存抽象没有对多线程和多进程环境、`TTL`、`TTI`、`Eviction`策略提供支持，由缓存实现自行处理

要使用缓存抽象，从两个方面入手：
- 缓存声明：确认需要缓存的方法和缓存策略
- 缓存配置：储存数据和从中读取数据的备份缓存
#### 声明式注解缓存
对于缓存声明，Spring的缓存抽象提供了一组Java注解：
- `@EnableCaching`: 启动缓存抽象。
- `@Cacheable`: 触发缓存的填充。
- `@CacheEvict`: 触发缓存驱逐。
- `@CachePut`: 更新缓存而不干扰方法的执行。
- `@Caching`: 将多个缓存操作重新分组，应用在一个方法上。
- `@CacheConfig`: 分享一些常见的类级别的缓存相关设置。
## Spring async
从Spring3开始提供了`@Async`注解，该注解可以被标在方法上，以便异步地调用该方法。调用者将在调用时立即返回，方法的实际执行将提交给`Spring TaskExecutor`的任务中，由指定的线程池中的线程执行。

> 注意：生产环境不建议使用默认配置，默认的线程池实现`SimpleAsyncTaskExecutor`不是真的线程池，每次调用都会创建一个新线程

> 异步不生效原因:
1. 没有在配置类添加`@EnableAsync`
2. 调用方和`@Async`方法在一个类中(因为是使用代理实现的异步，当同一个类内部调用时，实际上是绕过了代理，从而导致`@Async`注解无效)
3. 注解`@Async`的方法不是`public`方法
4. 注解`@Async`的返回值只能为`void`或`Future`
5. 在`@Async`方法上标注`@Transactional`是没用的，但在`@Async`方法调用的方法上标注`@Transcational`是有效的(看事务传播机制)
6. ....

> 同样备注下事务不生效的原因:
1. 方法自调用 (`spring`声明式事务底层实现是`aop`，如果内部调用方法再生成代理类后调用的还是原本没有添加事务的方法)
2. 异常被捕获了，或者抛出的异常不在回滚范围
3. 方法非`public`
4. 调用方法的类不是`spring bean`
5. 数据库不支持事务

#### 指定线程池
自定义线程池有如下模式:
- 重新实现接口`AsyncConfigurer`
- 继承`AsyncConfigurerSupport`
- 配置由自定义的`TaskExecutor`替代内置的任务执行器

配置由自定义的`TaskExecutor`替代内置的任务执行器
```java
@Configuration  
public class ThreadPoolConfig {  
  
    @Value("${user.task.execution.pool.queueCapacity:8}")  
    private Integer queueCapacity;  
  
    @Value("${user.task.execution.pool.corePoolSize:8}")  
    private Integer corePoolSize;  
  
    @Value("${user.task.execution.pool.maxPoolSize:8}")  
    private Integer maxPoolSize;  
  
    @Value("${user.task.execution.pool.keepAliveSeconds:5}")  
    private Integer keepAliveSeconds;  
  
    @Value("${user.task.execution.pool.threadNamePrefix:async-task-}")  
    private String threadNamePrefix;  
  
    @Bean("userAsyncExecutor")  
    public Executor userAsyncExecutor() {  
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();  
        // 线程池的任务队列容量 当线程池中的线程数达到 corePoolSize 时，新提交的任务会被放入这个队列中等待执行。如果队列已满，则根据拒绝策略进行处理。  
        executor.setQueueCapacity(queueCapacity);  
        // 线程池的核心线程数 在没有任务执行时，线程池的线程数会保持在这个数量 即使线程池中的线程处于空闲状态，也不会被销毁，除非设置了 allowCoreThreadTimeOut。  
        executor.setCorePoolSize(corePoolSize);  
        executor.setMaxPoolSize(maxPoolSize);  
        // 线程池维护线程所允许的空闲时间 当线程池中的线程数大于 corePoolSize 时，如果一个线程空闲时间达到了 keepAliveSeconds，那么该线程会被销毁直到线程池中的线程数不大于 corePoolSize        executor.setKeepAliveSeconds(keepAliveSeconds);  
        // 指定创建的线程的名称前缀 线程名称可以方便地用于日志记录和调试  
        executor.setThreadNamePrefix(threadNamePrefix);  
        // 拒绝策略 任务被拒绝时，直接在提交任务的线程中执行被拒绝的任务  
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());  
        executor.initialize();  
        return executor;  
    }  
  
}

@Slf4j  
@Service  
public class AsyncServer {  

    // 当只配置了一个线程池时，可以不用指定线程池名称  
    @Async("userAsyncExecutor")  
    public void async() {  
      log.info("async start");  
    }  
  
}
```

Java 中的线程池还提供了四种拒绝策略：
- `AbortPolicy`（默认策略）：这是默认的拒绝策略，当任务被拒绝时，会抛出 `RejectedExecutionException` 异常
- `DiscardPolicy`：当任务被拒绝时，会默默地丢弃被拒绝的任务，不会抛出异常也不会执行任务
- `DiscardOldestPolicy`：当任务被拒绝时，会丢弃队列中最老的一个任务，然后尝试重新提交被拒绝的任务
- `CallerRunsPolicy`：当任务被拒绝时，直接在提交任务的线程中执行被拒绝的任务

> 线程池大小设置(推荐)：  
> IO密集型: 频繁读取磁盘上的数据，或者需要通过网络远程调用接口。 2N  
> CPU密集型: 非常复杂的调用，循环次数很多，或者递归调用层次很深等。 N+1  
> 最佳线程数目算法: ((线程等待时间+线程CPU时间)/线程CPU时间)* CPU数目

> 获取N(CPU核数) `int availableProcessors = Runtime.getRuntime().availableProcessors();`

> 更加严谨的计算方法：最佳线程数 = N * (1 + WT / ST)  
> N:cpu核心数、WT:线程等待时间、ST:线程计算时间;WT = 线程总运行时间 - ST

#### 获取返回值
~~等着填坑~~
## 其他事项
1. 定制Banner,只需要添加banner.txt到src/main/resources中即可,[创建banner.txt](http://www.network-science.de/ascii/)
2. @Bean 的默认名字是方法名，不是返回值类名 [Bean详解](https://www.liaoxuefeng.com/wiki/1252599548343744/1308043627200545)
# Spring Web
`SpringBoot`的`web`模块，内置`Tomcat`使得项目能够直接运行，不再需要手动打包成`war`放入`Tomcat`运行，并能直接打包成`jar`运行

包含`Spring mvc` 的组件(如`Controller`、`Service`、`Component`、`Configuration`等),且能自动扫描位置，支持自动将对象转为`json`格式返回客户端(默认使用`jackson`)
```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
## 快速开始
项目结构:`config`、`controller`、`domain`、`filter`、`dao`、`service`、`utils`

`config`包放各种配置类 `@Configuration` 如: `WebConfiguration`、`SecurityConfiguration`
`controller`包与用户交互类:
1. `@Controller` 一般返回前端界面 其方法添加 `@GetMapping(url)` 、`@PostMapping(url)`
2. `@RestController` 一般返回`json`字符串，前后端分离 需要在类前添加 `@RequestMapping(url)` 方法前也需要添加`@GetMapping(url)` 、`@PostMapping(url)`
```java
// @ResponseBody 可以作用于方法也可作用于类  表示返回值直接返回客户端，不返回视图 
// 在普通Controller中，方法的返回值不是String,也不是ModelAndView时添加
// RestController 默认添加

// 获取请求参数 

@RestController
@RequestMapping("/images")
public void ImagesController{

    // RestFul 风格的api接口
    @GetMapping("/{id}")
    public String getImages(@PathVariable("id") String id){
        // 获取对应图片，将图片转为base64
        return "";
    }
    
    @GetMapping("/{id}/{key}")
    public String getImagesByKey(@PathVariable("id") String id, @PathVariable("key") String key){
        // 获取对应图片，将图片转为base64
        return "";
    }

    // 参数在请求路径和请求体中  如果参数名相同可以省略@RequestParam
    @GetMapping("/image")
    public String getImage(@RequestParam("id") String id) {
        // 获取对应图片，将图片转为base64
        return "";
    }
    
    // 参数为json字符串，直接转为对象读取 (请求头 Content-Type：application/json)
    @GetMapping("/input")
    public boolean storageImage(@RequestBody ImageData image) {
        // 存储image
        return true;
    }
}
```

全局异常处理
```java
@RestControllerAdvice  
public class GlobalExceptionHandler {  
  
    @ExceptionHandler(Exception.class)  
    public ResponseEntity<?> handleException(Exception e) {  
        return ResponseEntity.internalServerError().body(e.getMessage());  
    }  
}


// 更加详细
@RestController
@RequestMapping({"${server.error.path:${error.path:/error}}"})
public class ErrorPageController extends AbstractErrorController {

    public ErrorPageController(ErrorAttributes errorAttributes) {
        super(errorAttributes);
    }

    /**
     * 所有错误在这里统一处理，自动解析状态码和原因
     * @param request 请求
     * @return 失败响应
     */
    @RequestMapping
    public RestBean<?> error(HttpServletRequest request) {
        HttpStatus status = this.getStatus(request);
        Map<String, Object> errorAttributes = this.getErrorAttributes(request, this.getAttributeOptions());
        String message = this.convertErrorMessage(status)
                .orElse(errorAttributes.get("message").toString());
        return RestBean.failure(status.value(), message);
    }

    /**
     * 对于一些特殊的状态码，错误信息转换
     * @param status 状态码
     * @return 错误信息
     */
    private Optional<String> convertErrorMessage(HttpStatus status){
        String value = switch (status.value()) {
            case 400 -> "请求参数有误";
            case 404 -> "请求的接口不存在";
            case 405 -> "请求方法错误";
            case 500 -> "内部错误，请联系管理员";
            default -> null;
        };
        return Optional.ofNullable(value);
    }

    /**
     * 错误属性获取选项，这里额外添加了错误消息和异常类型
     * @return 选项
     */
    private ErrorAttributeOptions getAttributeOptions(){
        return ErrorAttributeOptions
                .defaults()
                .including(ErrorAttributeOptions.Include.MESSAGE,
                        ErrorAttributeOptions.Include.EXCEPTION);
    }
}
```

`filter`包放拦截器 有`servlet`中的`filter`，也有`Spring mvc`的`interceptor`，前者过滤所有请求，后者只过滤`controller`
```java
// 创建 filter  继承HttpFilter或OncePerRequestFilter
/**
* 限流 filter 
*/
@Slf4j
@Component
@Order(-999)        
public class FlowLimitingFilter extends HttpFilter {

    @Resource
    StringRedisTemplate template;
    //指定时间内最大请求次数限制
    @Value("${spring.web.flow.limit}")
    int limit;
    //计数时间周期
    @Value("${spring.web.flow.period}")
    int period;
    //超出请求限制封禁时间
    @Value("${spring.web.flow.block}")
    int block;

    @Resource
    FlowUtils utils;

    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        String address = request.getRemoteAddr();
        if (!tryCount(address))
            this.writeBlockMessage(response);
        else
            chain.doFilter(request, response);
    }

    /**
     * 尝试对指定IP地址请求计数，如果被限制则无法继续访问
     * @param address 请求IP地址
     * @return 是否操作成功
     */
    private boolean tryCount(String address) {
        synchronized (address.intern()) {
            if(Boolean.TRUE.equals(template.hasKey(Const.FLOW_LIMIT_BLOCK + address)))
                return false;
            String counterKey = Const.FLOW_LIMIT_COUNTER + address;
            String blockKey = Const.FLOW_LIMIT_BLOCK + address;
            return utils.limitPeriodCheck(counterKey, blockKey, block, limit, period);
        }
    }

    /**
     * 为响应编写拦截内容，提示用户操作频繁
     * @param response 响应
     * @throws IOException 可能的异常
     */
    private void writeBlockMessage(HttpServletResponse response) throws IOException {
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(RestBean.forbidden("操作频繁，请稍后再试").asJsonString());
    }
}


// 创建一个 interceptor 需要继承 HandlerInterceptor接口 Order()决定先后顺序数值越小，优先级越大
// 实现 preHandle()、postHandle()、afterCompletion()
@Order(0)
@Component
public class LoggerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return HandlerInterceptor.super.preHandle(request, response, handler);
    }
}

// interceptor 还需注册
@Configuration
public class WebConfiguration implements WebMvcConfigurer {
    @Autowired
    LoggerInterceptor loggerInterceptor;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 如果要过滤url registry.addInterceptor().addaddPathPatterns();
        registry.addInterceptor(loggerInterceptor); 
    }
}
```
## Jackson
[Jackson doc](https://www.baeldung.com/jackson)

可以通过自定义`Jackson2ObjectMapperBuilder`来定制`ObjectMapper`
```java
@Bean  
public Jackson2ObjectMapperBuilder jackson2ObjectMapperBuilder() {  
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd", Locale.CHINESE);  
    return new Jackson2ObjectMapperBuilder().serializers(new LocalDateTimeSerializer(dateTimeFormatter),  
                    new ZonedDateTimeSerializer(dateTimeFormatter))  // 修改时间格式
            .serializationInclusion(JsonInclude.Include.NON_NULL);   // 忽略null值
}
```

## 网络状态码
HTTP 响应状态码用来表明特定 [HTTP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP) 请求是否成功完成。 响应被归为以下五大类：
1. [信息响应](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status#%E4%BF%A1%E6%81%AF%E5%93%8D%E5%BA%94) (`100`–`199`)
2. [成功响应](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status#%E6%88%90%E5%8A%9F%E5%93%8D%E5%BA%94) (`200`–`299`)
3. [重定向消息](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status#%E9%87%8D%E5%AE%9A%E5%90%91%E6%B6%88%E6%81%AF) (`300`–`399`)
4. [客户端错误响应](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status#%E5%AE%A2%E6%88%B7%E7%AB%AF%E9%94%99%E8%AF%AF%E5%93%8D%E5%BA%94) (`400`–`499`)
5. [服务端错误响应](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status#%E6%9C%8D%E5%8A%A1%E7%AB%AF%E9%94%99%E8%AF%AF%E5%93%8D%E5%BA%94) (`500`–`599`)

| code | message                                        |
| ---- | ---------------------------------------------- |
| 200  | 请求成功                                           |
| 201  | 请求已成功，并因此创建了一个新的资源                             |
| 300  | 请求拥有多个可能的响应。用户代理或者用户应当从中选择一个                   |
| 301  | 请求资源的 URL 已永久更改(永久重定向)                         |
| 302  | 临时重定向                                          |
| 304  | 用于缓存的目的,告诉客户端可以继续使用相同的缓存版本                     |
| 400  | 客户端错误(如：请求语法错误、无效请求消息格式或者欺骗性请求路由),服务器无法或不会处理请求 |
| 401  | `unauthenticated`,客户端身份验证没通过                   |
| 403  | 客户端没有访问内容的权限,服务器知道客户端的身份                       |
| 404  | 无法识别 URL,或服务器找不到请求的资源                          |
| 410  | 请求的内容已从服务器中永久删除且没有转发地址                         |
| 411  | 请求头缺少`Content-Length`，且服务器需要`Content-Length`   |
| 414  | `url too long`                                 |
| 500  | 服务器遇到了不知道如何处理的情况                               |
| 501  | 服务器不支持请求方法，因此无法处理                              |
| 503  | 服务器没有准备好处理请求,常见原因是服务器因维护或重载而停机                 |
| 505  | 服务器不支持请求中使用的`HTTP`版本                           |
| 508  | 服务器在处理请求时检测到无限循环                               |

> 一般项目还会再封装一个响应体，将http协议的响应码与实际接口的结果响应码区分开来

# Spring validation
数据校验，可以避免参数注入、`SQL`注入、`XSS`攻击等安全问题
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

最佳实践
```java
@Slf4j
@Validated   //首先在Controller上开启接口校验
@Controller
public class TestController {
    @ResponseBody
    @PostMapping("/submit")
    public String submit(@Length(min = 3) String username, @Length(min = 10) String password){
        // ...
        return "请求成功!";
    }
    
    @ResponseBody
    @PostMapping("/submit")  //在参数上添加@Valid注解表示需要验证
    public String submit(@Valid Account account){
        // ...
        return "请求成功!";
    }
}

@Data
public class Account {
    //只需要在对应的字段上添加校验的注解即可
    @Length(min = 3)   
    String username;
    @Length(min = 10)
    String password;
}
```

![[常用的验证注解.png]]

# 数据库连接池
## HikariCP
`HikariCP`是一个高性能的Java连接池库，用于管理数据库连接。它专为高并发和低延迟的应用程序设计，提供了可靠、稳定且高效的连接池解决方案。（是SpringBoot默认的数据库连接池）

| 常用配置项               | 解释                                                         | 默认值             |
| ------------------- | ---------------------------------------------------------- | --------------- |
| autoCommit          | 从池返回的连接的默认自动提交行为                                           | true            |
| connectionTimeout   | 客户端等待来自池的连接的最大毫秒数                                          | 30000(ms)       |
| idleTimeout         | 连接允许在池中闲置的最长时间(0 或 >= 600000)                              | 600000          |
| maxLifetime         | 池中连接最长生命周期(0 或 >= 1800000)                                 | 1800000         |
| minimumldle         | 池中维护的最小空闲连接数(0<=minimumldle<=maximumPoolSize)              | maximumPoolSize |
| maximumPoolSize     | 池中最大连接数，其实就是线程池中队列的大小                                      | 10              |
| connectionTestQuery | 将在从池中向您提供连接之前执行的查询，以验证与数据库的连接是否仍然有效(如果驱动程序支持JDBC4，强烈建议不设置) | 无(select 1)     |
[一般来说，连接池数量 = (核心数 * 2) + 有效磁盘数](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)

[mysql性能配置推荐](https://github.com/brettwooldridge/HikariCP/wiki/MySQL-Configuration)
```properties
jdbcUrl=jdbc:mysql://localhost:3306/simpsons
username=test
password=test
dataSource.cachePrepStmts=true
dataSource.prepStmtCacheSize=250
dataSource.prepStmtCacheSqlLimit=2048
dataSource.useServerPrepStmts=true
dataSource.useLocalSessionState=true
dataSource.rewriteBatchedStatements=true
dataSource.cacheResultSetMetadata=true
dataSource.cacheServerConfiguration=true
dataSource.elideSetAutoCommits=true
dataSource.maintainTimeStats=false
```
## Druid
Druid是一个开源的Java数据库连接池，由阿里巴巴开发并维护。它提供了高性能、可靠性和可管理性的数据库连接池解决方案、数据库监控功能
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>${version}</version>
</dependency>
```

> `SpringBoot 3`使用`druid-spring-boot-3-starter` 

| 常用配置项                                     | 解释                                                                                                                 | 默认值       |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | --------- |
| initialSize                               | 初始化时建立物理连接的个数。初始化发生在显示调用init方法                                                                                     | 0         |
| maxActive                                 | 最大连接池数量                                                                                                            | 8         |
| minIdle                                   | 最小连接池数量                                                                                                            | maxActive |
| maxWait                                   | 获取连接时最大等待时间                                                                                                        |           |
| poolPreparedStatements                    | 是否缓存preparedStatement，也就是PSCache(对支持游标的数据库性能提升巨大，比如说oracle。在mysql下建议关闭)                                            | false     |
| maxPoolPreparedStatementPerConnectionSize | 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100 | -1        |
| validationQuery                           | 用来检测连接是否有效的sql                                                                                                     |           |
| filters                                   | 属性类型是字符串，通过别名的方式配置扩展插件，常用的插件有： 监控统计用的filter:stat 日志用的filter:log4j 防御sql注入的filter:wall                              |           |
常用配置
```yml
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/demo  
    username: root  
    password: password  
    type: com.alibaba.druid.pool.DruidDataSource  
    # druid 连接池管理  
    druid:  
      # 初始化时建立物理连接的个数  
      initial-size: 5  
      # 连接池的最小空闲数量  
      min-idle: 5  
      # 连接池最大连接数量  
      max-active: 20  
      # 获取连接时最大等待时间，单位毫秒  
      max-wait: 60000  
      # 申请连接的时候检测，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。  
      test-while-idle: true  
      # 既作为检测的间隔时间又作为testWhileIdel执行的依据  
      time-between-eviction-runs-millis: 60000  
      # 销毁线程时检测当前连接的最后活动时间和当前时间差大于该值时，关闭当前连接(配置连接在池中的最小生存时间)  
      min-evictable-idle-time-millis: 30000  
      # 用来检测数据库连接是否有效的sql 必须是一个查询语句(oracle中为 select 1 from dual)      validation-query: select 'x'  
      # 申请连接时会执行validationQuery检测连接是否有效,开启会降低性能,默认为true  
      test-on-borrow: false  
      # 归还连接时会执行validationQuery检测连接是否有效,开启会降低性能,默认为true  
      test-on-return: false  
      # 是否缓存preparedStatement, 也就是PSCache,PSCache对支持游标的数据库性能提升巨大，比如说oracle,在mysql下建议关闭。  
      pool-prepared-statements: false  
      # 置监控统计拦截的filters，去掉后监控界面sql无法统计，stat: 监控统计、Slf4j:日志记录、waLL: 防御sqL注入  
      filters: stat,wall,slf4j  
      # 要启用PSCache，必须配置大于0，当大于0时，poolPreparedStatements自动触发修改为true。在Druid中，不会存在Oracle下PSCache占用内存过多的问题，可以把这个数值配置大一些，比如说100  
      max-pool-prepared-statement-per-connection-size: -1  
      # 合并多个DruidDataSource的监控数据  
      use-global-data-source-stat: true  
      # 打开mergeSql功能；慢SQL记录  
      connection-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000  
        
      web-stat-filter:  
        # 是否启用StatFilter默认值true  
        enabled: true  
        # 添加过滤规则  
        url-pattern: /*  
        # 忽略过滤的格式  
        exclusions: /druid/*,*.js,*.gif,*.jpg,*.png,*.css,*.ico  
  
      stat-view-servlet:  
        # 是否启用StatViewServlet默认值true  
        enabled: true  
        # 访问路径为/druid时，跳转到StatViewServlet  
        url-pattern: /druid/*  
        # 是否能够重置数据  
        reset-enable: false  
        # 需要账号密码才能访问控制台，默认为root  
        login-username: druid  
        login-password: druid  
        # IP白名单  
        allow: 127.0.0.1  
        # IP黑名单（共同存在时，deny优先于allow）  
        deny:
```

# 数据持久化框架

## Mybaits
[Mybaits 官方文档(目前是mybaits-3)](https://mybatis.org/mybatis-3/zh_CN/index.html)
[MyBatis-Spring-Boot-Starter 官方文档](https://github.com/mybatis/spring-boot-starter/blob/master/mybatis-spring-boot-autoconfigure/src/site/zh_CN/markdown/index.md)
数据持久化框架,半`orm`需要自行编写`sql`语句，相比`jpa`更加灵活学习成本更低
```xml
<!--    Mybaits     -->
<dependency>  
    <groupId>org.mybatis.spring.boot</groupId>  
    <artifactId>mybatis-spring-boot-starter</artifactId>  
    <version>${mybaits.version}</version>  
</dependency>

<!--    MySQL驱动    -->
<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

> `mybatis-spring-boot-starter`里包含了`spring-boot-jdbc-starter`自带`HikariCP`数据库连接池，但没有数据库连接驱动

配置文件(更详细查看官方文档)
```yml
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/mbg  
    username: root  
    password: password  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    hikari:  
      pool-name: HikariCP  
      minimum-idle: 5  
      maximum-pool-size: 10  
mybatis:  
  mapper-locations: classpath:mapper/*.xml        # XML 映射文件的路径  
  type-aliases-package: org.bigboss.workdatabase.model   # 搜索类型别名的包名  
  configuration:  
    map-underscore-to-camel-case: true # 开启驼峰命名转换  
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 打印 SQL 语句  
    lazy-loading-enabled: true # 开启延迟加载
```

> `MyBatis-Spring-Boot-Starter` 将默认搜寻带有 `@Mapper` 注解的 mapper 接口。
> 如要指定一个自定义的注解或接口来扫描 mapper 接口，使用 `@MapperScan` 注解

### 缓存
一级缓存：作用域：`sqlSession`，默认开启
> 一级缓存失效场景：非同一`sqlSession`、查询条件不同、两次查询之间执行了增删改操作、两次查询之间手动清空过缓存

二级缓存：作用域：`sqlSessionFactory`
开启条件: 
1. 配置全局属性`cacheEnabled=true`(其实默认为true，无需配置)
2. 在映射文件中设置标签`<cache/>`
3. 二级缓存在`sqlSession`关闭或提交之后有效(反之在一级缓存中有效)
4. 查询数据的实体类必须实现序列化的接口
> 二级缓存失效场景：两次查询之间执行了增删改操作

### generator
使用Mybaits官方提供的[逆向工程工具](https://mybatis.org/generator/quickstart.html)，快速生成基础代码

使用`maven`插件的方式配置:
1. 在`POM.xml`中提供`mybaits`、`mybatis-generator-core`的依赖，`mybatis-generator-maven-plugin`的插件 (记得带上数据库连接驱动)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.bigboss</groupId>
    <artifactId>mybaits</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.15</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-core</artifactId>
            <version>1.4.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.2</version>
                <configuration>
                    <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.33</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
</project>
```

2. 在`resources`目录配置`generatorConfig.xml`和`config.properties`

config.properties
```properties
jdbc.driverClass=com.mysql.cj.jdbc.Driver  
jdbc.url=jdbc:mysql://localhost:3306/mbg  
jdbc.user=root  
jdbc.password=password
```
generatorConfig.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE generatorConfiguration  
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"  
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">  
<generatorConfiguration>  
    <properties resource="config.properties"/>  
  
    <context id="mysql" targetRuntime="MyBatis3" defaultModelType="flat">  
        <!-- 配置SQL语句中的前置分隔符 -->  
        <property name="beginningDelimiter" value="`"/>  
        <!-- 配置SQL语句中的后置分隔符 -->  
        <property name="endingDelimiter" value="`"/>  
        <!-- 配置生成Java文件的编码 -->  
        <property name="javaFileEncoding" value="UTF-8"/>  
  
        <!-- 为模型生成序列化方法 -->  
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>  
        <!-- 为生成的Java模型创建一个toString方法 -->  
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>  
        <!--为创建的model添加equal和hashcode方法-->  
        <plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"/>  
        <!-- 生成mapper.xml时覆盖原文件 -->  
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />  
  
        <!--关闭注释生成-->  
        <commentGenerator>  
            <property name="suppressAllComments" value="true"/>  
            <property name="suppressDate" value="true"/>  
            <property name="addRemarkComments" value="true"/>  
        </commentGenerator>  
        <jdbcConnection driverClass="${jdbc.driverClass}" connectionURL="${jdbc.url}"  
                        userId="${jdbc.user}" password="${jdbc.password}">  
            <!--高版本需要配置-->  
            <property name="nullCatalogMeansCurrent" value="true"/>  
        </jdbcConnection>        
        <javaModelGenerator targetPackage="com.bigboss.domain" targetProject="src/main/java"/>  
  
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources"/>  
  
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.bigboss.mapper" targetProject="src/main/java"/>  
  
        <table tableName="%">  
            <!-- 用来指定主键生成策略 -->  
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>  
        </table>    
    </context>
</generatorConfiguration>
```
3. 在`maven`的生命周期里双击`generator`即可

> 为避免自己写的被再次生成的覆盖，请将自动生成的代码与手写的分开 ~~官方默认不覆盖~~ 
### 衍生框架
[MyBatis-Flex - MyBatis-Flex 官方网站](https://mybatis-flex.com/)

## Spring Data JPA
[Spring Data Jpa 官方网站](https://spring.io/projects/spring-data-jpa)
[Spring Data Jpa doc 中文站](https://springdoc.cn/spring-data-jpa/)
数据持久化框架(官方推荐，默认使用的`orm`是`Hibernate`) 根据方法名自动生成调用数据库的代码(不用写sql和实现代码)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>  
    <groupId>com.mysql</groupId>  
    <artifactId>mysql-connector-j</artifactId>  
    <scope>runtime</scope>  
</dependency>
```

### 快速开始
常用配置文件
```yml
spring:  
  datasource:  
    url: jdbc:mysql://localhost:3306/database-name  
    username: root  
    password: password  
    driver-class-name: com.mysql.cj.jdbc.Driver  
    hikari:  
      pool-name: HikariCP  
      minimum-idle: 5  
      maximum-pool-size: 10 
  jpa:  
    database: mysql  
    show-sql: true  # 日志打印SQL语句
    hibernate:  
      ddl-auto: update  
    properties:  
      hibernate:  
        highlight_sql: true  # 高亮SQL语句
```

>  `ddl-auto` 用于设置表的创建和修改，可选: `none`、`create`、`create-drop`、`update`、`validate`
>  `none`：不做任何操作，需要自行建表
>  `create`：框架每次运行会删除所有表，然后重新创建
>  `create-drop`：同上，但程序结束时也会删除所有表
>  `update`：框架自动检查数据库表结构，于实体类不匹配自动修改表
>  `validate`：框架自动检查，不匹配抛出异常

实体类
```java
@Data 
@NoArgsConstructor
@Entity
@Table(name = "acount")   // 和数据库表名一致
public class Account {
    @Id                     //  此属性为主键
    @Column(name = "id")    //  对应表中id这一列
	@GeneratedValue(strategy = GenerationType.IDENTITY)   //  生成策略，这里配置为自增
	Long id;                //  包装类更好
	@Column(name = "name", nullable = false, length = 20)
	String name;
	@Column(name = "password", unique = true)
	String password;
}
```

`Repository`接口
```java
// JpaRepository<实体类,ID数据类型>  可以不写实现方法，直接使用接口
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

	// 如果 repository 本身没有需要的方法  需要自己添加，不用实现
	List<Account> findByNameLike(String str);

	// 使用JPQL
	@Query("from Account where name = :name")  
	Account findByName(@Param("name") String name);

	// 使用SQL
	@Query(value = "select * from account where name = :name", nativeQuery = true)
	Account findByName(@Param("name") String name);
}
```
常用的方法名
![[JPA方法名拼接词.png]]
`service`层调用
```java
@Server
public class AccountServer{
	@Resource
	AccountRepository repository;

	public void addOrUpdataAccount(Account account) {
		repository.save(account);     //  添加和更新都是 save()
	}
}
```

### 动态查询
动态添加查询条件，三种实现方式：`Query by Example`、`Specifications`、`Querydsl`
#### Query by Example
缺点：
- 只支持字符串`start/contains/ends/regex`匹配和其他属性类型的精确匹配
- 不支持嵌套或分组的属性
```java
@Test  
void testQBE() {  
    // 查询条件(可动态添加)  
    Student student = new Student();  
    student.setName("张三");  
    student.setDetail(new StudentDetail());  

	// 通过匹配器 对条件行为进行设置(非必需)
	ExampleMatcher matcher = ExampleMatcher.matching()  
        .withIgnorePaths("detail")    // 设置忽略属性  
        .withIgnoreCase("name")      // 设置忽略大小写  
        .withStringMatcher(ExampleMatcher.StringMatcher.ENDING)  // 对所有条件进行结尾匹配
        .withMatcher("name", m -> m.endsWith().ignoreCase());   // 匹配单个条件 第二个参数传入一个GenericPropertyMatcher对象，其方法可以链式调用
  
    // 通过Example构建查询条件  
    Example<Student> example = Example.of(student， matcher);  
  
    // 查询  
    List<Student> students = studentRepository.findAll(example);  
}
```

#### Specifications
类似`Query by Example`但可以匹配所有类型
```java
// 实现JpaSpecificationExecutor接口
public interface AccountRepository extends CrudRepository<Account, Long>, JpaSpecificationExecutor<Account> {  
}

@Test  
void useCriteriaBuilder() {  
    List<Account> accounts = repository.findAll(((root, query, criteriaBuilder) -> {  
  
        // 获取字段  
        Path<String> name = root.get("name");  
        Path<Integer> age = root.get("age");  
  
        // 为字段设置条件  
        List<Predicate> predicates = new ArrayList<>();  
  
        CriteriaBuilder.In<String> in = criteriaBuilder.in(name);  
        List<String> values = List.of("张三", "王五");  
        values.forEach(in::value);  
          
        predicates.add(in);  
        predicates.add(criteriaBuilder.equal(root.get("address").as(String.class), "黄山马图巷子"));  
        predicates.add(criteriaBuilder.greaterThan(age, 16));  
  
        return criteriaBuilder.and(predicates.toArray(new Predicate[0]));  
    }));  
  
    System.out.println(accounts);  
}

// 还有group...具体看源码 不过简单的count可以直接用方法名查询
@Test  
void useCriteriaQuery() {  
    List<Account> accounts = repository.findAll(((root, query, criteriaBuilder) -> {  
        return query  
                .where(criteriaBuilder.equal(root.get("address"), "黄山马图巷子"))  
                .orderBy(criteriaBuilder.asc(root.get("age")))      // 根据age升序排列  
                .getRestriction();  
    }));  
  
    System.out.println(accounts);  
}

// count   注意column使用having先要groupBy
@Test  
void useCriteriaQuery2() {  
    Long count = repository.count( ((root, query, criteriaBuilder) -> {  
        return query  
                .groupBy(root.get("address"), root.get("age"))  
                .having(criteriaBuilder.equal(root.get("address"), "黄山马图巷子"))  
                .having(criteriaBuilder.greaterThan(root.get("age"), 20))  
                .getRestriction();  
    }));  
  
    System.out.println(count);  
}
```

#### Querydsl
使用(因我的idea无法自动生成Q类，而运行报错暂时不整理)
### 关联查询
`jpa`并没有关联查询的配置，[请参考hibernate](https://hibernate.org/search/documentation/)

> 注意：
> 1. 保存关联数据时，如果使用已有的需要先从数据库中查出来(持久状态)
> 2. 如果一个业务方法中有多个持久化操作，需要加上`@Transactional`
> 3. 单元测试中使用`@Transactional`，按需添加`@Commit`
#### 一对一
接着上面Account，添加AccountDetail表 jpa会自动修改表结构，如添加外键等
```java
@Data   
@Entity  
public class AccountDetail {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    Integer id;  
  
    @Column(nullable = false)  
    String message;  
  
    LocalDateTime birtDay;  
}


// 修改Account类
...
public class Account {
	... 
	@JoinColumn(name = "detail_id")  
	@OneToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REMOVE}, orphanRemoval = true, fetch = FetchType.LAZY)  
	AccountDetail detail;
}
```

> 添加`cascade = CascadeType.ALL`设置关联操作 可选：`ALL`、`PERSIST`(插入)、`REMOVE`、`MERGE`(合并)  
  在关系后面添加`fetch= FetchType.LAZY`开启懒加载，注意懒加载属性需要在事务环境下获取，因为repository方法 调用完后Session会立即关闭 (非必需)  
  `orphanRemoval = true` 当`Account.setDetail(null)`会自动删除`account_detail`表里的对应数据(非必需)  
  `optional=false`限制关联对象不能为空

```java
@Test  
@Transactional  
void updateAccountAndDetail() {  
    repository.findById(1).ifPresent(account -> {  
        account.getDetail().setBirtDay(LocalDateTime.now().withNano(0));  
        System.out.println(repository.save(account));  
    });  
}
```

#### 双向一对一
双向关联会导致循环维护外键约束导致不能删除数据，需要将外键约束交给一方维护(添加`mappedBy`参数)
```java
// 修改Account
...
public class Account {
  ...
	@JoinColumn(name = "detail_id")  
	@OneToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REMOVE},  
				mappedBy = "account",       // 将外键约束执行交给另一方  
				orphanRemoval = true,  
				fetch = FetchType.LAZY)  
	AccountDetail detail;
  ...
}

// 修改AccountDetail
...
public class AccountDetail {
  ...
	@OneToOne  
	@JoinColumn  
	Account account;
  ...
}
```
#### 一对多、多对一
一对多、多对一都只有一个关联的字段，外键维护在一上，所以可以同时设置一对多、多对一

一对多
```java
@Data  
@Entity  
public class Classroom {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    Integer id;  
  
    String address;  
  
    // 声明在这里，但是外键会自动维护在student表里  
    @JoinColumn(name = "classroom_id")  
    @OneToMany(cascade = CascadeType.ALL)    // 一对多关系默认是懒加载 save()、delete()不需要事务执行
    List<Student> students;  
}

@Data  
@NoArgsConstructor  
@Entity  
public class Student {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    Integer id;  
  
    String name;  
  
    Integer age;  
  
    public Student(String name, Integer age) {  
        this.name = name;  
        this.age = age;  
    }  
}

// 更新 代码操作懒加载数据要添加 @Transactional
@Test  
@Transactional  
void addStudent() {  
    repository.findById(1).ifPresent(classroom -> {  
        classroom.getStudents().add(new Student("李四", 18));  
        System.out.println(repository.save(classroom));  
    });  
}
```

多对一 继续上面的例子 可以只修改Student
```java
@Data  
@NoArgsConstructor  
@Entity  
public class Student {  
   ...
	@JoinColumn(name = "classroom_id")  
	@ManyToOne(cascade = CascadeType.REMOVE)  
	Classroom classroom;
  ... 
}


@Test  
@Transactional  
void getStudent() {  
    Classroom classroom = new Classroom();  
    classroom.setId(1);  

    // 多表关联查询时，传入的classroom只需要设置Id即可   Student的repository
    List<Student> students = repository.findByClassroom(classroom);   
    System.out.println(students);        // 注意toString不要循环引用了会导致栈溢出
}
```

#### 多对多
`mysql`并没有多对多的关系，在建表时通常会将多对多拆成两个一对多(在多对多之间新建一张中间表)  

> 注意save时，set了多对多的对象，而对多对是懒加载，要开启事务

```java
// 这里举例老师和教室  在jpa中只需要声明多对多关系会自动创建一张中间表
@Data  
@Entity  
public class Classroom {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    Integer id;  
  
    String address;  
  
    // 声明在这里，但是外键会自动维护在student表里  
    @JoinColumn(name = "classroom_id")  
    @OneToMany(cascade = CascadeType.ALL)  // 默认开启懒加载
    List<Student> students;  
  
    // 单向多对多  
    @ManyToMany(cascade = CascadeType.PERSIST)  
    // 不写也会自动生成  joinColumns 本表在新建表里的外键名   inverseJoinColumns 另一个表...  
    @JoinTable(  
            name = "class_teacher",  
            joinColumns = {@JoinColumn(name = "classroom_id")},  
            inverseJoinColumns = {@JoinColumn(name = "teacher_id")}  
    )  
    List<Teacher> teachers;  
}

@Data  
@NoArgsConstructor  
@Entity  
public class Teacher {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    Integer id;  
  
    String name;  
  
    @ManyToMany(mappedBy = "teacher")  
    List<Classroom> classrooms;  
  
    public Teacher(String name) {  
        this.name = name;  
    }  
}

@Test  
void saveClassRoom() {  
    Classroom classroom = new Classroom();  
    classroom.setAddress("中角楼001");  
    classroom.setTeachers(List.of(new Teacher("张三丰"), new Teacher("张无忌")));  // 注意如果添加就需要设置 cascade
  
    repository.save(classroom);  
}
```

但上述方法无法为中间表添加字段，自行创建中间表解决
```java
@Entity
public class StudentCourseRegistration {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "student_id")
    private Student student;

    @ManyToOne
    @JoinColumn(name = "course_id")
    private Course course;

    @Temporal(TemporalType.DATE)
    private Date registrationDate;

    // 其他字段和方法
}
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "student")
    private List<StudentCourseRegistration> registrations;

    // 其他字段和方法
}

@Entity
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "course")
    private List<StudentCourseRegistration> registrations;

    // 其他字段和方法
}
```

### 乐观锁
防止并发修改  
在字段前中添加注解 `@Version` 如:`private @Version Long version`
### 审计
为每条数据添加额外信息，如：什么时候添加、谁添加、修改了哪里

开启自动审计需要在`@Configuration`类上添加`@EnableJpaAuditing`  
使用`@CreatedBy`和`@LastModifiedBy`来捕获创建或修改实体的用户，以及`@CreatedDate`和`@LastModifiedDate`来捕获变化发生的时间
```java
@Data  
@Entity  
@EntityListeners(AuditingEntityListener.class)
public class Account {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    Integer id;  
    @Column(nullable = false, unique = true)  
    String name;  
    @Column(nullable = false)  
    String password;  
    @Column(nullable = false, unique = true)  
    String email;  
    String role;  
    @CreatedDate  
	Instant createData;
    @LastModifiedDate  
    Instant modfiedDate;  
}
```

## Spring Data Redis
[Spring data Redis 官方网站](https://spring.io/projects/spring-data-redis)  
[Spring data Redis doc 中文站](https://springdoc.cn/spring-data-redis/)

`redis`的`java`客户端`Lettuce`和`Jedis`都没有数据库连接池，SpringDataRedis使用这两个客户端，因此需要自己添加连接池
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>  
    <groupId>org.apache.commons</groupId>  
    <artifactId>commons-pool2</artifactId>  
</dependency>
```

常用配置
```yml
spring:  
  data:  
    redis:  
      host: 127.0.0.1
      port: 6379
      database: 0
      timeout: 1000ms
      password:
      lettuce:  
        pool:  
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: -1ms
```

### RedisTemplate
`RedisTemplate`是`Redis`模块的中心类，提供了丰富的功能集，包括各种操作视图(针对`Redis`的各种类型进行操作)

> `template`是线程安全的，可以在多个实例中重复使用

`RedisConnection`提供了接受和返回二进制值（`byte` 数组）的低级方法，模板负责序列化和连接管理

`template`将Java对象转为原始数据通过`serializer`处理，包含两种类型：
1. 基于`RedisSerializer`的双向序列化器（serializer）。
2. 使用 `RedisElementReader`和`RedisElementWriter` 
    实现方式有：
	- `JdkSerializationRedisSerializer`，它被默认用于 `RedisCache` 和 `RedisTemplate`
	- `StringRedisSerializer`
	- `Jackson2JsonRedisSerializer`或`GenericJackson2JsonRedisSerializer`以`JSON`格式存储
	- etc.

> 众所周知，Java的序列化和反序列化并不安全，因此一般使用其他的消息格式(如JSON)来代替

配置一个`RedisTemplate`~~没修改序列化方式😀~~
```java
@Bean  
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {  
    RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();  
    redisTemplate.setConnectionFactory(redisConnectionFactory);
    return redisTemplate;  
}
```

> 由于存储在Redis中的key和value是很常见,`Spring Data Redis`提供了`StringRedisTemplate`和`StringRedisConnection`可以直接注入使用，其`Serializer`使用`StringRedisSerializer`

# Spring Security
[中文文档](https://springdoc.cn/spring-security/)  
网络安全框架，支持用户认证、授权等操作

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```
常见网站攻击方式:
1. CSRF(跨站请求伪造攻击) 访问钓鱼网站，网站非法获取你的cookie，然后使用你的cookie来访问其他网站
2. SFA(会话固定攻击) 在钓鱼网站登录正版网站，他使用自己的`JSESSIONID`发送登录请求
3. XSS(跨站脚本攻击) 在合法网站注入恶意脚本代码，用户访问恶意代码执行，窃取信息

> 常见`SFA`攻击方式:`1.会话传递(通过URL参数、表单隐藏字段、cookie等方式将会话ID传递给用户，当用户使用该会话ID登录时，攻击者就能利用该会话ID获取用户的权限) 2.会话劫持(劫持用户与服务器之间的通信流量，获取到用户的会话ID，然后利用该会话ID冒充用户进行操作或事先获取到会话ID，并将其分配给用户，之后通过其他方式欺骗用户登录)`
> 
> 常见`XSS`攻击方式: SQL注入、恶意代码嵌入URL、脚本注入

解决方法:
1. CSRF: 浏览器在不同域名的站点操作时，默认cookie会被自动屏蔽，而使得钓鱼网站不能调用其他网站的cookie(`SameSite` 安全机制)，也可以自己设置cookie的访问权限来避免
2. SFA: 将`JSESSIONID`设置为`HttpOnly`防止被跨站脚本读取或篡改，但在一些浏览器还是能读写，要彻底杜绝可以在用户登录后重新分配`JSESSIONID`
3. XSS: 不相信用户输入的一切事物，前后端接收输入，要对内容安全扫描
## 基础架构
`Spring Security`对`Servlet`的支持是基于`Servlet`过滤器(`Filter`)，这些`filter`可以用于许多不同的目的，如 [认证](https://springdoc.cn/spring-security/servlet/authentication/index.html)、 [授权](https://springdoc.cn/spring-security/servlet/authorization/index.html)、 [漏洞保护](https://springdoc.cn/spring-security/servlet/exploits/index.html) 等等

主要过滤链：`CsrfFilter`(防止CSRF攻击) -> `认证filter`(认证登陆) -> `ExceptionTranslationFilter`(处理 [`AccessDeniedException`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/AccessDeniedException.html)和[`AuthenticationException`](https://docs.spring.io/spring-security/site/docs/current/api//org/springframework/security/core/AuthenticationException.html) -> `AuthorizationFilter`(授权请求)
### 登陆异常处理
对于`ExceptionTranslationFilter`执行逻辑：
1. 首先`ExceptionTranslationFilter`调用`FilterChain.doFilter(request, response)`来调用应用程序的其他部分
2. 如果用户没有被认证，或者是一个`AuthenticationException`那么就开始认证
    - 清理掉SecurityContextHolder
    - 保存`HttpServletRequest`认证成功后，重放原始请求
    - `AuthenticationEntryPoint` 用于请求客户的凭证
3. 已经认证过，如果是 `AccessDeniedException`，那么就是 _Access Denied_。 `AccessDeniedHandler` 被调用来处理拒绝访问

> 如果应用程序没有抛出`AccessDeniedException`或`AuthenticationException`，那么`ExceptionTranslationFilter`就不会做任何事情

#### 客制异常处理
```java
// 通过配置exceptionHandling来自定义认证失败返回内容和执行过程
http.exceptionHandling((exceptions) -> exceptions  
        .authenticationEntryPoint(new RestfulAuthenticationEntryPoint())  
        .accessDeniedHandler(new RestfulAccessDeniedHandler())  
)
```

> `AuthenticationEntryPoint`用于发送一个要求客户端提供凭证的HTTP响应。

### 认证流程
用户是如何被重定向到登录表单的？
1. 用户向其未被授权的资源（`/private`）发出一个未经认证的请求
2. `AuthorizationFilter`通过抛出一个`AccessDeniedException`来表明请求未经认证被拒绝了
3. `ExceptionTranslationFilter`捕获到，调用`AuthenticationEntryPoint`后续处理(默认实现`LoginUrlAuthenticationEntryPoint`，返回302重定向到登陆界面)
4. 浏览器重定向到登录页面。

用户提交表单后流程：  
1. `UsernamePasswordAuthenticationFilter`从`HttpServletRequest`提取用户名和密码并创建`UsernamePasswordAuthenticationToken`(一种`Authentication`)  
2. `UsernamePasswordAuthenticationToken`被传入`AuthenticationManager`实例，进行认证。  
3. 认证失败：  
1. `SecurityContextHolder`清空  
2. `RememberMeServices.loginFail` 被调用  
3. `AuthenticationFailureHandler` 被调用  
4. 认证成功：  
1. `SessionAuthenticationStrategy`被通知有新登录  
2. `Authentication`被放入`SecurityContext`  
3. `ApplicationEventPublisher` 发布一个 `InteractiveAuthenticationSuccessEvent` 事件  
4. `AuthenticationSuccessHandler` 被调用

> 其他登录方式也大致如此(`Basic`、~~`Digest`~~)

> `AuthenticationManager`的细节取决于 [用户信息的存储方式](https://springdoc.cn/spring-security/servlet/authentication/passwords/storage.html)，因此我们可以暴露一个自定义的`UserDetailsService`作为一个`bean`来定义自定义认证。（`UserDetails`由`UserDetailsService`返回，`DaoAuthenticationProvider`从`UserDetails`中验证`Authentication`)

#### 客制表单登录
```java
http.formLogin(form -> form  
        // 客制登录页面url(默认走post)
        .loginProcessingUrl("/login").permitAll()
        // 客制登录成功回调方法(默认方法)  
        .successHandler(new SimpleUrlAuthenticationSuccessHandler())  
)

@Bean  
public UserDetailsService userDetailsService() {  
    //获取登录用户信息  
    return username -> memberService.loadUserByUsername(username);  
}
```

#### 客制认证凭证
`Spring Security`的认证模型的核心是`SecurityContextHolder`,因此最简单的方法是直接设置`SecurityContextHolder`来表明用户已被认证

> 流程：创建`SecurityContext` -> 创建`Authentication` -> 在`SecurityContextHolder`上设置`SecurityContext`

```java
// 创建认证凭证
SecurityContext context = SecurityContextHolder.createEmptyContext();  
Authentication authentication = new UsernamePasswordAuthenticationToken(userDetails, password, authorities);  
context.setAuthentication(authentication);  
SecurityContextHolder.setContext(context);

// 获取并使用认证凭证
SecurityContext context = SecurityContextHolder.getContext();  
Authentication authentication = context.getAuthentication();  
String username = authentication.getName();  
Object principal = authentication.getPrincipal();  
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```
### 权限
从上文可知权限认证是由`AccessDecisionManager`执行，那他如何读取访问用户权限呢？  
通过调用`GrantedAuthority`接口的`String getAuthority();`获得当前用户的权限。在用户/密码存储中有一种方式[UserDetailsService](https://springdoc.cn/spring-security/servlet/authentication/passwords/user-details-service.html#servlet-authentication-userdetailsservice)他返回的`UserDetails`需要我们实现一个`Collection<? extends GrantedAuthority> getAuthorities();`方法，现在就知道为什么要实现这个方法了

> `GrantedAuthority`返回的权限是一个精确的`String`表示，如果返回的结果不能被精确的表示为`String`，`getAuthority()`应该返回`null`

> `GrantedAuthority` 的常用实现类`SimpleGrantedAuthority`，使用其可以将自定义的`String`为权限

默认情况下角色权限规则使用`ROLE_`作为前缀，可以配置一个`GrantedAuthorityDefaults`来自定义前缀

> 在配置`hasRole("USER")`时需要获取的精确`String`为`ROLE_USER`
> 请使用`static`方法配置`GrantedAuthorityDefaults`，以确保在初始化`spring security`的`method security`之前发布

为角色设置包含关系
```java
@Bean
static RoleHierarchy roleHierarchy() {
    var hierarchy = new RoleHierarchyImpl();
    hierarchy.setHierarchy("ROLE_ADMIN > ROLE_STAFF\n" +
            "ROLE_STAFF > ROLE_USER\n" +
            "ROLE_USER > ROLE_GUEST");
    return hierarchy;
}

// and, if using method security also add
@Bean
static MethodSecurityExpressionHandler methodSecurityExpressionHandler(RoleHierarchy roleHierarchy) {
	DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
	expressionHandler.setRoleHierarchy(roleHierarchy);
	return expressionHandler;
}
```

### 注销
默认情况下，`Spring Security`会建立一个`/logout`端点，所以不需要额外的代码

> 由于`LogoutFilter`在`filter chain`中出现在`AuthorizationFilter`之前，所以默认情况下不需要明确允许 `/logout` 端点

```java
http.logout(logout -> logout  
        // 客制退出登录url  
        .logoutUrl("/api/logout")  
        // 客制退出登录处理  
        .logoutSuccessHandler((request, response, authentication) -> {})  
)
```

## JWT
[JWT官方文档](https://jwt.io/introduction)  
`“JSON Web Token (JWT) is defined a compact and self-contained way for securely transmitting information between parties as a JSON object.”`

JWT组成：`Header`、`Payload`、`Signature` looks like：xxx.yyy.zzz  
`Header`：通常包括两部分，令牌类型和使用的加密方法(在生产中还可添加一个`KeyID`，用来确定返回哪个密钥)  
`Payload`：由`claims`组成，`claims`有三种 _registered_, _public_, and _private_ claims

- _registered_:非必需但建议包含的内容，比如 **iss**(issuer), **exp**(expiration time), **sub**(subject), **aud**(audience)
- _public_：自定义的内容，比如 name("张三")
- _private_:除上面两者之外的，~~其实我也不知道什么时候用~~  

An example payload could be:
```json
{  
  "sub": "1234567890",  
  "name": "John Doe",  
  "admin": true  
}
```

`Signature`：由前两部分通过加密算法计算而来，`HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`

> 注意：`JWT`并不规定`Header`和`Payload`必须加密，一般直接转为`base64`传输，而`Signature`的作用是检验前两者是否被篡改。因此在非主动加密`Header`和`Payload`时，不要存入敏感信息

可以用作请求头或放在请求体里，请求头格式：`Authorization: Bearer <token>`

# Spring Mail
邮件发送模块 常用的邮件发送接收协议:`SMTP`和`POP3`
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

`SMTP`邮件发送，常见邮件服务SMTP主机host
```
QQ邮箱：SMTP服务器是smtp.qq.com，端口是465/587
163邮箱：SMTP服务器是smtp.163.com，端口是465
Gmail邮箱：SMTP服务器是smtp.gmail.com，端口是465/587
```
## 快速开始
常用配置
```yml
spring:  
  mail:  
    host: smtp.163.com
    username: xxxxx
    password: xxxxx
    default-encoding: UTF-8
    nickname: xxx
    port: 465
    properties:  
      mail:  
        smtp:  
          auth: true
          socketFactory:  
            class: javax.net.ssl.SSLSocketFactory
            port: 465
          ssl:  
            enable: true
          starttls:  
            enable: true
            required: true
```

注入`JavaMailSender` 进行邮件发送
```java
@Slf4j
@Component
public class sendMessage{
    @Autowired
    JavaMailSender sender;
    
    // 配置文件没找到spring.mail.username则默认为maifuqla@163.com
    @Value("${spring.mail.username:maifuqla@163.com}")
    String from;
    
    public void sendTxtMessage() {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setSubject("这是邮件主题");
        message.setText("这是邮件正文");
        message.setTo("邮件接收方");
        message.setFrom(from);
        try {
            sender.send(message);
            log.info("邮件发送成功")
        }catch (Exception e) {
            log.error("邮件发送失败")
        }
    }
    
    // 发送带有附件的 html 邮件
    public void sendAppendixMessage(String to, String subject, String content) {
        MimeMessage message = sender.createMimeMessage();
        try {
            MimeMessageHelper helper = new MimeMessageHelper(message, true); // true 表示多部分，可添加附件
            helper.setFrom(from);
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(content, true);  // true 表示为 html 邮件
            
            // 添加图片  html 引用 <img src='cid:logo'>
            helper.addInline("logo", new ClassPathResource("img/logo.jpg"));
            // 添加附件
            helper.addAttachment("logo.pdf", new ClassPathResource("doc/logo.pdf"));
            
            sender.send(message);
            log.info("邮件发送成功");
        }catch (Exception e) {
            log.error("邮件发送失败");
        }
    }
    
    // 发送模板解析引擎解析的 html 邮件   使用Thymeleaf
	public void sendTemplateMessage(String to, String subject, Map<String, Object> valueMap, String templateName) throws MessagingException {  
	    MimeMessage message = sender.createMimeMessage();  
	  
	    MimeMessageHelper helper = new MimeMessageHelper(message, true);  
	    message.setFrom(nickname + "<" + from + ">");  
	    helper.setTo(to);  
	    helper.setSubject(subject);  
	  
	    // 利用 Thymeleaf 引擎渲染 HTML    Context context = new Context();  
	    context.setVariables(valueMap);  // 设置注入的变量  
	    String content = templateEngine.process(templateName, context);  
	    helper.setText(content, true);  
	  
	    sender.send(message);  
	}
}
```
# 消息队列
消息队列是一种应用程序间异步服务调用的一种软件架构模式；它主要用来传递消息，在分布式系统中起到重要的作用
## RabbitMQ
[官方文档](https://www.rabbitmq.com/docs) 轻量、高效、开源、功能全面的消息队列`AMQP`协议，官方提供docker版本  

> 默认端口：5672 web端口：15672
### 常用命令
```bash
# 查看所有用户
rabbitmqctl list_users   
rabbitmqctl add_user username password
# 配置权限  vhost configure writer read
rabbitmqctl set_permissions -p "/" username ".*" ".*" ".*"  
rabbitmqctl list_user_permissions username
# 删除默认用户
rabbitmqctl delete_user guest

# 查看所有可用插件
rabbitmq-plugins list
# 开启web管理插件
rabbitmq-plugins enable rabbitmq_management  # disbale 关闭
# 开启日志管理插件
rabbitmq-plugins enable rabbitmq_tracing
```
### 核心概念
`VirtualHost`(虚拟消息服务器，他们之间相互隔离类似于`mysql`的`database`)  
`Exchange`(交换机，消息从生产者出来由交换机路由到队列中)  
`Queue`(队列，存储消息)  
`Binding`(绑定,定义`exchange`的路由规则)  
`Channel`(信道，客户端与`RabbitMQ Broker`的通信通道，`connection`的多路复用客户)  
`Connection`(客户端与`RabbitMQ Broker`之间的TCP连接)  
`RoutingKey`(路由键，将生产者的消息分配到交换机上)  
`BindingKey`(绑定键，将交换机的消息绑定到队列上)

交换机可选模式：`direct`、`topic`、`headers`、`fanout`  
`fanout`:将收到的所有消息广播到它知道的所有的队列  
`direct`:通过`bindingKey`、`routingKey`精准匹配传递消息  
`topic`:`routingKey`的约定`content_01.content_02.`传递消息

消息传递流程：  
生产者 -> bindingKey -> exchange -> routingKey -> queue -> comsumer

> `routingKey`还有两个特殊字符：`#`、`*` 
> 如：action.animal.reason -> run.rabbit.# -> 所有跑动的兔子

### 工作模式
1. "Hello World"
2. "Work Queues"
3. "Publish/Subscribe"
4. "Routing"
5. "Topics"
6. "RPC"
7. "Publisher Confirms"

### Spring 集成
[Spring amqp 官方网站](https://spring.io/projects/spring-amqp) [Spring amqp doc 中文站](https://springdoc.cn/spring-amqp/)
```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-amqp</artifactId>  
</dependency>
```

常用配置项
```xml
rabbitmq:  
  host: 127.0.0.1
  port: 5672  
  virtual-host: /
  username: admin
  password: admin
```

#### 配置消息转换器
```java
@Bean  
public MessageConverter messageConverter() {  
    return new Jackson2JsonMessageConverter();  
}
```

#### 配置死信
```java
@Bean(name = "deathDeclarable")  
public Declarables deathDeclarables() {  
    Queue queue = new Queue("death_queue");  
    Exchange exchange = ExchangeBuilder.directExchange("death_exchange")  
            .durable(true)  
            .build();  
    Binding binding = BindingBuilder.bind(queue)  
            .to(exchange)  
            .with("death_key")  
            .noargs();  
    return new Declarables(queue, exchange, binding);  
}  
  
@Bean("test_queue")  
public Queue testQueue() {  
    return QueueBuilder.durable("test_queue")  
            .deadLetterExchange("death_exchange")  
            .deadLetterRoutingKey("death_key")  
            .build();  
}
```
> 死信交换机也是普通交换机，只是人家出问题就将消息传了过来,ta就成了死信交换机

#### 发送延时消息
1. 下载[延时消息插件](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases)，将其放在`RabbitMQ`的`/plugins`文件夹
2. 启动延时插件`rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange`
3. 创建延时交换机和队列并绑定
4. 发送延时信息
```java
@Bean  
public Declarables delayDeclarables() {  
    Queue queue = new Queue("delay_queue");  
    Exchange exchange = ExchangeBuilder.directExchange("delay_exchange")  
            .delayed()  
            .durable(true)  
            .build();  
    Binding binding = BindingBuilder.bind(queue)  
            .to(exchange)  
            .with("delay_key")  
            .noargs();  
    return new Declarables(queue, exchange, binding);  
}


@Test
void testSendDelayMessage() {
	rabbitTemplate.convertAndSend("delay_exchange", "delay_key", "delayed message", new MessagePostProcessor() {  
	    @Override  
	    public Message postProcessMessage(Message message) throws AmqpException {  
       message.getMessageProperties().setDelayLong(10000L);  
	       return message;  
	    }  
	});
}
```
#### 消费者Retry
自动配置(自动 ACK) 在配置项添加  
消费者运行时抛出异常会自动重试超过最大次数会自动放入死信
```yml
listener:  
  simple:  
    retry:  
      enabled: true  
      max-attempts: 3  # 最大重试次数
      initial-interval: 1000ms  # 第一次重试等待时间
      multiplier: 5  # 每次重试时间倍率
      max-interval: 5000ms # 最大重试时间
```
手动配置(手动 ACK) 挖个坑先
```java
@RabbitListener(queues = "test_queue", ackMode = "MANUAL")
```
## Kafka


