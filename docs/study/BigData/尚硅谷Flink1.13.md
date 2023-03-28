「尚硅谷大数据技术之flink(java) 含书签.pdf」https://www.aliyundrive.com/s/rLf7szByhuw 



# 第 1 章 初识Flink

> 特点： 实时流处理（低延迟）， 高吞吐量



>  四层API

1. SQL
2. Table API
3. DataStream/DataSet API

4. 有状态流处理



> Spark VS Flink

Spark工作中没用过，就是批处理框架，Spark Streaming也可以进行流处理，但实际上还是微型的批处理

Flink是真正的流处理



讲讲你对flink的理解

```
flink是一个高性能，分布式，可扩展的流式处理框架，它可用于实时数据处理，批处理等场景，它提供了丰富的API和工具，支持基于事件时间和处理时间的窗口计算，基于状态的迭代计算等一系列复杂的数据处理操作。

flink的核心思想是将数据处理看作一系列函数操作，然后把这一系列的函数操作根据方向，依赖关系组装成一个有向无环图（DAG）。 flink会对数据进行分区，调度和负载均衡，以保证高效的数据处理。

flink提供了精确一次语义（Exactly-one Semantics）的支持，即能够在保证数据处理的正确性的前提下，最大程度地保证数据只被处理一次。
```



flink怎么保证精确一次语义？(chatgpt)

```
在分布式计算系统中，精确一次语义（Exactly-Once Semantics）是指在进行数据处理时，保证每条数据只被处理一次，并且将结果准确地输出到外部存储系统中。Flink 通过以下两种机制来保证精确一次语义：

	1. Checkpointing：Checkpoint 是 Flink 中用于实现容错性的机制之一。Flink 会定期生成 Checkpoint，并将当前状态写入持久化存储中。在出现故障时，Flink 可以从上一个 Checkpoint 中恢复应用程序状态和数据处理进度。通过使用精确一次语义的 Checkpoint，可以确保所有进入应用程序的数据都被完整地处理且仅被处理一次。

	2. 事务性 Sink：Flink 提供了支持事务性输出的 Sink 算子，如 Flink JDBC Sink 和 Flink Elasticsearch Sink。这些 Sink 算子可以通过事务机制来确保数据在输出到外部存储时的一致性和可靠性。在这些 Sink 算子中，每个 Checkpoint 都对应着一个事务，所有在该 Checkpoint 前到达的数据都将被写入该事务。如果出现故障，则会回滚到最近一个成功的 Checkpoint，然后重新处理从该 Checkpoint 开始的数据。

需要注意的是，要想实现精确一次语义，还需要考虑数据源的幂等性问题。幂等性是指对于相同的输入数据，在不同时间和位置上处理的结果应该是相同的。在 Flink 应用程序中，需要保证数据源和算子都满足幂等性要求
```





# 第 2 章 Flink快速上手

wordcount案例



# 第 3 章 Flink 部署

## 集群启动

Flink集群

![image-20220615080801768](尚硅谷Flink1.13.assets/image-20220615080801768.png)



集群启动配置

vim flink-conf.yaml

```yaml
jobmanager.rpc.address: hadoop102
```

vim workers 

```
hadoop103
hadoop104
```





## 部署模式



### 1.会话模式

![image-20220615081604974](尚硅谷Flink1.13.assets/image-20220615081604974.png)

先启动集群再提交作业，这种方式其实就是会话模式。

会话模式比较适合于单个规模小、执行时间短的大量作业。



### 2.单作业模式

会话模式因为资源共享会导致很多问题，所以为了更好地隔离资源，我们可以考虑为每个

提交的作业启动一个集群，这就是所谓的单作业（Per-Job）模式

![image-20220615081851721](尚硅谷Flink1.13.assets/image-20220615081851721.png)

单作业模式也很好理解，就是严格的一对一，集群只为这个作业而生。同样由客户端运行应用程序，然后启动集群，作业被提交给 JobManager，进而分发给 TaskManager 执行。作业完成后，集群就会关闭，所有资源也会释放。这样一来，每个作业都有它自己的 JobManager管理，占用独享的资源，即使发生故障，它的 TaskManager 宕机也不会影响其他作业。

**注意： 单作业模式一般需要借助一些资源管理框架来启动集群，比如 YARN、Kubernetes。**





### 3.应用模式

![image-20220615082214582](尚硅谷Flink1.13.assets/image-20220615082214582.png)

应用模式与单作业模式，都是提交作业之后才创建集群；单作业模式是通过客户端来提交的，客户端解析出的每一个作业对应一个集群；而应用模式下，是直接由 JobManager 执行应用程序的，并且即使应用包含了多个作业，也只创建一个集群。



**注意： 这里对应用模式其实没有理解到位**





### 总结

总结一下，在会话模式下，集群的生命周期独立于集群上运行的任何作业的生命周期，并且提交的所有作业共享资源。而单作业模式为每个提交的作业创建一个集群，带来了更好的资源隔离，这时集群的生命周期与作业的生命周期绑定。最后，应用模式为每个应用程序创建一个会话集群，在 JobManager 上直接调用应用程序的 main()方法。



## 独立模式



## Yarn模式



## K8s模式





# 第 4 章 Flink 运行时架构



![image-20230327120251210](尚硅谷Flink1.13.assets/image-20230327120251210.png)

## flink图

说一下flink中数据流图，作业图，执行图的转换过程以及是谁负责作图的？(chatgpt)

```
在 Flink 中，作业的执行过程可以用三种图来描述：数据流图、作业图和执行图。

数据流图（Dataflow Graph）：表示 Flink 作业中数据流处理的逻辑结构，即数据如何从源头经过各个算子转换最终到达目标。数据流图是静态的，也就是说，它反映的是代码中定义的程序结构，而不考虑实际运行时的情况。在 Flink 中，数据流图通常是由开发者编写的代码自动生成的。

作业图（Job Graph）：表示 Flink 作业中逻辑上需要执行的任务和它们之间的依赖关系，包括数据源、算子、数据分区、数据交换等信息。与数据流图不同，作业图是动态生成的，它根据数据流图和当前集群状态进行优化和调度。作业图的生成是由 Flink 的 JobGraphGenerator 负责完成的。

执行图（Execution Graph）：表示 Flink 作业的实际执行过程，包括每个 Task 在哪个 TaskManager 上执行、Task 之间如何链式调用、数据交换如何进行等信息。执行图是动态生成的，它是根据作业图和当前集群状态生成的。执行图的生成是由 Flink 的 ExecutionGraphBuilder 负责完成的。

总体来说，Flink 的数据流图、作业图和执行图都是由 Flink 运行时系统自动生成的，并且在整个作业的执行过程中都扮演着重要的角色。其中，数据流图主要用于代码开发和调试阶段，作业图
```

逻辑流图（StreamGraph）→ 作业图（JobGraph）→ 执行图（ExecutionGraph）→ 物理图（Physical Graph）。

物理图就是包含数据存放位置的执行图





## flink作业提交流程

讲一讲flink作业提交的流程（chatgpt）

```
当我们要提交一个 Flink 作业时，一般的流程如下：

编写 Flink 应用程序代码：使用 Flink 提供的 API 编写具体的应用程序代码。

打包应用程序：将应用程序代码打包成 JAR 文件或者其他可执行文件，以便在 Flink 集群上运行。

启动 Flink 集群：启动 Flink 集群，可以通过脚本或命令行等方式启动。

提交应用程序：使用 Flink 提供的客户端工具 flink run 提交应用程序。flink run 命令将会向 JobManager 发送作业提交请求，同时将应用程序 JAR 包上传到指定的集群节点上。

JobManager 接收并处理请求：JobManager 接收到作业提交请求后，会对作业进行解析、生成 ExecutionGraph 并执行任务调度计算，同时将控制权交给 ResourceManager。

ResourceManager 分配资源：ResourceManager 接收到控制权后，根据作业的需求，为该作业分配所需的资源，如 TaskManager 资源、内存等。

TaskManager 运行任务：TaskManager 在接收到 ResourceManager 分配的任务资源后，开始运行相应的任务，并不断向 JobManager 汇报任务状态。

作业执行完毕：当所有任务都执行完毕后，Flink 会自动关闭作业并释放所有资源。

总之，Flink 作业提交的流程是先将应用程序打包成可执行文件，然后通过 flink run 工具提交给 JobManager，在整个过程中 Flink 集群会对作业进行调度、资源分配和任务执行等操作。
```



```
这里首先要说明一下“客户端”。其实客户端并不是处理系统的一部分，它只负责作业的提交。具体来说，就是调用程序的 main 方法，将代码转换成“数据流图”（Dataflow Graph），并最终生成作业图（JobGraph），一并发送给 JobManager。提交之后，任务的执行其实就跟客户端没有关系了

TaskManager 启动之后，JobManager 会与它建立连接，并将作业图（JobGraph）转换成可执行的“执行图”（ExecutionGraph）分发给可用的 TaskManager，然后就由 TaskManager 具体执行任务。
```



## JobManger 结构

JobManger 又包含 3 个不同的组件

**JobMaster，ResourceManager， Dispatcher**

```
jobMaster会将作业图转换成执行图， 还会负责检查点的协调
resourceManager负责资源的分配和管理， slot就是资源调度的基本单位
dispatcher 主要负责提供restapi， 用来提交应用， 并且负责为每一个新提交的作业启动一个新的jobMaster. dispatcher还会启动一个WebUI页面，方便展示监控信息
```





## flink程序结构

Flink是流式计算框架。它的程序结构，其实就是定义了一连串的处理操作，每一个数据输入之后都会依次调用每一步计算。在Flink代码中，我们定义的每一个处理转换操作都叫作“算子”（Operator），所以我们的程序可以看作是一串算子构成的管道，数据则像水流一样有序地流过。



所有的Flink程序都可以归纳为由三部分构成：Source、Transformation和Sink。

​	Source 表示“源算子”，负责读取数据源。

​	Transformation 表示“转换算子”，利用各种算子进行处理加工。

​	Sink 表示“下沉算子”，负责数据的输出。



## flink分区

讲一下flink的分区（chatgpt）

```
在 Flink 中，数据流可以被分成多个分区，每一个分区包含了数据流中的一部分数据。Flink 根据分区将数据流切分为多个子任务来并行执行计算。Flink 支持以下几种常见的分区方式：

    1. Round Robin 分区：将数据轮询地均匀地分配到下游算子的不同分区中。
    2. Hash 分区：根据指定的 Key 值进行哈希操作，将 Key 相同的数据分配到同一个分区中。
    3. Range 分区：根据 Key 的大小和范围将数据流分割成多个分区，比如按照时间戳或者价格分区等。
    4. Broadcast 分区：将数据流广播到所有下游算子中，每个下游算子都会接收到完整的数据流。
    5. Rebalance 分区：将数据流重新平衡，即将数据均匀地分配到不同的分区中，以便更好地实现负载均衡。
    6. Global 分区：将整个数据流看作是一个分区，将其发送到下游算子的一个并行任务中。

需要根据具体的业务场景和算子特性选择合适的分区方式，以提高应用程序的执行效率和可靠性。同时，在 Flink 应用程序中也可以自定义分区器，通过继承 Partitioner 接口来实现自定义分区逻辑。
```



讲一下flink的算子的并行度

```
在 Flink 中，算子并行度（Parallelism）是指一个算子运行时使用的并发任务数。Flink 通过将一个算子拆分为多个子任务来实现并行计算，每个子任务处理输入数据流中的部分数据。算子并行度通常可以通过 setParallelism() 方法进行设置。

在 Flink 应用程序中，可以在算子级别或者作业级别上设置并行度。如果没有显式设置算子的并行度，则会使用 Flink 的默认并行度。下面是一些常见的设置并行度的方法：
	1.setParallelism() 方法：该方法可以在算子级别上设置并行度，例如 source、map、filter 等算子。
	2.setMaxParallelism() 方法：该方法同样是在算子级别上设置，并行度的最大值，对于一些需要后续扩展性的算子比较有用。
	3.ExecutionConfig.setParallelismDefault() 方法：该方法可以在应用程序级别上设置默认并行度，对所有算子都生效。
	4.flink-conf.yaml 文件：也可以通过修改 flink-conf.yaml 配置文件来设置默认并行度。

需要注意的是，并行度的设置要基于资源可用性、数据倾斜等因素进行合理调整，以充分利用集群资源并提高应用程序的执行效率。
```

flink 并行度配置的优先级问题

