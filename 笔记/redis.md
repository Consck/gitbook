> 《Redis入门指南》（第2版）

优点：
1. redis支持的键值数据类型如下：
   * 字符串类型
   * 散列类型
   * 列表类型
   * 集合类型
   * 有序集合类型
2. 对不同的数据类型提供了方便的操作方式

    eg.使用集合类型存储文章标签，Redis可以对标签进行交集、并集等集合运算操作

Redis & memcached 区别：
* Redis是单线程模型，memcached支持多线程
* 功能丰富，memcached几乎所有功能都成为了Redis 3.0的子集
  
Redis功能：
* 为键设置生存时间(Time To Live,TTL),到期后键会自动被删除
* 限定数据占用的最大内存空间，在数据达到空间限制后按照一定规则自动淘汰不需要的键
* Redis的列表类型可以用来实现队列，并支持阻塞式读取
* 支持”发布/订阅“的消息模式
  
# Redis下载安装包并编译：

## POSIX系统

```
wget http://download.redis.io/redis-stable.tar.gz
tar xzf redis-stable.tar.gz
cd redis-stable
make
```
编译后在Redis源代码目录的src文件夹中可以找到若干个可执行程序，最好在编译后直接执行make install命令来将这些可执行程序复制到/usr/local/bin 目录中以便以后执行程序时可以不用输入完整的路径。

在实际运行Redis前推荐使用make test命令测试Redis是否编译正确，尤其是在编译一个不稳定版本的Redis时。

## OS X系统

```
安装homebrew
ruby -e "$(curl -fsSKL raw.github.com/mxcl/homebrew/go)"

若已经安装过homebrew，则更新
brew update

安装Redis
brew install redis
```

OS X系统从Tiger版本开始引入了launchd工具来管理后台程序，如果想让Redis随系统自动运行可以通过命令配置launchd：

```
ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

通过launchd运行的Redis会加载位于/usr/local/etc/redis.conf的配置文件

# Redis启动和停止
在编译后执行make install命令，这些程序会被复制到/usr/local/bin目录内，所以在命令行中直接输入程序名称即可执行

| 文件名 | 说明 | 
|:--------|:--------:|
|redis-server|redis服务器|
|redis-cli|redis命令行客户端|
|redis-benchmark|redis性能测试工具|
|redis-check-aof|AOF文件修复工具|
|redis-check-dump|RDB文件检查工具|
|redis-sentinel|Sentinel服务器(仅在2.8版以后)|

redis-server是Redis的服务器，启动Redis即运行Redis-server；redis-cli是Redis自带的Redis命令行客户端，是学习redis的重要工具。```redis-cli SHUTDOWN ```
当Redis收到命令后，会先断开所有客户端连接，然后根据配置执行持久化，最后完成退出。

Redis服务器默认会使用6379端口，通过--port参数可以自定义端口号

