# 锁

## 类别

* 悲观锁：Java中的重量级锁，如synchronize；数据库行锁
* 乐观锁：Java中的轻量级锁，如volatile和CAS；数据库版本号
* 分布式锁(Redis锁)

## 乐观锁

每次去获取共享数据的时候会认为别人不会修改，所以不上锁，但是在更新的时候会判断这期间有没有人去更新这个数据

乐观锁使用在前，判断在后

```
reduce()
{
    select total_amount from table_1
    if(total_amount < amount ){
          return failed.  
    }  
    //其他业务逻辑
    update total_amount = total_amount - amount where total_amount > amount; }
```

## 悲观锁

每次共享数据的时候会加一把锁，等使用完之后释放锁

悲观锁判断在前，使用在后

```
reduce()
{
    //其他业务逻辑
    int num = update total_amount = total_amount - amount where total_amount > amount; 
   if(num ==1 ){
          //业务逻辑.  
    } 
}
```

## 扣减操作案例

库存数据只有100个。并发情况下第1笔请求卖出100个，第2批卖出100元，导致当前的库存数量为负数。遇到这种场景应该如何破解呢？

### 方案1：同步排它锁

缺点：

* 线程串行导致性能问题，性能消耗比较大
* 无法解决分布式部署情况下跨进程问题

### 方案2：数据库行锁

相对于排它锁解决了跨进程的问题，缺点：

* 性能问题，在数据库层面会一直阻塞，直到事务提交，也是串行执行
* 设置事务的隔离级别是read committed，否则并发情况下，另外的事务无法看到提交的数据，依然会导致超卖
* 容易打满数据库连接，如果事务中有第三方接口交互(可能超时),会导致这个事务的连接一直阻塞，打满数据库连接
* 容易产生交叉死锁，如果多个业务的加锁控制不好，就会发生AB两条记录的交叉死锁

### 方案3：Redis分布式锁

优点：可以避免大量对数据库排他锁的征用，提高系统的响应能力

缺点：

* 设置锁和设置超时时间的原子性
> redis加锁命令setnx，设置锁的过期时间是expire，解锁命令是del。分布式环境下，A获取到了锁之后，因为线程A的业务代码耗时过长，导致锁的超时时间，锁自动失效。后续线程B就意外的持有了锁，之后线程A再次恢复执行，直接用del命令释放锁，这样就错误的将线程B同样Key的锁误删除了。代码耗时过长还是比较常见的场景，假如你的代码中有外部通讯接口调用，就容易产生这样的场景。
* 不设置超时时间的缺点
> 如果线程持有锁的过程中突然服务宕机了，这样锁就永远无法失效了。同样的也存在锁超时时间设置是否合理的问题。先给锁设置一个超时时间，然后启动一个守护线程，让守护线程在一段时间之后重新去设置这个锁的超时时间，续命锁的实现过程就是写一个守护线程，然后去判断对象锁的情况，快失效的时候，再次进行重新加锁，但是一定要判断锁的对象是同一个，不能乱续。主线程业务执行完了，守护线程也需要销毁，避免资源浪费.
* 服务宕机或线程阻塞超时的情况
* 超时时间设置不合理的情况

### 方案4：数据库乐观锁

数据库乐观锁加锁的一个原则就是尽量想办法减少锁的范围

```
update total_amount = total_amount - amount where total_amount > amount
```

* 利用事务回滚写法

```
reduce()
{
    select total_amount from table_1
    if(total_amount < amount ){
          return failed.  
    }  
    //其他业务逻辑
    int num = update total_amount = total_amount - amount where total_amount > amount;   
    if(num==0) throw Exception;}
```

* 先执行update业务逻辑，执行成功再去执行逻辑操作

```
reduce()
{
    //其他业务逻辑
    int num = update total_amount = total_amount - amount where total_amount > amount;    if(num ==1 ){
          //业务逻辑.  
    }  else{    throw Exception;  }
}
```


# Nginx和Tomcat区别
Nginx处理高并发和静态资源优于Tomcat，动态资源可以放在Tomcat
# 什么是反向代理
正向代理：客户端发送请求到代理服务器，代理服务器将请求转发至原始服务器。代理服务器所代理的对象是很多个客户端。
反向代理：对于客户而言反向代理就像原始服务器，客户不需要作任何设置，客户端发送请求，直接发送到代理服务器，代理服务器判断向何处转发请求，并将获得的内容返回给客户端。反向代理是对多个服务器进行代理。
