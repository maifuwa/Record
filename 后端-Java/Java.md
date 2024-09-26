# 新特性
## Stream流
使用`Stream`函数式编程
```java
// 数组Stream包装
Stream.of("a", "b", "c");
// 集合类Stream包裹
List.of("a", "b", "c").stream();

// 使用filter过滤元素 传入一个方法来判断是否符合条件， true保存，false过滤掉
List.of("a", "b", "c").stream().filter(s -> "a".equals(s));

// 使用map修改元素 传入一个方法返回新元素(可以修改元素类型)
List.of("a", "b", "c").stream().map(String::toUpperCase);

// 使用reduce聚合元素 传入一个参数和一个方法与所有元素运算后返回结果
List.of(1,2,3,4).stream().reduce(0, (acc, n) -> acc + n); // acc 是上一次计算的结果

// 输出数组或集合
List.of("a", "b", "c").stream().toArray(String[]::new);  
Stream.of("a", "b", "c").toList();

List.of("a", "b", "c").stream().collect(Collectors.toSet());

List.of("a", "b", "c").stream().collect(Collectors.toMap(s -> s.toLowerCase(), s -> s.toUpperCase()));

// 分组输出 

// 其他
sorted  //排序
distinct // 去重
skip(2) // 跳过前2个元素
limit(2) // 截取前2个元素
flatMap // 一对多转换，将每个元素映射为一个流，将多个流扁平为一个单一的流 
parallel // 并行流  启用多线程，会自动以多个线程执行后续方法

Stream.concat(s1, s2) // 合并两个Stream，静态方法


// 使用 stream 分割 list
private static <T> List<List<T>> splitList(List<T> list, int size) {  
    Map<Integer, List<T>> result = IntStream.range(0, list.size())  
            .boxed()  
            .collect(Collectors.groupingBy(i -> i / size, Collectors.mapping(list::get, Collectors.toList())));  
  
    return List.copyOf(result.values());  
}

// Collectors.mapping(Function<? super T,? extends U> mapper,Collector<? super U,A,R> downstream) 前面是类型转换方法, 后面是聚合方法

//<T,K> Collector<T,?,Map<K,List<T>>> Collectors.groupingBy(Function<? super T,? extends K> classifier) 传入分类函数 
fullPathList.stream().collect(Collectors.groupingBy(FullPath::getStatisType));

// Collectors.groupingBy(Function<? super T,? extends K> classifier, Collector<? super T,A,D> downstream)  传入分类函数和聚合方法

```

Java 的 Stream 在以下情况下可能会有多线程风险：
1. **使用并行流（Parallel Stream）时**：
    - 并行流会将任务分成多个子任务并行执行，如果操作不是线程安全的，可能会导致数据竞争和不一致性
2. **共享可变状态**：
    - 如果流操作中使用了共享的可变状态（例如，外部变量或集合），并且这些状态在多个线程中被修改，可能会导致线程安全问题
3. **非线程安全的终端操作**：
    - 终端操作（例如 `collect`）如果使用了非线程安全的集合或数据结构，可能会导致并发修改异常或数据不一致
4. **自定义的非线程安全的中间操作**：
    - 如果自定义了中间操作，并且这些操作不是线程安全的，也可能会导致多线程问题
