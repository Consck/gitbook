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

在程序启动时，会将配置的规则信息读入flowRules。

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

第一步：遍历所有流控规则FlowRule，读取flowRules值

第二步：针对每个规则，调用canPassCheck进行校验






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





