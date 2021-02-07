# 代码规范
* 修改（包括新增）操作必须打印日志
* 条件分支必须打印条件值，重要参数必须打印
* 数据量大的时候需要打印数据量
* 不要依赖debug，多依赖日志
* 代码开发测试完成之后不要急着提交，先跑一遍看看日志是否看得懂
  
# Controller规范
* 所有函数返回统一的ResultBean/PageResultBean格式
* ResultBean/PageResultBean是controller专用的，不允许往后传
* Controller做参数格式的转换，不允许把json，map这类对象传到services去，也不允许services返回json、map
* 参数中一般情况不允许出现Request，Response这些对象
* 不需要打印日志

# MDC
MDC是SLF4J中的一个类，通过MDC我们可以很方便的实现同一个线程内（包括父线程和子线程之间）的日志的追踪。

添加pom依赖：
```java
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.21</version>
</dependency>

<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

resource目录下新建logback.xml文件：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd hh:mm:ss} [%thread] [traceId = %X{traceId}] [%logger{32}] - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
    </root>
</configuration>
```

使用时在类上加```@Slf4j```注解,
```java
public static void main(String[] args) {
    MDC.put("traceId", UUID.randomUUID().toString());
    //log.info("MdcTest");
    MDC.clear();
    //log.info("clear test");
    MDC.remove("traceId");
}
```
