## 1. flink 远程调试

https://blog.csdn.net/chenyulancn/article/details/104511292

```shell
# jobmanager debug端口
env.java.opts.jobmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5006"
# taskmanager debug端口
env.java.opts.taskmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005"
```





## 2. flink conf

参考https://nightlies.apache.org/flink/flink-docs-master/docs/deployment/config/#memory-configuration

```
taskmanager.memory.process.size: 2g
taskmanager.memory.managed.size: 256m
```



```shell
jobmanager.rpc.address: localhost
jobmanager.rpc.port: 6123
jobmanager.memory.process.size: 2g

taskmanager.memory.process.size: 7g
taskmanager.memory.jvm-metaspace.size: 512m
taskmanager.memory.managed.size: 1g
taskmanager.memory.network.max: 256m
taskmanager.numberOfTaskSlots: 4

parallelism.default: 1

jobmanager.execution.failover-strategy: region
rest.port: 8081

io.tmp.dirs: /data/tmp/io
web.tmpdir: /data/tmp/web/tmpdir
web.upload.dir: /data/tmp/web/upload


# for debug
env.java.opts.jobmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5006"
env.java.opts.taskmanager: "-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005"
rest.flamegraph.enabled: true

# hashmap or rocksdb
state.backend: rocksdb
state.checkpoints.dir: file:///data/state_checkpoints

heartbeat.timeout: 600000
```







## 3. flink 单元测试

https://nightlies.apache.org/flink/flink-docs-master/zh/docs/dev/datastream/testing/#%e6%b5%8b%e8%af%95-flink-%e4%bd%9c%e4%b8%9a

> JUnit 规则 MiniClusterWithClientResource

Apache Flink 提供了一个名为 `MiniClusterWithClientResource` 的 Junit 规则，用于针对本地嵌入式小型集群测试完整的作业。 叫做 `MiniClusterWithClientResource`.

要使用 `MiniClusterWithClientResource`，需要添加一个额外的依赖项（测试范围）。

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-test-utils</artifactId>
    <version>1.15-SNAPSHOT</version>    
    <scope>test</scope>
</dependency>
```



IncrementMapFunction.java

```java
import org.apache.flink.api.common.functions.MapFunction;

public class IncrementMapFunction implements MapFunction<Long, Long> {

    @Override
    public Long map(Long record) throws Exception {
        return record + 1L;
    }
}

```



ExampleIntegrationTest.java

```java
import org.apache.flink.runtime.testutils.MiniClusterResourceConfiguration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.test.util.MiniClusterWithClientResource;
import org.junit.ClassRule;
import org.junit.Test;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

import static org.junit.Assert.*;

public class ExampleIntegrationTest {

    @ClassRule
    public static MiniClusterWithClientResource flinkCluster =
            new MiniClusterWithClientResource(
                    new MiniClusterResourceConfiguration.Builder()
                            .setNumberSlotsPerTaskManager(2)
                            .setNumberTaskManagers(1)
                            .build());

    @Test
    public void testIncrementPipeline() throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // configure your test environment
        env.setParallelism(2);

        // values are collected in a static variable
        CollectSink.values.clear();

        // 相当于
        /*env.addSource(new SourceFunction<Long>() {
            @Override
            public void run(SourceContext<Long> ctx) throws Exception {
                ctx.collect(1L);
                ctx.collect(21L);
                ctx.collect(22L);
            }

            @Override
            public void cancel() {

            }
        })
        .map(new IncrementMapFunction())
        .addSink(new CollectSink());*/

        // create a stream of custom elements and apply transformations
        env.fromElements(1L, 21L, 22L)
                .map(new IncrementMapFunction())
                .addSink(new CollectSink());

        // execute
        env.execute();

        System.out.println(CollectSink.values);  // 2L, 22L, 23L
        // verify your results
        assertTrue(CollectSink.values.containsAll(Arrays.asList(2L, 22L, 23L)));
    }

    // create a testing sink
    private static class CollectSink implements SinkFunction<Long> {

        // must be static
        public static final List<Long> values = Collections.synchronizedList(new ArrayList<>());

        @Override
        public void invoke(Long value, SinkFunction.Context context) throws Exception {
            values.add(value);
        }
    }
}
```

https://nightlies.apache.org/flink/flink-docs-release-1.14/api/java/org/apache/flink/streaming/api/windowing/triggers/CountTrigger.html



## 4. flink 异步调用

> flink 异步 + httpasyncclient 异步

https://blog.csdn.net/u010271601/article/details/104803866

```java
package com.github.async;

import org.apache.flink.streaming.api.datastream.AsyncDataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import java.util.concurrent.TimeUnit;

public class HttpAsyncQuery {
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<String> lines = env.socketTextStream("10.60.62.123", 7777);