```
在 Flink 中，算子的并行度可以在多个级别进行设置，因此存在不同级别配置的优先级。具体来说，Flink 算子的并行度配置的优先级从高到低依次是：
	1. Operator 实例级别：通过 setParallelism() 方法为算子实例单独设置的并行度，这种设置方式的优先级最高。
	2.TaskManager 内存的 slot 数量：每个 TaskManager 节点都有一定数量的 Task slot，在应用程序启动时可以通过 TaskManagerOptions.NUM_TASK_SLOTS 参数进行设置。如果算子的并行度大于 Task slot 的数量，则会出现资源不足的情况。
	3.JobManager 内存的 slot 数量：JobManager 也有一定数量的 Job slot，可以通过 SlotSharingGroup 和 Co-locationGroup 等机制向其中分配 Task slot。如果 JobManager 内存的 slot 数量小于 TaskManager 内存中可供使用的 slot 总数，则会出现无法分配 Task slot 的情况。
	4.ExecutionConfig 设置的默认并行度：可以通过 ExecutionConfig.setParallelismDefault() 方法设置默认的并行度值，在没有显式设置算子并行度时将使用该值作为默认并行度。
	5.flink-conf.yaml 配置文件中的默认并行度：在没有任何上述配置时，Flink 会使用 flink-conf.yaml 配置文件中的默认并行度来执行计算任务。

当存在多个级别的并行度设置时，Flink 会按照上述优先级顺序进行并行度配置。
```





## 算子链

算子之前传输数据的方式有哪些？

```
在 Flink 中，算子之间通常通过数据流进行数据传输。Flink 支持以下几种常见的数据传输方式：

	1. 点对点直接传输：每个算子将数据直接发送给下游算子。这种方式适用于数据量较小、计算复杂度较低的场景。

	2. Shuffle-based 数据传输：将数据根据 Key 值哈希分区，并将同一分区内的数据发送到同一个下游算子中。此种数据传输方式会涉及到网络传输和磁盘 I/O 操作，因此在大规模数据处理和高并发操作场景下需要考虑性能问题。

	3. 广播（Broadcast）传输：将数据广播到所有下游算子中，每个下游算子都可以使用该数据进行计算。这种方式适用于数据量较小且不易被修改的场景。

	4. Forwarding 数据传输：将数据简单地从上游算子直接转发到下游算子中，无需做任何额外的处理。这种方式适用于数据经过多次复杂变换后仍保持稳定的场景。 (1和4好像重合了？)

除了以上几种数据传输方式，Flink 还支持自定义数据传输方式，可以根据具体业务场景和计算特点来选择合适的数据传输方式，以提高应用程序的执行效率和可靠性。
```



算子之间的数据传输如果是一对一（one to one）的，那么两个算子可以合并成一个算子链



可以设置算子链

```
// 禁用算子链
.map(word -> Tuple2.of(word, 1L)).disableChaining();
// 从当前算子开始新链
.map(word -> Tuple2.of(word, 1L)).startNewChain()
```



## slot

slot 目前仅仅用来隔离内存，不会涉及 CPU 的隔离。在具体应用时，可以将 slot 数量配置为机器的 CPU 核心数，尽量避免不同任务之间对 CPU 的竞争。这也是开发环境默认并行度设为机器 CPU 数量的原因。



slot和并行度的关系

slot必须大于并行度，flink程序才能正常运行





# 第 5 章 DataStream API（基础篇）



## 执行环境

StreamExecutionEnvironment 类 创建执行环境， 有3个静态方法

```
getExecutionEnvironment()
createLocalEnvironment()
createRemoteEnvironment(JobManager主机名，JobManager进程端口号，提交给JobManager的JAR包路径)
```





## 物理分区

顾名思义，“分区”（partitioning）操作就是要将数据进行重新分布，传递到不同的流分区去进行下一步处理。

keyBy，它就是一种按照键的哈希值来进行重新分区的操作。只不过这种分区操作只能保证把数据按key“分开”，至于分得均不均匀、每个 key 的数据具体会分到哪一区去，这些是完全无从控制的——所以我们有时也说，keyBy 是一种逻辑分区（logical partitioning）操作。



有些时候，我们还需要手动控制数据分区分配策略。比如当发生数据倾斜的时候，系统无法自动调整，这时就需要我们重新进行负载均衡，将数据流较为平均地发送到下游任务操作分区中去。Flink 对于经过转换操作之后的 DataStream，提供了一系列的底层操作接口，能够帮我们实现数据流的手动重分区。



为了同 keyBy 相区别，我们把这些操作统称为“物理分区”操作。物理分区与 keyBy 另一大区别在于，keyBy 之后得到的是一个 KeyedStream，而物理分区之后结果仍是 DataStream，且流中元素数据类型保持不变。从这一点也可以看出，分区算子并不对数据进行转换处理，只是定义了数据的传输方式。



###  1. 随机分区（shuffle）

最简单的重分区方式就是直接“洗牌”。通过调用 DataStream 的.shuffle()方法，将数据随机地分配到下游算子的并行任务中去。

随机分区服从均匀分布（uniform distribution），所以可以把流中的数据随机打乱，均匀地传递到下游任务分区

![image-20230327172402711](尚硅谷Flink1.13.assets/image-20230327172402711.png)



### 2. 轮询分区（Round-Robin）

轮询也是一种常见的重分区方式。简单来说就是“发牌”，按照先后顺序将数据做依次分发，通过调用 DataStream 的.rebalance()方法，就可以实现轮询重分区。rebalance使用的是 Round-Robin 负载均衡算法，可以将输入流数据平均分配到下游的并行任务中去。

![image-20230327172411130](尚硅谷Flink1.13.assets/image-20230327172411130.png)

### 3. 重缩放分区（rescale）

重缩放分区和轮询分区非常相似。当调用 rescale()方法时，其实底层也是使用 Round-Robin算法进行轮询，但是只会将数据轮询发送到下游并行任务的一部分中

也就是说，“发牌人”如果有多个，那么 rebalance 的方式是每个发牌人都面向所有人发牌；而 rescale的做法是分成小团体，发牌人只给自己团体内的所有人轮流发牌。

![image-20230327172625863](尚硅谷Flink1.13.assets/image-20230327172625863.png)

当下游任务（数据接收方）的数量是上游任务（数据发送方）数量的整数倍时，rescale的效率明显会更高。比如当上游任务数量是 2，下游任务数量是 6 时，上游任务其中一个分区的数据就将会平均分配到下游任务的 3 个分区中。



由于 rebalance 是所有分区数据的“重新平衡”，当 TaskManager 数据量较多时，这种跨节点的网络传输必然影响效率；而如果我们配置的 task slot 数量合适，用 rescale 的方式进行“局部重缩放”，就可以让数据只在当前 TaskManager 的多个 slot 之间重新分配，从而避免了网络传输带来的损耗。

从底层实现上看，rebalance 和 rescale 的根本区别在于任务之间的连接机制不同。rebalance将会针对所有上游任务（发送数据方）和所有下游任务（接收数据方）之间建立通信通道，这是一个笛卡尔积的关系；而 rescale 仅仅针对每一个任务和下游对应的部分任务之间建立通信通道，节省了很多资源。



示例代码

```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;


public class RescaleTest {

    private static final Logger LOG = LogManager.getLogger(RescaleTest.class);

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 这里使用了并行数据源的富函数版本
        // 这样可以调用 getRuntimeContext 方法来获取运行时上下文的一些信息
        env.addSource(new RichParallelSourceFunction<Integer>() {
            @Override
            public void run(SourceContext<Integer> sourceContext) throws Exception {
                for (int i = 1; i <= 80; i++) {
                    // 将奇数发送到索引为 1 的并行子任务
                    // 将偶数发送到索引为 0 的并行子任务
                    if (i % 4 == getRuntimeContext().getIndexOfThisSubtask() % 4) {
                        LOG.warn("子任务索引：" + getRuntimeContext().getIndexOfThisSubtask() + "收集数据：" + i);
                        sourceContext.collect(i);
                    }
                }
            }

            @Override
            public void cancel() {
            }
        })
            .setParallelism(4)
            .rescale()
            .print().setParallelism(8); // 注意flink task manager slot要大于8
        env.execute();
    }
}
```



### 4. 广播（broadcast）

这种方式其实不应该叫做“重分区”，因为经过广播之后，数据会在不同的分区都保留一份，可能进行重复处理。可以通过调用 DataStream 的 broadcast()方法，将输入数据复制并发送到下游算子的所有并行任务中去。



### 5. 全局分区（global）

全局分区也是一种特殊的分区方式。这种做法非常极端，通过调用.global()方法，会将所有的输入流数据都发送到下游算子的第一个并行子任务中去。这就相当于强行让下游任务并行度变成了 1，所以使用这个操作需要非常谨慎，可能对程序造成很大的压力。



### 6. 自定义分区（Custom）

当 Flink 提 供 的 所 有 分 区 策 略 都 不 能 满 足 用 户 的 需 求 时 ， 我 们 可 以 通 过 使 用partitionCustom()方法来自定义分区策略。在调用时，方法需要传入两个参数，第一个是自定义分区器（Partitioner）对象，第二个是应用分区器的字段，它的指定方式与 keyBy 指定 key 基本一样：可以通过字段名称指定，也可以通过字段位置索引来指定，还可以实现一个 KeySelector。

例如，我们可以对一组自然数按照奇偶性进行重分区。代码如下：







## 源算子

### 一般的源算子

```
DataStream<String> stream = env.addSource(...);
```



### 从集合中读取数据

```
ArrayList<Event> clicks = new ArrayList<>();
DataStream<Event> stream = env.fromCollection(clicks);
```



### 从文件中读取数据

```
DataStream<String> stream = env.readTextFile("clicks.csv");
```

说明:

- 参数可以是目录，也可以是文件；

- 路径可以是相对路径，也可以是绝对路径；

- 相对路径是从系统属性 user.dir 获取路径: idea 下是 project 的根目录, standalone 模式下是集群节点根目录；

- 也可以从 hdfs 目录下读取, 使用路径 hdfs://..., 由于 Flink 没有提供 hadoop 相关依赖, 

    - 需要 pom 中添加相关依赖

        ```xml
        <dependency>
             <groupId>org.apache.hadoop</groupId>
             <artifactId>hadoop-client</artifactId>
             <version>2.7.5</version>
             <scope>provided</scope>
        </dependency>
        ```

        

### 从 Socket 读取数据

```
DataStream<String> stream = env.socketTextStream("localhost", 7777); // linux下命令： nv -l 7777 
```



### 从 Kafka 读取数据

引入依赖

```xml
<dependency>
     <groupId>org.apache.flink</groupId>
     <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
     <version>${flink.version}</version>
</dependency>
```

示例代码

```java
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;


/*
第一个参数 topic，定义了从哪些主题中读取数据。可以是一个 topic，也可以是 topic列表，还可以是匹配所有想要读取的 topic 的正则表达式。当从多个 topic 中读取数据时，Kafka 连接器将会处理所有 topic 的分区，将这些分区的数据放到一条流中去。

第二个参数是一个 DeserializationSchema 或者 KeyedDeserializationSchema。Kafka 消息被存储为原始的字节数据，所以需要反序列化成 Java 或者 Scala 对象。上面代码中使用的 SimpleStringSchema，是一个内置的 DeserializationSchema，它只是将字节数组简单地反序列化成字符串。DeserializationSchema 和 KeyedDeserializationSchema 是公共接口，所以我们也可以自定义反序列化逻辑。

第三个参数是一个 Properties 对象，设置了 Kafka 客户端的一些属性。
*/
public class SourceKafkaTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "hadoop102:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer",  "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",  "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("auto.offset.reset", "latest");

        DataStreamSource<String> stream = env.addSource(
            new FlinkKafkaConsumer<String>(
                "clicks",
                new SimpleStringSchema(),
                properties
            ));
        
        stream.print("Kafka");
        env.execute();
    }
}


```



### 自定义Source

创建一个自定义的数据源，实现 SourceFunction 接口。主要重写两个关键方法：run()和 cancel()。

- run()方法：使用运行时上下文对象（SourceContext）向下游发送数据；
- cancel()方法：通过标识位控制退出循环，来达到中断数据源的效果。

```
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Calendar;  // todo这里不是线程安全的，使用时考虑一下用java.time包替换
import java.util.Random;

public class ClickSource implements SourceFunction<Event> {
    // 声明一个布尔变量，作为控制数据生成的标识位
    private Boolean running = true;

    @Override
    public void run(SourceContext<Event> ctx) throws Exception {
        Random random = new Random(); // 在指定的数据集中随机选取数据
        String[] users = {"Mary", "Alice", "Bob", "Cary"};
        String[] urls = {"./home", "./cart", "./fav", "./prod?id=1", "./prod?id=2"};
        while (running) {
            ctx.collect(new Event(
                users[random.nextInt(users.length)],
                urls[random.nextInt(urls.length)],
                Calendar.getInstance().getTimeInMillis()
            ));
            // 隔 1 秒生成一个点击事件，方便观测
            Thread.sleep(1000);
        }
    }

    @Override
    public void cancel() {
        running = false;
    }
}
```



要注意的是 SourceFunction 接口定义的数据源，并行度只能设置为 1，如果数据源设置为大于 1 的并行度，则会抛出异常。

想要自定义并行的数据源的话，需要使用 ParallelSourceFunction

示例程序：

