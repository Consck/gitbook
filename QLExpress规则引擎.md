# QLExpress规则引擎
[gitee地址](https://gitee.com/cuibo119/QLExpress)

特性：
- 线程安全，引擎运算过程中产生的临时变量都是threadLocal类型。
- 高效执行，比较耗时的脚本编译过程可以缓存在本地机器，运行时的临时变量创建采用了缓冲池的技术，和groovy性能相当。
- 弱类型脚本语言，和groovy，JavaScript语法类似，虽然比强类型脚本语言要慢一些，但是使业务的灵活度大大增强。
- 安全控制，可以通过设置相关运行参数，预防死循环、高危系统api调用等情况。
- 代码精简，依赖最小，250K的jar包适合所有Java的运行环境，在Android系统的低端POS机也得到广泛运用。

# 调用

导入依赖包：
```
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>QLExpress</artifactId>
  <version>3.2.0</version>
</dependency>
```

使用示例：
```
ExpressRunner runner = new ExpressRunner();
DefaultContext<String, Object> context = new DefaultContext<String, Object>();
context.put("a",1);
context.put("b",2);
context.put("c",3);
String express = "a+b*c";
Object r = runner.execute(express, context, null, true, false);
System.out.println(r);
```





























