# 限流规则FlowSlot

使用Apollo持久化限流规则后，服务请求限流生效逻辑流程

## 服务启动即读入限流信息

1. 将限流规则持久化到Apollo配置(随时可配置)，也可以直接写在代码中(不可配置)

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

2. 读取限流规则：

```
ReadableDataSource<String, List<FlowRule>> flowRuleDataSource = new ApolloDataSource<>("application", "sentinel.flow", "",
                source -> JSON.parseObject(source, new TypeReference<List<FlowRule>>() {
                }));
//Listen to the SentinelProperty for FlowRules.
FlowRuleManager.register2Property(flowRuleDataSource.getProperty());
```
- String namespaceName：命名空间，could not be null or empty
- String ruleKey：Apollo中对应的key，could not be null or empty
- String defaultRuleValue：默认value，一般设置"[]"
- Converter<String, T> parser：限流参数配置，将字符串配置转换为实际流规则的解析器

ApolloDataSource类中完成两个主要操作：
- initializeConfigChangeListener(): 初始化Apollo，添加监听`config.addChangeListener(configChangeListener, Sets.newHashSet(ruleKey))`，当配置值修改立马生效。
- loadAndUpdateRules(): 保存限流规则

[picture]: https://github.com/Consck/gitbook/raw/master/picture/sentinel%20rule.jpg

![picture]

在程序启动时，会将配置的规则信息读入flowRules，并根据流控规则通过`FlowRuleUtil.buildFlowRuleMap(conf)`初始化TrafficShapingController实现类，共包含四个DefaultController、RateLimiterController、WarmUpController、WarmUpRateLimiterController。

```
//当SentinelProperty updateValue需要通知监听器时，该类将保存回调方法
private static final class FlowPropertyListener implements PropertyListener<List<FlowRule>> {

        @Override
        public void configUpdate(List<FlowRule> value) {
            Map<String, List<FlowRule>> rules = FlowRuleUtil.buildFlowRuleMap(value);
            if (rules != null) {
                flowRules.clear();
                flowRules.putAll(rules);
            }
            RecordLog.info("[FlowRuleManager] Flow rules received: " + flowRules);
        }

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

## 当请求到达FlowSlot节点，判断是否限流

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

[picture1]: https://github.com/Consck/gitbook/raw/master/picture/sentinel%20rule.jpg

![picture1]

过程中有可能抛出两种异常，在StatisticSlot文件的entry中有捕获处理。


----

### 限流规则参数

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





