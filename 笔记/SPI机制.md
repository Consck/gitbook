# Java SPI概念

接口属于实现方的情况，实现方提供了接口和实现，我们可以引用接口来达到调用某实现类的功能，这就是API，它具有以下特征：

- 概念上更接近实现方
- 组织上位于实现方所在的包中
- 实现和接口在一个包中

当接口属于调用方时，我们就将其称为SPI，全称为：`service provider interface`，SPI的规则如下：

- 概念上更依赖调用方
- 组织上位于调用方所在的包中
- 实现位于独立的包中（也可认为在提供方中）


# Java SPI机制

`java spi`的具体约定为:当服务的提供者，提供了服务接口的一种实现之后，在jar包的`META-INF/services/`目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。而当外部程序装配这个模块的时候，就能通过该jar包`META-INF/services/`里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。 基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。jdk提供服务实现查找的一个工具类：`java.util.ServiceLoader`


# 示例

> sentinel源码ProcessorSlots接口的实现类

1.在resources文件目录下创建`META-INF/services/`文件夹，并创建名为：`com.alibaba.csp.sentinel.slotchain.ProcessorSlot`文件，注意名字需要与接口文件位置一致。文件内容为接口的实现类。

```
# Sentinel default ProcessorSlots
# 调用链路构建
com.alibaba.csp.sentinel.slots.nodeselector.NodeSelectorSlot
# 统计簇点构建
com.alibaba.csp.sentinel.slots.clusterbuilder.ClusterBuilderSlot
com.alibaba.csp.sentinel.slots.logger.LogSlot
# 监控统计
com.alibaba.csp.sentinel.slots.statistic.StatisticSlot
# 来源访问控制
com.alibaba.csp.sentinel.slots.block.authority.AuthoritySlot
# 系统保护
com.alibaba.csp.sentinel.slots.system.SystemSlot
# 流量控制
com.alibaba.csp.sentinel.slots.block.flow.FlowSlot
# 熔断降级
com.alibaba.csp.sentinel.slots.block.degrade.DegradeSlot
```

2.通过`java.util.ServiceLoader`工具类装载实例化

```
ServiceLoader<T> serviceLoader = ServiceLoader.load(clazz);

ServiceLoader.load(clazz, clazz.getClassLoader());
```