        SingleOutputStreamOperator<String> result = AsyncDataStream.unorderedWait(
                lines,                          //输入的数据流
                new AsyncHttpQueryFunction(),   //异步查询的Function实例
                2000,                   //超时时间
                TimeUnit.MILLISECONDS,         //时间单位
                10                     //最大异步并发请求数量（并发的线程队列数）
        );

        result.print();

        env.execute("HttpAsyncQuery");
    }
}
```



```java
package com.github.async;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.async.ResultFuture;
import org.apache.flink.streaming.api.functions.async.RichAsyncFunction;
import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.nio.client.CloseableHttpAsyncClient;
import org.apache.http.impl.nio.client.HttpAsyncClients;
import org.apache.http.util.EntityUtils;
import java.util.Collections;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.Future;
import java.util.function.Consumer;
import java.util.function.Supplier;

public class AsyncHttpQueryFunction extends RichAsyncFunction<String, String> {

    private CloseableHttpAsyncClient httpclient = null;

    @Override
    public void open(Configuration parameters) throws Exception {
        // 要是api有问题？比如api不可用这个异常情况？
        RequestConfig requestConfig = RequestConfig.custom()
                .setSocketTimeout(3000)
                .setConnectTimeout(3000) //设置HttpClient连接池超时时间
                .build();

        httpclient = HttpAsyncClients.custom()
                .setMaxConnTotal(10) //连接池最大连接数
                .setDefaultRequestConfig(requestConfig)
                .build();
        httpclient.start();
        System.out.println("====");
    }

    // 关闭连接
    @Override
    public void close() throws Exception {
        httpclient.close();
    }
// 异步处理函数的主执行方法
    @Override
    public void asyncInvoke(String line, ResultFuture<String> resultFuture) throws Exception {

        // try 处理异常的json解析
        try {
            // 1. 创建httpGet请求
            HttpGet httpGet = new HttpGet("http://10.60.62.123:5000/");

            // 2、提交httpclient异步请求，获取异步请求的future对象
            Future<HttpResponse> future = httpclient.execute(httpGet, null);//callback是回调函数（也可通过回调函数拿结果）

            // 3、从成功的Future中取数据
            CompletableFuture<String> completableFuture =
                    CompletableFuture.supplyAsync(new Supplier<String>() {

                        @Override
                        public String get() {

                            // 用try包住，处理get不到值时的报错程序
                            try {
                                HttpResponse response = future.get();

                                if (response.getStatusLine().getStatusCode() == 200) {

                                    //拿出响应的实例对象, 将对象toString
                                    HttpEntity entity = response.getEntity();
                                    return EntityUtils.toString(entity);
                                } else {
                                    return null;
                                }

                            } catch (Exception e) {
                                // 拿不到的返回null(还没有查询到结果，就从future取了)
                                return null;
                            }
                        }
                    });


            // 4、将取出的数据，存入ResultFuture，返回给方法
            completableFuture.thenAccept(new Consumer<String>() {
                @Override
                public void accept(String result) {

                    //complete()里面需要的是Collection集合，但是一次执行只返回一个结果
                    //所以集合使用singleton单例模式，集合中只装一个对象
                    resultFuture.complete(Collections.singleton(result));
                }
            });

        } catch (Exception e) {
            resultFuture.complete(Collections.singleton(null));
        }
    }


}
```




# flink pulsar的应用

反序列化成实体类

```java
// 13.3的版本，用这个，以后有机会用再研究，这里只给个方向
// pulsar-flink-connector-origin-1.13.1.5-rc3.jar
public class AvroPulsarSink  extends FlinkPulsarSink {
    public AvroPulsarSink(PulsarSinkParams params,Class clazz) {
        super(params.getAdminUrl(),
                Optional.of(params.getTopic()),
                PulsarClientUtils.newClientConf(checkNotNull(params.getServiceUrl()), new Properties()),
                params.getProperties(),
                new PulsarSerializationSchemaWrapper.Builder(
                        AvroSerializationSchema.forSpecific(clazz))
                        .usePojoMode(clazz, RecordSchemaType.AVRO)
                        .build(),
                PulsarSinkSemantic.AT_LEAST_ONCE);
    }
}
```

```java
// 13.3的版本，用这个，以后有机会用再研究，这里只给个方向
// pulsar-flink-connector-origin-1.13.1.5-rc3.jar
public class AvroPulsarSource<T>  extends FlinkPulsarSource<T>  {

    public AvroPulsarSource(PulsarSourceParams params) throws ClassNotFoundException {
        super(
                params.getAdminUrl(),
                createClientConfigurationData((params.getServiceUrl()), createProperites(params.getTopic(),params.getPulsar_subscriptionname())),
                (PulsarDeserializationSchema<T>) PulsarDeserializationSchema.valueOnly(
                        AvroDeserializationSchema.forSpecific(((Class<SpecificRecordBase>) Class.forName(params.getSchemaClass())))),
                createProperites(params.getTopic(),params.getPulsar_subscriptionname()));
    }

