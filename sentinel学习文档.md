# sentinel学习文档
[sentinel]:https://github.com/alibaba/Sentinel
[sentinel]：(面向云原生微服务的高可用流控防护组件)
## sentinel项目结构
1. sentinel-core 核心模块，限流、降级、系统保护等都在这里实现
1. sentinel-dashboard 控制台模块，可以对连接上的sentinel客户端实现可视化的管理
1. sentinel-transport 传输模块，提供了基本的监控服务端和客户端的API接口以及一些基于不同库的实现
1. sentinel-extension 扩展模块，主要对DataSource进行了部分扩展实现
1. sentinel-adapter 适配器模块，主要实现了对一些常见框架的适配
1. sentinel-demo 样例模块，可参考怎么使用sentinel进行限流、降级等
1. sentinel-benchmark 基准测试模块，对核心代码的精确性提供基准测试

## sentinel参数配置

一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：

- resource：资源名，即限流规则的作用对象
- count: 限流阈值
- grade: 限流阈值类型：0 代表根据并发数量来限流，1 代表根据 QPS 来进行流量控制
- limitApp: 流控针对的调用来源，若为 default 则不区分调用来源
- strategy: 调用关系限流策略
- controlBehavior: 流量控制效果（直接拒绝、Warm Up、匀速排队）
> 注意：匀速排队模式暂时不支持 QPS > 1000 的场景。

## sentinel-dashboard控制台接入

* 添加依赖
```
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-transport-simple-http</artifactId>
	<version>1.7.2</version>
</dependency>
```
* 触发客户端连接控制台
```
@Configuration
public  class  Env{
	static{
		InitExecutor.doInit();
	}
}
```
* 增加JVM启动参数

`-Dproject.name=project -Dcsp.sentinel.dashboard.server=127.0.0.1:8080`
### 相关问题
1. 项目启动报错，内容如下：

> `Circular view path [again]: would dispatch back to the current handler URL [/again] again. Check you. .....`

分析原因：依赖包版本冲突问题

解决办法：
* 限流依赖包，版本需大于触发控制台依赖包
```
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-annotation-aspectj</artifactId>
	<version>1.8.0</version>
</dependency>
```
* 触发控制台依赖包
```
<dependency>
	<groupId>com.alibaba.csp</groupId>
	<artifactId>sentinel-transport-simple-http</artifactId>
	<version>1.7.2</version>
</dependency>
```

### 公司sentinel接入说明
* 引入依赖包

该sdk已经将上述接入方式全部封装
```
<dependency>
    <groupId>com.wosai</groupId>
    <artifactId>sentinel-sdk-starter</artifactId>
    <version>1.0.9-SNAPSHOT</version>
</dependency>
```
* 升级jsonrpc版本为1.5.4

```
<dependency>
    <groupId>com.wosai.middleware</groupId>
    <artifactId>jsonrpc4j</artifactId>
    <version>1.5.4</version>
</dependency>
```

* 拦截器配置

```
<bean class="com.googlecode.jsonrpc4j.spring.AutoJsonRpcServiceImplExporter">
    <property name="interceptorList" ref="jsonRpcSentinelInterceptor"/>
</bean>
 
<bean id="jsonRpcSentinelInterceptor" class="com.wosai.sentinel.sdk.listener.SentinelInterceptor"/>
```

* 限流日志文件配置


	`-Dcsp.sentinel.log.dir=/app/log/xx`


具体可参考：[wiki](https://confluence.wosai-inc.com/pages/viewpage.action?pageId=213352508)


## sentinel-core项目学习
```
根本找不到项目起点，一团乱麻的赶脚，不知从何看起。先从文档看起吧。。。
2020-08-31
```

### com.alibaba.csp.sentinel.slots.block.Rule文件解析

所有规则的基本接口，仅包含getResource方法待实现，获取此规则的目标资源

### AbstractRule类实现Rule接口

抽象类包含resource、limitApp变量和equals、limitAppEquals、hashCode方法

equals方法作用：主要用来对比两个限流规则是否一样，对resource、limitApp进行对比

limitAppEquals方法作用：对比limitApp参数

hashCode方法作用：计算resource、limitApp的哈希值

### FlowRule类继承AbstractRule类

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

***
> 2020-08-31 记

# sentinel-core文件解析

com.alibaba.csp.sentinel.init文件作为客户端接入

| 日志目录    | 默认日志目录为：user.home/logs/csp，可以通过csp.sentinel.log.dir参数设置 | 
|:------------|:----------|
| 日志名称    | appName-metrics.log.yyyy-MM-dd.n，可以通过csp.sentinel.app.name指定每创建一个日志文件n会递增，可以通过logNameUsePid将pid加入到日志文件名中，默认false。例如：com-alibaba-csp-sentinel-dashboard-DashboardApplication-metrics.log.2020-08-25.5 | 
| 日志大小    | 默认50M，可以通过csp.sentinel.metric.file.single.size设置 |
| 日志数量    | 默认最多6个文件，可以通过csp.sentinel.metric.file.total.count设置 |




# sentinel-extension扩展模块
## sentinel-datasource-apollo提供与Apollo之间的适配
限流规则可在Apollo进行配置，当值发生修改时，可以立马被读取到
































