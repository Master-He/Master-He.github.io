# pulsar笔记

参考 https://blog.frognew.com/2021/10/learning-apache-pulsar-03.html



未整理

```
# docker 进入pulsar容器
# 1. 租户，命名空间，主题创建
./pulsar-admin tenants create 20715
./pulsar-admin namespaces create 20715/default
./pulsar-admin topics create-partitioned-topic -p 1 persistent://20715/default/network_log
# 2. 查看topic信息 web api
http://10.60.62.123:9090/admin/v2/persistent/20715/default/log/partitioned-stats
http://10.60.62.123:9090/admin/v2/persistent/20715/default/reduced/partitioned-stats
http://10.60.62.123:9090/admin/v2/persistent/20715/default/alert/partitioned-stats
# 3. 删除schema
/pulsar/bin/pulsar-admin schemas delete persistent://20715/default/network_security_log
# 4. 列出topic
./pulsar-admin topics list 20715/default

# 功能测试(shell命令)
./pulsar-client --url pulsar://10.60.62.123:6650 consume persistent://20715/default/reduced -n 0 -s "hwj730"
```



游标回退

```shell
# 查看订阅者
./pulsar-admin topics subscriptions 20715/default/log

# 游标回退
./pulsar-admin topics reset-cursor --subscription alpha_app_test_5 --time 30m persistent://20715/default/log
```

# 一、pulsar简介





## 基本概念

<img src="pulsar.assets/pulsar-logical-arch.png" alt="pulsar-logical-arch.png" style="zoom:80%;" />



### 租户(Tenant)

租户可以跨集群分布，表了组织中特定的业务单元，产品线、核心功能，这些由组织不同的部门或团队负责。每个租户都可以有单独的认证和授权机制，可以针对租户设置存储配额、消息生存时间TTL和隔离策略。

### 命名空间(Namespace)

命名空间是租户的管理单元，每个租户下可以创建多个命名空间。可以通过在命名空间上设置的配置策略来管理该命名空间下的Topic，这样就可以在命名空间的级别上为该命名空间的所有Topic设置访问权限、调整复制设置、管理跨集群跨地域的消息复制，控制消息过期时间。

### 主题(Topic)和分区主题(Partitioned Topic)

在Pulsar中所有消息的读取和写入都是和Topic进行，Pulsar的Topic本身并不区分`发布订阅模式`或者`生产消费模式(独占, 一个消息只能被一个消费者消费)`，Pulsar是依赖于各种`订阅类型`来控制消息的使用模式。



# 二、pulsar安装

docker安装 https://pulsar.apache.org/docs/zh-CN/next/standalone-docker/



# 三、 pulsar基本使用



## pulsar-admin命令

使用`pulsar-admin`查看一下集群列表，可以看到只有一个名称为`standalone`的集群:

```shell
cd /pulsar/bin
./pulsar-admin clusters list
```

查看一下租户列表，可以看到默认已经创建了`public`, `pulsar`, `sample`三个租户:

```shell
./pulsar-admin tenants list
```

创建一个名称为`study`的租户

```shell
./pulsar-admin tenants create study
./pulsar-admin tenants list
```

在study租户下创建名称为app1的命名空间:

```shell
./pulsar-admin namespaces create study/app1
./pulsar-admin namespaces list study
```

在app1的命名空间下创建一个名称为`topic-1`的topic:

```shell
./pulsar-admin topics create persistent://study/app1/topic-1
# 创建分区
/pulsar/bin/pulsar-admin topics create-partitioned-topic -p 1 persistent://study/app1/topic-1
./pulsar-admin topics list study/app1
```

查看topics的订阅者

```shell
./pulsar-admin topics subscriptions study/app1/topic-1
```





实际上，pulsar-admin命令行工具是使用Pulsar Admin REST API接口来完成工作的。Pulsar的Web接口在8080端口上。 可以直接以HTTP请求的形式访问Pulsar Admin REST API接口，例如查看study租户下的命名空间:

```
curl 127.0.0.1:8080/admin/v2/namespaces/study
# 结果： ["study/app1"]
```

https://pulsar.apache.org/admin-rest-api/是Pulsar Admin REST API的接口文档。



## 游标回退脚本

```python
#!/usr/bin/env python
# encoding: utf-8
import os
import datetime

pulsar_home = "/opt/pulsar/apache-pulsar-2.8.1"
pulsar_tenant = "tenant"
pulsar_namespace = "new_new_schema_avro"
pulsar_topic_list = ["http_log", "dns_log", "dce_rpc_log", "ssl_log", "icmp_log", "db2_log", "ldap_log", "redis_log",
                     "redis_login_log", "snmp_log", "sql_server_log", "http_login_log", "mysql_log", "ntlm_log",
                     "rdp_log", "ssh_log", "ftp_login_log", "pop3_login_log", "smtp_login_log", "oracle_log", "tcp_log",
                     "imap_login_log", "telnet_login_log", "smb_log"]
subscription_list = ["alpha_app", "alpha_engine"]

mark_time = datetime.datetime(2022, 5, 19)
now_time = datetime.datetime.now()
beforeDays = (now_time - mark_time).days


for sub in subscription_list:
    for topic in pulsar_topic_list:
        print("{}/bin/pulsar-admin topics reset-cursor --subscription {} --time {}d persistent://{}/{}/{}".format(
            pulsar_home, sub, beforeDays, pulsar_tenant, pulsar_namespace, topic))
        os.system("{}/bin/pulsar-admin topics reset-cursor --subscription {} --time {}d persistent://{}/{}/{}".format(
            pulsar_home, sub, beforeDays, pulsar_tenant, pulsar_namespace, topic))

```





# 四、pulsar web 管理

https://github.com/apache/pulsar-manager





