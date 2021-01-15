# sentinel学习文档

[sentinel]:https://github.com/alibaba/Sentinel

[gitee](https://gitee.com/nilera/Sentinel/wikis/Home)

[sentinel]：(面向云原生微服务的高可用流控防护组件)

Sentinel以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

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

