    public static Properties createProperites(String topic,String pulsar_subscriptionname) {
        Properties prop = new Properties();

        prop.setProperty("topic", topic);
        prop.setProperty("subscriptionName", pulsar_subscriptionname);

        return prop;
    }
    
	public static ClientConfigurationData createClientConfigurationData(String serviceUrl, Properties properties) {
        ClientConfigurationData data = PulsarClientUtils.newClientConf(checkNotNull(serviceUrl), properties);

        return data;
    }

}
```



```java
// 1.14.3 处理方法（参考别人，真的垃圾）
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-pulsar_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
 </dependency>
    
PulsarSource.builder()
.setDeserializationSchema(PulsarDeserializationSchema.flinkSchema(
                            new PulsarByteDeserializer()
                    ))
// 序列化了一个寂寞！！！
public class PulsarByteDeserializer implements DeserializationSchema<byte[]> {
    @Override
    public TypeInformation<byte[]> getProducedType() {
        return TypeInformation.of(byte[].class);
    }

    @Override
    public byte[] deserialize(byte[] message) throws IOException {
        return message;
    }

    @Override
    public boolean isEndOfStream(byte[] nextElement) {
        return false;
    }
}
// 用flink map反序列化？？？ 傻逼操作吧， 合道上面不行？？？
public N_alert_tmg map(byte[] value) throws Exception {
        N_alert_tmg result;
        ByteArrayInputStream in = new ByteArrayInputStream(value);
        DatumReader<N_alert_tmg> userDatumReader = new SpecificDatumReader<>(N_alert_tmg.SCHEMA$);
        BinaryDecoder decoder = DecoderFactory.get().directBinaryDecoder(in, null);
        try {
            result = userDatumReader.read(null, decoder);
            // 事件时间还搞成这个，沃日尼玛？？？优秀！草！傻逼代码！
            result.setInsertTimestamp(System.currentTimeMillis() / 1000);
        } catch (Exception e) {
            log.error(e.toString());
            return null;
        }
        return result;
    }
```

```
// 1.14.3 sink 处理方法
public static FlinkPulsarProducer<N_alert_tmg_reduced> getSink(SinkParams params) {
        ClientConfigurationData clientConf = new ClientConfigurationData();
        clientConf.setServiceUrl(params.getServiceUrl());
        if (params.getEnableAuth()) {
            clientConf.setAuthentication((new AuthenticationToken(params.getToken())));
        } else {
            clientConf.setAuthentication((new AuthenticationDisabled()));
        }
        ProducerConfigurationData producerConf = new ProducerConfigurationData();
        producerConf.setMessageRoutingMode(MessageRoutingMode.SinglePartition);
        producerConf.setTopicName(params.getSelectedTopic());
        producerConf.setBlockIfQueueFull(true);
        producerConf.setMaxPendingMessagesAcrossPartitions(params.getMaxPendingMessagesAcrossPartitions());
        producerConf.setMaxPendingMessages(params.getMaxPendingMessages());
        
        return new FlinkPulsarProducer<>(
                clientConf,
                producerConf,
                AvroSerializationSchema.forSpecific(N_alert_tmg_reduced.class),
                null,
                null
        );
    }
```




#  源码阅读

```xml
<dependency>
    <groupId>io.streamnative.connectors</groupId>
    <artifactId>pulsar-flink-connector_2.12</artifactId>
    <version>1.14.3.4</version>
</dependency>
```



```java
import org.apache.flink.streaming.connectors.pulsar.FlinkPulsarSource;

FlinkPulsarSource
  run()
  	this.pulsarFetcher = createFetcher(ctx, ...)
    	new PulsarFetcher(sourceContext, ...);
    pulsarFetcher.runFetchLoop();
		createAndStartReaderThread(topicsToAssign, exceptionProxy));
			readerT = createReaderThread(exceptionProxy, state);
				new ReaderThread()
			readerT.setDaemon(true);
			readerT.start();

ReaderThread
  run()
    handleTooLargeCursor();
  	createActualReader(); // log.info("Create a reader at topic...")
		ReaderBuilder<T> readerBuilder = CachedPulsarClient.getOrCreate(clientConf)
		readerBuilder.create();
	Message<T> message = reader.readNext(pollTimeoutMs, TimeUnit.MILLISECONDS);  // 重点
	emitRecord(message); // while true
		MessageId messageId = message.getMessageId();
		final T record = deserializer.deserialize(message);
		owner.emitRecordsWithTimestamps(record, state, messageId, message.getEventTime());
			sourceContext.collectWithTimestamp(record, timestamp);
```





