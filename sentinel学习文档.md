# sentinel学习文档

[sentinel]:https://github.com/alibaba/Sentinel

[gitee](https://gitee.com/nilera/Sentinel/wikis/Home)

[sentinel]：(面向云原生微服务的高可用流控防护组件)

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

### sentinel接入说明

* 引入依赖包

该sdk已经将上述接入方式全部封装

```
<dependency>
    <groupId>***</groupId>
    <artifactId>sentinel-sdk-starter</artifactId>
    <version>1.0.9-SNAPSHOT</version>
</dependency>
```

* 升级jsonrpc版本为1.5.4

```
<dependency>
    <groupId>***</groupId>
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

> 限流规则可在Apollo进行配置，当值发生修改时，可以立马被读取到
