```java
package hwj;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.ParallelSourceFunction;

import java.util.Random;

public class ParallelSourceExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.addSource(new CustomSource()).setParallelism(2).print();
        env.execute();
    }

    public static class CustomSource implements ParallelSourceFunction<Integer> {
        private boolean running = true;
        private Random random = new Random();

        @Override
        public void run(SourceContext<Integer> sourceContext) throws Exception {
            while (running) {
                sourceContext.collect(random.nextInt());
            }
        }

        @Override
        public void cancel() {
            running = false;
        }
    }
}
```







###  Flink 支持的数据类型

Flink 在内部，Flink对支持不同的类型进行了划分，这些类型可以在 **Types 工具类**中找到

1）基本类型

- 所有 Java 基本类型及其包装类，再加上 Void、String、Date、BigDecimal 和 BigInteger。

2）数组类型

- 包括基本类型数组（PRIMITIVE_ARRAY）和对象数组(OBJECT_ARRAY)

3）复合数据类型

- Java 元组类型（TUPLE）：这是 Flink 内置的元组类型，是 Java API 的一部分。最多25 个字段，也就是从 Tuple0~Tuple25，不支持空字段

-  Scala 样例类及 Scala 元组：不支持空字段

- 行类型（ROW）：可以认为是具有任意个字段的元组,并支持空字段

-  POJO：Flink 自定义的类似于 Java bean 模式的类

4）辅助类型

- Option、Either、List、Map 等 

5）泛型类型（GENERIC）

Flink 对 POJO 类型的要求如下：

-  类是公共的（public）和独立的（standalone，也就是说没有非静态的内部类）；

- 类有一个公共的无参构造方法；

- 类中的所有字段是 public 且非 final 的；或者有一个公共的 getter 和 setter 方法，这些方法需要符合 Java bean 的命名规范。



> 类型提示

回忆一下之前的 word count 流处理程序，我们在将 String 类型的每个词转换成（word，count）二元组后，就明确地用 returns 指定了返回的类型。因为对于 map 里传入的 Lambda 表达式，系统只能推断出返回的是 Tuple2 类型，而无法得到 Tuple2<String, Long>。只有显式地告诉系统当前的返回类型，才能正确地解析出完整数据。

```java
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG));  // 基本类型
```

这是一种比较简单的场景，二元组的两个元素都是基本数据类型。那如果元组中的一个元素又有泛型，该怎么处理呢？Flink 专门提供了 TypeHint 类，它可以捕获泛型的类型信息，并且一直记录下来，为运行时提供足够的信息。我们同样可以通过.returns()方法，明确地指定转换之后的 DataStream 里元素的类型。

```java
returns(new TypeHint<Tuple2<Integer, SomeType>>(){})  // SomeType是泛型
```



## 转换算子

### 基本算子

map： 消费一个元素，产生一个元素

filter： 根据情况过滤一个元素

flatmap ： 消费一个元素，可以产生0个或多个元素， 所以 flatMap 方法也可以实现 map 方法和 filter 方法的功能



### 聚合算子Aggregation

keyBy 和聚合是成对出现的，先分区、后聚合，得到的依然是一个 DataStream。而且经过简单聚合之后的数据流，元素的数据类型保持不变



一个聚合算子，会为每一个key保存一个聚合的值，在Flink中我们把它叫作“状态”（state）。所以每当有一个新的数据输入，算子就会更新保存的聚合结果，并发送一个带有更新后聚合值的事件到下游算子。



#### 按键分区（keyBy）

基于不同的 key，流中的数据将被分配到不同的分区中去

keyBy()方法需要传入一个参数，这个参数指定了一个或一组 key

有很多不同的方法来指定 key：

- 对于 Tuple 数据类型，可以指定字段的位置或者多个位置的组合；
- 对于 POJO 类型，可以指定字段的名称（String）；
- 可以传入 Lambda 表达式或者实现一个键选择器（KeySelector），用于说明从数据中提取 key 的逻辑。



keyBy 得到的结果将不再是 DataStream，而是会将 DataStream 转换为KeyedStream。KeyedStream 可以认为是“分区流”或者“键控流”，它是对 DataStream 按照key 的一个逻辑分区，所以泛型有两个类型：除去当前流中的元素类型外，还需要指定 key 的类型。



#### 简单聚合

sum()

min()

max()

minBy() 

- 在输入流上针对指定字段求最小值。不同的是，min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值；而 minBy()则会返回包含字段最小值的整条数据。

maxBy()

- 两者区别与min()/minBy()完全一致



min和minBy的却别

```java
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransTupleAggregationTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
            StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStreamSource<Tuple3<String, Integer, String>> stream = env.fromElements(
            Tuple3.of("a", 8, "A"),
            Tuple3.of("a", 3, "B"),
            Tuple3.of("b", 9, "C"),
            Tuple3.of("b", 4, "D")
        );
        stream.keyBy(r -> r.f0).min("f1").print();
//        stream.keyBy(r -> r.f0).minBy("f1").print();
        env.execute();
    }
}

/* min()的结果
(a,8,A)
(a,3,A)
(b,9,C)
(b,4,C)
*/

/* minBy()的结果(其他字段也会被替换)
(a,8,A)
(a,3,B)
(b,9,C)
(b,4,D)
*/
```



#### 归约聚合（reduce）

reduce 同简单聚合算子一样，也要针对每一个 key 保存状态。因为状态不会清空，所以我们需要将 reduce 算子作用在一个有限 key 的流上



举例

我们将数据流按照用户 id 进行分区，然后用一个 reduce 算子实现 sum 的功能，统计每个用户访问的频次；进而将所有统计结果分到一组，用另一个 reduce 算子实现 maxBy 的功能，记录所有用户中访问频次最高的那个，也就是当前访问量最大的用户是谁。

```java
import com.github.chapter05.ClickSource;
import com.github.chapter05.Event;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class TransReduceTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
            StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 这里的 ClickSource()使用了之前自定义数据源小节中的 ClickSource()
        env.addSource(new ClickSource())
            // 将 Event 数据类型转换成元组类型
            .map(new MapFunction<Event, Tuple2<String, Long>>() {
                @Override
                public Tuple2<String, Long> map(Event e) throws Exception {
                    return Tuple2.of(e.user, 1L);
                }
            })
            .keyBy(r -> r.f0) // 使用用户名来进行分流
            .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                @Override
                public Tuple2<String, Long> reduce(Tuple2<String, Long> value1,
                                                   Tuple2<String, Long> value2) throws Exception {
                    // 每到一条数据，用户 pv 的统计值加 1
                    return Tuple2.of(value1.f0, value1.f1 + value2.f1);
                }
            })
            .keyBy(r -> true) // 为每一条数据分配同一个 key，将聚合结果发送到一条流中 去
            .reduce(new ReduceFunction<Tuple2<String, Long>>() {
                @Override
                public Tuple2<String, Long> reduce(Tuple2<String, Long> value1,
                                                   Tuple2<String, Long> value2) throws Exception {
                    // 将累加器更新为当前最大的 pv 统计值，然后向下游发送累加器的值
                    return value1.f1 > value2.f1 ? value1 : value2;
                }
            })
            .print();
        env.execute();
    }

}
```



### 用户自定义函数（UDF）

前面讲到的Source 算子，其实也是需要自定义类实现一个 SourceFunction 接口。

接口有一个共同特点：全部都以算子操作名称 + Function 命名

- map算子需要实现MapFunction接口

- reduce算子需要实现ReduceFunction接口

这就是所谓的用户自定义函数（user-defined function，UDF）。



**Rich Function Classes**

所有的Flink函数类都有其Rich版本。富函数类一般是以抽象类的形式出现的。例如：RichMapFunction、RichFilterFunction、RichReduceFunction等。

富函数类可以获取**运行环境的上下文**，并拥有一些**生命周期方法**，所以可以实现更复杂的功能。

- 增加了生命周期方法 open()，  close()
    - open()方法，是 Rich Function 的初始化方法，也就是会开启一个算子的生命周期。当一个算子的实际工作方法例如 map()或者 filter()方法被调用之前，open()会首先被调用。所以像文件 IO 的创建，数据库连接的创建，配置文件的读取等等这样一次性的工作，都适合在 open()方法中完成。
    - close()方法，是生命周期中的最后一个调用的方法，类似于解构方法。一般用来做一些清理工作。
- 增加了运行环境的上下文



## 输出算子

​		在 Flink 中，如果我们希望将数据写入外部系统，其实并不是一件难事。我们知道所有算子都可以通过实现函数类来自定义处理逻辑，所以只要有读写客户端，与外部系统的交互在任何一个处理算子中都可以实现。例如在 MapFunction 中，我们完全可以构建一个到 Redis 的连接，然后将当前处理的结果保存到 Redis 中。如果考虑到只需建立一次连接，我们也可以利用RichMapFunction，在 open() 生命周期中做连接操作。

​		这样看起来很方便，却会带来很多问题。Flink 作为一个快速的分布式实时流处理系统，对稳定性和容错性要求极高。一旦出现故障，我们应该有能力恢复之前的状态，保障处理结果的正确性。这种性质一般被称作“状态一致性”。Flink 内部提供了一致性检查点（checkpoint）来保障我们可以回滚到正确的状态；但如果我们在处理过程中任意读写外部系统，发生故障后就很难回退到从前了



总结： 在map，filter算子中连接数据库，**“精确一次（exactly once）”的状态一致性不能得到保证**，状态一致性由flink的checkpoint来保证）



一般情况下 Sink 算子的创建是通过调用 DataStream 的.addSink()方法实现的。

```java
stream.addSink(new SinkFunction(…));
```

addSource 的参数需要实现一个 SourceFunction 接口；类似地，addSink 方法同样需要传入一个参数，实现的是 SinkFunction 接口。在这个接口中只需要重写一个方法 invoke(),用来将指定的值写入到外部系统中。这个方法在每条数据记录到来时都会调用：

```java
default void invoke(IN value, Context context) throws Exception
```



SinkFuntion 多数情况下同样并不需要我们自己实现。Flink 官方提供了一部分的框架的 Sink 连接器。

![image-20230327182341478](尚硅谷Flink1.13.assets/image-20230327182341478.png)



### 输出到文件

Flink 本来也有一些非常简单粗暴的输出到文件的预实现方法：如 writeAsText()、writeAsCsv()

但我们知道，这种方式是不支持同时写入一份文件的；所以我们往往会将最后的 Sink 操作并行度设为 1，这就大大拖慢了系统效率；而且对于故障恢复后的状态一致性，也没有任何保证。所以目前这些简单的方法已经要被弃用。



Flink 为此专门提供了一个流式文件系统的连接器：**StreamingFileSink**，它继承自抽象类RichSinkFunction，而且集成了 Flink 的检查点（checkpoint）机制，用来保证精确一次（exactly once）的一致性语义。



怎么理解 StreamingFileSink 的原理(chatgpt)

```
StreamingFileSink 是 Flink 中的一个 Sink，用于将数据流写入本地或远程文件系统中的文件。其原理是将接收到的数据流按照指定的格式和规则写入文件系统中。当存在多个并行的任务时，StreamingFileSink 会对这些任务进行均衡以保证数据的顺序性和完整性。此外，StreamingFileSink 还支持一些高级功能，如恰好一次语义（exactly-once semantics）和可靠性检查点（reliable checkpoints），以确保数据写入的准确性和容错性。
```



StreamingFileSink 支持行编码（Row-encoded）和批量编码（Bulk-encoded，比如 Parquet） 格式。这两种不同的方式都有各自的构建器（builder），调用方法也非常简单，可以直接调用StreamingFileSink 的静态方法

- 行编码：StreamingFileSink.forRowFormat（basePath，rowEncoder）。

- 批量编码：StreamingFileSink.forBulkFormat（basePath，bulkWriterFactory）。



以行编码为例： 将一些测试数据直接写入文件

```java
import org.apache.flink.api.common.serialization.SimpleStringEncoder;
import org.apache.flink.core.fs.Path;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.filesystem.StreamingFileSink;
import org.apache.flink.streaming.api.functions.sink.filesystem.rollingpolicies.DefaultRollingPolicy;

import java.util.concurrent.TimeUnit;

public class SinkToFileTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(4);

        DataStreamSource<Event> stream = env.fromElements(
            new Event("Mary", "./home", 1000L),
            new Event("Bob", "./cart", 2000L),
            new Event("Alice", "./prod?id=100", 3000L),
            new Event("Alice", "./prod?id=200", 3500L),
            new Event("Bob", "./prod?id=2", 2500L),
            new Event("Alice", "./prod?id=300", 3600L),
            new Event("Bob", "./home", 3000L),
            new Event("Bob", "./prod?id=1", 2300L),
            new Event("Bob", "./prod?id=3", 3300L)
        );


        StreamingFileSink<String> fileSink = StreamingFileSink
            .<String>forRowFormat(
                new Path("/output"),
                new SimpleStringEncoder<>("UTF-8")
            )
            .withRollingPolicy(
                DefaultRollingPolicy.builder()
                    .withRolloverInterval(TimeUnit.MINUTES.toMillis(15))
                    .withInactivityInterval(TimeUnit.MINUTES.toMillis(5))
                    .withMaxPartSize(1024 * 1024 * 1024)
                    .build())
            .build();

        // 将 Event 转换成 String 写入文件
        stream.map(Event::toString).addSink(fileSink);
        env.execute();
    }
}
```

