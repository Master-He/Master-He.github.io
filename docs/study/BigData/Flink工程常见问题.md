## 1. flink 远程调试

https://blog.csdn.net/chenyulancn/article/details/104511292





## 2. flink单元测试

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