## Optional
非空判断
```java
public void lowerCase(String str) {
    if (str != null && !str.isBlank()) {
        sout(str.toLowerCase());
    }
}

// 使用 Optional
public void lowerCaseByOptional(String str) {
    Optional
        .ofNullable(str)    // 将str包装进Optional   也可使用.of()包装，但不接受null
        // 对象不为null执行 传入方法(接受一个参数，返回值为void)  isPresent()
        .ifPresent(System.out::println);    
}

// 可以直接从Optional中获取被包装的对象 Optional.ofNullable(object).get(); .orElse();即使不为null也会执行里面的代码
public String other(String str) {
    return Optional.ofNullable(str).orElseGet("str 为 null");    // 当对象为null时返回 "str 为 null"
}

// 为空时传入一个方法，抛出异常
Optional.ofNullable(str).orElseThrow(() => new RuntimeException("str为空"))

// 也提供 map、flatMap、filter 等流式操作
```
## 日期
新的日期`api`
本地日期、时间 `LocalDateTime`、`LocalDate`、 `LocalTime`
```java
/* 
* 注意ISO 8601标准格式
* 日期：yyyy-MM-dd
* 时间：HH:mm:ss
* 带毫秒的时间：HH:mm:ss.SSS
* 日期和时间：yyyy-MM-dd'T'HH:mm:ss
* 带毫秒的日期和时间：yyyy-MM-dd'T'HH:mm:ss.SSS
*/

// 要直接获取本地当前时间
LocalDateTime dt = LocalDateTime.now();   // 2023-09-26T10:53:08.692623418
// 将时间拆成日期和时间
LocalDate d = dt.toLocalDate(); // 转换到当前日期  
LocalTime t = dt.toLocalTime(); // 转换到当前时间

// 自定义时间
LocalDateTime dt = LocalDateTime.parse("2023-09-26T10:53:08.692623418");

// 自定义输出格式
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");  
System.out.println(dtf.format(dt));      // 2023/09/26 11:00:49
// 自定义解析格式
LocalDateTime dt = LocalDateTime.parse("2023-09-26T10:53:08.692623418", dtf);

// 日期、时间计算
LocalDateTime newDate = dt.plusDay(5).minusHours(3);    // 增加五天，减少三小时

// 直接修改日期、时间
LocalDateTime dt = LocalDateTime.now().withNano(0)   // 将时间后面的纳秒归零

// 比较前后使用 isBefore() isAfter()
```

时间之间的时区转换`ZonedDateTime`
```java
// 创建ZonedDateTime实例
ZonedDateTime zdt = ZonedDateTime.now();  // 默认时区(根据电脑默认时区)  中国的ZoneId: Asia/Shanghai
ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York"));  // 获取指定时区时间

// 由LocalDateTime转换
ZonedDateTime zbj = LocalDateTime.now().atZone(ZoneId.systemDefault()); 

// 时区转换  注意时区转换，千万不要自己计算时差，各地有各地的时区选择和时间政策(夏令时...)
ZonedDateTime zny = zbj.withZoneSameInstant(ZoneId.of("America/New_York"));

// 也能进行同上的日期加减操作
```

格式化显示`DateTimeFormatter`  (不是不变对象，而且线程安全)
```java
// 基础使用
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");

// 指定Locale本地化 中国：Locale.CHINA  E表示星期数
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("E, yyyy-MMMM-dd HH:mm", Locale.US);
```

时间戳`Instant`
```java
// 和原本的 Date 差不多
Instant now = Instant.now();
Instant ins = Instant.ofEpochSecond(1568568760);

// 转换为时间戳(long)
now.getEpochSecond();  // 秒
now.toEpochMilli();    // 毫秒

// 转换为ZonedDateTime
ZonedDateTime zdt = ins.atZone(ZoneId.systemDefault());
```

# 日志
日志目前分为三层：接口层、实现层、适配层
> 接口层`interface`：只定义接口
> 实现层：真正干活的，但不是接口层的实现
> 适配层`adapter`：将接口层和实现层连接起来

适配层绑定实现层桥接到接口层，比如：`Slf4j(Slf4j-api)` -> `slf4j-jcl` -> `JCL`
每个实现层有很多适配层桥接到不同接口层，反正也亦然

> 适配层会依赖实现层和接口层，项目需要添加日志只需要选择实现层和接口层然后添加两者适配层的依赖即可(注意与其他库的接口层接口匹配)

常见的日志解决方案: `Slf4j` + `Logback` 、`Slf4j` + `Log4j 2`、`Log4j 2`、`Logging`
# Lombok
自动为`pojo`添加`get/set`方法、构造器方法、支持链式访问`pojo` 还支持日志输出

> get/set`@Data`、全参构造`@AllArgsConstructor`、无参构造`@NoArgsConstructor`、为`final`或`not-null`字段生成构造`@RequiredArgsConstructor`、日志`@Slf4j`