这里我们创建了一个简单的文件 Sink，通过.withRollingPolicy()方法指定了一个“滚动策略”。“滚动”的概念在日志文件的写入中经常遇到：因为文件会有内容持续不断地写入，所以我们应该给一个标准，到什么时候就开启新的文件，将之前的内容归档保存。也就是说，上面的代码设置了在以下 3 种情况下，我们就会滚动分区文件：
- 至少包含 15 分钟的数据
- 最近 5 分钟没有收到新的数据
- 文件大小已达到 1 GB



### 输出到kafka

```java
package com.github.chapter05;

import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

import java.util.Properties;

public class SinkToKafkaTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        Properties properties = new Properties();
        properties.put("bootstrap.servers", "hadoop102:9092");
        DataStreamSource<String> stream = env.readTextFile("input/clicks.csv");
        stream
            .addSink(new FlinkKafkaProducer<String>(
                "clicks",
                new SimpleStringSchema(),
                properties
            ));
        env.execute();
    }
}
```



这里我们可以看到，addSink 传入的参数是一个 FlinkKafkaProducer。这也很好理解，因为需要向 Kafka 写入数据，自然应该创建一个生产者。FlinkKafkaProducer 继承了抽象类TwoPhaseCommitSinkFunction，这是一个实现了“两阶段提交”的 RichSinkFunction。两阶段提交提供了 Flink 向 Kafka 写入数据的事务性保证，能够真正做到精确一次（exactly once）的状态一致性。



### 输出到redis



（1）导入的 Redis 连接器依赖

```xml
<dependency>
 <groupId>org.apache.bahir</groupId>
 <artifactId>flink-connector-redis_2.11</artifactId>
 <version>1.0</version>
</dependency>
```

（2）启动 Redis 集群

这里我们为方便测试，只启动了单节点 Redis。

（3）编写输出到 Redis 的示例代码

连接器为我们提供了一个 RedisSink，它继承了抽象类 RichSinkFunction，这就是已经实现好的向 Redis 写入数据的 SinkFunction。我们可以直接将 Event 数据输出到 Redis：

```java
package com.github.chapter05;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.redis.RedisSink;
import org.apache.flink.streaming.connectors.redis.common.config.FlinkJedisPoolConfig;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommand;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisCommandDescription;
import org.apache.flink.streaming.connectors.redis.common.mapper.RedisMapper;

/*
    这里 RedisSink 的构造方法需要传入两个参数：
    JFlinkJedisConfigBase：Jedis 的连接配置
    RedisMapper：Redis 映射类接口，说明怎样将数据转换成可以写入 Redis 的类型
    接下来主要就是定义一个 Redis 的映射类，实现 RedisMapper 接口。
*/
public class SinkToRedisTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
            StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        // 创建一个到 redis 连接的配置
        FlinkJedisPoolConfig conf = new FlinkJedisPoolConfig.Builder().setHost("hadoop102").build();
        env.addSource(new ClickSource())
            .addSink(new RedisSink<Event>(conf, new MyRedisMapper()));
        env.execute();
    }
	
    public static class MyRedisMapper implements RedisMapper<Event> {
        @Override
        public String getKeyFromData(Event e) {
            return e.user;
        }

        @Override
        public String getValueFromData(Event e) {
            return e.url;
        }

        @Override
        public RedisCommandDescription getCommandDescription() {
            return new RedisCommandDescription(RedisCommand.HSET, "clicks");
        }
    }
}
```





### 输出到Elasticsearch

（1）添加 Elasticsearch 连接器依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-elasticsearch7_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
```

（2）启动 Elasticsearch 集群

（3）编写输出到 Elasticsearch 的示例代码

```java
package com.github.chapter05;

import org.apache.flink.api.common.functions.RuntimeContext;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.elasticsearch.ElasticsearchSinkFunction;
import org.apache.flink.streaming.connectors.elasticsearch.RequestIndexer;
import org.apache.flink.streaming.connectors.elasticsearch7.ElasticsearchSink;
import org.apache.http.HttpHost;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.Requests;

import java.util.ArrayList;
import java.util.HashMap;


/*
与 RedisSink类似 ，连接器也为我们实现了写入到Elasticsearch 的SinkFunction——ElasticsearchSink。
区别在于，这个类的构造方法是私有（private）的，我们需要使用 ElasticsearchSink 的 Builder 内部静态类，
调用它的 build()方法才能创建出真正的SinkFunction。而 Builder 的构造方法中又有两个参数：
    - httpHosts：连接到的 Elasticsearch 集群主机列表
    - elasticsearchSinkFunction：这并不是我们所说的 SinkFunction，而是用来说明具体处理逻辑、
    准备数据向 Elasticsearch 发送请求的函数
具体的操作需要重写中 elasticsearchSinkFunction 中的 process 方法，我们可以将要发送的
数据放在一个 HashMap 中，包装成 IndexRequest 向外部发送 HTTP 请求
*/
public class SinkToEsTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        DataStreamSource<Event> stream = env.fromElements(
            new Event("Mary", "./home", 1000L),
            new Event("Bob", "./cart", 2000L),
            new Event("Alice", "./prod?id=100", 3000L),
            new Event("Alice", "./prod?id=200", 3500L),
            new Event("Bob", "./prod?id=2", 2500L),
            new Event("Alice", "./prod?id=300", 3600L),
            new Event("Bob", "./home", 3000L),
            new Event("Bob", "./prod?id=1", 2300L),
            new Event("Bob", "./prod?id=3", 3300L)
        );
        ArrayList<HttpHost> httpHosts = new ArrayList<>();
        httpHosts.add(new HttpHost("hadoop102", 9200, "http"));
        // 创建一个 ElasticsearchSinkFunction
        ElasticsearchSinkFunction<Event> elasticsearchSinkFunction = new ElasticsearchSinkFunction<Event>() {
            @Override
            public void process(Event element, RuntimeContext ctx, RequestIndexer indexer) {
                HashMap<String, String> data = new HashMap<>();
                data.put(element.user, element.url);
                IndexRequest request = Requests.indexRequest()
                    .index("clicks")
                    // .type("type") // Es 6 必须定义 type
                    .source(data);
                indexer.add(request);
            }
        };
        stream.addSink(new ElasticsearchSink.Builder<Event>(httpHosts, elasticsearchSinkFunction).build());
        env.execute();
    }
}
```





### 输出到MySQL(JDBC)

（1）添加依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
    <version>${flink.version}</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
```

（2）启动 MySQL，在 database 库下建表 clicks

```
mysql> create table clicks(
 -> user varchar(20) not null,
 -> url varchar(100) not null);
```

（3）编写输出到 MySQL 的示例代码

```java
package com.github.chapter05;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.connector.jdbc.JdbcConnectionOptions;
import org.apache.flink.connector.jdbc.JdbcExecutionOptions;
import org.apache.flink.connector.jdbc.JdbcSink;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class SinkToMySQL {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStreamSource<Event> stream = env.fromElements(
            new Event("Mary", "./home", 1000L),
            new Event("Bob", "./cart", 2000L),
            new Event("Alice", "./prod?id=100", 3000L),
            new Event("Alice", "./prod?id=200", 3500L),
            new Event("Bob", "./prod?id=2", 2500L),
            new Event("Alice", "./prod?id=300", 3600L),
            new Event("Bob", "./home", 3000L),
            new Event("Bob", "./prod?id=1", 2300L),
            new Event("Bob", "./prod?id=3", 3300L)
        );

        stream.addSink(JdbcSink.sink(
            "INSERT INTO clicks (user, url) VALUES (?, ?)",
            (statement, r) -> {
                statement.setString(1, r.user);
                statement.setString(2, r.url);
            },
            JdbcExecutionOptions.builder()
                .withBatchSize(1000)
                .withBatchIntervalMs(200)
                .withMaxRetries(5)
                .build(),
            new JdbcConnectionOptions.JdbcConnectionOptionsBuilder()
                .withUrl("jdbc:mysql://localhost:3306/userbehavior")
                // 对于 MySQL 5.7，用"com.mysql.jdbc.Driver"
                .withDriverName("com.mysql.cj.jdbc.Driver")
                .withUsername("username")
                .withPassword("password")
                .build()
            )
        );
        env.execute();
    }
}
```





### 自定义sink

Flink 并没有提供 HBase 的连接器，所以需要我们自己写

创建 HBase 的连接以及关闭 HBase 的连接需要分别放在 open()方法和 close()方法中。



（1）导入依赖

```xml
<dependency>
    <groupId>org.apache.hbase</groupId>
    <artifactId>hbase-client</artifactId>
    <version>${hbase.version}</version>
</dependency>
```

（2）编写输出到 HBase 的示例代码

```java
package com.github.chapter05;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Put;
import org.apache.hadoop.hbase.client.Table;

import java.nio.charset.StandardCharsets;

public class SinkCustomToHBase {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
            StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.fromElements("hello", "world")
            .addSink(
                new RichSinkFunction<String>() {
                    // 管理 Hbase 的配置信息,这里因为 Configuration 的重名问题，将类以完整路径导入
                    public org.apache.hadoop.conf.Configuration configuration;
                    public Connection connection; // 管理 Hbase 连接

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        super.open(parameters);
                        configuration = HBaseConfiguration.create();
                        configuration.set("hbase.zookeeper.quorum", "hadoop102:2181");
                        connection = ConnectionFactory.createConnection(configuration);
                    }

                    @Override
                    public void invoke(String value, Context context) throws Exception {
                        Table table = connection.getTable(TableName.valueOf("test")); // 表名为 test
                        Put put = new Put("rowkey".getBytes(StandardCharsets.UTF_8)); // 指定 row key
                        put.addColumn(
                            "info".getBytes(StandardCharsets.UTF_8), // 指定列名
                            value.getBytes(StandardCharsets.UTF_8), // 写入的数据
                            "1".getBytes(StandardCharsets.UTF_8) // 写入的数据
                        );
                        table.put(put); // 执行 put 操作
                        table.close(); // 将表关闭
                    }

                    @Override
                    public void close() throws Exception {
                        super.close();
                        connection.close(); // 关闭连接
                    }
                }
            );
        env.execute();
    }
}
```



（3）可以在 HBase 查看插入的数据。

# 第 6 章 Flink 中的时间和窗口

## 时间语义

flink1.12之前默认的是处理时间, flink1.12和flink1.12之后默认的是事件时间

处理时间

事件时间



## 水位线



### 怎么理解水位线watermark

