[源码](https://github.com/alibaba/Sentinel)

# 一、Sentinel块

1. <font color=#FF0000 >sentinel-core 核心模块，限流</font>、降级、系统保护等都在这里实现
1. sentinel-dashboard 控制台模块，可以对连接上的sentinel客户端实现可视化的管理
1. sentinel-transport 传输模块，提供了基本的监控服务端和客户端的API接口以及一些基于不同库的实现
1. sentinel-extension 扩展模块，主要对DataSource进行了部分扩展实现
1. sentinel-adapter 适配器模块，主要实现了对一些常见框架的适配
1. sentinel-demo 样例模块，可参考怎么使用sentinel进行限流、降级等
1. sentinel-benchmark 基准测试模块，对核心代码的精确性提供基准测试

# 二、Sentinel配置

添加依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
    <version>1.8.0</version>
</dependency>
```

创建限流规则代码示例(也可将数据持久化到Apollo或数据库)：

```java
@Configuration
public class AopConfiguration implements InitializingBean {
    @Bean
    public SentinelResourceAspect sentinelResourceAspect(){
        return new SentinelResourceAspect();
    }

    @Override
    public void afterPropertiesSet() throws Exception{
        List<FlowRule> rules = new ArrayList<>();
        FlowRule rule = new FlowRule();
        //资源名，方法名等
        rule.setResource("Query");
        //限流策略，0代表直接流控
        rule.setStrategy(0);
        //限流类型，根据QPS来进行流控
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        //限流阈值
        rule.setCount(5);
        //流量控制效果，2代表均匀等待
        rule.setControlBehavior(2);
        //将限流规则假如list，可配置多条限流规则
        rules.add(rule);
        FlowRuleManager.loadRules(rules);
    }
}
```


# 三、Sentinel使用

通过注解的方法使用限流服务

`@SentinelResource(value="Query", blockHandler = "blockHandlerMethod", blockHandlerClass = BlockHandler.class)`

## 1. SentinelResource注解

该注解可作用于方法和接口、类、枚举;会在class字节码文件中存在，在运行时可以通过反射获取到。共包含以下字段信息，可赋值：

| 字段名  | 含义  | 具体要求  |
|:----------|:----------|:----------|
| value    | 资源名    |     |
| entryType    | 标记流量的方向    | 取值IN/OUT，默认为out    |
| resourceType    | 资源的类型    | 1.7.0版本开始新增字段，默认为0    |
| blockHandler    | 被限流后执行的方法名    | 必须是 public；<br>返回类型与原方法一致；<br>参数类型需要和原方法相匹配，并在最后加 BlockException 类型的参数。<br>默认需和原方法在同一个类中。<br>若希望使用其他类的函数，可配置blockHandlerClass ，并指定blockHandlerClass里面的方法    |
| blockHandlerClass    | 被限流执行方法所在的类名    | 对应的处理函数必须static修饰，否则无法解析    |
| fallback    | 在抛出异常的时候提供fallback处理逻辑    | 默认为""<br>返回类型与原方法一致；<br>参数类型需要和原方法相匹配，Sentinel 1.6开始，也可在方法最后加 Throwable 类型的参数；<br>默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定fallbackClass里面的方法。<br>1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。<br>可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理    |
| defaultFallback    | 通用的 fallback 逻辑    |   默认为""<br>若同时配置了 fallback 和 defaultFallback，以fallback为准 |
| fallbackClass    | Cell 2    | 默认为{}<br>对应的处理函数必须static修饰，否则无法解析  |
| exceptionsToTrace    | 需要trace的异常    | 默认为Throwable.class    |
| exceptionsToIgnore    | 指定排除掉哪些异常    | 默认为{} <br> 排除的异常不会计入异常统计，也不会进入fallback逻辑，而是原样抛出 |


## 2. Sentinel配置限流规则参数

一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：

- resource：资源名，即限流规则的作用对象
- count: 限流阈值
- grade: 限流阈值类型：0 代表根据并发数量来限流，1 代表根据 QPS 来进行流量控制
- limitApp: 流控针对的调用来源，若为 default 则不区分调用来源
- strategy: 调用关系限流策略
- controlBehavior: 流量控制效果（直接拒绝、Warm Up、匀速排队）
> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。

无参构造函数：默认limitApp为"default"

带资源名构造：设置资源名及limitApp值

流控主要由3个因素组成：grade、strategy、controlBehavior

grade默认取值：1

```
public static final int FLOW_GRADE_THREAD = 0; 线程数
public static final int FLOW_GRADE_QPS = 1; QPS
```

strategy默认取值：0

```
public static final int STRATEGY_DIRECT = 0; 直接流控
public static final int STRATEGY_RELATE = 1; 相关流控
public static final int STRATEGY_CHAIN = 2; 链流控制
```

controlBehavior默认取值：0

```
public static final int CONTROL_BEHAVIOR_DEFAULT = 0; 直接拒绝
public static final int CONTROL_BEHAVIOR_WARM_UP = 1; 冷启动
public static final int CONTROL_BEHAVIOR_RATE_LIMITER = 2; 均匀等待
public static final int CONTROL_BEHAVIOR_WARM_UP_RATE_LIMITER = 3; 冷启动+均匀等待
```

count：阈值

warmUpPeriodSec默认取值： 10，配合冷启动策略使用

maxQueueingTimeMs默认取值： 500，配合均匀等待策略使用

类中还包含equals、hashCode、toString方法

> 限流规则可在Apollo进行配置，当值发生修改时，可以立马被读取到