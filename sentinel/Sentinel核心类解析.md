# Sentinel-核心类解析

Sentinel的核心骨架，将不同的Slot按照顺序串在一起(责任链模式)，从而将不同的功能(限流、降级、系统保护)组合在一起。slot chain其实可以分为两部分：统计数据构建部分(statistic)和判断部分(rule checking)。


[picture]: https://github.com/Consck/gitbook/raw/master/sentinel-slot-chain-architecture.png

![picture]


## context

Context代表调用链路上下文，贯穿一次调用链中的所有Entry。Context维持着入口节点(entranceNode)、本次调用链路的curNode、调用来源(origin)等信息。Context名称即为调用链路入口名称。

Context维持的方式：通过ThreadLocal传递，只有在入口enter的时候生效。由于Context是通过ThreadLocal传递的，因此对于异步调用链路，线程切换的时候会丢掉Context，因此需要手动通过ContextUtil.runOnContext(context, f)来变换context。

## Entry

每一次资源调用都会创建一个Entry。Entry包含了资源名、curNode(当前统计节点)、originNode(来源统计节点)等信息。

CtEntry为普通的Entry，在调用SphU.entry(xxx)的时候创建。特性：Linked entry within current context(内部维护者parent和child)

**需要注意的一点**：CtEntry构造函数中会做调用链的变换，即将当前Entry接到传入Context的调用链路上(setUpEntryFor)。

资源调用结束时需要entry.exit()。exit操作会过一遍slot chain exit，恢复调用栈，exit context然后清空entry中的context防止重复调用。

## Node

Sentinel里面的各种种类的统计节点：

- StatisticNode：最为基础的统计节点，包含秒级和分钟级两个滑动窗口结构。
- DefaultNode：链路节点，用于统计调用链路上某个资源的数据，维持树状结构。
- ClusterNode：簇点，用于统计每个资源全局的数据(不区分调用链路)，以及存放该资源的按来源区分的调用数据(类型为StatisticNode)。特别地，Constants.ENTRY_NODE节点用于统计全局的入口资源数据。
- EntranceNode：入口节点，特殊的链路节点，对应某个Context入口的所有调用数据。Constants.ROOT节点也是入口节点。

构建的时机：

- EntranceNode在ContextUtil.enter(xxx)的时候就创建了，然后塞到Context里面。
- NodeSelectorSlot：根据context创建DefaultNode，然后set CurNode to Context。
- ClusterBuilderSlot：首先根据resourceName创建ClusterNode，并且set clusterNode to defaultNode；然后再根据origin创建来源节点(类型为StatisticNode)，并且set originNode to curEntry 。

几种Node的维度(数目)：

- ClusterNode的维度是resource
- DefaultNode的维度是resource * Context，存在每个NodeSelectorSlot的map里面
- EntranceNode的维度是Context，存在ContextUtil类的contextNameNodeMap里面
- 来源节点(类型为StatisticNode)的维度是resource * origin，存在每个ClusterNode的originCountMap里面

# Sentinel工作流程

在Sentinel里面，所有的资源都对应一个资源名称(resourceName)，每次资源调用都会创建一个Entry对象。Entry可以通过对主流框架的适配自动创建，也可以通过注解的方式或调用SphU API显示创建。Entry创建的时候，同时也会创建一系列功能插槽(slot chain)，这些插槽有不同的职责：

- NodeSelectorSlot 负责收集资源的路径，并将这些资源的调用路径，以树状结构存储起来，用于根据调用路径来限流降级；
- ClusterBuilderSlot 则用于存储资源的统计信息以及调用者信息，例如该资源的 RT, QPS, thread count 等等，这些信息将用作为多维度限流，降级的依据；
- StatistcSlot 则用于记录，统计不同纬度的 runtime 信息；
- FlowSlot 则用于根据预设的限流规则，以及前面 slot 统计的状态，来进行限流；
- AuthorizationSlot 则根据黑白名单，来做黑白名单控制；
- DegradeSlot 则通过统计信息，以及预设的规则，来做熔断降级；
- SystemSlot 则通过系统的状态，例如 load1 等，来控制总的入口流量；

