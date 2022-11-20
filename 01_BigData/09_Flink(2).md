

[TOC]



# 第 7 章 处理函数(81-88)

## 7.1 基本处理函数（ProcessFunction）

>   在处理函数中，我们直面的就是数据流中最基本的元素：数据事件（event）、状态（state）以及时间（time）。这就相当于对流有了完全的控制权。处理函数比较抽象，没有具体的操作，所以对于一些常见的简单应用（比如求和、开窗口）会显得有些麻烦；不过正是因为它不限定具体做什么，所以理论上我们可以做任何事情，实现所有需求。所以可以说，==处理函数是我们进行 Flink 编程的“大招”，轻易不用，一旦放出来必然会扫平一切==。

  处理函数主要是定义数据流的转换操作，所以也可以把它归到转换算子中。我们知道在Flink 中几乎所有转换算子都提供了对应的函数类接口，处理函数也不例外；它所对应的函数类，就叫作 ProcessFunction。

### 7.1.1 处理函数的功能和使用

  是无论那种算子，如果我们想要访问事件的时间戳，或者当前的水位线信息，都是完全做不到的。在定义生成规则之后，水位线会源源不断地产生，像数据一样在任务间流动，可我们却不能像数据一样去处理它；跟时间相关的操作，目前我们只会用窗口来处理。==而在很多应用需求中，要求我们对时间有更精细的控制，需要能够获取水位线，甚至要“把控时间”、定义什么时候做什么事，这就不是基本的时间窗口能够实现的了==。

  于是必须祭出大招——处理函数（ProcessFunction）了。处理函数提供了一个“定时服务”（TimerService），我们可以通过它访问流中的事件（event）、时间戳（timestamp）、水位线（watermark），甚至可以注册“定时事件”。而且处理函数继承了 AbstractRichFunction 抽象类，所以拥有富函数类的所有特性，同样可以访问状态（state）和其他运行时信息。此外，处理函数还可以直接将数据输出到侧输出流（side output）中。所以，处理函数是最为灵活的处理方法，可以实现各种自定义的业务逻辑；同时也是整个 DataStream API 的底层基础。

  处理函数的使用与基本的转换操作类似，只需要直接基于 DataStream 调用.process()方法就可以了。方法需要传入一个 ProcessFunction 作为参数，用来定义处理逻辑。

```
stream.process(new MyProcessFunction())
```

  这里 ProcessFunction 不是接口，而是一个抽象类，继承了 AbstractRichFunction；MyProcessFunction 是它的一个具体实现。所以所有的处理函数，都是富函数（RichFunction），富函数可以调用的东西这里同样都可以调用。

**下面是一个具体的应用示例：**

```java
package com.fang.chapter07;


import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;

public class ProcessFunctionTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env
                .addSource(new ClinkSource())
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Event>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                                    @Override
                                    public long extractTimestamp(Event event, long l) {
                                        return event.timestamp;
                                    }
                                })
                )
                .process(new ProcessFunction<Event, String>() {
                    @Override
                    public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
                        if (value.user.equals("Mary")) {
                            out.collect(value.user);
                        } else if (value.user.equals("Bob")) {
                            out.collect(value.user);
                            out.collect(value.user);
                        }
                        System.out.println(ctx.timerService().currentWatermark());
                    }
                })
                .print();
        env.execute();
    }
}
```

![image-20221120172711849](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221120172711849.png)

### 7.1.2 ProcessFunction 解析

  在源码中我们可以看到，抽象类 ProcessFunction 继承了 AbstractRichFunction，有两个泛型类型参数：I 表示 Input，也就是输入的数据类型；O 表示 Output，也就是处理完成之后输出的数据类型。

  内部单独定义了两个方法：一个是必须要实现的抽象方法.processElement()；另一个是非抽象方法.onTimer()。

