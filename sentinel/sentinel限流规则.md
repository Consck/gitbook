# 通过Apollo配置限流规则后sentinel都做了什么

## 1. 在服务启动时加载配置的限流信息

- 将限流规则持久化到Apollo配置(随时可配置)，也可以直接写在代码中(不可配置)

```
[
   {
        "resource": "TestResource",
        "controlBehavior": 0,
        "count": 5.0,
        "grade": 1,
        "limitApp": "default",
        "strategy": 0
   }
]
```

- 读取限流规则：

```java
ReadableDataSource<String, List<FlowRule>> flowRuleDataSource = 
 new ApolloDataSource<>("application", "sentinel.flow", "",
 source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>(){}));
//Listen to the SentinelProperty for FlowRules.
FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
```

| 参数  | 含义  |
|:----------|:----------|
| namespaceName    | 命名空间，could not be null or empty    |
| ruleKey    | Apollo中对应的key，could not be null or empty    |
| defaultRuleValue    | 默认value，一般设置"[]"    |
| Converter<String, T> parser    | 限流参数配置，将字符串配置转换为实际流规则的解析器    |

ApolloDataSource类中完成两个主要操作：
- initializeConfigChangeListener(): 初始化Apollo，添加监听，当配置值修改立马生效。
- loadAndUpdateRules(): 保存限流规则

[picture1]: https://github.com/Consck/gitbook/raw/master/picture/sentinel%20rule.jpg

![picture1]

在程序启动时，会将配置的规则信息读入flowRules，并根据流控规则通过`FlowRuleUtil.buildFlowRuleMap(conf)`初始化TrafficShapingController实现类，共包含四个DefaultController、RateLimiterController、WarmUpController、WarmUpRateLimiterController。

```java
//当SentinelProperty updateValue需要通知监听器时，该类将保存回调方法
private static final class FlowPropertyListener 
      implements PropertyListener<List<FlowRule>> {
        //更新Apollo规则配置
        @Override
        public void configUpdate(List<FlowRule> value) {
            Map<String, List<FlowRule>> rules = FlowRuleUtil.buildFlowRuleMap(value);
            if (rules != null) {
                flowRules.clear();
                flowRules.putAll(rules);
            }
            RecordLog.info("[FlowRuleManager] Flow rules received: " + flowRules);
        }
        //初始化读取Apollo规则配置
        @Override
        public void configLoad(List<FlowRule> conf) {
            Map<String, List<FlowRule>> rules = FlowRuleUtil.buildFlowRuleMap(conf);
            if (rules != null) {
                flowRules.clear();
                flowRules.putAll(rules);
            }
            RecordLog.info("[FlowRuleManager] Flow rules loaded: " + flowRules);
        }
    }

```

## 2. 通过SentinelResource注解使用限流

通过`SentinelResourceAspect`类定义注解运行逻辑。`@Pointcut`注解定义切入点，匹配当前执行方法持有指定注解的方法；`@Around`注解定义在调用具体方法前和调用后来完成一些具体的任务。具体执行逻辑如下：

1. 根据方法入参获取出注解所在的类、方法等信息创建Method实例
1. 获取注解信息，包含资源名、流量方向、资源类型。资源名若为空，则默认为方法名。
1. 判断达到被限流条件，执行对应的处理逻辑。

流程如下：

[picture2]: https://github.com/Consck/gitbook/raw/master/picture/%E6%B3%A8%E8%A7%A3%E9%99%90%E6%B5%81%E8%BF%90%E8%A1%8C%E9%80%BB%E8%BE%91.png

![picture2]

###  entryWithPriority方法详解
#### - 校验全局上下文
从`ThreadLocal<Context>`实例中`contextHolder.get()`校验以下几点：
- 若为NullContext，表示上下文的数量已经超过了阈值，所以这里只初始化条目。不会进行规则检查。
- 若为null，则进行调用链初始化,该类用于跳过上下文名称检查。

> `defaultContextName`值为`sentinel_default_context`，创建调用入口节点`DefaultNode`类实例，其中默认名为`defaultContextName`，流量方向为IN，用于保存特定上下文中特定资源名的统计信息。

- 若全局开关关闭，不进行规则检查。一般情况下均设定为true。

#### - 构造ProcessorSlot链及slot结构

`ResourceWrapper r1 = new StringResourceWrapper("firstRes", EntryType.IN);`

调用方法`ctSph.lookProcessChain(r1)`获取责任链，结果如下：

[picture3]: https://github.com/Consck/gitbook/raw/master/picture/slot.jpg

![picture3]

通过`ServiceLoader.load(clazz, clazz.getClassLoader())`获取出Slot实现类，每个实现类通过`@SpiOrder(-6000)`注解带入一个value，通过value的值进行排序，最终加载出已排序的实例列表。通过Java SPI机制加载以下几个实例，并按照从小到大构造调用链，顺序为：NodeSelectorSlot > ClusterBuilderSlot > LogSlot > StatisticSlot > AuthoritySlot > SystemSlot > FlowSlot > DegradeSlot


| ProcessorSlot实现类  | value值  |
|:----------|:----------|
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

