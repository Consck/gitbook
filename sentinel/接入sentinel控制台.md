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

