## NodeSelectorSlot

主要负责收集资源的路径，并将这些资源的调用路径以树状结构存储起来，用于根据调用路径进行流量控制。

```
ContextUtil.enter("entrance1", "appA");
 Entry nodeA = SphU.entry("nodeA");
 if (nodeA != null) {
    nodeA.exit();
 }
 ContextUtil.exit();
```

上述代码通过` ContextUtil.enter()` 创建了一个名为 `entrance1` 的上下文，同时指定调用发起者为 `appA`；接着通过 `SphU.entry()`请求一个 `token`，如果该方法顺利执行没有抛 BlockException，表明 `token` 请求成功。

以上代码在内存中生成以下结构：

			
		machine-root
                 /     
                /
         EntranceNode1
              /
             /   
      DefaultNode(nodeA)


注意：每个 DefaultNode 由资源 ID 和输入名称来标识。换句话说，一个资源 ID 可以有多个不同入口的 DefaultNode。

```
ContextUtil.enter("entrance1", "appA");
  Entry nodeA = SphU.entry("nodeA");
  if (nodeA != null) {
    nodeA.exit();
  }
  ContextUtil.exit();

  ContextUtil.enter("entrance2", "appA");
  nodeA = SphU.entry("nodeA");
  if (nodeA != null) {
    nodeA.exit();
  }
  ContextUtil.exit();
```

以上代码将在内存中生成以下结构：

			machine-root
                   /         \
                  /           \
          EntranceNode1   EntranceNode2
                /               \
               /                 \
       DefaultNode(nodeA)   DefaultNode(nodeA)

上面的结构可以通过调用 `curl http://localhost:8719/tree?type=root` 来显示

## ClusterBuilderSlot

从插槽用于构建资源ClusterNode以及调用来源节点。ClusterNode保持某个资源运行统计信息(响应时间、QPS、block数目、线程数、异常数等)以及调用来源统计信息列表。调用来源的名称由ContextUtil.enter(contextName, origin)中的origin标记。可通过如下命令查看某个资源不同调用者的访问情况：`curl http://localhost:8719/origin?id=caller：`

## StatisticSlot

StatisticSlot 是Sentinel的核心功能插槽之一，用于统计实时的调用数据。

- clusterNode：资源唯一标识的ClusterNode的实时统计
- origin：根据来自不同调用者的统计信息
- defaultnode：根据入口上下文区分的资源ID的runtime统计
- 入口流量的统计
- entry的时候：依次执行后面的判断slot。每个slot触发流控的话会抛出异常(BlockException的子类)。若有BlockException抛出，则记录block数据；若无异常抛出则算作可通过，记录pass数据。
- exit的时候：若无error(无论是业务异常还是流控异常)。记录complete(success)以及RT，线程数-1.
- 记录数据的维度：线程数+1、记录当前DefaultNode数据、记录对应的originNode数据(若存在origin)、累计IN统计数据(若流量类型为IN)。

sentinel底层采用高性能的滑动窗口数据结构LeapArray来统计实时的秒级指标数据，可以很好地支撑写多于读的高并发场景。

## FlowSlot

这个slot主要根据预设的资源的统计信息，按照固定的次序，依次生效。如果一个资源对应两条或者多条流控规则，则会根据如下次序依次检验，直到全部通过或者有一个规则生效为止：

```
指定应用生效的规则，即针对调用方限流的；
调用方为other的规则；
调用方为default的规则。
```

## DegradeSlot

主要针对资源的平均响应时间(RT)以及异常比率，来决定资源是否在接下来的时间被自动熔断掉。

## SystemSlot

根据对于当前系统的整体情况，对入口资源的调用进行动态调配。其原理是让入口的流量和当前系统的预计容量达到一个动态平衡。

注意系统规则只对入口流量起作用(调用类型为EntryType.IN)，对出口流量无效。可通过SphU.entry(res, entryType)指定调用类型，如果不指定，默认是EntryType.OUT.


