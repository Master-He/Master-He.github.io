「尚硅谷大数据技术之flink(java) 含书签.pdf」https://www.aliyundrive.com/s/rLf7szByhuw 



# DataStream API

## 执行环境（Execution Environment）

StreamExecutionEnvironment 类的方法

```
getExecutionEnvironment
createLocalEnvironment
createRemoteEnvironment
```



# Flink中的时间和窗口

## 怎么理解水位线watermark
watermark是当前事件的时钟，这个时钟还有一点延迟
watermark可以理解为把原本的窗口标准稍微放宽了一点。（比如原本5s，设置延迟时间=2s，那么实际等到7s的数据到达时，才认为是[0,5）的桶需要关闭了）

## 如何生成水位线watermark
每来一个数据记录最大的时间戳，然后周期性的生成watermark，watermark的值为（当前数据流中最大的时间戳 - 延迟时间）

## 如何确定延迟时间
先假设延迟时间是0秒， 然后分析数据的乱序程度，数据5来了，同时周期性地生成了水位线w(5)
4 —> 3  —> 5 —> 1  —>2   
此时5之后来的最小的数是3， 5-3=2就是乱序程度
这个乱序程度就可以作为延迟时间

## 水位线的特性

- 水位线是插入到数据流中的一个标记，可以认为是一个特殊的数据
- 水位线主要的内容是一个时间戳，用来表示当前事件时间的进展， 它是基于数据的时间戳生成的
- 水位线的时间戳必须单调递增，以确保任务的事件时间时钟一直向前推进
- 水位线可以通过设置延迟，来保证正确处理乱序数据
- **一个水位线 Watermark(t)，表示在当前流中事件时间已经达到了时间戳 t, 这代表 t 之前的所有数据都到齐了，之后流中不会出现时间戳 t’ ≤ t 的数据**

## 水位线代码

设置生成水位线的周期
env.getConfig.setAutoWatermarkInterval(100);

## 窗口