```java
AbstractLinkedProcessorSlot<?> first = new AbstractLinkedProcessorSlot<Object>() {

        @Override
        public void entry(Context context, ResourceWrapper resourceWrapper, Object t, 
   int count, boolean prioritized, Object... args)
            throws Throwable {
            super.fireEntry(context, resourceWrapper, t, count, prioritized, args);
        }

        @Override
        public void exit(Context context, ResourceWrapper resourceWrapper, int count, 
    Object... args) {
            super.fireExit(context, resourceWrapper, count, args);
        }

    };
    AbstractLinkedProcessorSlot<?> end = first;
```

责任链包含多个节点，整体结构如下。每个slot分别执行不同的功能，进行不同的规则校验。

[picture4]: https://github.com/Consck/gitbook/raw/master/picture/slot%E7%BB%93%E6%9E%84.jpg

![picture4]

## 3.当请求到达FlowSlot节点时判断是否pass

chain.entry方法会经过FlowSlot中的entry(),调用checkFlow进行流控规则判断

第一步：遍历所有流控规则FlowRule，获取`flowRules.get(resource)`对应的限流配置参数

第二步：调用canPassCheck进行校验。根据来源和策略获取Node,从而拿到统计的runtime信息；使用流量控制器检查是否让流量通过。

1.获取不同处理策略的Node

- 当前上下文context中调用来源为limitApp配置值，且不为default或者other：
	
	strategy设定为直接流控：`context.getOriginNode();`

- limitApp配置值为default：
	
	strategy设定为直接流控：`node.getClusterNode();`

- limitApp配置值为other，且调用来源不为other：
	
	strategy设定为直接流控：`context.getOriginNode();`

> strategy设定为相关流控：`ClusterBuilderSlot.getClusterNode(refResource);`

> strategy设定为链流控制：DefaultNode实例

假设我们对接口UseService配置限流1000QPS，这3种场景分别如下。
* 第一种：目的是优先保障重要来源的流量，我们需要区分调用来源，将限流规则细化。
* 对A应用配置500QPS，对B应用配置200QPS，此时会产生两条规则：A应用请求的流量限制在500，B应用请求的流量限制在200
* 第二种：没有特别重要来源的配置。我们不想区分调用来源，所有入口调用UserService共享一个规则，所有client加起来总流量只能通过1000QPS
* 第三种：配合第一种场景使用，在长尾应用多的情况下不想对每个应用进行设置，没有具体设置的应用都将命中。

2.通过`rule.getRater()`获取controlBehavior配置值，判断canPass

> 例如CONTROL_BEHAVIOR_DEFAULT直接拒绝

[picture5]: https://github.com/Consck/gitbook/raw/master/picture/%E7%9B%B4%E6%8E%A5%E6%8B%92%E7%BB%9D%E7%AD%96%E7%95%A5%E9%80%BB%E8%BE%91%E5%A4%84%E7%90%86.jpg

![picture5]

| 字段  | 含义  | 
|:--------|:------------|
| curCount    | 已通过请求量  | 
| acquireCount    | 当前请求量    | 
| prioritized    | 请求是否有优先级,值定为false   | 
| FlowException   | 请求被限流    | 
| PriorityWaitException    | 请求被降级等待    | 
过程中有可能抛出两种异常，在StatisticSlot文件的entry中有捕获处理。

### 请求等待

```Java
public long tryOccupyNext(long currentTime, int acquireCount, double threshold) {
        double maxCount = threshold * IntervalProperty.INTERVAL / 1000;
        long currentBorrow = rollingCounterInSecond.waiting();
        if (currentBorrow >= maxCount) {
            //返回最长等待时间
            return OccupyTimeoutProperty.getOccupyTimeout();
        }

        int windowLength = IntervalProperty.INTERVAL / SampleCountProperty.SAMPLE_COUNT;
        long earliestTime = currentTime - currentTime % windowLength + windowLength - IntervalProperty.INTERVAL;

        int idx = 0;
        long currentPass = rollingCounterInSecond.pass();
        while (earliestTime < currentTime) {
            long waitInMs = idx * windowLength + windowLength - currentTime % windowLength;
            if (waitInMs >= OccupyTimeoutProperty.getOccupyTimeout()) {
                break;
            }
            long windowPass = rollingCounterInSecond.getWindowPass(earliestTime);
            if (currentPass + currentBorrow + acquireCount - windowPass <= maxCount) {
                return waitInMs;
            }
            earliestTime += windowLength;
            currentPass -= windowPass;
            idx++;
        }

        return OccupyTimeoutProperty.getOccupyTimeout();
    }
```

若返回请求等待时间<最大时间长度(默认500ms，可通过Apollo配置)，则进行请求等待。


### 限流日志
日志存放文件路径：`-Dcsp.sentinel.log.dir=/app/log`

格式：`|--timestamp-|------date time----|-resource-|p |block|s |e|rt`

| 参数  | 含义  |
|:----------|:----------|
| p    | 通过的请求    |
| block    | 被阻止的请求    |
| s    | 成功执行完成的请求个数    |
| e    | 用户自定义的异常    |
| rt    | 平均响应时长    |

-------

> 滑动窗口

例：限制每60秒内只能接受客户端10个请求，如果超过10个请求则直接拒绝访问服务。固定速率 10R/M。

滑动窗口计数器算法原理：创建6个独立的格子，每个格子都有自己独立的计数器。每个格子独立计数10秒。

> 令牌桶

令牌桶分为2个动作，动作1(固定速率往桶中存入令牌)、动作2(客户端如果想访问请求，先从桶中获取token)。