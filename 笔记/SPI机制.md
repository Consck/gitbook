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


# 注解值获取

定义一个注解接口，包含value值
```Java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
public @interface SpiOrder {

    /**
     * Represents the lowest precedence.
     */
    int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
    /**
     * Represents the highest precedence.
     */
    int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

    /**
     * The SPI precedence value. Lowest precedence by default.
     *
     * @return the precedence value
     */
    int value() default LOWEST_PRECEDENCE;
}
```
对NetHandler.class类加注解`@SpiOrder(value = -1000)`之后使用命令`NetHandler.class.getAnnotation(com.project.adapter.annotation.SpiOrder.class).value()`即可获取到注解中value的值。

# 自定义注解前后执行逻辑
## 声明一个注解类
```Java
@Target({ElementType.METHOD,ElementType.PARAMETER,ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyAnnotationImpl.class)
@Documented
public @interface Annotation{
	String message() default "value默认值";
	Class<?>[] groups() default{};
	Class<?extendsPayload>[] payload() default{};
	String value();
}
```
其中用的一些含义：
* @Target：被描述的注解可以用到什么地方
	1. ElementType.TYPE：用于描述类、接口或enum声明
	2. ElementType.FIELD：用于描述域
	3. ElementType.METHOD：用于描述方法
	4. ElementType.PARAMETER：用于描述参数
	5. ElementType.CONSTRUCRTOR：用于描述构造器
* @Retention：被描述的注解在什么范围内有效
	1. RetentionPolicy.RUNTIME：在运行时有效(运行时保留)
	2. RetentionPolicy.CLASS：在class文件中有效
	3. RetentionPolicy.SOURCE：在源文件中有效
* @Documented：描述其他类型的注解应该被作为被标注的程序成员的公共API
* @Inherited：某个被标注的类型是被继承的。如果注解用在类上，则注解将被用于该类的子类
## 切面编程
注解前后可以增加一些执行逻辑
```
@Component
@Aspect
public class KthLogAspect {
    @Pointcut("@annotation(com.project.adapter.annotation.KthLog)")
    private void pointcut(){}
    @Around("pointcut() && @annotation(logger)")
    public Object advice(ProceedingJoinPoint joinPoint, KthLog logger){
        Object result = null;
        Object[] args = joinPoint.getArgs();
        for(int i = 0;i<args.length;i++){
            args[i] = (int)args[i] - 1;
            System.out.println("此处可以对参数" + i + "进行操作");
        }
        try{
            System.out.println("执行方法前");
            result = joinPoint.proceed(args);
            System.out.println("执行方法后");
        }catch (Throwable throwable){
            /**
             * 使用proceed需要捕获Throwable异常
             */
            throwable.printStackTrace();
        }
        System.out.println("此处可以对result结果进行操作");
        return result;
    }
}
```