
----


`cloneable`接口和`clone`方法

`Cloneable` 是 Java里的一个标记接口，让这个类支持 `clone()` 复制对象，否则调用 `clone()` 会报不支持克隆异常

通常需要重写`clone`方法：因为 `Object` 里的 `clone()` 是 `protected`，外部一般不能直接调

`super.clone()` 默认做的是***浅拷贝***
- 引用类型字段：复制的是*引用地址*，不是对象本身


---

**函数式接口**是指**仅包含一个抽象方法**的接口（可用 `@FunctionalInterface`注解标记，帮助编译器检查）
核心作用是：**允许用“行为”（方法）作为参数传递**（类似“把函数当参数用”，这是函数式编程的思想）


---


**方法引用**是 Java 8 引入的语法糖，**用于直接引用已有类或对象的方法**，而不需要重新实现该方法。本质是：当你需要一个“函数式接口的实现”时，直接用已有的方法代替。
语法格式之一是：**`类名/实例名::方法名`**

```Java
@FunctionalInterface
public interface Refreshable {
    boolean refresh(long version, long timestamp, ProcessType processType);
}

class SourceAwGeoService {
	boolean refreshScene(long version, long timestamp, ProcessType processType)
}

submit(sourceAwGeoService::refreshScene, ...)
```
submit方法第一个参数需要一个Refreshable类型对象（Refreshable是函数式接口，其实现类实例可作为参数）
sourceAwGeoService::refreshScene 是方法引用，它表示：“调用 sourceAwGeoService的 refreshScene方法”。
Java 会自动将 `sourceAwGeoService::refreshScene`识别为 `Refreshable`接口的一个**实现**（因为 `refreshScene`的参数列表 `(long, long, ProcessType)`和返回值 `boolean`，与 `Refreshable`接口的 `refresh`方法的签名完全一致）
"方法引用所对应的方法，能否适配函数式接口的那个唯一抽象方法"


---


RabbitMQ
生产者 -> Exchange -> Queue -> 消费者
- 生产者 `rabbitTemplate.convertAndSend(...)`
- message：发出的消息
- queue：RabbitMQ 里真正存消息的地方，消费者监听 queue，一条消息通常：
	1. 先发到 exchange
	2. 再被路由到 queue
	3. 然后消费者从 queue 里取
- 消费者 `@RabbitListener`
- exchange交换机：exchange**不存消息**，它只负责：收到消息后，决定**把消息投递到哪些 queue**
- binding：把某个 queue 和某个 exchange **建立关系**
- routingKey路由键：发消息时带的一个“路由标记”

exchange常见类型
- FanoutExchange 广播模式
	- 不看 routingKey
	- 发到这个 exchange 的消息，会发给所有绑定的队列
- DirectExchange 精确匹配模式
	- 看 routingKey
	- queue 绑定时也会指定 bindingKey
	- 只有 routingKey 和 bindingKey 一样，消息才会进这个 queue
- TopicExchange 模糊匹配模式
	- routingKey 支持通配符
	- 常用于按主题分类

RabbitMQ里常见的几个机制
异步、解耦、削峰、多消费者、广播


---


`LocalDateTime.of(LocalDate.now(), LocalTime.MIN).toInstant(ZoneOffset.of("+8")).toEpochMilli()`
- LocalDate.now()：取当前日期
- LocalTime.MIN：表示一天的最小时间
- LocalDateTime.of(LocalDate.now(), LocalTime.MIN)：把“今天的日期”和“00:00”拼成一个时间
- .toInstant(ZoneOffset.of("+8"))：把这个“本地时间”按东八区来解释，并转换成 Instant（）
- .toEpochMilli()：把这个时间点转成：从 1970-01-01 00:00:00 UTC 到该时刻的毫秒数


----

jdbcTemplate.queryForObject

    @Autowired
    @Qualifier("mysqlJdbcTemplate")
    private JdbcTemplate jdbcTemplate;


        int ipVersion = this.getMetadata().getIpVersion();


AWReader 类



形参是接口，实参可以是这个接口的实现类


KeyHolder holder = new GeneratedKeyHolder();

//对响应进行流式处理而不是将其全部加载到内存中