watermark是当前事件的时钟，这个时钟还有一点延迟
watermark可以理解为把原本的窗口标准稍微放宽了一点。（比如原本5s，设置延迟时间=2s，那么实际等到7s的数据到达时，才认为是[0,5）的桶需要关闭了）

水位线的默认计算公式：水位线 = 观察到的最大事件时间 – 最大延迟时间 – 1 毫秒。



### 如何生成水位线watermark

每来一个数据记录最大的时间戳，然后周期性的生成watermark，watermark的值为（当前数据流中最大的时间戳 - 延迟时间）

或者每来一个数据记录最大的时间戳，然后遇到某个特定的时间戳后生成watermark



在 Flink的DataStream API中 ，有一个单独用于生成水位线的方法：.assignTimestampsAndWatermarks() 它主要用来为流中的数据分配时间戳，并生成水位线来指示事件时间

具体使用时，直接用 DataStream 调用该方法即可，与普通的 transform 方法完全一样。

```java
DataStream<Event> withTimestampsAndWatermarks = stream.assignTimestampsAndWatermarks(<watermark strategy>);
```





### 如何确定延迟时间

先假设延迟时间是0秒， 然后分析数据的乱序程度，数据5来了，同时周期性地生成了水位线w(5)
4 —> 3  —> 5 —> 1  —>2   
此时5之后来的最小的数是3， 5-3=2就是乱序程度
这个乱序程度就可以作为延迟时间



工作中的经验，如果乱序程度特别厉害，特别是数据是第三方的数据时，数据的乱序程度非常大。
比如数据的时间顺序是1->2->10->3->20->1000->4->5,  这种情况就使用处理时间比较合适了



### 水位线的特性

- 水位线是插入到数据流中的一个标记，可以认为是一个特殊的数据
- 水位线主要的内容是一个时间戳，用来表示当前事件时间的进展， 它是基于数据的时间戳生成的
- 水位线的时间戳必须单调递增，以确保任务的事件时间时钟一直向前推进
- 水位线可以通过设置延迟，来保证正确处理乱序数据
- **一个水位线 Watermark(t)，表示在当前流中事件时间已经达到了时间戳 t, 这代表 t 之前的所有数据都到齐了，之后流中不会出现时间戳 t’ ≤ t 的数据**



### 水位线周期

设置生成水位线的周期
env.getConfig.setAutoWatermarkInterval(100);





### 如何生成水位线

```java
stream.assignTimestampsAndWatermarks(<watermark strategy>); // 水位线生成策略（Watermark Strategies）
```

WatermarkStrategy中包含了

- 一个“时间戳分配器” TimestampAssigner 
- 一个“水位线生成器” WatermarkGenerator。

```java
public interface WatermarkStrategy<T> extends TimestampAssignerSupplier<T>, WatermarkGeneratorSupplier<T> {
	@Override
    TimestampAssigner<T> createTimestampAssigner(TimestampAssignerSupplier.Context context);
 
    @Override
    WatermarkGenerator<T> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context);
    
}

// TimestampAssigner：主要负责从流中数据元素的某个字段中提取时间戳，并分配给元素。时间戳的分配是生成水位线的基础。
// WatermarkGenerator：主要负责按照既定的方式，基于时间戳生成水位线。在WatermarkGenerator 接口中，主要又有两个方法：onEvent()和 onPeriodicEmit()。

// onEvent：每个事件（数据）到来都会调用的方法，它的参数有当前事件、时间戳，以及允许发出水位线的一个 WatermarkOutput，可以基于事件做各种操作

// onPeriodicEmit：周期性调用的方法，可以由 WatermarkOutput 发出水位线。周期时间为处理时间，可以调用环境配置的.setAutoWatermarkInterval()方法来设置，默认为200ms。

// env.getConfig().setAutoWatermarkInterval(60 * 1000L);
```



###  Flink 内置水位线生成器

两个生成器可以通过调用 WatermarkStrategy 的静态辅助方法来创建。它们都是周期性生成水位线的，分别对应着处理有序流和乱序流的场景。

（1）有序流

```java
// 需要注意的是，时间戳和水位线的单位，必须都是毫秒。
stream.assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
            @Override
            public long extractTimestamp(Event element, long recordTimestamp) {
                return element.timestamp;
            }
        })
);
```



（2）乱序流

由于乱序流中需要等待迟到数据到齐，所以必须设置一个固定量的延迟时间（Fixed Amount of Lateness）。**这时生成水位线的时间戳，就是当前数据流中最大的时间戳减去延迟的结果**，相当于把表调慢，当前时钟会滞后于数据的最大时间戳。调用 WatermarkStrategy. forBoundedOutOfOrderness()方法就可以实现。这个方法需要传入一个 maxOutOfOrderness 参数，表示“最大乱序程度”，它表示数据流中乱序数据时间戳的最大差值；如果我们能确定乱序程度，那么设置对应时间长度的延迟，就可以等到所有的乱序数据了。

```java
public class WatermarkTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.addSource(new ClickSource())
            // 插入水位线的逻辑
            .assignTimestampsAndWatermarks(
                // 针对乱序流插入水位线，延迟时间设置为 5s
                WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(5))
                    .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                        // 抽取时间戳的逻辑
                        @Override
                        public long extractTimestamp(Event element, long recordTimestamp) {
                            return element.timestamp;
                        }
                    })
            )
            .print();
        env.execute();
    }
}
```



事实上，有序流的水位线生成器本质上和乱序流是一样的，相当于延迟设为 0 的乱序流水位线生成器，两者完全等同：

```java
WatermarkStrategy.forMonotonousTimestamps()
WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(0))
```



#### 需要注意的是

乱序流中生成的水位线真正的时间戳，其实是 当前最大时间戳 – 延迟时间 – 1，这里的单位是毫秒。为什么要减 1 毫秒呢？我们可以回想一下水位线的特点：时间戳为 t 的水位线，表示时间戳≤t 的数据全部到齐，不会再来了。如果考虑有序流，也就是延迟时间为 0 的情况，那么时间戳为 7 秒的数据到来时，之后其实是还有可能继续来 7 秒的数据的；所以生成的水位线不是 7 秒，而是 6 秒 999 毫秒，7 秒的数据还可以继续来。这一点可以在 BoundedOutOfOrdernessWatermarks 的源码中明显地看到：

```java
public void onPeriodicEmit(WatermarkOutput output) {
     output.emitWatermark(new Watermark(maxTimestamp - outOfOrdernessMillis - 1));
}
```



### 自定义水位线策略

在 WatermarkStrategy 中，时间戳分配器 TimestampAssigner 都是大同小异的，指定字段提取时间戳就可以了；而不同策略的关键就在于 WatermarkGenerator 的实现。整体说来，Flink有两种不同的生成水位线的方式：一种是周期性的（Periodic），另一种是断点式的（Punctuated）

还记得 WatermarkGenerator 接口中的两个方法吗？——onEvent()和 onPeriodicEmit()，前者是在每个事件到来时调用，而后者由框架周期性调用。周期性调用的方法中发出水位线，自然就是周期性生成水位线；而在事件触发的方法中发出水位线，自然就是断点式生成了。两种方式的不同就集中体现在这两个方法的实现上。



#### （1）周期性水位线生成器（Periodic Generator）



```java
package com.github.chapter06;

import com.github.chapter05.ClickSource;
import com.github.chapter05.Event;
import org.apache.flink.api.common.eventtime.*;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

// 自定义水位线的产生
public class CustomWatermarkTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.addSource(new ClickSource())
            .assignTimestampsAndWatermarks(new CustomWatermarkStrategy())
            .print();
        env.execute();
    }

    public static class CustomWatermarkStrategy implements WatermarkStrategy<Event> {
        @Override
        public TimestampAssigner<Event> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
            return new SerializableTimestampAssigner<Event>() {
                @Override
                public long extractTimestamp(Event element, long recordTimestamp) {
                    return element.timestamp; // 告诉程序数据源里的时间戳是哪一个字段
                }
            };
        }

        @Override
        public WatermarkGenerator<Event> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new CustomPeriodicGenerator();
        }
    }

    public static class CustomPeriodicGenerator implements WatermarkGenerator<Event> {
        private Long delayTime = 5000L; // 延迟时间
        private Long maxTs = Long.MIN_VALUE + delayTime + 1L; // 观察到的最大时间戳

        @Override
        public void onEvent(Event event, long eventTimestamp, WatermarkOutput
            output) {
            // 每来一条数据就调用一次
            maxTs = Math.max(event.timestamp, maxTs); // 更新最大时间戳
        }

        @Override
        public void onPeriodicEmit(WatermarkOutput output) {
            // 发射水位线，默认 200ms 调用一次, 这个是系统框架周期性地调用
            output.emitWatermark(new Watermark(maxTs - delayTime - 1L));
        }
    }
}
```





#### （2）断点式水位线生成器（Punctuated Generator）



```java
package com.github.chapter06;

import com.github.chapter05.ClickSource;
import com.github.chapter05.Event;
import org.apache.flink.api.common.eventtime.*;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

// 自定义水位线的产生
public class CustomWatermarkTest2 {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.addSource(new ClickSource())
            .assignTimestampsAndWatermarks(new CustomWatermarkStrategy())
            .print();
        env.execute();
    }

    public static class CustomWatermarkStrategy implements WatermarkStrategy<Event> {
        @Override
        public TimestampAssigner<Event> createTimestampAssigner(TimestampAssignerSupplier.Context context) {
            return new SerializableTimestampAssigner<Event>() {
                @Override
                public long extractTimestamp(Event element, long recordTimestamp) {
                    return element.timestamp; // 告诉程序数据源里的时间戳是哪一个字段
                }
            };
        }

        @Override
        public WatermarkGenerator<Event> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context) {
            return new CustomPeriodicGenerator();
        }
    }

    public static class CustomPeriodicGenerator implements WatermarkGenerator<Event> {

        @Override
        public void onEvent(Event r, long eventTimestamp, WatermarkOutput output) {
            // 只有在遇到特定的 itemId 时，才发出水位线
            if (r.user.equals("Mary")) {
                output.emitWatermark(new Watermark(r.timestamp - 1));
            }
        }

        @Override
        public void onPeriodicEmit(WatermarkOutput output) {
            // 不需要做任何事情，因为我们在 onEvent 方法中发射了水位线
        }
    }
}
```





### 水位线传递

![image-20220617220150762](尚硅谷Flink1.13.assets/image-20220617220150762.png)



## 窗口

### 窗口概念

![image-20220616083142593](尚硅谷Flink1.13.assets/image-20220616083142593.png)



如果乱序流设置了水位线延迟时间 delay，也只需要等到时间戳为 end + delay 的数据，就可以关窗了。

这样就可以不再考虑时间边界问题。



### 窗口分类

> 按驱动类型分类

时间窗口，计数窗口

> 按具体实现分类

滚动窗口，滑动窗口，会话窗口，全局窗口（全局窗口需要自定义触发器，不然不会进行计算处理）



### 窗口API

```java
//其中.window()方法需要传入一个窗口分配器，它指明了窗口的类型；而后面的.aggregate()方法传入一个窗口函数作为参数，它用来定义窗口具体的处理逻辑。窗
stream.keyBy(<key selector>)
	.window(<window assigner>)
	.aggregate(<window function>)
```



###  窗口分配器



窗口分配器（Window Assigners）最通用的定义方式，就是调用.window()方法。这个方法需要传入一个WindowAssigner 作为参数，返回 WindowedStream。如果是非按键分区窗口，那么直接调用.windowAll()方法，同样传入一个 WindowAssigner，返回的是 AllWindowedStream。

```java
// 滚动处理时间窗口
stream.keyBy(...).window(TumblingProcessingTimeWindows.of(Time.seconds(5)))
// 加参数： 北京时间东八区（UTC+B）时间偏移量, 北京时间每天 0 点开启的滚动窗口
stream.keyBy(...).window(TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8)))

// 滑动处理时间窗口
stream.keyBy(...).window(SlidingProcessingTimeWindows.of(Time.seconds(10), Time.seconds(5)))

// 处理时间会话窗口
stream.keyBy(...).window(ProcessingTimeSessionWindows.withGap(Time.seconds(10)))
// 处理时间会话窗口
stream.keyBy(...).window(
    ProcessingTimeSessionWindows.withDynamicGap(new SessionWindowTimeGapExtractor<Tuple2<String, Long>>() {
        @Override
        public long extract(Tuple2<String, Long> element) { 
            // 提取 session gap 值返回, 单位毫秒
            return element.f0.length() * 1000;
        }
    })
)
    
// 滚动事件时间窗口
stream.keyBy(...).window(TumblingEventTimeWindows.of(Time.seconds(5)))
// 滑动事件时间窗口
stream.keyBy(...).window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
// 事件时间会话窗口
stream.keyBy(...).window(EventTimeSessionWindows.withGap(Time.seconds(10)))

// 计数窗口
stream.keyBy(...).countWindow(10)
// 滑动计数窗口
stream.keyBy(...).countWindow(10，3)
    
    
// 全局窗口
stream.keyBy(...).window(GlobalWindows.create());
```



### 窗口函数

定义了窗口分配器，我们只是知道了数据属于哪个窗口，可以将数据收集起来了；至于收集起来到底要做什么，其实还完全没有头绪。所以在窗口分配器之后，必须再接上一个定义窗口如何进行计算的操作，这就是所谓的“窗口函数”（window functions）。

![image-20220616085030423](尚硅谷Flink1.13.assets/image-20220616085030423.png)



#### 增量聚合函数

增量聚合函数（incremental aggregation functions）

典型的增量聚合函数有两个：ReduceFunction 和 AggregateFunction。 



>  ReduceFunction

ReduceFunction 中需要重写一个 reduce 方法，它的两个参数代表输入的两个元素，而归约最终输出结果的数据类型，与输入的数据类型必须保持一致。也就是说，中间聚合的状态和输出的结果，都和输入的数据类型是一样的。



> AggregateFunction

AggregateFunction 可以看作是 ReduceFunction 的通用版本，这里有三种类型：输入类型（IN）、累加器类型（ACC）和输出类型（OUT）。输入类型 IN 就是输入流中元素的数据类型；累加器类型 ACC 则是我们进行聚合的中间状态类型；而输出类型当然就是最终计算结果的类型了

接口中有四个方法：

- createAccumulator()：创建一个累加器，这就是为聚合创建了一个初始状态，每个聚合任务只会调用一次。

-  add()：将输入的元素添加到累加器中。这就是基于聚合状态，对新来的数据进行进一步聚合的过程。方法传入两个参数：当前新到的数据 value，和当前的累加器accumulator；返回一个新的累加器值，也就是对聚合状态进行更新。每条数据到来之后都会调用这个方法。

- getResult()：从累加器中提取聚合的输出结果。也就是说，我们可以定义多个状态，然后再基于这些聚合的状态计算出一个结果进行输出。比如之前我们提到的计算平均值，就可以把 sum 和 count 作为状态放入累加器，而在调用这个方法时相除得到最终结果。这个方法只在窗口要输出结果时调用。

- merge()：合并两个累加器，并将合并后的状态作为一个累加器返回。这个方法只在需要合并窗口的场景下才会被调用；最常见的合并窗口（Merging Window）的场景就是会话窗口（Session Windows）。



#### 全窗口函数

全窗口函数（full window functions）

窗口操作中的另一大类就是全窗口函数。与增量聚合函数不同，全窗口函数需要先收集窗口中的数据，并在内部缓存起来，等到窗口要输出结果的时候再取出数据进行计算。

为什么还需要有全窗口函数呢？

这是因为有些场景下，我们要做的计算必须基于全部的数据才有效，这时做增量聚合就没什么意义了（**比如计算中位数**）

另外，输出的结果有可能要包含上下文中的一些信息（比如**窗口的起始时间**），这是增量聚合函数做不到的。所以，我们还需要有更丰富的窗口计算方式，这就可以用全窗口函数来实现。

在 Flink 中，全窗口函数也有两种：WindowFunction 和 ProcessWindowFunction。



>WindowFunction 

WindowFunction 是老版本的通用窗口函数接口， 它的作用可以被 ProcessWindowFunction 全覆盖，所以之后可能会逐渐弃用。



> ProcessWindowFunction

ProcessWindowFunction 是 Window API 中最底层的通用窗口函数接口。

ProcessWindowFunction 还可以获取到一个“上下文对象”（Context）。这个上下文对象非常强大，不仅能够获取窗口信息，还可以访问当前的时间和状态信息。这里的时间就包括了处理时间（processing time）和事件时间水位线（event time watermark）。这就使得 ProcessWindowFunction 更加灵活、功能更加丰富。



#### 增量聚合和全窗口函数的结合使用

在实际应用中，我们往往希望兼具这两者的优点，把它们结合在一起使用。Flink 的Window API 就给我们实现了这样的用法。

我们之前在调用 WindowedStream 的.reduce()和.aggregate()方法时，只是简单地直接传入了一个 ReduceFunction 或 AggregateFunction 进行增量聚合。除此之外，其实还可以传入第二个参数：一个全窗口函数，可以是 WindowFunction 或者ProcessWindowFunction。

这样调用的处理机制是：基于第一个参数（增量聚合函数）来处理窗口数据，每来一个数据就做一次聚合；等到窗口需要触发计算时，则调用第二个参数（全窗口函数）的处理逻辑输出结果。需要注意的是，这里的全窗口函数就不再缓存所有数据了，而是直接将增量聚合函数的结果拿来当作了 Iterable 类型的输入。一般情况下，这时的可迭代集合中就只有一个元素了。

**窗口处理的主体其实还是增量聚合**，而引入全窗口函数又可以获取到更多的信息包装输出，这样的结合兼具了两种窗口函数的优势，在保证处理性能和实时性的同时支持了更加丰富的应用场景





### 其他API

#### 触发器 Trigger

触发器主要是用来控制窗口什么时候触发计算。所谓的“触发计算”，本质上就是执行窗口函数，所以可以认为是计算得到结果并输出的过程。



Trigger 是窗口算子的内部属性，每个窗口分配器（WindowAssigner）都会对应一个默认的触发器；对于 Flink 内置的窗口类型，它们的触发器都已经做了实现。例如，所有事件时间窗口，默认的触发器都是 EventTimeTrigger；类似还有 ProcessingTimeTrigger 和 CountTrigger。



Trigger 是一个抽象类，自定义时必须实现下面四个抽象方法：

- onElement()：窗口中每到来一个元素，都会调用这个方法。

- onEventTime()：当注册的事件时间定时器触发时，将调用这个方法。 

- onProcessingTime ()：当注册的处理时间定时器触发时，将调用这个方法。

- clear()：当窗口关闭销毁时，调用这个方法。一般用来清除自定义的状态。



这几个方法的参数中

都有一个“触发器上下文”（TriggerContext）对象，可以用来注册定时器回调（callback）。这里提到的“定时器”（Timer），其实就是我们设定的一个“闹钟”，代表未来某个时间点会执行的事件；当时间进展到设定的值时，就会执行定义好的操作。



上面的前三个方法可以响应事件，那它们又是怎样跟窗口操作联系起来的呢？这就需要了解一下它们的返回值。这三个方法返回类型都是 TriggerResult，这是一个枚举类型（enum），其中定义了对窗口进行操作的四种类型。

- CONTINUE（继续）：什么都不做

- FIRE（触发）：触发计算，输出结果

- PURGE（清除）：清空窗口中的所有数据，销毁窗口

- FIRE_AND_PURGE（触发并清除）：触发计算输出结果，并清除窗口



在日常业务场景中，我们经常会开比较大的窗口来计算每个窗口的pv 或者 uv 等数据。但窗口开的太大，会使我们看到计算结果的时间间隔变长。所以我们可以使用触发器，来隔一段时间触发一次窗口计算。



#### 移除器 Evictor



####  允许延迟 Allowed Lateness





# 第 7 章 处理函数



## 7.5 侧输出流





# 第 8 章 多流转换



## 分流



## 合流



union connect



# 第 9 章 状态编程



状态的分类

托管状态（Managed State）和原始状态（Raw State）

托管状态就是由 Flink 统一管理的，状态的存储访问、故障恢复和重组等一系列问题都由 Flink 实现，我们只要调接口就可以

原始状态则是自定义的，相当于就是开辟了一块内存，需要我们自己管理，实现状态的序列化和故障恢复。



## 托管状态

我们知道在 Flink 中，一个算子任务会按照并行度分为多个并行子任务执行，而不同的子任务会占据不同的任务槽（task slot）。

**<u>由于不同的 slot 在计算资源上是物理隔离的，所以 Flink能管理的状态在并行任务间是无法共享的</u>**，每个状态只能针对当前子任务的实例有效。而很多有状态的操作（比如聚合、窗口）都是要先做 keyBy 进行按键分区的。按键分区之后，任务所进行的所有计算都应该只针对当前 key 有效，所以状态也应该按照 key 彼此隔离。在这种情况下，状态的访问方式又会有所不同。

所以我们又可以将托管状态分为两类：算子状态和按键分区状态。







## 按键分区状态（Keyed State）

状态是根据输入流中定义的键（key）来维护和访问的，所以只能定义在按键分区流（KeyedStream）中，也就 keyBy 之后才可以使用

![image-20220619112542871](尚硅谷Flink1.13.assets/image-20220619112542871.png)

​		按键分区状态应用非常广泛。之前讲到的聚合算子必须在 keyBy 之后才能使用，就是因为聚合的结果是以 Keyed State 的形式保存的。另外，也可以通过富函数类（Rich Function）来自定义 Keyed State

​		所以只要提供了富函数类接口的算子，也都可以使用 Keyed State。所以即使是 map、filter 这样无状态的基本转换算子，我们也可以通过富函数类给它们“追加”Keyed State，或者实现 CheckpointedFunction 接口来定义 Operator State；从这个角度讲，Flink 中所有的算子都可以是有状态的，不愧是“有状态的流处理”。

​		无论是 Keyed State 还是 Operator State，它们都是在本地实例上维护的，也就是说每个并行子任务维护着对应的状态，**算子的子任务之间状态不共享**。关于状态的具体使用，我们会在下面继续展开讲解。



​		按键分区状态（Keyed State）顾名思义，是任务按照键（key）来访问和维护的状态。它的特点非常鲜明，就是以 key 为作用范围进行隔离。

​		我们知道，**在进行按键分区（keyBy）之后，具有相同键的所有数据，都会分配到同一个并行子任务中**；

​		所以如果当前任务定义了状态，Flink 就会在当前并行子任务实例中，为每个键值维护一个状态的实例。于是当前任务就会为分配来的所有数据，按照 key 维护和处理对应的状态。

​		因为一个并行子任务可能会处理多个 key 的数据，所以 Flink 需要对 Keyed State 进行一些特殊优化。在底层，Keyed State 类似于一个分布式的映射（map）数据结构，所有的状态会根据 key 保存成键值对（key-value）的形式。这样当一条数据到来时，任务就会自动将状态的访问范围限定为当前数据的 key，从 map 存储中读取出对应的状态值。所以具有相同 key 的所有数据都会到访问相同的状态，而不同 key 的状态之间是彼此隔离的。

​		这种将状态绑定到 key 上的方式，相当于使得状态和流的逻辑分区一一对应了：不会有别的 key 的数据来访问当前状态；而当前状态对应 key 的数据也只会访问这一个状态，不会分发到其他分区去。这就保证了对状态的操作都是本地进行的，对数据流和状态的处理做到了分区一致性。

​		另外，在应用的并行度改变时，状态也需要随之进行重组。不同 key 对应的 Keyed State可以进一步组成所谓的键组（key groups），每一组都对应着一个并行子任务。键组是 Flink 重新分配 Keyed State 的单元，键组的数量就等于定义的最大并行度。当算子并行度发生改变时，Keyed State 就会按照当前的并行度重新平均分配，保证运行时各个子任务的负载相同。

​		需要注意，**使用 Keyed State 必须基于 KeyedStream。没有进行 keyBy 分区的 DataStream，即使转换算子实现了对应的富函数类，也不能通过运行时上下文访问 Keyed State。**



### 支持的结构类型

>  值状态（ValueState）

```java
public interface ValueState<T> extends State {
    T value() throws IOException;
    void update(T value) throws IOException;
}
```

在具体使用时，为了让运行时上下文清楚到底是哪个状态，我们还需要创建一个“状态描述器”（StateDescriptor）来提供状态的基本信息。例如源码中，ValueState 的状态描述器构造  方法如下：

```java
public ValueStateDescriptor(String name, Class<T> typeClass) {
    super(name, typeClass, null);
}
```

这里需要传入状态的名称和类型——这跟我们声明一个变量时做的事情完全一样。有了这个描述器，运行时环境就可以获取到状态的控制句柄（handler）了。



> 列表状态（ListState）

将需要保存的数据，以列表（List）的形式组织起来。在 ListState<T>接口中同样有一个类型参数 T，表示列表中数据的类型。ListState 也提供了一系列的方法来操作状态，使用方式与一般的 List 非常相似。

- Iterable<T> get()：获取当前的列表状态，返回的是一个可迭代类型 Iterable<T>； 

- update(List<T> values)：传入一个列表 values，直接对状态进行覆盖；

- add(T value)：在状态列表中添加一个元素 value； 

- addAll(List<T> values)：向列表中添加多个元素，以列表 values 形式传入。

ListState 的状态描述器就叫作 ListStateDescriptor，用法跟 ValueStateDescriptor完全一致。



> 映射状态（MapState）

把一些键值对（key-value）作为状态整体保存起来，可以认为就是一组 key-value 映射的列表。对应的 MapState<UK, UV>接口中，就会有 UK、UV 两个泛型，分别表示保存的 key 和 value 的类型。同样，MapState 提供了操作映射状态的方法，与 Map 的使用非常类似。

- UV get(UK key)：传入一个 key 作为参数，查询对应的 value 值；

- put(UK key, UV value)：传入一个键值对，更新 key 对应的 value 值；

- putAll(Map<UK, UV> map)：将传入的映射 map 中所有的键值对，全部添加到映射状态中；

- remove(UK key)：将指定 key 对应的键值对删除；

- boolean contains(UK key)：判断是否存在指定的 key，返回一个 boolean 值。

另外，MapState 也提供了获取整个映射相关信息的方法：

- Iterable<Map.Entry<UK, UV>> entries()：获取映射状态中所有的键值对；

- Iterable<UK> keys()：获取映射状态中所有的键（key），返回一个可迭代 Iterable 类型；

- Iterable<UV> values()：获取映射状态中所有的值（value），返回一个可迭代 Iterable类型；

- boolean isEmpty()：判断映射是否为空，返回一个 boolean 值。



> 归约状态（ReducingState）

​		类似于值状态（Value），不过需要对添加进来的所有数据进行归约，将归约聚合之后的值作为状态保存下来。

​		ReducintState<T>这个接口调用的方法类似于 ListState，只不过它保存的只是一个聚合值，所以调用.add()方法时，不是在状态列表里添加元素，而是直接把新数据和之前的状态进行归约，并用得到的结果更新状态。

​		归约逻辑的定义，是在归约状态描述器（ReducingStateDescriptor）中，通过传入一个归约函数（ReduceFunction）来实现的。这里的归约函数，就是我们之前介绍 reduce 聚合算子时讲到的 ReduceFunction，所以**状态类型跟输入的数据类型是一样的。**

```java
// 这里的描述器有三个参数，其中第二个参数就是定义了归约聚合逻辑的 ReduceFunction，另外两个参数则是状态的名称和类型。
public ReducingStateDescriptor( String name, ReduceFunction<T> reduceFunction, Class<T> typeClass) {...}
```



>  聚合状态（AggregatingState）

与归约状态非常类似，聚合状态也是一个值，用来保存添加进来的所有数据的聚合结果。与 ReducingState 不同的是，它的聚合逻辑是由在描述器中传入一个更加一般化的聚合函数（AggregateFunction）来定义的



### 代码实现

在 Flink 中，状态始终是与特定算子相关联的；算子在使用状态前首先需要“注册”，其实就是告诉 Flink 当前上下文中定义状态的信息，这样运行时的 Flink 才能知道算子有哪些状态。

状态的注册，主要是通过“状态描述器”（StateDescriptor）来实现的。状态描述器中最重要的内容，就是状态的名称（name）和类型（type）。

以 ValueState 为例，我们可以定义值状态描述器如下：

```java
ValueStateDescriptor<Long> descriptor = new ValueStateDescriptor<>(
	"my state", // 状态名称
	Types.LONG // 状态类型
);
```

代码中完整的操作是，首先定义出状态描述器；然后调用.getRuntimeContext()方法获取运行时上下文；继而调用 RuntimeContext 的获取状态的方法，将状态描述器传入，就可以得到对应的状态了。

因为状态的访问需要获取运行时上下文，这只能在富函数类（Rich Function）中获取到，所以自定义的 Keyed State 只能在富函数中使用。当然，底层的处理函数（Process Function）本身继承了 AbstractRichFunction 抽象类，所以也可以使用。

在富函数中，调用.getRuntimeContext()方法获取到运行时上下文之后，RuntimeContext 有以下几个获取状态的方法：

```java
ValueState<T> getState(ValueStateDescriptor<T>)
MapState<UK, UV> getMapState(MapStateDescriptor<UK, UV>)
ListState<T> getListState(ListStateDescriptor<T>)
ReducingState<T> getReducingState(ReducingStateDescriptor<T>)
AggregatingState<IN, OUT> getAggregatingState(AggregatingStateDescriptor<IN, ACC, OUT>)
```

对于不同结构类型的状态，只要传入对应的描述器、调用对应的方法就可以了。

获取到状态对象之后，就可以调用它们各自的方法进行读写操作了。另外，所有类型的状态都有一个方法.clear()，用于清除当前状态。



代码中使用状态的整体结构如下：

```java
package com.github.chapter09.section01;

import org.apache.flink.api.common.functions.RichFlatMapFunction;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.util.Collector;

public class MyFlatMapFunction extends RichFlatMapFunction<Long, String> {
    // 声明状态
    private transient ValueState<Long> state;

    @Override
    public void open(Configuration config) {
        // 在 open 生命周期方法中获取状态
        ValueStateDescriptor<Long> descriptor = new ValueStateDescriptor<>(
            "my state", // 状态名称
            Types.LONG // 状态类型
        );
        state = getRuntimeContext().getState(descriptor);
    }

    @Override
    public void flatMap(Long input, Collector<String> out) throws Exception {
        // 访问状态
        Long currentState = state.value();
        currentState += 1; // 状态数值加 1
        // 更新状态
        state.update(currentState);
        if (currentState >= 10) {
            out.collect("state: "+currentState);
            state.clear(); // 清空状态
        }
    }
}
```

因为 RichFlatmapFunction 中的.flatmap()是每来一条数据都会调用一次的，所以我们不应该在这里调用运行时上下文的.getState()方法，而是在生命周期方法.open()中获取状态对象。另外还有一个问题，我们获取到的状态对象也需要有一个变量名称 state（注意这里跟状态的名称 my state 不同），但这个变量不应该在 open 中声明——否则在.flatmap()里就访问不到了。所以我们还需要在外面直接把它定义为类的属性，这样就可以在不同的方法中通用了。而在外部又不能直接获取状态，因为编译时是无法拿到运行时上下文的。所以最终的解决方案就变成了：在外部声明状态对象，在 open 生命周期方法中通过运行时上下文获取状态。

这里需要注意，这种方式定义的都是 Keyed State，它对于每个 key 都会保存一份状态实例。所以对状态进行读写操作时，获取到的状态跟当前输入数据的 key 有关；只有相同 key的数据，才会操作同一个状态，不同 key 的数据访问到的状态值是不同的。而且上面提到的.clear()方法，也只会清除当前 key 对应的状态。

另外，状态不一定都存储在内存中，也可以放在磁盘或其他地方，具体的位置是由一个可配置的组件来管理的，这个组件叫作“状态后端”（State Backend）。





## 状态生存时间（TTL）

如果用一个进程不停地扫描所有状态看是否过期，显然会占用大量资源做无用功。状态的失效其实不需要立即删除，所以我们可以给状态附加一个属性，也就是状态的“失效时间”。状态创建的时候，设置 失效时间 = 当前时间 + TTL；之后如果有对状态的访问和修改，我们可以再对失效时间进行更新；当设置的清除条件被触发时（比如，状态被访问的时候，或者每隔一段时间扫描一次失效状态），就可以判断状态是否失效、从而进行清除了。

配置状态的 TTL 时，需要创建一个 StateTtlConfig 配置对象，然后调用状态描述器的.enableTimeToLive()方法启动 TTL 功能。

```java
StateTtlConfig ttlConfig = StateTtlConfig
 .newBuilder(Time.seconds(10))
 .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
 .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
 .build();
ValueStateDescriptor<String> stateDescriptor = new ValueStateDescriptor<>("mystate", String.class);
stateDescriptor.enableTimeToLive(ttlConfig);
```

这里用到了几个配置项:

.newBuilder()

状态 TTL 配置的构造器方法，必须调用，返回一个 Builder 之后再调用.build()方法就可以得到 StateTtlConfig 了。方法需要传入一个 Time 作为参数，这就是设定的状态生存时间。

.setUpdateType()

设置更新类型。更新类型指定了什么时候更新状态失效时间，这里的 OnCreateAndWrite表示只有创建状态和更改状态（写操作）时更新失效时间。另一种类型 OnReadAndWrite 则表示无论读写操作都会更新失效时间，也就是只要对状态进行了访问，就表明它是活跃的，从而延长生存时间。这个配置默认为 OnCreateAndWrite。

.setStateVisibility()

设置状态的可见性。所谓的“状态可见性”，是指因为清除操作并不是实时的，所以当状态过期之后还有可能基于存在，这时如果对它进行访问，能否正常读取到就是一个问题了。这里设置的 NeverReturnExpired 是默认行为，表示从不返回过期值，也就是只要过期就认为它已经被清除了，应用不能继续读取；这在处理会话或者隐私数据时比较重要。对应的另一种配置是 ReturnExpireDefNotCleanedUp，就是如果过期状态还存在，就返回它的值。

除此之外，TTL 配置还可以设置在保存检查点（checkpoint）时触发清除操作，或者配置增量的清理（incremental cleanup），还可以针对 RocksDB 状态后端使用压缩过滤器（compaction filter）进行后台清理。

这里需要注意，**目前的 TTL 设置只支持处理时间。**另外，所有集合类型的状态（例如ListState、MapState）在设置 TTL 时，都是针对每一项（per-entry）元素的。也就是说，一个列表状态中的每一个元素，都会以自己的失效时间来进行清理，而不是整个列表一起清理。





## 算子状态（Operator State）

状态作用范围限定为当前的算子任务实例，也就是只对当前并行子任务实例有效。这就意味着对于一个并行子任务，占据了一个“分区”，它所处理的所有数据都会访问到相同的状态，状态对于同一任务而言是共享的。 如图所示

![image-20220619112217255](尚硅谷Flink1.13.assets/image-20220619112217255.png)





除按键分区状态（Keyed State）之外，另一大类受控状态就是算子状态（Operator State）。从某种意义上说，算子状态是更底层的状态类型，因为它只针对当前算子并行任务有效，不需要考虑不同 key 的隔离。算子状态功能不如按键分区状态丰富，应用场景较少，它的调用方法也会有一些区别。

算子状态也支持不同的结构类型，主要有三种：ListState、UnionListState 和 BroadcastState。 



> 列表状态（ListState） 

当算子并行度进行缩放调整时，算子的列表状态中的所有元素项会被统一收集起来，相当于把多个分区的列表合并成了一个“大列表”，然后再均匀地分配给所有并行任务。这种“均匀分配”的具体方法就是“轮询”（round-robin），与之前介绍的 rebanlance 数据传输方式类似，是通过逐一“发牌”的方式将状态项平均分配的。这种方式也叫作“平均分割重组”（even-split redistribution）。

算子状态中不会存在“键组”（key group）这样的结构，所以**为了方便重组分配**，就把它直接定义成了“列表”（list）。这也就解释了，为什么算子状态中没有最简单的值状态（ValueState）。

```java
ListStateDescriptor<String> descriptor = new ListStateDescriptor<>(
    "buffered-elements",
    Types.of(String));
ListState<String> checkpointedState = context.getOperatorStateStore().getListState(descriptor);
```



我们已经知道，状态从本质上来说就是算子并行子任务实例上的一个特殊本地变量。它的特殊之处就在于 Flink 会提供完整的管理机制，来保证它的持久化保存，以便发生故障时进行状态恢复；另外还可以针对不同的 key 保存独立的状态实例。按键分区状态（Keyed State）对这两个功能都要考虑；而算子状态（Operator State）并不考虑 key 的影响，所以主要任务就是要让 Flink 了解状态的信息、将状态数据持久化后保存到外部存储空间。



看起来算子状态的使用应该更加简单才对。不过仔细思考又会发现一个问题：我们对状态进行持久化保存的目的是为了故障恢复；在发生故障、重启应用后，数据还会被发往之前分配的分区吗？显然不是，因为并行度可能发生了调整，不论是按键（key）的哈希值分区，还是直接轮询（round-robin）分区，数据分配到的分区都会发生变化。这很好理解，当打牌的人数从 3 个增加到 4 个时，即使牌的次序不变，轮流发到每个人手里的牌也会不同。数据分区发生变化，带来的问题就是，怎么保证原先的状态跟故障恢复后数据的对应关系呢？

对于 Keyed State 这个问题很好解决：状态都是跟 key 相关的，而相同 key 的数据不管发往哪个分区，总是会全部进入一个分区的；于是只要将状态也按照 key 的哈希值计算出对应的分区，进行重组分配就可以了。恢复状态后继续处理数据，就总能按照 key 找到对应之前的状态，就保证了结果的一致性。所以 Flink 对 Keyed State 进行了非常完善的包装，我们不需实现任何接口就可以直接使用。

而对于 Operator State 来说就会有所不同。因为不存在 key，所有数据发往哪个分区是不可预测的；也就是说，当发生故障重启之后，我们不能保证某个数据跟之前一样，进入到同一个并行子任务、访问同一个状态。所以 Flink 无法直接判断该怎样保存和恢复状态，而是提供了接口(CheckpointedFunction 接口)，让我们根据业务需求自行设计状态的快照保存（snapshot）和恢复（restore）逻辑。



在 Flink 中，对状态进行持久化保存的快照机制叫作“检查点”（Checkpoint）。于是使用算子状态时，就需要对检查点的相关操作进行定义，实现一个 CheckpointedFunction 接口。

```java
public interface CheckpointedFunction {
	// 保存状态快照到检查点时，调用这个方法
	void snapshotState(FunctionSnapshotContext context) throws Exception
	// 初始化状态时调用这个方法，也会在恢复状态时调用
 	void initializeState(FunctionInitializationContext context) throws Exception;
}
```

每次应用保存检查点做快照时，都会调用.snapshotState()方法，将状态进行外部持久化。而在算子任务进行初始化时，会调用. initializeState()方法。这又有两种情况：一种是整个应用第一次运行，这时状态会被初始化为一个默认值（default value）；另一种是应用重启时，从检查点（checkpoint）或者保存点（savepoint）中读取之前状态的快照，并赋给本地状态。所以，接口的.snapshotState()方法定义了检查点的快照保存逻辑，而. initializeState()方法不仅定义了初始化逻辑，也定义了恢复逻辑。

这里需要注意，CheckpointedFunction 接口中的两个方法，分别传入了一个上下文（context）作为参数。不同的是，.snapshotState()方法拿到的是快照的上下文 FunctionSnapshotContext，它可以提供检查点的相关信息，不过无法获取状态句柄；而. initializeState()方法拿到的是FunctionInitializationContext，这是函数类进行初始化时的上下文，是真正的“运行时上下文”。FunctionInitializationContext 中提供了“算子状态存储”（OperatorStateStore）和“按键分区状态存储（” KeyedStateStore），在这两个存储对象中可以非常方便地获取当前任务实例中的 OperatorState 和 Keyed State。

例子：

```java
package com.github.chapter09.section05;

import com.github.chapter05.ClickSource;
import com.github.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.runtime.state.FunctionInitializationContext;
import org.apache.flink.runtime.state.FunctionSnapshotContext;
import org.apache.flink.streaming.api.checkpoint.CheckpointedFunction;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;

import java.util.ArrayList;
import java.util.List;

public class BufferingSinkExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        env.enableCheckpointing(1000);
        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
            .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event element, long
                        recordTimestamp) {
                        return element.timestamp;
                    }
                })
            );
        stream.print("input");
        // 批量缓存输出
        stream.addSink(new BufferingSink(10));
        env.execute();
    }
	
    public static class BufferingSink implements SinkFunction<Event>, CheckpointedFunction {
        private final int threshold;
        private transient ListState<Event> checkPointedState;
        private List<Event> bufferedElements;

        public BufferingSink(int threshold) {
            this.threshold = threshold;
            this.bufferedElements = new ArrayList<>();
        }

        @Override
        public void invoke(Event value, Context context) throws Exception {
            bufferedElements.add(value);
            if (bufferedElements.size() == threshold) {
                for (Event element : bufferedElements) {
                    // 输出到外部系统，这里用控制台打印模拟
                    System.out.println(element);
                }
                System.out.println("==========输出完毕=========");
                bufferedElements.clear();
            }
        }

        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
            checkPointedState.clear();
            // 把当前局部变量中的所有元素写入到检查点中
            for (Event element : bufferedElements) {
                checkPointedState.add(element);
            }
        }

        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {
            ListStateDescriptor<Event> descriptor = new ListStateDescriptor<>(
                "buffered-elements",
                Types.POJO(Event.class));

            checkPointedState = context.getOperatorStateStore().getListState(descriptor);

            // 如果是从故障中恢复，就将 ListState 中的所有元素添加到局部变量中
            if (context.isRestored()) {
                System.out.println("======从故障中恢复=======");
                for (Event element : checkPointedState.get()) {
                    bufferedElements.add(element);
                }
            }
        }
    }
}
```







> 联合列表状态（UnionListState） 

与 ListState 类似，联合列表状态也会将状态表示为一个列表。它与常规列表状态的区别在于，算子并行度进行缩放调整时对于状态的分配方式不同。UnionListState 的重点就在于“联合”（union）。在并行度调整时，常规列表状态是轮询分配状态项，而联合列表状态的算子则会直接广播状态的完整列表。这样，并行度缩放之后的并行子任务就获取到了联合后完整的“大列表”，可以自行选择要使用的状态项和要丢弃的状态项。这种分配也叫作“联合重组”（union redistribution）。如果列表中状态项数量太多，为资源和效率考虑一般不建议使用联合重组的方式。



>  广播状态（BroadcastState）

有时我们希望算子并行子任务都保持同一份“全局”状态，用来做统一的配置和规则设定。这时所有分区的所有数据都会访问到同一个状态，状态就像被“广播”到所有分区一样，这种特殊的算子状态，就叫作广播状态（BroadcastState）



在代码上，可以直接调用 DataStream 的.broadcast()方法，传入一个“映射状态描述器”（MapStateDescriptor）说明状态的名称和类型，就可以得到一个“广播流”（BroadcastStream）；进而将要处理的数据流与这条广播流进行连接（connect），就会得到“广播连接流”（BroadcastConnectedStream）。**注意广播状态只能用在广播连接流中**



广播连接流

```java
MapStateDescriptor<String, Rule> ruleStateDescriptor = new MapStateDescriptor<>(...);
BroadcastStream<Rule> ruleBroadcastStream = ruleStream.broadcast(ruleStateDescriptor);
DataStream<String> output = stream
    .connect(ruleBroadcastStream)
	.process( new BroadcastProcessFunction<>() {...} );
```

这里我们定义了一个“规则流”ruleStream，里面的数据表示了数据流 stream 处理的规则，规则的数据类型定义为 Rule。于是需要先定义一个 MapStateDescriptor 来描述广播状态，然后传入 ruleStream.broadcast()得到广播流，接着用 stream 和广播流进行连接。这里状态描述器中的 key 类型为 String，就是为了区分不同的状态值而给定的 key 的名称对 于 广 播 连 接 流 调 用 .process() 方 法 ， 可 以 传 入 “ 广 播 处 理 函 数 ”KeyedBroadcastProcessFunction 或者 BroadcastProcessFunction 来进行处理计算。广播处理函数里面有两个方法.processElement()和.processBroadcastElement()，源码中定义如下：

```java
public abstract class BroadcastProcessFunction<IN1, IN2, OUT> extends BaseBroadcastProcessFunction {
	...
 	public abstract void processElement(IN1 value, ReadOnlyContext ctx, Collector<OUT> out) throws Exception;
 	public abstract void processBroadcastElement(IN2 value, Context ctx, Collector<OUT> out) throws Exception;
...
}
```

这里的.processElement()方法，处理的是正常数据流，第一个参数 value 就是当前到来的流数据；而.processBroadcastElement()方法就相当于是用来处理广播流的，它的第一个参数 value就是广播流中的规则或者配置数据。两个方法第二个参数都是一个上下文 ctx，都可以通过调用.getBroadcastState()方法获取到当前的广播状态；区别在于，.processElement()方法里的上下文 是 “ 只 读 ” 的 （ ReadOnly ）， 因 此 获 取 到 的 广 播 状 态 也 只 能 读 取 不 能 更 改 ；而.processBroadcastElement()方法里的 Context 则没有限制，可以根据当前广播流中的数据更新状态。

```java
Rule rule = ctx.getBroadcastState( new MapStateDescriptor<>("rules", Types.String, Types.POJO(Rule.class))).get("my rule");
```

通过调用 ctx.getBroadcastState()方法，传入一个 MapStateDescriptor，就可以得到当前的叫作“rules”的广播状态；调用它的.get()方法，就可以取出其中“my rule”对应的值进行计算处理。





## 状态持久化和状态后端



### 检查点

在 Flink 的状态管理机制中，很重要的一个功能就是对状态进行持久化（persistence）保存，这样就可以在发生故障后进行重启恢复。Flink 对状态进行持久化的方式，就是将当前所有分布式状态进行“快照”保存，写入一个“检查点”（checkpoint）或者保存（savepoint）保存到外部存储系统中。具体的存储介质，一般是分布式文件系统（distributed file system）。



**默认情况下，检查点是被禁用的，需要在代码中手动开启。直接调用执行环境的.enableCheckpointing()方法就可以开启检查点。**

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();
env.enableCheckpointing(1000);
```



除了检查点之外，Flink 还提供了“保存点”（savepoint）的功能。保存点在原理和形式上跟检查点完全一样，也是状态持久化保存的一个快照；区别在于，保存点是自定义的镜像保存，所以不会由 Flink 自动创建，而需要用户手动触发。这在有计划地停止、重启应用时非常有用。



### 状态后端

检查点的保存离不开 JobManager 和 TaskManager，以及外部存储系统的协调。在应用进行检查点保存时，首先会由 JobManager 向所有 TaskManager 发出触发检查点的命令；TaskManger 收到之后，将当前任务的所有状态进行快照保存，持久化到远程的存储介质中；完成之后向 JobManager 返回确认信息。这个过程是分布式的，当 JobManger 收到所有TaskManager 的返回信息后，就会确认当前检查点成功保存



![image-20220620004526137](尚硅谷Flink1.13.assets/image-20220620004526137.png)

在 Flink 中，状态的存储、访问以及维护，都是由一个可插拔的组件决定的，这个组件就叫作状态后端（state backend）。状态后端主要负责两件事：一是本地的状态管理，二是将检查点（checkpoint）写入远程的持久化存储。



状态后端是一个“开箱即用”的组件，可以在不改变应用程序逻辑的情况下独立配置。Flink 中提供了两类不同的状态后端，一种是“哈希表状态后端”（HashMapStateBackend），另一种是“内嵌 RocksDB 状态后端”（EmbeddedRocksDBStateBackend）。如果没有特别配置，系统默认的状态后端是 HashMapStateBackend。 



（1）哈希表状态后端（HashMapStateBackend）

这种方式就是我们之前所说的，把状态存放在内存里。具体实现上，哈希表状态后端在内部会直接把状态当作对象（objects），保存在 Taskmanager 的 JVM 堆（heap）上。普通的状态，以及窗口中收集的数据和触发器（triggers），都会以键值对（key-value）的形式存储起来，所以底层是一个哈希表（HashMap），这种状态后端也因此得名。

对于检查点的保存，一般是放在持久化的分布式文件系统（file system）中，也可以通过配置“检查点存储”（CheckpointStorage）来另外指定。

HashMapStateBackend 是将本地状态全部放入内存的，这样可以获得最快的读写速度，使计算性能达到最佳；代价则是内存的占用。它适用于具有大状态、长窗口、大键值状态的作业，对所有高可用性设置也是有效的。



（2）内嵌 RocksDB 状态后端（EmbeddedRocksDBStateBackend）

RocksDB 是一种内嵌的 key-value 存储介质，可以把数据持久化到本地硬盘。配置EmbeddedRocksDBStateBackend 后，会将处理中的数据全部放入 RocksDB 数据库中，RocksDB默认存储在 TaskManager 的本地数据目录里。

与 HashMapStateBackend 直接在堆内存中存储对象不同，这种方式下状态主要是放在RocksDB 中的。数据被存储为序列化的字节数组（Byte Arrays），读写操作需要序列化/反序列化，因此状态的访问性能要差一些。另外，因为做了序列化，key 的比较也会按照字节进行，而不是直接调用.hashCode()和.equals()方法

对于检查点，同样会写入到远程的持久化文件系统中。

EmbeddedRocksDBStateBackend 始终执行的是异步快照，也就是不会因为保存检查点而阻塞数据的处理；而且它还提供了增量式保存检查点的机制，这在很多情况下可以大大提升保存效率。

由于它会把状态数据落盘，而且支持增量化的检查点，所以在状态非常大、窗口非常长、键/值状态很大的应用场景中是一个好选择，同样对所有高可用性设置有效。



> 如何选择正确的状态后端

HashMap 和 RocksDB 两种状态后端最大的区别，就在于本地状态存放在哪里：前者是内存，后者是 RocksDB。在实际应用中，选择那种状态后端，主要是需要根据业务需求在处理性能和应用的扩展性上做一个选择。

HashMapStateBackend 是内存计算，读写速度非常快；但是，状态的大小会受到集群可用内存的限制，如果应用的状态随着时间不停地增长，就会耗尽内存资源。

而 RocksDB 是硬盘存储，所以可以根据可用的磁盘空间进行扩展，而且是唯一支持增量检查点的状态后端，所以它非常适合于超级海量状态的存储。不过由于每个状态的读写都需要做序列化/反序列化，而且可能需要直接从磁盘读取数据，这就会导致性能的降低，平均读写性能要比 HashMapStateBackend 慢一个数量级。

我们可以发现，实际应用就是权衡利弊后的取舍。最理想的当然是处理速度快且内存不受限制可以处理海量状态，那就需要非常大的内存资源了，这会导致成本超出项目预算。比起花更多的钱，稍慢的处理速度或者稍小的处理规模，老板可能更容易接受一点



> 状态后端的配置

在不做配置的时候，应用程序使用的默认状态后端是由集群配置文件 flink-conf.yaml 中指定的，配置的键名称为 state.backend。

这个默认配置对集群上运行的所有作业都有效，我们可以通过更改配置值来改变默认的状态后端。另外，我们还可以在代码中为当前作业单独配置状态后端，这个配置会覆盖掉集群配置文件的默认值。

（1）在 flink-conf.yaml 中，可以使用 state.backend 来配置默认状态后端。

配置项的可能值为 hashmap，这样配置的就是 HashMapStateBackend；也可以是 rocksdb，这样配置的就是EmbeddedRocksDBStateBackend。另外，也可以是一个实现了状态后端工厂StateBackendFactory 的类的完全限定类名。

下面是一个配置 HashMapStateBackend 的例子：

```yaml
# 默认状态后端
state.backend: hashmap
# 存放检查点的文件路径
state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints
```

（2）为每个作业（Per-job）单独配置状态后端

每个作业独立的状态后端，可以在代码中，基于作业的执行环境直接设置。代码如下：

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new HashMapStateBackend());
```

如果想要设置 EmbeddedRocksDBStateBackend，可以用下面的配置方式

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new EmbeddedRocksDBStateBackend());
```

需要注意，如果想在 IDE 中使用 EmbeddedRocksDBStateBackend，需要为 Flink 项目添加依赖：

```xml
<dependency>
    <groupId>org.apache.flink</groupId> 
	<artifactId>flink-statebackend-rocksdb_${scala.binary.version}</artifactId>
 	<version>1.13.0</version>
</dependency>
```

而由于 Flink 发行版中默认就包含了 RocksDB，所以只要我们的代码中没有使用 RocksDB的相关内容，就不需要引入这个依赖。即使我们在 flink-conf.yaml 配置文件中设定了state.backend 为 rocksdb，也可以直接正常运行，并且使用 RocksDB 作为状态后端。





# 第 10 章 容错机制







# 第 11 章 Table API 和 SQL



# 第 12 章 Flink CEP


