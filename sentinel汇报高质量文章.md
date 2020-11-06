# Sentinel介绍篇

[源码](https://github.com/alibaba/Sentinel)

## 源码结构简介

1. sentinel-core 
1. sentinel-dashboard 
1. sentinel-transport 
1. sentinel-extension 
1. sentinel-adapter 
1. sentinel-demo 
1. sentinel-benchmark 

# Sentinel使用篇

# Sentinel核心篇

通过注解的方法接入限流服务

## SentinelResource注解

该注解可作用于方法和接口、类、枚举;会在class字节码文件中存在，在运行时可以通过反射获取到。共包含以下字段信息，可赋值：

- value：资源名
- entryType：标记流量的方向，取值IN/OUT，默认为out
- resourceType：1.7.0版本开始新增字段，资源的类型。默认为0。
- blockHandler：被限流后执行的方法名。具体要求如下：

       必须是 public；
       返回类型与原方法一致；
       参数类型需要和原方法相匹配，并在最后加 BlockException 类型的参数。
       默认需和原方法在同一个类中。若希望使用其他类的函数，可配置blockHandlerClass ，并指定blockHandlerClass里面的方法。
- blockHandlerClass：被限流执行方法所在的类名。

       对应的处理函数必须static修饰，否则无法解析

- fallback：默认为""，用于在抛出异常的时候提供fallback处理逻辑。fallback函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。

       返回类型与原方法一致；
       参数类型需要和原方法相匹配，Sentinel 1.6开始，也可在方法最后加 Throwable 类型的参数；
       默认需和原方法在同一个类中。若希望使用其他类的函数，可配置 fallbackClass ，并指定fallbackClass里面的方法。
       1.6.0 之前的版本 fallback 函数只针对降级异常（DegradeException）进行处理，不能针对业务异常进行处理。
- defaultFallback：默认为""。用于通用的 fallback 逻辑。

       若同时配置了 fallback 和 defaultFallback，以fallback为准。

- fallbackClass：默认为{}。
  
       对应的处理函数必须static修饰，否则无法解析。

- exceptionsToTrace：默认为Throwable.class。需要trace的异常。
- exceptionsToIgnore：默认为{}。指定排除掉哪些异常。

       排除的异常不会计入异常统计，也不会进入fallback逻辑，而是原样抛出。

## SentinelResource注解执行逻辑

通过`SentinelResourceAspect`类定义注解运行逻辑。`@Pointcut`注解定义切入点，匹配当前执行方法持有指定注解的方法；`@Around`注解定义在调用具体方法前和调用后来完成一些具体的任务。具体执行逻辑如下：

1. 根据方法入参获取出注解所在的类、方法等信息创建Method实例
1. 获取注解信息，包含资源名、流量方向、资源类型。资源名若为空，则默认为方法名。
1. 判断达到被限流条件，执行对应的处理逻辑。

## entryWithPriority方法详解
### 校验全局上下文
从`ThreadLocal<Context>`实例中`contextHolder.get()`校验以下几点：
- 若为NullContext，则表示上下文的数量已经超过了阈值。不执行任何规则检查。
- 若为null，则进行调用链初始化。

      `defaultContextName`值为`sentinel_default_context`，创建调用入口节点`DefaultNode`类实例，其中默认名为`defaultContextName`，流量方向为IN，用于保存特定上下文中特定资源名的统计信息。

- 若全局开关关闭，不进行规则检查。一般情况下均设定为true。

### 构造ProcessorSlot链




# Sentinel扩展篇

