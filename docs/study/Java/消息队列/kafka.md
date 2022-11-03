​	参考文章

https://kafka.apache.org/quickstart

https://zhuanlan.zhihu.com/p/95215691



# zookper

参考文章： 

https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247485115&idx=1&sn=5d269f40f820c82b460993669ca6242e&chksm=ebd747badca0ceac9953f82e08b1d1a49498ebd4af77ec5d628a0682bb9f0ac5ab347411f654&token=1741918942&lang=zh_CN#rd



杂项

- ZooKeeper可以作为**分布式锁**的一种实现。

- Kafka也需要依赖ZooKeeper。Kafka使用ZooKeeper**管理自己的元数据配置**。

- 同样ZooKeeper也可以作为**注册中心**。



简单概括一下：

- ZooKeeper主要**服务于分布式系统**，可以用ZooKeeper来做：统一配置管理、统一命名服务、分布式锁、集群管理。
- 使用分布式系统就无法避免对节点管理的问题(需要实时感知节点的状态、对节点进行统一管理等等)，而由于这些问题处理起来可能相对麻烦和提高了系统的复杂性，ZooKeeper作为一个能够**通用**解决这些问题的中间件就应运而生了。



开始学习：

ZooKeeper的数据结构，跟Unix文件系统非常类似，可以看做是一颗**树**，每个节点叫做**ZNode**。每一个节点可以通过**路径**来标识，结构图如下：

<img src="kafka.assets/640" alt="图片" style="zoom:67%;" />

那ZooKeeper这颗"树"有什么特点呢？？ZooKeeper的节点我们称之为**Znode**，Znode分为**两种**类型：

- **短暂/临时(Ephemeral)**：当客户端和服务端断开连接后，所创建的Znode(节点)**会自动删除**
- **持久(Persistent)**：当客户端和服务端断开连接后，所创建的Znode(节点)**不会删除**



ZooKeeper和Redis一样，也是C/S结构(分成客户端和服务端)

<img src="kafka.assets/640" alt="图片" style="zoom: 67%;" />



其他细节看参考文章。。。







# console client

```shell
# 消费者
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning
# 生产者
./kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic
```



# python client

```python
# 消费者
from kafka import KafkaConsumer
# To consume latest messages and auto-commit offsets
consumer = KafkaConsumer('my-topic',
                         group_id='my-group',
                         bootstrap_servers=['localhost:9092'])
for message in consumer:
    # message value and key are raw bytes -- decode if necessary!
    # e.g., for unicode: `message.value.decode('utf-8')`
    print("%s:%d:%d: key=%s value=%s" % (message.topic, message.partition,
                                         message.offset, message.key,
                                         message.value))
```

```python
# 生产者
import time
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'])

# Asynchronous by default
while True:
    future = producer.send('my-topic', b'raw_bytes')
    time.sleep(1)
```





