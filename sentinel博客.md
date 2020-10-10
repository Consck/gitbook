# sentinel为Dubbo服务保驾护航

[学习链接](http://dubbo.apache.org/zh-cn/blog/sentinel-introduction-for-dubbo.html)

## Dubbo是什么

Dubbo：[参考链接](https://www.jianshu.com/p/3090d63e9cb3)

[picture]: https://github.com/Consck/project/raw/master/images/1598854823247.jpg

![picture]

Sentinel-adapter文件夹下提供各种相关适配依赖，[sentinel-apache-dubbo-adapter](https://github.com/apache/dubbo-sentinel-support)模块用来提供与Dubbo适配，包括针对服务提供方的过滤器和服务消费方的过滤器。

[源码Demo链接](https://github.com/alibaba/Sentinel/tree/master/sentinel-demo/sentinel-demo-dubbo)

## Sentinel控制台监控数据持久化【Apollo】

目前 Sentinel 控制台中监控数据聚合后直接存在内存中，未进行持久化，且仅保留最近 5 分钟的监控数据。若需要监控数据持久化的功能，可以自行扩展实现 MetricsRepository 接口，然后注册成 Spring Bean 并在相应位置通过 @Qualifier 注解指定对应的 bean name 即可。

- 1：控制台只负责把规则存储到Apollo（或其他配置中心中间件ZK、Nacos）。
- 2：客户端系统要自己去拉取Apollo的配置
- 3：控制台中维护的规则有一个动作是调用Http ApiClient推送到你的系统，这也是原本就有的，但是也只是点击保存那一刻给你推一次，当你自己的系统重启，那还是清空了。

## Sentinel一体化监控解决方案 CrateDB+Grafana

将数据持久化后配置在Grafana，可以查看多个接口的流控指标

## 限流降级神器-哨兵(sentinel)原理分析

### 责任链模式

抽象处理者（Handler）角色：定义出一个处理请求的接口。如果需要，接口可以定义出一个方法以设定和返回对下家的引用。这个角色通常由一个Java抽象类或者Java接口实现。上图中Handler类的聚合关系给出了具体子类对下家的引用，抽象方法handleRequest()规范了子类处理请求的操作。

具体处理者（ConcreteHandler）角色：具体处理者接到请求后，可以选择将请求处理掉，或者将请求传给下家。由于具体处理者持有对下家的引用，因此，如果需要，具体处理者可以访问下家。