```
public abstract class ProcessFunction<I, O> extends AbstractRichFunction {
 ...
	public abstract void processElement(I value, Context ctx, Collector<O> out) 
throws Exception;

	public void onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) 
	throws Exception {}
	...
}
```

**1.抽象方法.processElement()**

  用于“处理元素”，定义了处理的核心逻辑。这个方法对于流中的每个元素都会调用一次，参数包括三个：输入数据值 value，上下文 ctx，以及“收集器”（Collector）out。方法没有返回值，处理之后的输出数据是通过收集器 out 来定义的。

⚫ value：当前流中的输入元素，也就是正在处理的数据，类型与流中数据类型一致。

⚫ ctx：类型是 ProcessFunction 中定义的内部抽象类 Context，表示当前运行的上下文，可以获取到当前的时间戳，并提供了用于查询时间和注册定时器的“定时服务”(TimerService)，以及可以将数据发送到“侧输出流”（side output）的方法.output()。

Context 抽象类定义如下：

```
public abstract class Context {
 public abstract Long timestamp();
 public abstract TimerService timerService();
 public abstract <X> void output(OutputTag<X> outputTag, X value);
}
```

⚫ out：“收集器”（类型为 Collector），用于返回输出数据。使用方式与 flatMap

  算子中的收集器完全一样，直接调用 out.collect()方法就可以向下游发出一个数据。这个方法可以多次调用，也可以不调用。

  通过几个参数的分析不难发现，ProcessFunction 可以轻松实现 flatMap 这样的基本转换功能（当然 map、filter 更不在话下）；而通过富函数提供的获取上下文方法.getRuntimeContext()，也可以自定义状态（state）进行处理，这也就能实现聚合操作的功能了。关于自定义状态的具体实现，我们会在后续“状态管理”一章中详细介绍。

**2.非抽象方法.onTimer()**

  用于定义定时触发的操作，这是一个非常强大、也非常有趣的功能。这个方法只有在注册好的定时器触发的时候才会调用，而定时器是通过“定时服务”TimerService 来注册的。打个比方，注册定时器（timer）就是设了一个闹钟，到了设定时间就会响；而.onTimer()中定义的，就是闹钟响的时候要做的事。所以它本质上是一个基于时间的“回调”（callback）方法，通过时间的进展来触发；在事件时间语义下就是由水位线（watermark）来触发了。

```java
package com.fang.chapter07;

import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;

import java.sql.Timestamp;

public class ProcessingTimeTimerTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 处理时间语义，不需要分配时间戳和watermark
        SingleOutputStreamOperator<Event> stream = env.addSource(new ClinkSource());

        // 要用定时器，必须基于KeyedStream
        stream.keyBy(data -> true)
                .process(new KeyedProcessFunction<Boolean, Event, String>() {
                    @Override
                    public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
                        Long currTs = ctx.timerService().currentProcessingTime();
                        out.collect("数据到达，到达时间：" + new Timestamp(currTs));
                        // 注册一个10秒后的定时器
                        ctx.timerService().registerProcessingTimeTimer(currTs + 10 * 1000L);
                    }

                    @Override
                    public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
                        out.collect("定时器触发，触发时间：" + new Timestamp(timestamp));
                    }
                })
                .print();
        env.execute();
    }
}
```

![image-20221120173520224](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221120173520224.png)