```xml
<!-- 普通项目引入 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>{lombok-version}</version>
    <scope>annotationProcessor</scope>
</dependency>   
```

`@Builder` 链式调用创建类
```java
@Data  
@Builder  
public class Article {  
    String title;  
    @Builder.Default
    Instant createdTime = Instant.now();  
}

void someTest() {
	Article article = Article.builder()  
	    .title("title")  
	    .build();
}
```

> `@Builder`模式字段有默认值时，需要在字段上添加`@Builder.Default`

# fastJson2
[fastjson2 APIdoc](https://javadoc.io/doc/com.alibaba.fastjson2/fastjson2/2.0.40/com/alibaba/fastjson2/JSONObject.html)
`fastjson2` 是阿里巴巴的一个Json解析工具效率高于Jackson、Gosn，能够同时用于web、android项目且api一致
```xml
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>{fastjson2.version}</version>
</dependency>
```
## Json解析
```java
String json = "...";
byte[] json = "...";

JSONObject data = JSON.parseObject(json);
JSONArray data = JSON.parseArray(json);

Account account = JSON.parseObject(json, Account.class);

// 会自动忽略 null字段
String json = JSON.toJSONString(account);
byte[] byJson = JSON.toJSONBytes(account);
```

## JSONObject、JSONArray
获取简单属性 注意只能拿到第一层的数据
```java
String json = "{\"id\": 2,\"name\": \"fastjson2\"}";
JSONObject obj = JSON.parseObject(json);

int id = obj.getIntValue("id");
JSONObject id = obj.getJSONObject("id");


String json = "[2, \"fastjson2\"]";
JSONArray array = JSON.parseArray(json);

int id = array.getIntValue(0);
String name = array.getString(1);
```
转化为实体类
```java
JSONArray array = ...
JSONObject obj = ...

Account account = array.getObject(0, Account.class);
Account account = obj.getObject("Account", Account.class);

List<Account> accounts = array.toJavaList(Account.class);
Account account = obj.toJavaObject(Account.class);
```
实体类转Json
```java
Account account = ...

// 实体类需要有setter/getter方法
// 不忽略 null字段 
String json = JSONObject.toJSONString(account, JSONWriter.Feature.WriteNulls);
```
# Guava
`Guava`是由`Google`开发和维护的一套`Java`库，提供实用的工具类和`api`，包括集合操作、并发工具、字符串处理、`I/O`操作、数学工具等等
## 缓存
`Guava`基于`ConcurrentHashMap`实现缓存，它使用了分段锁机制来保证线程安全，并提供了多种缓存清理策略
## 限流
使用`guava`的限流工具`RateLimiter`做单体限流，`RateLimiter` 基于令牌桶算法，可以应对突发流量。还提供了**平滑预热限流**的算法实现
> 平滑突发限流就是按照指定的速率放令牌到桶里，而平滑预热限流会有一段预热时间，预热时间之内，速率会逐渐提升到配置的速率。

简单使用
```java
@Test  
void testRateLimiter() {  
	// 1s 放入 5个 令牌
    RateLimiter rateLimiter = RateLimiter.create(5.0);  
    // 1s 放入 5个 令牌,在3s 内发牌速率会逐渐提升到 0.2s 放 1 个令牌到桶里
    // RateLimiter rateLimiter = RateLimiter.create(5, 3, TimeUnit.SECONDS);
    for (int i = 0; i < 10; i++) {  
	    // 申请令牌，并设置最大超时时间为 1ms
        boolean tryAcquire = rateLimiter.tryAcquire(1, TimeUnit.MILLISECONDS);  
        log.info("tryAcquire: {}", tryAcquire);  
    }  
}
```

配合`Spring aop`使用
```java
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.METHOD})  
@Documented  
public @interface Limit {  
  
    String key() default "";  
  
    double permitsPerSecond();  
  
    long timeout();  
  
    TimeUnit timeunit() default TimeUnit.MILLISECONDS;  
  
    String message() default "系统繁忙，请稍后再试";  
}
 
@Slf4j  
@Aspect  
@Component  
public class LimitAop {  
  
    private final Map<String, RateLimiter> limiterMap = Maps.newConcurrentMap();  
  
    @Around("@annotation(com.example.demo.Limit)")  
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {  
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();  
        Method method = signature.getMethod();  
        Limit limit = method.getAnnotation(Limit.class);  
        if (limit != null) {  
            String key = limit.key();  
            RateLimiter rateLimiter = null;  
            if (!limiterMap.containsKey(key)) {  
                rateLimiter = RateLimiter.create(limit.permitsPerSecond());  
                limiterMap.put(key, rateLimiter);  
                log.info("create rateLimiter key={}, capacity={}", key, limit.permitsPerSecond());  
            }  
            rateLimiter = limiterMap.get(key);  
  
            boolean acquire = rateLimiter.tryAcquire(limit.timeout(), limit.timeunit());  
  
            if (!acquire) {  
                log.info("Current limit, key={}", key);  
                throw new RuntimeException(limit.message());  
            }  
        }  
  
        return joinPoint.proceed();  
    }  
  
}
```
# Apache Commons
`Apache Commons`是`Apache`基金会的项目，目的是提供可重用的、开源的`Java`代码
### Lang3
补充`Java`标准库做的工具库，为文本处理、日期操作、数学计算、系统属性、反射、字符串操作进行了补充(jdk >= 8)
> 主要提供了 `StringUtils`、`ArrayUtils`、`NumberUtils`、`DataUtils`、`RandomStringUtils`、`SystemUtils`、`WordUtils`

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>${commons-lang3.version}</version>
</dependency>
```

### Net
实现了许多基本Internet协议的客户端。库的目的是提供基本的协议访问，而不是更高级别的抽象

```xml
<dependency>
    <groupId>commons-net</groupId>
    <artifactId>commons-net</artifactId>
    <version>${{commons-net.version}}</version>
</dependency>
```

# 定时任务
定时后单次执行(`delay`)，约定周期定时执行(`initialDelay`和`period`)
## ScheduledExecutorService
Java标准库中的定时任务服务，常用实现类`ScheduledThreadPoolExecutor` jdk > 1.5
> `ScheduledThreadPoolExecutor` 继承了`ThreadPoolExecutor`且实现了`ScheduledExecutorService`，所以它既是定时器又是线程池

创建实例
![[ScheduledThreadPoolExecutor实例化方法.png]]
```java
// 使用的等待队列为`DelayedWorkQueue`无界队列，再多的任务都能扔进来，有可能OOM
public ScheduledThreadPoolExecutor(int corePoolSize) {  
    super(corePoolSize, Integer.MAX_VALUE,  
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,  
          new DelayedWorkQueue());  
}
```

提交任务
![[ScheduledThreadPoolExecutor提交任务.png]]
> `fixedRate`和`fixedDelay` 前者按照`period`执行新任务，后者等待当前任务执行完后再等待`period`执行新任务

## DelayQueue
`Java`标准库中的延时队列，基于优先级队列和堆排序算法
1. 定义`Task`类并实现`Delayed`
2. 创建`DelayQueue`队列，将`task`传入队列
3. 从队列中取出`task`，执行

最佳实践
```java
public class DelayedTask implements Delayed {  
  
    private final long executeTime;  
  
    private final Runnable task;  
  
    public DelayedTask(long delay, Runnable task) {  
        this.executeTime = System.currentTimeMillis() + delay;  
        this.task = task;  
    }  
  
    public void execute() {  
        task.run();  
    }  
  
    @Override  
    public long getDelay(TimeUnit unit) {  
        return unit.convert(executeTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);  
    }  
  
    @Override  
    public int compareTo(Delayed o) {  
        return Long.compare(this.executeTime, ((DelayedTask) o).executeTime);  
    }  
}

// 阻塞调用
public static void main(String[] args) {  
    DelayQueue<DelayedTask> delayedTasks = new DelayQueue<>();  
    DelayedTask task1 = new DelayedTask(1000, () -> System.out.println("task1"));  
    delayedTasks.add(task1);  
    while (!delayedTasks.isEmpty()) {  
        DelayedTask task = delayedTasks.poll();  
        if (task != null) {  
            task.execute();  
        }  
    }  
}
```
## Quartz
[官方文档](https://www.quartz-scheduler.org/documentation/quartz-2.2.2/tutorials/)
提供了灵活可靠的方式来管理和执行定时任务的任务调度框架

关键API: `Scheduler`、`Job`、`JobDetail`、`JobBuilder`、`Trigger`、`TriggerBuilder`
`Scheduler`: 执行任务的调度器
`Job`: 执行的任务
`JobDetail`: 任务的详细信息(调度器存储这个)
`Trigger`: 任务的触发器(时间触发)
### JobStore
`JobStore`负责跟踪提供给调度程序的所有工作数据
> 可以提供给用于生成调度程序实例的`SchedulerFactory`的属性文件或对象中声明使用哪个`JobStore`(及其配置)
> 永远不要在代码中直接使用`JobStore`实例

常用的三种`JobStore`实例：`RAMJobStore`、`JobStoreTX`、`JobStoreCMT`
- `RAMJobStore`基于`RAM`存储数据，因此速度最快配置简单，但当程序结束所有调度信息都会丢失
- `JobStoreTX`、`JobStoreCMT`使用数据库存储数据(`JDBC`)，如果不需要将调度命令（例如添加和删除触发器）绑定到其他事务选择前者，后者反之

> 使用`JDBCJobStore`需要配置数据源和初始化数据库，`Quartz`适配了大部分数据库，内置了数据库连接驱动并可自动初始化数据库

配置数据源：
- 让`Quartz`创建和管理数据源本身 - 通过提供数据库的所有连接信息
- 让`Quartz`使用由`Quartz`在其中运行的应用程序服务器管理的数据源 - 通过为`JDBCJobStore`提供数据源的`JNDI`名称

数据库驱动(`DriverDelegate`)：常用的`StdJDBCDelegate`，对于使用其发生问题时可选择对应数据库对应的驱动，从`org.quartz.impl.jdbcjobstore`包和其子包中寻找

> 如果线程池时常满载，对于数据库连接池的大小应为`threadPool + 2`
### Cron
定时任务表达式，格式：`second minute hour day month week [year]`
可填数组或字符`* ? - , / L W C #`
> `*` 表示每一个，放在`day`就是每天
> `-` 表示范围，如`5-7`放在`day`上就是`5号~7号`
> `,` 表示列表，如`5,7`放在`day`上就是`5号和7号`
> `/` 表示等长序列，如`1/5`放在`day`上就是`1号、6号...(每5天执行一次任务)`
> `?` 表示无意义，只能在`day`和`week`中使用
> `L` 表示`Last`，只能在`day`和`week`中使用
> `W` 表示距当前最近的工作日不可以跨月，只能在`day`中使用

如：`0/20 0/30 * * 7,8 ? *` 表示`Every 20 seconds, every 30 minutes, every hour, every day, only in July and August`

# 生成ID
[ID使用场景](https://javaguide.cn/distributed-system/distributed-id-design.html)
## UUID
UUID(128 bit)
```java
// 36f257af-599b-4ea0-8fc1-d0854bfc6e08
String uuid = UUID.randomUUID().toString();
uuid.length() = 32;
```
优点：生成快、操作简单  
缺点：存储空间大、无顺序、不安全(基于 MAC 地址生成 UUID 的算法会造成 MAC 地址泄露)、没有业务意义、存在ID重复问题(机器时间不对，可能出现)

## 雪花算法(Snowflake)
Snowflake(64bit) Twitter开源的ID生成算法
![[snowflake-distributed-id-schematic-diagram.png]]
- **sign(1bit)**:符号位（标识正负），始终为 0，代表生成的 ID 为正数。
- **timestamp (41 bits)**:一共 41 位，用来表示时间戳，单位是毫秒，可以支撑 2 ^41 毫秒（约 69 年）
- **datacenter id + worker id (10 bits)**:一般来说，前 5 位表示机房 ID，后 5 位表示机器 ID（实际项目中可以根据实际情况调整）。这样就可以区分不同集群/机房的节点。
- **sequence (12 bits)**:一共 12 位，用来表示序列号。 序列号为自增值，代表单台机器每毫秒能够产生的最大 ID 数(2^12 = 4096),也就是说单台机器每毫秒最多可以生成 4096 个 唯一 ID

优点：生成快、有序、灵活(可加入业务信息)  
缺点：时钟敏感(时钟回拨)、突发性能有上限

第三方实现：[Leaf](https://github.com/Meituan-Dianping/Leaf)：美团开源的分布式ID解决方案，提供了段号模式和Sonwflake、[IdGenerator](https://github.com/yitter/IdGenerator)：小体量项目使用

简单实现
```java
public class SnowflakeIdGenerator {  
    // 起始的时间戳  
    private final static long START_TIMESTAMP = 1704038400000L; // 2024-01-01 00:00:00  
  
    // 每部分占用的位数  
    private final static long SEQUENCE_BIT = 12; // 序列号占用的位数  
    private final static long WORKER_BIT = 10; // 工作机器ID占用的位数  
  
    // 每部分的最大值  
    private final static long MAX_SEQUENCE = ~(-1L << SEQUENCE_BIT);  
    private final static long MAX_WORKER_ID = ~(-1L << WORKER_BIT);  
  
    // 每部分向左的位移  
    private final static long WORKER_LEFT = SEQUENCE_BIT;  
    private final static long TIMESTAMP_LEFT = SEQUENCE_BIT + WORKER_BIT;  
  
    private long workerId; // 工作机器ID  
    private long sequence = 0L; // 序列号  
    private long lastTimestamp = -1L; // 上次生成ID的时间戳  
  
    public SnowflakeIdGenerator(long workerId) {  
        if (workerId > MAX_WORKER_ID || workerId < 0) {  
            throw new IllegalArgumentException("Worker ID can't be greater than " + MAX_WORKER_ID + " or less than 0");  
        }  
        this.workerId = workerId;  
    }  
  
    public synchronized long nextId() {  
        long timestamp = System.currentTimeMillis();  
  
        if (timestamp < lastTimestamp) {  
            throw new RuntimeException("Clock moved backwards. Refusing to generate ID");  
        }  
  
        if (timestamp == lastTimestamp) {  
            sequence = (sequence + 1) & MAX_SEQUENCE;  
            if (sequence == 0) {  
                timestamp = tilNextMillis(lastTimestamp);  
            }  
        } else {  
            sequence = 0L;  
        }  
  
        lastTimestamp = timestamp;  
  
        return ((timestamp - START_TIMESTAMP) << TIMESTAMP_LEFT)  
                | (workerId << WORKER_LEFT)  
                | sequence;  
    }  
  
    private long tilNextMillis(long lastTimestamp) {  
        long timestamp = System.currentTimeMillis();  
        while (timestamp <= lastTimestamp) {  
            timestamp = System.currentTimeMillis();  
        }  
        return timestamp;  
    }  
}
```

# 限流算法
限流就是对请求的速率进行限制，避免瞬时的大量请求击垮软件系统
## 单机限流
常用的限流算法有:
- 固定窗口计数器算法
- 滑动窗口计数器算法
- 漏桶算法
- 令牌桶算法

> 令牌桶算法：系统向令牌桶中放入令牌，每个令牌代表一个可用的访问请求。当一个请求到达时，必须从令牌桶中获取一个令牌，只有获取到令牌的请求才会被允许进入系统，否则将被限制或延迟处理。令牌桶算法可以限制平均速率和应对突然激增的流量，还可以动态调整生成令牌的速率。不过，如果令牌产生速率和桶的容量设置不合理，可能会出现问题比如大量的请求被丢弃、系统过载

限流对象：ip、业务ID、用户个性化、调用关系...

