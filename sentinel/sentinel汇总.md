[源码](https://github.com/alibaba/Sentinel)

# 一、源码结构简介

1. <font color=#FF0000 >sentinel-core 核心模块，限流</font>、降级、系统保护等都在这里实现
1. sentinel-dashboard 控制台模块，可以对连接上的sentinel客户端实现可视化的管理
1. sentinel-transport 传输模块，提供了基本的监控服务端和客户端的API接口以及一些基于不同库的实现
1. sentinel-extension 扩展模块，主要对DataSource进行了部分扩展实现
1. sentinel-adapter 适配器模块，主要实现了对一些常见框架的适配
1. sentinel-demo 样例模块，可参考怎么使用sentinel进行限流、降级等
1. sentinel-benchmark 基准测试模块，对核心代码的精确性提供基准测试

# 二、Sentinel使用篇

添加依赖：

```
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-annotation-aspectj</artifactId>
    <version>1.8.0</version>
</dependency>
```

创建限流规则代码示例(也可将数据持久化到Apollo或数据库)：

```
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
        rule.setResource("Query");
        rule.setStrategy(0);
        rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
        rule.setCount(5);
        rule.setControlBehavior(2);
        rules.add(rule);
        FlowRuleManager.loadRules(rules);
    }
}
```

通过注解的方式进行限流：

`@SentinelResource(value="Query", blockHandler = "blockHandlerMethod", blockHandlerClass = BlockHandler.class)`


# 三、Sentinel核心篇

通过注解的方法接入限流服务

## 1. SentinelResource注解

该注解可作用于方法和接口、类、枚举;会在class字节码文件中存在，在运行时可以通过反射获取到。共包含以下字段信息，可赋值：

- value：资源名
- entryType：标记流量的方向，取值IN/OUT，默认为out
- resourceType：1.7.0版本开始新增字段，资源的类型。默认为0。
- blockHandler：被限流后执行的方法名。具体要求如下：

       必须是 public；
       返回类型与原方法一致；
       参数类型需要和原方法相匹配，并在最后加 BlockException 类型的参数。
       默认需和原方法在同一个类中。若希望使用其他类的函数，可配置blockHandlerClass ，并指定blockHandlerClass里面的方法。
- blockHandlerClass：被限流执行方法所在的类名。

       对应的处理函数必须static修饰，否则无法解析

- fallback：默认为""，用于在抛出异常的时候提供fallback处理逻辑。fallback函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。

       返回类型与原方法一致；
       参数类型需要和原方法相匹配，Sentinel 1.6开始，也可在方法最后加 Throwable 类型的参数；
       默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定fallbackClass里面的方法。
       1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。
- defaultFallback：默认为""。用于通用的 fallback 逻辑。

       若同时配置了 fallback 和 defaultFallback，以fallback为准。

- fallbackClass：默认为{}。
  
       对应的处理函数必须static修饰，否则无法解析。

- exceptionsToTrace：默认为Throwable.class。需要trace的异常。
- exceptionsToIgnore：默认为{}。指定排除掉哪些异常。

       排除的异常不会计入异常统计，也不会进入fallback逻辑，而是原样抛出。

## 2. SentinelResource注解执行逻辑

通过`SentinelResourceAspect`类定义注解运行逻辑。`@Pointcut`注解定义切入点，匹配当前执行方法持有指定注解的方法；`@Around`注解定义在调用具体方法前和调用后来完成一些具体的任务。具体执行逻辑如下：

1. 根据方法入参获取出注解所在的类、方法等信息创建Method实例
1. 获取注解信息，包含资源名、流量方向、资源类型。资源名若为空，则默认为方法名。
1. 判断达到被限流条件，执行对应的处理逻辑。

流程如下：

[picture]: https://github.com/Consck/gitbook/raw/master/picture/%E6%B3%A8%E8%A7%A3%E9%99%90%E6%B5%81%E8%BF%90%E8%A1%8C%E9%80%BB%E8%BE%91.png

![picture]

## 3. entryWithPriority方法详解
### 3.1 校验全局上下文
从`ThreadLocal<Context>`实例中`contextHolder.get()`校验以下几点：
- 若为NullContext，则表示上下文的数量已经超过了阈值。不执行任何规则检查。
- 若为null，则进行调用链初始化。

      `defaultContextName`值为`sentinel_default_context`，创建调用入口节点`DefaultNode`类实例，其中默认名为`defaultContextName`，流量方向为IN，用于保存特定上下文中特定资源名的统计信息。

- 若全局开关关闭，不进行规则检查。一般情况下均设定为true。

### 3.2 构造ProcessorSlot链及slot结构

`ResourceWrapper r1 = new StringResourceWrapper("firstRes", EntryType.IN);`

调用方法`ctSph.lookProcessChain(r1)`获取责任链，结果如下：

[picture1]: https://github.com/Consck/gitbook/raw/master/picture/slot.jpg

![picture1]

通过`ServiceLoader.load(clazz, clazz.getClassLoader())`获取出Slot实现类，每个实现类通过`@SpiOrder(-6000)`注解带入一个value，通过value的值进行排序，最终加载出已排序的实例列表。


| ProcessorSlot实现类  | value值  |
|:----------|:----------|
| NodeSelectorSlot    | @SpiOrder(-10000)    |
| AuthoritySlot    | @SpiOrder(-6000)    |
| ClusterBuilderSlot    | @SpiOrder(-9000)    |
| DegradeSlot    | @SpiOrder(-1000)   |
| DemoSlot    | @SpiOrder(-3500)    |
| FlowSlot    | @SpiOrder(-2000)    |
| GatewayFlowSlot    | @SpiOrder(-4000)   |
| LogSlot    | @SpiOrder(-8000)    |
| NodeSelectorSlot   | @SpiOrder(-10000)    |
| ParamFlowSlot    | @SpiOrder(-3000)    |
| StatisticSlot   | @SpiOrder(-7000)  |
| SystemSlot  | @SpiOrder(-5000)    |


责任链初始化为DefaultProcessorSlotChain实例，包含first节点和end节点，指向同一个节点。通过`SpiLoader.loadPrototypeInstanceListSorted(ProcessorSlot.class)`加载出所有的slot类，并依次加入链尾，构造出完整的责任链。

```
AbstractLinkedProcessorSlot<?> first = new AbstractLinkedProcessorSlot<Object>() {

        @Override
        public void entry(Context context, ResourceWrapper resourceWrapper, Object t, int count, boolean prioritized, Object... args)
            throws Throwable {
            super.fireEntry(context, resourceWrapper, t, count, prioritized, args);
        }

        @Override
        public void exit(Context context, ResourceWrapper resourceWrapper, int count, Object... args) {
            super.fireExit(context, resourceWrapper, count, args);
        }

    };
    AbstractLinkedProcessorSlot<?> end = first;
```

责任链包含多个节点，整体结构如下。每个slot分别执行不同的功能，进行不同的规则校验。

[picture2]: https://github.com/Consck/gitbook/raw/master/picture/slot%E7%BB%93%E6%9E%84.jpg

![picture2]