```java
package com.fang.chapter07;

import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.util.Collector;

public class EventTimeTimerTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new CustomSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        }));

        // 基于KeyedStream定义事件时间定时器
        stream.keyBy(data -> true)
                .process(new KeyedProcessFunction<Boolean, Event, String>() {
                    @Override
                    public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
                        out.collect("数据到达，时间戳为：" + ctx.timestamp());
                        out.collect("数据到达，水位线为：" + ctx.timerService().currentWatermark() + "\n -------分割线-------");
                        // 注册一个10秒后的定时器
                        ctx.timerService().registerEventTimeTimer(ctx.timestamp() + 10 * 1000L);
                    }

                    @Override
                    public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
                        out.collect("定时器触发，触发时间：" + timestamp);
                    }
                })
                .print();

        env.execute();
    }

    // 自定义测试数据源
    public static class CustomSource implements SourceFunction<Event> {
        @Override
        public void run(SourceContext<Event> ctx) throws Exception {
            // 直接发出测试数据
            ctx.collect(new Event("Mary", "./home", 1000L));
            // 为了更加明显，中间停顿5秒钟
            Thread.sleep(5000L);

            // 发出10秒后的数据
            ctx.collect(new Event("Mary", "./home", 11000L));
            Thread.sleep(5000L);

            // 发出10秒+1ms后的数据
            ctx.collect(new Event("Alice", "./cart", 11001L));
            Thread.sleep(5000L);
        }

        @Override
        public void cancel() { }
    }
}
```

![image-20221120175057652](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221120175057652.png)

### 7.1.3 处理函数的分类

**Flink 提供了 8 个不同的处理函数：**

**（1）ProcessFunction**

  最基本的处理函数，基于 DataStream 直接调用.process()时作为参数传入。

**（2）KeyedProcessFunction**

  对流按键分区后的处理函数，基于 KeyedStream 调用.process()时作为参数传入。==要想使用定时器，比如基于 KeyedStream。==

**（3）ProcessWindowFunction**

  开窗之后的处理函数，也是全窗口函数的代表。基于 WindowedStream 调用.process()时作为参数传入。

**（4）ProcessAllWindowFunction**

  同样是开窗之后的处理函数，基于 AllWindowedStream 调用.process()时作为参数传入。

**（5）CoProcessFunction**

  合并（connect）两条流之后的处理函数，基于 ConnectedStreams 调用.process()时作为参数传入。关于流的连接合并操作，我们会在后续章节详细介绍。

**（6）ProcessJoinFunction**

  间隔连接（interval join）两条流之后的处理函数，基于 IntervalJoined 调用.process()时作为参数传入。

**（7）BroadcastProcessFunction**

  广播连接流处理函数，基于 BroadcastConnectedStream 调用.process()时作为参数传入。这里的“广播连接流”BroadcastConnectedStream，是一个未 keyBy 的普通 DataStream 与一个广播流（BroadcastStream）做连接（conncet）之后的产物。

**（8）KeyedBroadcastProcessFunction**

  按键分区的广播连接流处理函数，同样是基于 BroadcastConnectedStream 调用.process()时作为参数传入。与 BroadcastProcessFunction 不同的是，这时的广播连接流，是一个 KeyedStream与广播流（BroadcastStream）做连接之后的产物。接下来，我们就对 KeyedProcessFunction 和 ProcessWindowFunction 的具体用法展开详细说明。

## 7.2 按键分区处理函数（KeyedProcessFunction）

### 7.2.1 定时器（Timer）和定时服务（TimerService）





### 7.2.2 KeyedProcessFunction 的使用





## 7.3 窗口处理函数

### 7.3.1 窗口处理函数的使用



### 7.3.2 ProcessWindowFunction 解析



## 7.4 应用案例——Top N

### 7.4.1 使用 ProcessAllWindowFunction



### 7.4.2 使用 KeyedProcessFunction



## 7.5 侧输出流（Side Output）

# 第 8 章 多流转换(89-97)

## 8.1 分流

### 8.1.1 简单实现

### 8.1.2 使用侧输出流

## 8.2 基本合流操作

### 8.2.1 联合（Union）

### 8.2.2 连接（Connect）

## 8.3 基于时间的合流——双流联结（Join）

### 8.3.1 窗口联结（Window Join）

### 8.3.2 间隔联结（Interval Join）

### 8.3.3 窗口同组联结（Window CoGroup）

# 第 9 章 状态编程(98-115)



# 第 10 章 容错机制(116-126)



# 第 11 章 Table API 和 SQL(127-156)



# 第 12 章 Flink CEP(157-171)

