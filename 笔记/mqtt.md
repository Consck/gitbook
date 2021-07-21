# MQTT介绍

EMQ X 支持基于 Ekka 库的集群自动发现 (Autocluster)。Ekka 是为 Erlang/OTP 应用开发的集群管理库，支持 Erlang 节点自动发现 (Service Discovery)、自动集群 (Autocluster)、脑裂自动愈合 (Network Partition Autoheal)、自动删除宕机节点 (Autoclean)。

MQTT客户端订阅主题时，所在节点订阅成功后广播通知其他节点：某个主题被本节点订阅
MQTT客户端发布消息时，所在节点会根据消息主题，检索订阅并路由消息到相关节点
EMQ X消息服务器同一集群的所有节点，都会复制一份主题(topic)->节点(node)映射的路由表
EMQ X消息服务器每个集群节点，都保存一份主题树(Topic Trie)和路由表

注意： 节点名格式为 Name@Host, Host 必须是 IP 地址或 FQDN (主机名。域名)

## emqx_retainer插件

配置路径:etc/plugins/emqx_retainer.conf

### Qos取值
- 0：订阅者最多收到一次消息
- 1：订阅者最少收到一次消息
- 2：订阅者最少收到一次消息，且同一条消息仅允许被处理一次，需要客户端去重

## 集群节点发现端口 & RPC端口

- epmd模式：若预先设置环境变量WITH_EPMD=1，启动emqx时会使用启动epmd(监听端口4369)做节点发现
  如果集群节点间存在防火墙，防火墙需要为每个节点开通TCP 4369端口，用来让各节点能互相访问
  防火墙还需要开通一个 TCP 从 node.dist_listen_min(包含) 到 node.dist_listen_max(包含) 的端口段， 这两个配置的默认值都是 6369。
- ekka模式：若环境变量WITH_EPMD没有设置，则启动emqx时不启用epmd，而使用emqx ekka的节点发现
  跟empd 模式不同，在ekka 模式下，集群发现端口的映射关系是约定好的，而不是动态的。 node.dist_listen_min and node.dist_listen_max 两个配置在ekka 模式下不起作用。
  如果集群节点间存在防火墙，防火墙需要放开这个约定的端口。约定端口的规则如下：
        ListeningPort = BasePort + Offset
  其中 BasePort 为 4370 (不可配置), Offset 为节点名的数字后缀. 如果节点名没有数字后缀的话， Offsset 为 0。

  举例来说, 如果 emqx.conf 里配置了节点名：node.name = emqx@192.168.0.12，那么监听端口为 4370， 但对于 emqx1 (或者 emqx-1) 端口就是 4371，以此类推。

  每个节点还需要监听一个 RPC 端口，也需要被防火墙也放开。跟上面说的ekka 模式下的集群发现端口一样，这个 RPC 端口也是约定式的。

  RPC 端口的规则跟ekka 模式下的集群发现端口类似，只不过 BasePort = 5370。

  就是说，如果 emqx.conf 里配置了节点名：node.name = emqx@192.168.0.12，那么监听端口为 5370， 但对于 emqx1 (或者 emqx-1) 端口就是 5371，以此类推。


# MQTT使用

```
<dependency>
    <groupId>org.eclipse.paho</groupId>
    <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
    <version>1.2.1</version>
</dependency>
```