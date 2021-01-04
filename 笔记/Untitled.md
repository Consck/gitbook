所有规则的基本接口，仅包含getResource方法待实现，获取此规则的目标资源

### AbstractRule类实现Rule接口

抽象类包含resource、limitApp变量和equals、limitAppEquals、hashCode方法

equals方法作用：主要用来对比两个限流规则是否一样，对resource、limitApp进行对比

limitAppEquals方法作用：对比limitApp参数

hashCode方法作用：计算resource、limitApp的哈希值

### FlowRule类继承AbstractRule类
