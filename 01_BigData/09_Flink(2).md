

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
                }).print();
        env.execute();
    }
}
```

![image-20221120172711849](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221120172711849.png)

### 7.1.2 ProcessFunction 解析

  在源码中我们可以看到，抽象类 ProcessFunction 继承了 AbstractRichFunction，有两个泛型类型参数：I 表示 Input，也就是输入的数据类型；O 表示 Output，也就是处理完成之后输出的数据类型。

  内部单独定义了两个方法：一个是必须要实现的抽象方法.processElement()；另一个是非抽象法.onTimer()。

```
public abstract class ProcessFunction<I, O> extends AbstractRichFunction {

	public abstract void processElement(I value, Context ctx, Collector<O> out) 
throws Exception;

	public void onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) 
	throws Exception {}

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

  KeyedProcessFunction 的一个特色，就是可以灵活地使用定时器。定时器（timers）是处理函数中进行时间相关操作的主要机制。在.onTimer()方法中可以实现定时处理的逻辑，而它能触发的前提，就是之前曾经注册过定时器、并且现在已经到了触发时间。注册定时器的功能，是通过上下文中提供的“定时服务”（TimerService）来实现的。定时服务与当前运行的环境有关。前面已经介绍过，ProcessFunction 的上下文（Context）

中提供了.timerService()方法，可以直接返回一个 TimerService 对象：

```
public abstract TimerService timerService();
```

TimerService 是 Flink 关于时间和定时器的基础服务接口，包含以下六个方法：

```java
// 获取当前的处理时间
long currentProcessingTime();

// 获取当前的水位线（事件时间）
long currentWatermark();

// 注册处理时间定时器，当处理时间超过 time 时触发
void registerProcessingTimeTimer(long time);

// 注册事件时间定时器，当水位线超过 time 时触发
void registerEventTimeTimer(long time);

// 删除触发时间为 time 的处理时间定时器
void deleteProcessingTimeTimer(long time);

// 删除触发时间为 time 的处理时间定时器
void deleteEventTimeTimer(long time);
```

### 7.2.2 KeyedProcessFunction 的使用

  KeyedProcessFunction 可以说是处理函数中的“嫡系部队”，可以认为是 ProcessFunction 的一个扩展。我们只要基于 keyBy 之后的 KeyedStream，直接调用.process()方法，这时需要传入的参数就是 KeyedProcessFunction 的实现类。

```
stream.keyBy( t -> t.f0 )
		.process(new MyKeyedProcessFunction())
```

类似地，KeyedProcessFunction 也是继承自 AbstractRichFunction 的一个抽象类，源码中定义如下：

```java
public abstract class KeyedProcessFunction<K, I, O> extends AbstractRichFunction 
{
...
  public abstract void processElement(I value, Context ctx, Collector<O> out) throws Exception;
  public void onTimer(long timestamp, OnTimerContext ctx, Collector<O> out) throws Exception {
  }
  public abstract class Context {...}
...
}
```



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

## 7.3 窗口处理函数

### 7.3.1 窗口处理函数的使用

  进行窗口计算，我们可以直接调用现成的简单聚合方法（sum/max/min）,也可以通过调用.reduce()或.aggregate()来自定义一般的增量聚合函数（ReduceFunction/AggregateFucntion）；而对于更加复杂、需要窗口信息和额外状态的一些场景，我们还可以直接使用全窗口函数、把数据全部收集保存在窗口内，等到触发窗口计算时再统一处理。窗口处理函数就是一种典型的全窗口函数。

  窗 口 处 理 函 数 ProcessWindowFunction 的 使 用 与 其 他 窗 口 函 数 类 似 ， 也 是 基 于WindowedStream 直接调用方法就可以，只不过这时调用的是.process()。

```
stream.keyBy( t -> t.f0 )
 .window( TumblingEventTimeWindows.of(Time.seconds(10)) )
 .process(new MyProcessWindowFunction())
```

### 7.3.2 ProcessWindowFunction 解析

  ProcessWindowFunction 既是处理函数又是全窗口函数。从名字上也可以推测出，它的本质似乎更倾向于“窗口函数”一些。事实上它的用法也确实跟其他处理函数有很大不同。我们可以从源码中的定义看到这一点：

```
public abstract class ProcessWindowFunction<IN, OUT, KEY, W extends Window>
 extends AbstractRichFunction {
		public abstract void process(
 			KEY key, Context context, Iterable<IN> elements, Collector<OUT> out) throws 
		Exception;
		public void clear(Context context) throws Exception {}
		public abstract class Context implements java.io.Serializable {...}
}
```

**ProcessWindowFunction 依然是一个继承了 AbstractRichFunction 的抽象类，它有四个类型**

参数：

​		⚫ IN：input，数据流中窗口任务的输入数据类型。

​		⚫ OUT：output，窗口任务进行计算之后的输出数据类型。

​		⚫ KEY：数据中键 key 的类型。

​		⚫ W：窗口的类型，是 Window 的子类型。一般情况下我们定义时间窗口，W就是 TimeWindow。

## 7.4 应用案例——Top N

### 7.4.1 使用 ProcessAllWindowFunction

  一种最简单的想法是，我们干脆不区分 url 链接，而是将所有访问数据都收集起来，统一进行统计计算。所以可以不做 keyBy，直接基于 DataStream 开窗，然后使用全窗口函数ProcessAllWindowFunction 来进行处理。

  在窗口中可以用一个 HashMap 来保存每个 url 的访问次数，只要遍历窗口中的所有数据，自然就能得到所有 url 的热门度。最后把 HashMap 转成一个列表 ArrayList，然后进行排序、取出前两名输出就可以了。

```java
package com.fang.chapter07;


import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.windowing.ProcessAllWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.HashMap;

public class ProcessAllWindowTopN {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> eventStream = env.addSource(new ClinkSource())
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Event>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                                    @Override
                                    public long extractTimestamp(Event element, long recordTimestamp) {
                                        return element.timestamp;
                                    }
                                })
                );

        // 只需要url就可以统计数量，所以转换成String直接开窗统计
        SingleOutputStreamOperator<String> result = eventStream
                .map(new MapFunction<Event, String>() {
                    @Override
                    public String map(Event value) throws Exception {
                        return value.url;
                    }
                })
                .windowAll(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))    // 开滑动窗口
                .process(new ProcessAllWindowFunction<String, String, TimeWindow>() {
                    @Override
                    public void process(Context context, Iterable<String> elements, Collector<String> out) throws Exception {
                        HashMap<String, Long> urlCountMap = new HashMap<>();
                        // 遍历窗口中数据，将浏览量保存到一个 HashMap 中
                        for (String url : elements) {
                            if (urlCountMap.containsKey(url)) {
                                long count = urlCountMap.get(url);
                                urlCountMap.put(url, count + 1L);
                            } else {
                                urlCountMap.put(url, 1L);
                            }
                        }
                        ArrayList<Tuple2<String, Long>> mapList = new ArrayList<Tuple2<String, Long>>();
                        // 将浏览量数据放入ArrayList，进行排序
                        for (String key : urlCountMap.keySet()) {
                            mapList.add(Tuple2.of(key, urlCountMap.get(key)));
                        }
                        mapList.sort(new Comparator<Tuple2<String, Long>>() {
                            @Override
                            public int compare(Tuple2<String, Long> o1, Tuple2<String, Long> o2) {
                                return o2.f1.intValue() - o1.f1.intValue();
                            }
                        });
                        // 取排序后的前两名，构建输出结果
                        StringBuilder result = new StringBuilder();
                        result.append("========================================\n");
                        for (int i = 0; i < 2; i++) {
                            Tuple2<String, Long> temp = mapList.get(i);
                            String info = "浏览量No." + (i + 1) +
                                    " url：" + temp.f0 +
                                    " 浏览量：" + temp.f1 +
                                    " 窗口结束时间：" + new Timestamp(context.window().getEnd()) + "\n";

                            result.append(info);
                        }
                        result.append("========================================\n");
                        out.collect(result.toString());
                    }
                });

        result.print();

        env.execute();
    }
}


```

![image-20221121143520945](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221121143520945.png)

### 7.4.2 使用 KeyedProcessFunction

![image-20221121143616931](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221121143616931.png)

```java
package com.fang.chapter07;



import com.fang.chapter05.ClinkSource;
import com.fang.chapter05.Event;
import com.fang.chapter06.UrlViewCount;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.Comparator;

public class KeyedProcessTopN {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 从自定义数据源读取数据
        SingleOutputStreamOperator<Event> eventStream = env.addSource(new ClinkSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        }));

        // 需要按照url分组，求出每个url的访问量
        SingleOutputStreamOperator<UrlViewCount> urlCountStream =
                eventStream.keyBy(data -> data.url)
                        .window(SlidingEventTimeWindows.of(Time.seconds(10), Time.seconds(5)))
                        .aggregate(new UrlViewCountAgg(),
                                new UrlViewCountResult());


        // 对结果中同一个窗口的统计数据，进行排序处理
        SingleOutputStreamOperator<String> result = urlCountStream.keyBy(data -> data.windowEnd)
                .process(new TopN(2));

        result.print("result");

        env.execute();
    }

    // 自定义增量聚合
    public static class UrlViewCountAgg implements AggregateFunction<Event, Long, Long> {
        @Override
        public Long createAccumulator() {
            return 0L;
        }

        @Override
        public Long add(Event value, Long accumulator) {
            return accumulator + 1;
        }

        @Override
        public Long getResult(Long accumulator) {
            return accumulator;
        }

        @Override
        public Long merge(Long a, Long b) {
            return null;
        }
    }

    // 自定义全窗口函数，只需要包装窗口信息
    public static class UrlViewCountResult extends ProcessWindowFunction<Long, UrlViewCount, String, TimeWindow> {

        @Override
        public void process(String url, Context context, Iterable<Long> elements, Collector<UrlViewCount> out) throws Exception {
            // 结合窗口信息，包装输出内容
            Long start = context.window().getStart();
            Long end = context.window().getEnd();
            out.collect(new UrlViewCount(url, elements.iterator().next(), start, end));
        }
    }

    // 自定义处理函数，排序取top n
    public static class TopN extends KeyedProcessFunction<Long, UrlViewCount, String>{
        // 将n作为属性
        private Integer n;
        // 定义一个列表状态
        private ListState<UrlViewCount> urlViewCountListState;

        public TopN(Integer n) {
            this.n = n;
        }

        @Override
        public void open(Configuration parameters) throws Exception {
            // 从环境中获取列表状态句柄
            urlViewCountListState = getRuntimeContext().getListState(
                    new ListStateDescriptor<UrlViewCount>("url-view-count-list",
                            Types.POJO(UrlViewCount.class)));
        }

        @Override
        public void processElement(UrlViewCount value, Context ctx, Collector<String> out) throws Exception {
            // 将count数据添加到列表状态中，保存起来
            urlViewCountListState.add(value);
            // 注册 window end + 1ms后的定时器，等待所有数据到齐开始排序
            ctx.timerService().registerEventTimeTimer(ctx.getCurrentKey() + 1);
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            // 将数据从列表状态变量中取出，放入ArrayList，方便排序
            ArrayList<UrlViewCount> urlViewCountArrayList = new ArrayList<>();
            for (UrlViewCount urlViewCount : urlViewCountListState.get()) {
                urlViewCountArrayList.add(urlViewCount);
            }
            // 清空状态，释放资源
            urlViewCountListState.clear();

            // 排序
            urlViewCountArrayList.sort(new Comparator<UrlViewCount>() {
                @Override
                public int compare(UrlViewCount o1, UrlViewCount o2) {
                    return o2.count.intValue() - o1.count.intValue();
                }
            });

            // 取前两名，构建输出结果
            StringBuilder result = new StringBuilder();
            result.append("========================================\n");
            result.append("窗口结束时间：" + new Timestamp(timestamp - 1) + "\n");
            for (int i = 0; i < this.n; i++) {
                UrlViewCount UrlViewCount = urlViewCountArrayList.get(i);
                String info = "No." + (i + 1) + " "
                        + "url：" + UrlViewCount.url + " "
                        + "浏览量：" + UrlViewCount.count + "\n";
                result.append(info);
            }
            result.append("========================================\n");
            out.collect(result.toString());
        }
    }
}


```

![image-20221121143339525](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221121143339525.png)

## 7.5 侧输出流（Side Output）

  处理函数还有另外一个特有功能，就是将自定义的数据放入“侧输出流”（side output）输出。这个概念我们并不陌生，之前在讲到窗口处理迟到数据时，最后一招就是输出到侧输出流。而这种处理方式的本质，其实就是处理函数的侧输出流功能。

  我们之前讲到的绝大多数转换算子，输出的都是单一流，流里的数据类型只能有一种。而侧输出流可以认为是“主流”上分叉出的“支流”，所以可以由一条流产生出多条流，而且这些流中的数据类型还可以不一样。利用这个功能可以很容易地实现“分流”操作。

  具体应用时，只要在处理函数的.processElement()或者.onTimer()方法中，调用上下文的.output()方法就可以了。

```java
DataStream<Integer> stream = env.addSource(...);
SingleOutputStreamOperator<Long> longStream = stream.process(new ProcessFunction<Integer, Long>() {

 @Override
 public void processElement( Integer value, Context ctx, Collector<Integer> out) throws Exception {
    // 转换成 Long，输出到主流中
    out.collect(Long.valueOf(value));
    // 转换成 String，输出到侧输出流中
    ctx.output(outputTag, "side-output: " + String.valueOf(value));
  }
});
```

这里 output()方法需要传入两个参数，第一个是一个“输出标签”OutputTag，用来标识侧

输出流，一般会在外部统一声明；第二个就是要输出的数据。

我们可以在外部先将 OutputTag 声明出来：

```
OutputTag<String> outputTag = new OutputTag<String>("side-output") {};
```

如果想要获取这个侧输出流，可以基于处理之后的 DataStream 直接调用.getSideOutput()

方法，传入对应的 OutputTag，这个方式与窗口 API 中获取侧输出流是完全一样的。

```
DataStream<String> stringStream = longStream.getSideOutput(outputTag);
```

# 第 8 章 多流转换(89-97)

## 8.1 分流

  ==所谓“分流”，就是将一条数据流拆分成完全独立的两条、甚至多条流==。也就是基于一个DataStream，得到完全平等的多个子 DataStream。一般来说，我们会定义一些筛选条件，==将符合条件的数据拣选出来放到对应的流里==。

![image-20221127142725094](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221127142725094.png)

### 8.1.1 过滤器filter

  其实根据条件筛选数据的需求，本身非常容易实现：只要针对同一条流多次独立调用.filter()方法进行筛选，就可以得到拆分之后的流了。

```java
import com.atguigu.chapter05.ClickSource;
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

public class SplitStreamByFilter {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        SingleOutputStreamOperator<Event> stream = env
                .addSource(new ClickSource());
        
        // 筛选Mary的浏览行为放入MaryStream流中
        DataStream<Event> MaryStream = stream.filter(new FilterFunction<Event>() {
            @Override
            public boolean filter(Event value) throws Exception {
                return value.user.equals("Mary");
            }
        });
        // 筛选Bob的购买行为放入BobStream流中
        DataStream<Event> BobStream = stream.filter(new FilterFunction<Event>() {
            @Override
            public boolean filter(Event value) throws Exception {
                return value.user.equals("Bob");
            }
        });
        // 筛选其他人的浏览行为放入elseStream流中
        DataStream<Event> elseStream = stream.filter(new FilterFunction<Event>() {
            @Override
            public boolean filter(Event value) throws Exception {
                return !value.user.equals("Mary") && !value.user.equals("Bob") ;
            }
        });

        MaryStream.print("Mary pv");
        BobStream.print("Bob pv");
        elseStream.print("else pv");

        env.execute();

    }
}

```

![image-20221127143748458](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221127143748458.png)

### 8.1.2 使用侧输出流（SideOutput）

​		在 Flink 1.13 版本中，已经弃用了.split()方法，取而代之的是直接用处理函数（process function）的侧输出流（side output）。
  我们知道，处理函数本身可以认为是一个转换算子，它的输出类型是单一的，处理之后得到的仍然是一个 DataStream； ==而侧输出流则不受限制，可以任意自定义输出数据，它们就像从“主流”上分叉出的“支流”。尽管看起来主流和支流有所区别，不过实际上它们都是某种类型的 DataStream，所以本质上还是平等的。== 利用侧输出流就可以很方便地实现分流操作，而且得到的多条 DataStream 类型可以不同，这就给我们的应用带来了极大的便利

```java
import com.atguigu.chapter05.ClickSource;
import com.atguigu.chapter05.Event;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.util.OutputTag;


public class SplitStreamByOutputTag {
    // 定义输出标签，侧输出流的数据类型为三元组(user, url, timestamp)
    private static OutputTag<Tuple3<String, String, Long>> MaryTag = new OutputTag<Tuple3<String, String, Long>>("Mary-pv"){};
    
    private static OutputTag<Tuple3<String, String, Long>> BobTag = new OutputTag<Tuple3<String, String, Long>>("Bob-pv"){};
    
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        SingleOutputStreamOperator<Event> stream = env
                .addSource(new ClickSource());

        SingleOutputStreamOperator<Event> processedStream = stream.process(new ProcessFunction<Event, Event>() {
            
            @Override
            public void processElement(Event value, Context ctx, Collector<Event> out) throws Exception {
                if (value.user.equals("Mary")){
                    ctx.output(MaryTag, new Tuple3<>(value.user, value.url, value.timestamp));
                    
                } else if (value.user.equals("Bob")){
                    ctx.output(BobTag, new Tuple3<>(value.user, value.url, value.timestamp));
                    
                } else {
                    out.collect(value);
                }
            }
        });

        processedStream.getSideOutput(MaryTag).print("Mary pv");
        processedStream.getSideOutput(BobTag).print("Bob pv");
        processedStream.print("else");

        env.execute();
    }
}
```

![image-20221127144037993](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221127144037993.png)

## 8.2 基本合流操作

### 8.2.1 联合（Union）

![image-20221130093816683](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221130093816683.png)

  在代码中，我们只要基于 DataStream 直接调用.union()方法，传入其他 DataStream 作为参数，就可以实现流的联合了；得到的依然是一个 DataStream：

```
stream1.union(stream2, stream3, ...)
```

==注意：union()的参数可以是多个 DataStream，所以联合操作可以实现多条流的合并。==

**注意:对于合流之后的水位线，也是要以最小的那个为准**

​		**数据类型不能改变**

```java
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.util.Collector;

import java.time.Duration;

public class UnionTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);



        SingleOutputStreamOperator<Event> stream1 = env.socketTextStream("hadoop102", 7777)
                .map(data -> {
                    String[] field = data.split(",");
                    return new Event(field[0].trim(), field[1].trim(), Long.valueOf(field[2].trim()));
                })
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(2))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );
        stream1.print("stream1");

        SingleOutputStreamOperator<Event> stream2 = env.socketTextStream("hadoop103", 7777)
                .map(data -> {
                    String[] field = data.split(",");
                    return new Event(field[0].trim(), field[1].trim(), Long.valueOf(field[2].trim()));
                })
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ofSeconds(5))
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );

        stream2.print("stream2");

        // 合并两条流
        stream1.union(stream2)
                .process(new ProcessFunction<Event, String>() {
                    @Override
                    public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
                        out.collect("水位线：" + ctx.timerService().currentWatermark());
                    }
                })
                .print();


        env.execute();
    }
}
```

![image-20221223135455973](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221223135455973.png)

### 8.2.2 连接（Connect）

![image-20221130093835415](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221130093835415.png)

**1.连接流（ConnectedStreams）**

```java
import org.apache.flink.streaming.api.datastream.ConnectedStreams;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.CoMapFunction;

public class ConnectTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStream<Integer> stream1 = env.fromElements(1,2,3);
        DataStream<Long> stream2 = env.fromElements(1L,2L,3L);

        ConnectedStreams<Integer, Long> connectedStreams = stream1.connect(stream2);
        SingleOutputStreamOperator<String> result = connectedStreams.map(new CoMapFunction<Integer, Long, String>() {
            @Override
            public String map1(Integer value) {
                return "Integer: " + value;
            }

            @Override
            public String map2(Long value) {
                return "Long: " + value;
            }
        });

        result.print();

        env.execute();
    }
}
```

![image-20221223140620202](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221223140620202.png)

**2.CoProcessFunction**

  **对于连接流 ConnectedStreams 的处理操作，需要分别定义对两条流的处理转换，因此接口中就会有两个相同的方法需要实现，用数字“1”“2”区分，在两条流中的数据到来时分别调用。我们把这种接口叫作“协同处理函数”（co-process function）。**

抽象类 CoProcessFunction 在源码中定义如下：

```java
public abstract class CoProcessFunction<IN1, IN2, OUT> extends 
AbstractRichFunction {

	public abstract void processElement1(IN1 value, Context ctx, Collector<OUT> out) throws 	Exception;
	
	public abstract void processElement2(IN2 value, Context ctx, Collector<OUT> out) throws 	Exception;
	
	public void onTimer(long timestamp, OnTimerContext ctx, Collector<OUT> out) throws 			Exception {}
	
	public abstract class Context {...}

}
```

  我们可以看到，很明显 CoProcessFunction 也是“处理函数”家族中的一员，用法非常相似。它需要实现的就是 processElement1()、processElement2()两个方法，在每个数据到来时，会根据来源的流调用其中的一个方法进行处理。CoProcessFunction 同样可以通过上下文 ctx 来访问 timestamp、水位线，并通过 TimerService 注册定时器；另外也提供了.onTimer()方法，用于定义定时触发的处理操作。

  下面是 CoProcessFunction 的一个具体示例：我们可以实现一个实时对账的需求，也就是app 的支付操作和第三方的支付操作的一个双流 Join。App 的支付事件和第三方的支付事件将会互相等待 5 秒钟，如果等不来对应的支付事件，那么就输出报警信息。程序如下：

```java
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.api.java.tuple.Tuple4;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.CoProcessFunction;
import org.apache.flink.util.Collector;

// 实时对账
public class BillCheckExample {

    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 来自app的支付日志
        SingleOutputStreamOperator<Tuple3<String, String, Long>> appStream = env.fromElements(
                Tuple3.of("order-1", "app", 1000L),
                Tuple3.of("order-2", "app", 2000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple3<String, String, Long>>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple3<String, String, Long> element, long recordTimestamp) {
                        return element.f2;
                    }
                })
        );

        // 来自第三方支付平台的支付日志
        SingleOutputStreamOperator<Tuple4<String, String, String, Long>> thirdpartStream = env.fromElements(
                Tuple4.of("order-1", "third-party", "success", 3000L),
                Tuple4.of("order-3", "third-party", "success", 4000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple4<String, String, String, Long>>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple4<String, String, String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple4<String, String, String, Long> element, long recordTimestamp) {
                        return element.f3;
                    }
                })
        );

        // 检测同一支付单在两条流中是否匹配，不匹配就报警
        appStream.connect(thirdpartStream)
                .keyBy(data -> data.f0, data -> data.f0)
                .process(new OrderMatchResult())
                .print();

        env.execute();
    }

    // 自定义实现CoProcessFunction
    public static class OrderMatchResult extends CoProcessFunction<Tuple3<String, String, Long>, Tuple4<String, String, String, Long>, String>{
        // 定义状态变量，用来保存已经到达的事件
        private ValueState<Tuple3<String, String, Long>> appEventState;
        private ValueState<Tuple4<String, String, String, Long>> thirdPartyEventState;

        @Override
        public void open(Configuration parameters) throws Exception {
            appEventState = getRuntimeContext().getState(
                    new ValueStateDescriptor<Tuple3<String, String, Long>>("app-event", Types.TUPLE(Types.STRING, Types.STRING, Types.LONG))
            );

            thirdPartyEventState = getRuntimeContext().getState(
                    new ValueStateDescriptor<Tuple4<String, String, String, Long>>("thirdparty-event", Types.TUPLE(Types.STRING, Types.STRING, Types.STRING,Types.LONG))
            );
        }

        @Override
        public void processElement1(Tuple3<String, String, Long> value, Context ctx, Collector<String> out) throws Exception {
            // 看另一条流中事件是否来过
            if (thirdPartyEventState.value() != null){
                out.collect("对账成功：" + value + "  " + thirdPartyEventState.value());
                // 清空状态
                thirdPartyEventState.clear();
            } else {
                // 更新状态
                appEventState.update(value);
                // 注册一个5秒后的定时器，开始等待另一条流的事件
                ctx.timerService().registerEventTimeTimer(value.f2 + 5000L);
            }
        }

        @Override
        public void processElement2(Tuple4<String, String, String, Long> value, Context ctx, Collector<String> out) throws Exception {
            if (appEventState.value() != null){
                out.collect("对账成功：" + appEventState.value() + "  " + value);
                // 清空状态
                appEventState.clear();
            } else {
                // 更新状态
                thirdPartyEventState.update(value);
                // 注册一个5秒后的定时器，开始等待另一条流的事件
                ctx.timerService().registerEventTimeTimer(value.f3 + 5000L);
            }
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            // 定时器触发，判断状态，如果某个状态不为空，说明另一条流中事件没来
            if (appEventState.value() != null) {
                out.collect("对账失败：" + appEventState.value() + "  " + "第三方支付平台信息未到");
            }
            if (thirdPartyEventState.value() != null) {
                out.collect("对账失败：" + thirdPartyEventState.value() + "  " + "app信息未到");
            }
            appEventState.clear();
            thirdPartyEventState.clear();
        }
    }}
```

![image-20221223144920814](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221223144920814.png)

**3.广播连接流（BroadcastConnectedStream）**

  关于两条流的连接，还有一种比较特殊的用法：DataStream 调用.connect()方法时，传入的参数也可以不是一个 DataStream，而是一个“广播流”（BroadcastStream），这时合并两条流得到的就变成了一个“广播连接流”（BroadcastConnectedStream）。

## 8.3 基于时间的合流——双流联结（Join）

### 8.3.1 窗口联结（Window Join）

  Flink 为这种场景专门提供了一个窗口联结（window join）算子，可以定义时间窗口，并将两条流中共享一个公共键（key）的数据放在窗口中进行配对处理。

![image-20221130093911782](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221130093911782.png)



**1.窗口联结的调用**

```
stream1.join(stream2)
 	.where(<KeySelector>)
 	.equalTo(<KeySelector>)
 	.window(<WindowAssigner>)
 	.apply(<JoinFunction>)
```

**2. 窗口联结的处理流程**

```java
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.JoinFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

// 基于窗口的join
public class WindowJoinTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStream<Tuple2<String, Long>> stream1 = env
                .fromElements(
                        Tuple2.of("a", 1000L),
                        Tuple2.of("b", 1000L),
                        Tuple2.of("a", 2000L),
                        Tuple2.of("b", 2000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple2<String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(
                                        new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                                            @Override
                                            public long extractTimestamp(Tuple2<String, Long> stringLongTuple2, long l) {
                                                return stringLongTuple2.f1;
                                            }
                                        }
                                )
                );

        DataStream<Tuple2<String, Long>> stream2 = env
                .fromElements(
                        Tuple2.of("a", 3000L),
                        Tuple2.of("b", 3000L),
                        Tuple2.of("a", 4000L),
                        Tuple2.of("b", 4000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple2<String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(
                                        new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                                            @Override
                                            public long extractTimestamp(Tuple2<String, Long> stringLongTuple2, long l) {
                                                return stringLongTuple2.f1;
                                            }
                                        }
                                )
                );

        stream1
                .join(stream2)
                .where(r -> r.f0)
                .equalTo(r -> r.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .apply(new JoinFunction<Tuple2<String, Long>, Tuple2<String, Long>, String>() {
                    @Override
                    public String join(Tuple2<String, Long> left, Tuple2<String, Long> right) throws Exception {
                        return left + "=>" + right;
                    }
                })
                .print();

        env.execute();
    }
}

```

![image-20221223164452487](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221223164452487.png)

### 8.3.2 间隔联结（Interval Join）

![image-20221130093933294](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221130093933294.png)

  间隔联结在代码中，是基于 KeyedStream 的联结（join）操作。DataStream 在 keyBy 得到KeyedStream 之后，可以调用.intervalJoin()来合并两条流，传入的参数同样是一个 KeyedStream，两者的 key 类型应该一致；得到的是一个 IntervalJoin 类型。后续的操作同样是完全固定的：先通过.between()方法指定间隔的上下界，再调用.process()方法，定义对匹配数据对的处理操作。调用.process()需要传入一个处理函数，这是处理函数家族的最后一员：“处理联结函数”ProcessJoinFunction。

```java
stream1
	.keyBy(<KeySelector>)
	.intervalJoin(stream2.keyBy(<KeySelector>))
 	.between(Time.milliseconds(-2), Time.milliseconds(1))
 	.process (new ProcessJoinFunction<Integer, Integer, String(){
 @Override
 public void processElement(Integer left, Integer right, Context ctx, Collector<String> out)  	  {
 		out.collect(left + "," + right);
 	}
 });
```

  在电商网站中，某些用户行为往往会有短时间内的强关联。我们这里举一个例子，我们有两条流，一条是下订单的流，一条是浏览数据的流。我们可以针对同一个用户，来做这样一个联结。也就是使用一个用户的下订单的事件和这个用户的最近十分钟的浏览数据进行一个联结查询。

```java
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

// 基于间隔的join
public class IntervalJoinTest {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple3<String, String, Long>> orderStream = env.fromElements(
                Tuple3.of("Mary", "order-1", 5000L),
                Tuple3.of("Alice", "order-2", 5000L),
                Tuple3.of("Bob", "order-3", 20000L),
                Tuple3.of("Alice", "order-4", 20000L),
                Tuple3.of("Cary", "order-5", 51000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Tuple3<String, String, Long>>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                    @Override
                    public long extractTimestamp(Tuple3<String, String, Long> element, long recordTimestamp) {
                        return element.f2;
                    }
                })
        );

        SingleOutputStreamOperator<Event> clickStream = env.fromElements(
                new Event("Bob", "./cart", 2000L),
                new Event("Alice", "./prod?id=100", 3000L),
                new Event("Alice", "./prod?id=200", 3500L),
                new Event("Bob", "./prod?id=2", 2500L),
                new Event("Alice", "./prod?id=300", 36000L),
                new Event("Bob", "./home", 30000L),
                new Event("Bob", "./prod?id=1", 23000L),
                new Event("Bob", "./prod?id=3", 33000L)
        ).assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                    @Override
                    public long extractTimestamp(Event element, long recordTimestamp) {
                        return element.timestamp;
                    }
                })
        );

        orderStream.keyBy(data -> data.f0)
                .intervalJoin(clickStream.keyBy(data -> data.user))
                .between(Time.seconds(-5), Time.seconds(10))
                .process(new ProcessJoinFunction<Tuple3<String, String, Long>, Event, String>() {
                    @Override
                    public void processElement(Tuple3<String, String, Long> left, Event right, Context ctx, Collector<String> out) throws Exception {
                        out.collect(right + " => " + left);
                    }
                })
                .print();

        env.execute();
    }}

```

![image-20221223162950631](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221223162950631.png)



### 8.3.3 窗口同组联结（Window CoGroup）

  除窗口联结和间隔联结之外，Flink 还提供了一个“窗口同组联结”（window coGroup）操作。它的用法跟 window join 非常类似，也是将两条流合并之后开窗处理匹配的元素，调用时只需要将.join()换为.coGroup()就可以了。

```
stream1.coGroup(stream2)
 .where(<KeySelector>)
 .equalTo(<KeySelector>)
 .window(TumblingEventTimeWindows.of(Time.hours(1)))
 .apply(<CoGroupFunction>)
```

  内部的.coGroup()方法，有些类似于 FlatJoinFunction 中.join()的形式，同样有三个参数，分别代表两条流中的数据以及用于输出的收集器（Collector）。==不同的是，这里的前两个参数不再是单独的每一组“配对”数据了，而是传入了可遍历的数据集合。也就是说，现在不会再去计算窗口中两条流数据集的笛卡尔积，而是直接把收集到的所有数据一次性传入，== 至于要怎样配对完全是自定义的。这样.coGroup()方法只会被调用一次，而且即使一条流的数据没有任何另一条流的数据匹配，也可以出现在集合中、当然也可以定义输出结果了。

  所以能够看出，coGroup 操作比窗口的 join 更加通用，不仅可以实现类似 SQL 中的“内连接”（inner join），也可以实现左外连接（left outer join）、右外连接（right outer join）和全外连接（full outer join）。事实上，窗口 join 的底层，也是通过 coGroup 来实现的。

```java
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.CoGroupFunction;

import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

// 基于窗口的join
public class CoGroupTest {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        DataStream<Tuple2<String, Long>> stream1 = env
                .fromElements(
                        Tuple2.of("a", 1000L),
                        Tuple2.of("b", 1000L),
                        Tuple2.of("a", 2000L),
                        Tuple2.of("b", 2000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple2<String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(
                                        new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                                            @Override
                                            public long extractTimestamp(Tuple2<String, Long> stringLongTuple2, long l) {
                                                return stringLongTuple2.f1;
                                            }
                                        }
                                )
                );

        DataStream<Tuple2<String, Long>> stream2 = env
                .fromElements(
                        Tuple2.of("a", 3000L),
                        Tuple2.of("b", 3000L),
                        Tuple2.of("a", 4000L),
                        Tuple2.of("b", 4000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy
                                .<Tuple2<String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(
                                        new SerializableTimestampAssigner<Tuple2<String, Long>>() {
                                            @Override
                                            public long extractTimestamp(Tuple2<String, Long> stringLongTuple2, long l) {
                                                return stringLongTuple2.f1;
                                            }
                                        }
                                )
                );

        stream1
                .coGroup(stream2)
                .where(r -> r.f0)
                .equalTo(r -> r.f0)
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .apply(new CoGroupFunction<Tuple2<String, Long>, Tuple2<String, Long>, String>() {
                    @Override
                    public void coGroup(Iterable<Tuple2<String, Long>> iter1, Iterable<Tuple2<String, Long>> iter2, Collector<String> collector) throws Exception {
                        collector.collect(iter1 + "=>" + iter2);
                    }
                })
                .print();

        env.execute();
    }
}
```

![image-20221223163625593](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221223163625593.png)

# 第 9 章 状态编程(98-115)

  状态就如同事务处理时数据库中保存的信息一样，是用来辅助进行任务计算的数据。而在 Flink 这样的分布式系统中，我们不仅需要定义出状态在任务并行时的处理方式，还需要考虑如何持久化保存、以便发生故障时正确地恢复。这就需要一套完整的管理机制来处理所有的状态。

## 9.1 Flink 中的状态

  在流处理中，数据是连续不断到来和处理的。每个任务进行计算处理时，可以基于当前数据直接转换得到输出结果；也可以依赖一些其他数据。这些由一个任务维护，并且用来计算输出结果的所有数据，就叫作这个任务的状态。

### 9.1.1 有状态算子

  在 Flink 中，算子任务可以分为无状态和有状态两种情况。

  无状态的算子任务只需要观察每个独立事件，根据当前输入的数据直接转换输出结果

![image-20221224092948762](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224092948762.png)

  而有状态的算子任务，则除当前数据之外，还需要一些其他数据来得到计算结果。这里的“其他数据”，就是所谓的状态（state），最常见的就是之前到达的数据，或者由之前数据计算出的某个结果。

![image-20221224092957580](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224092957580.png)

有状态算子的一般处理流程，具体步骤如下。
（1）算子任务接收到上游发来的数据；
（2）获取当前状态；
（3）根据业务逻辑进行计算，更新状态；
（4）得到计算结果，输出发送到下游任务。

### 9.1.2 状态的管理

  在传统的事务型处理架构中，这种额外的状态数据是保存在数据库中的。而对于实时流处理来说，这样做需要频繁读写外部数据库，如果数据规模非常大肯定就达不到性能要求了。所以 Flink 的解决方案是，将状态直接保存在内存中来保证性能，并通过分布式扩展来提高吞吐量。

  在 Flink 中，每一个算子任务都可以设置并行度，从而可以在不同的 slot 上并行运行多个实例，我们把它们叫作“并行子任务”。而状态既然在内存中，那么就可以认为是子任务实例上的一个本地变量，能够被任务的业务逻辑访问和修改。

  这样看来状态的管理似乎非常简单，我们直接把它作为一个对象交给 JVM 就可以了。然而大数据的场景下，我们必须使用分布式架构来做扩展，在低延迟、高吞吐的基础上还要保证容错性，一系列复杂的问题就会随之而来了。

⚫ 状态的访问权限。我们知道 Flink 上的聚合和窗口操作，一般都是基于 KeyedStream的，数据会按照 key 的哈希值进行分区，聚合处理的结果也应该是只对当前 key 有效。然而同一个分区（也就是 slot）上执行的任务实例，可能会包含多个 key 的数据，它们同时访问和更改本地变量，就会导致计算结果错误。所以这时状态并不是单纯的本地变量。

⚫ 容错性，也就是故障后的恢复。状态只保存在内存中显然是不够稳定的，我们需要将它持久化保存，做一个备份；在发生故障后可以从这个备份中恢复状态。

⚫ 我们还应该考虑到分布式应用的横向扩展性。比如处理的数据量增大时，我们应该相应地对计算资源扩容，调大并行度。这时就涉及到了状态的重组调整。

  可见状态的管理并不是一件轻松的事。好在 Flink 作为有状态的大数据流式处理框架，已经帮我们搞定了这一切。Flink 有一套完整的状态管理机制，将底层一些核心功能全部封装起来，包括状态的高效存储和访问、持久化保存和故障恢复，以及资源扩展时的调整。这样，我们只需要调用相应的 API 就可以很方便地使用状态，或对应用的容错机制进行配置，从而将更多的精力放在业务逻辑的开发上。

### 9.1.3 状态的分类

**1.托管状态（Managed State）和原始状态（Raw State）**

  Flink 的状态有两种：托管状态（Managed State）和原始状态（Raw State）。==托管状态就是由 Flink 统一管理的，状态的存储访问、故障恢复和重组等一系列问题都由 Flink 实现，我们只要调接口就可以；而原始状态则是自定义的，相当于就是开辟了一块内存，需要我们自己管理，实现状态的序列化和故障恢复。==

  具体来讲，托管状态是由 Flink 的运行时（Runtime）来托管的；在配置容错机制后，状态会自动持久化保存，并在发生故障时自动恢复。当应用发生横向扩展时，状态也会自动地重组分配到所有的子任务实例上。对于具体的状态内容，Flink 也提供了值状态（ValueState）、列表状态（ListState）、映射状态（MapState）、聚合状态（AggregateState）等多种结构，内部支持各种数据类型。聚合、窗口等算子中内置的状态，就都是托管状态；我们也可以在富函数类（RichFunction）中通过上下文来自定义状态，这些也都是托管状态。

  而对比之下，原始状态就全部需要自定义了。Flink 不会对状态进行任何自动操作，也不知道状态的具体数据类型，只会把它当作最原始的字节（Byte）数组来存储。我们需要花费大量的精力来处理状态的管理和维护。

  所以只有在遇到托管状态无法实现的特殊需求时，我们才会考虑使用原始状态；一般情况下不推荐使用。绝大多数应用场景，我们都可以用 Flink 提供的算子或者自定义托管状态来实现需求。

**2.算子状态（Operator State）和按键分区状态（Keyed State）**

接下来我们的重点就是托管状态（Managed State）。

  我们知道在 Flink 中，一个算子任务会按照并行度分为多个并行子任务执行，而不同的子任务会占据不同的任务槽（task slot）。由于不同的 slot 在计算资源上是物理隔离的，所以 Flink能管理的状态在并行任务间是无法共享的，每个状态只能针对当前子任务的实例有效。而很多有状态的操作（比如聚合、窗口）都是要先做 keyBy 进行按键分区的。按键分区之后，任务所进行的所有计算都应该只针对当前 key 有效，所以状态也应该按照 key 彼此隔离。在这种情况下，状态的访问方式又会有所不同。

基于这样的想法，我们又可以将托管状态分为两类：算子状态和按键分区状态。

**（1）算子状态（Operator State）**
  状态作用范围限定为当前的算子任务实例，也就是只对当前并行子任务实例有效。这就意味着对于一个并行子任务，占据了一个“分区”，==它所处理的所有数据都会访问到相同的状态==，状态对于同一任务而言是共享的。

![image-20221224095401246](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224095401246.png)

​		算子状态可以用在所有算子上，使用的时候其实就跟一个本地变量没什么区别——因为本地变量的作用域也是当前任务实例。在使用时，我们还需进一步实现 CheckpointedFunction 接口。
**（2）按键分区状态（Keyed State）**
  状态是根据输入流中定义的键（key）来维护和访问的，所以只能定义在按键分区流（KeyedStream）中，也就 keyBy 之后才可以使用。

![image-20221224095411637](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224095411637.png)

  按键分区状态应用非常广泛。之前讲到的聚合算子必须在 keyBy 之后才能使用，就是因为聚合的结果是以 Keyed State 的形式保存的。另外，也可以通过富函数类（Rich Function）来自定义 Keyed State，所以只要提供了富函数类接口的算子，也都可以使用 Keyed State。所以即使是 map、filter 这样无状态的基本转换算子，我们也可以通过富函数类给它们“追加”Keyed State，或者实现 CheckpointedFunction 接口来定义 Operator State；从这个角度讲，Flink 中所有的算子都可以是有状态的，不愧是“有状态的流处理”。

  无论是 Keyed State 还是 Operator State，它们都是在本地实例上维护的，也就是说每个并行子任务维护着对应的状态，算子的子任务之间状态不共享。关于状态的具体使用，我们会在下面继续展开讲解。

## 9.2 按键分区状态（Keyed State）

### 9.2.1 基本概念和特点

  按键分区状态（Keyed State）顾名思义，是任务按照键（key）来访问和维护的状态。它的特点非常鲜明，就是以 key 为作用范围进行隔离。

  在进行按键分区（keyBy）之后，具有相同键的所有数据，都会分配到同一个并行子任务中；所以如果当前任务定义了状态，Flink 就会在当前并行子任务实例中，为每个键值维护一个状态的实例。于是当前任务就会为分配来的所有数据，按照 key 维护和处理对应的状态。

### 9.2.2 支持的结构类型

**1.值状态（ValueState）**

  顾名思义，状态中只保存一个“值”（value）。ValueState<T>本身是一个接口，源码中定义如下：

```java
public interface ValueState<T> extends State {
	T value() throws IOException;
	void update(T value) throws IOException;
}
```

⚫ T value()：获取当前状态的值；
⚫ update(T value)：对状态进行更新，传入的参数 value 就是要覆写的状态值。

**2.列表状态（ListState）**

  将需要保存的数据，以列表（List）的形式组织起来。在 ListState<T>接口中同样有一个类型参数 T，表示列表中数据的类型。ListState 也提供了一系列的方法来操作状态，使用方式与一般的 List 非常相似。

⚫ Iterable<T> get()：获取当前的列表状态，返回的是一个可迭代类型 Iterable<T>；
⚫ update(List<T> values)：传入一个列表 values，直接对状态进行覆盖；
⚫ add(T value)：在状态列表中添加一个元素 value；
⚫ addAll(List<T> values)：向列表中添加多个元素，以列表 values 形式传入。

**3.映射状态（MapState）**

  把一些键值对（key-value）作为状态整体保存起来，可以认为就是一组 key-value 映射的列表。对应的 MapState<UK, UV>接口中，就会有 UK、UV 两个泛型，分别表示保存的 key和 value 的类型。同样，MapState 提供了操作映射状态的方法，与 Map 的使用非常类似。

⚫ UV get(UK key)：传入一个 key 作为参数，查询对应的 value 值；
⚫ put(UK key, UV value)：传入一个键值对，更新 key 对应的 value 值；
⚫ putAll(Map<UK, UV> map)：将传入的映射 map 中所有的键值对，全部添加到映射状态中；
⚫ remove(UK key)：将指定 key 对应的键值对删除；
⚫ boolean contains(UK key)：判断是否存在指定的 key，返回一个 boolean 值。另外，MapState 也提供了获取整个映射相关信息的方法：
⚫ Iterable<Map.Entry<UK, UV>> entries()：获取映射状态中所有的键值对；
⚫ Iterable<UK> keys()：获取映射状态中所有的键（key），返回一个可迭代 Iterable 类型；
⚫ Iterable<UV> values()：获取映射状态中所有的值（value），返回一个可迭代 Iterable类型；
⚫ boolean isEmpty()：判断映射是否为空，返回一个 boolean 值。

**4.归约状态（ReducingState）**

  类似于值状态（Value），不过需要对添加进来的所有数据进行归约，将归约聚合之后的值作为状态保存下来。ReducintState<T>这个接口调用的方法类似于 ListState，只不过它保存的只是一个聚合值，所以调用.add()方法时，不是在状态列表里添加元素，而是直接把新数据和之前的状态进行归约，并用得到的结果更新状态。

  归约逻辑的定义，是在归约状态描述器（ReducingStateDescriptor）中，通过传入一个归约函数（ReduceFunction）来实现的。这里的归约函数，就是我们之前介绍 reduce 聚合算子时讲到的 ReduceFunction，所以状态类型跟输入的数据类型是一样的。

```java
public ReducingStateDescriptor(
 String name, ReduceFunction<T> reduceFunction, Class<T> typeClass) {...}
```

  这里的描述器有三个参数，其中第二个参数就是定义了归约聚合逻辑的 ReduceFunction，另外两个参数则是状态的名称和类型。

**5.聚合状态（AggregatingState）**

  与归约状态非常类似，聚合状态也是一个值，用来保存添加进来的所有数据的聚合结果。与 ReducingState 不同的是，它的聚合逻辑是由在描述器中传入一个更加一般化的聚合函（AggregateFunction）来定义的；这也就是之前我们讲过的 AggregateFunction，里面通过一个

  累加器（Accumulator）来表示状态，所以聚合的状态类型可以跟添加进来的数据类型完全不同，使用更加灵活。

  同样地，AggregatingState 接口调用方法也与 ReducingState 相同，调用.add()方法添加元素时，会直接使用指定的 AggregateFunction 进行聚合并更新状态。

### 9.2.3 代码实现

  在 Flink 中，状态始终是与特定算子相关联的；算子在使用状态前首先需要“注册”，其实就是告诉 Flink 当前上下文中定义状态的信息，这样运行时的 Flink 才能知道算子有哪些状态。
  状态的注册，主要是通过“状态描述器”（StateDescriptor）来实现的。状态描述器中最重要的内容，就是状态的名称（name）和类型（type）。我们知道 Flink 中的状态，可以认为是加了一些复杂操作的内存中的变量；而当我们在代码中声明一个局部变量时，都需要指定变量类型和名称，名称就代表了变量在内存中的地址，类型则指定了占据内存空间的大小。同样地，我们一旦指定了名称和类型，Flink 就可以在运行时准确地在内存中找到对应的状态，进而返回状态对象供我们使用了。所以在一个算子中，我们也可以定义多个状态，只要它们的名称不同就可以了。
  另外，状态描述器中还可能需要传入一个用户自定义函数（user-defined-function，UDF），用来说明处理逻辑，比如前面提到的 ReduceFunction 和 AggregateFunction。

**1.值状态（ValueState）**

```java
import com.atguigu.chapter05.ClickSource;
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;



public class PeriodicPvExample{

    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );

        //stream.print("input");

        // 统计每个用户的pv，隔一段时间（10s）输出一次结果
        stream.keyBy(data -> data.user)
                .process(new PeriodicPvResult())
                .print("***********************");

        env.execute();
    }

    // 注册定时器，周期性输出pv
    public static class PeriodicPvResult extends KeyedProcessFunction<String ,Event, String>{
        // 定义两个状态，保存当前pv值，以及定时器时间戳
        ValueState<Long> countState;
        ValueState<Long> timerTsState;

        @Override
        public void open(Configuration parameters) throws Exception {
            countState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("count", Long.class));
            timerTsState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("timerTs", Long.class));
        }

        @Override
        public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
            // 更新count值
            Long count = countState.value();
            if (count == null){
                countState.update(1L);
            } else {
                countState.update(count + 1);
            }
            // 注册定时器
            if (timerTsState.value() == null){
                ctx.timerService().registerEventTimeTimer(value.timestamp + 10 * 1000L);
                timerTsState.update(value.timestamp + 10 * 1000L);
            }
        }

        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            out.collect(ctx.getCurrentKey() + " pv: " + countState.value());
            // 清空状态
            timerTsState.clear();
        }
    }}
```

![image-20221224112255726](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224112255726.png)

**2.列表状态（ListState）**

```java
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple3;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.CoProcessFunction;
import org.apache.flink.util.Collector;

public class TwoStreamFullJoinExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Tuple3<String, String, Long>> stream1 = env
                .fromElements(
                        Tuple3.of("a", "stream-1", 1000L),
                        Tuple3.of("b", "stream-1", 2000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Tuple3<String, String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                                    @Override
                                    public long extractTimestamp(Tuple3<String, String, Long> t, long l) {
                                        return t.f2;
                                    }
                                })
                );

        SingleOutputStreamOperator<Tuple3<String, String, Long>> stream2 = env
                .fromElements(
                        Tuple3.of("a", "stream-2", 3000L),
                        Tuple3.of("b", "stream-2", 4000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Tuple3<String, String, Long>>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Tuple3<String, String, Long>>() {
                                    @Override
                                    public long extractTimestamp(Tuple3<String, String, Long> t, long l) {
                                        return t.f2;
                                    }
                                })
                );

        stream1.keyBy(r -> r.f0)
                .connect(stream2.keyBy(r -> r.f0))
                .process(new CoProcessFunction<Tuple3<String, String, Long>, Tuple3<String, String, Long>, String>() {
                    private ListState<Tuple3<String, String, Long>> stream1ListState;
                    private ListState<Tuple3<String, String, Long>> stream2ListState;

                    @Override
                    public void open(Configuration parameters) throws Exception {
                        super.open(parameters);
                        stream1ListState = getRuntimeContext().getListState(
                                new ListStateDescriptor<Tuple3<String, String, Long>>("stream1-list", Types.TUPLE(Types.STRING, Types.STRING))
                        );
                        stream2ListState = getRuntimeContext().getListState(
                                new ListStateDescriptor<Tuple3<String, String, Long>>("stream2-list", Types.TUPLE(Types.STRING, Types.STRING))
                        );
                    }

                    @Override
                    public void processElement1(Tuple3<String, String, Long> left, Context context, Collector<String> collector) throws Exception {
                        stream1ListState.add(left);
                        for (Tuple3<String, String, Long> right : stream2ListState.get()) {
                            collector.collect(left + " => " + right);
                        }
                    }

                    @Override
                    public void processElement2(Tuple3<String, String, Long> right, Context context, Collector<String> collector) throws Exception {
                        stream2ListState.add(right);
                        for (Tuple3<String, String, Long> left : stream1ListState.get()) {
                            collector.collect(left + " => " + right);
                        }
                    }
                })
                .print();

        env.execute();
    }
}
```

![image-20221224112342935](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224112342935.png)

**3.映射状态（MapState）**

```java
import com.atguigu.chapter05.ClickSource;
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.MapState;
import org.apache.flink.api.common.state.MapStateDescriptor;

import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;

import org.apache.flink.util.Collector;

import java.sql.Timestamp;


// 使用KeyedProcessFunction模拟滚动窗口
public class FakeWindowExample {


    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );

        // 统计每10s窗口内，每个url的pv
        stream.keyBy(data -> data.url)
                .process(new FakeWindowResult(10000L))
                .print();

        env.execute();
    }

    public static class FakeWindowResult extends KeyedProcessFunction<String, Event, String>{
        // 定义属性，窗口长度
        private Long windowSize;

        public FakeWindowResult(Long windowSize) {
            this.windowSize = windowSize;
        }

        // 声明状态，用map保存pv值（窗口start，count）
        MapState<Long, Long> windowPvMapState;

        @Override
        public void open(Configuration parameters) throws Exception {
            windowPvMapState = getRuntimeContext().getMapState(new MapStateDescriptor<Long, Long>("window-pv", Long.class, Long.class));
        }

        @Override
        public void processElement(Event value, Context ctx, Collector<String> out) throws Exception {
            // 每来一条数据，就根据时间戳判断属于哪个窗口
            Long windowStart = value.timestamp / windowSize * windowSize;
            Long windowEnd = windowStart + windowSize;

            // 注册 end -1 的定时器，窗口触发计算
            ctx.timerService().registerEventTimeTimer(windowEnd - 1);

            // 更新状态中的pv值
            if (windowPvMapState.contains(windowStart)){
                Long pv = windowPvMapState.get(windowStart);
                windowPvMapState.put(windowStart, pv + 1);
            } else {
                windowPvMapState.put(windowStart, 1L);
            }
        }

        // 定时器触发，直接输出统计的pv结果
        @Override
        public void onTimer(long timestamp, OnTimerContext ctx, Collector<String> out) throws Exception {
            Long windowEnd = timestamp + 1;
            Long windowStart = windowEnd - windowSize;
            Long pv = windowPvMapState.get(windowStart);
            out.collect( "url: " + ctx.getCurrentKey()
                    + " 访问量: " + pv
                    + " 窗口：" + new Timestamp(windowStart) + " ~ " + new Timestamp(windowEnd));

            // 模拟窗口的销毁，清除map中的key
            windowPvMapState.remove(windowStart);
        }
    }    
}
```

![image-20221224165828696](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221224165828696.png)

**4.聚合状态（AggregatingState）**

  我们举一个简单的例子，对用户点击事件流每 5 个数据统计一次平均时间戳。这是一个类似计数窗口（CountWindow）求平均值的计算，这里我们可以使用一个有聚合状态的RichFlatMapFunction 来实现。

```java
import com.atguigu.chapter05.ClickSource;
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.functions.AggregateFunction;
import org.apache.flink.api.common.functions.RichFlatMapFunction;
import org.apache.flink.api.common.state.AggregatingState;
import org.apache.flink.api.common.state.AggregatingStateDescriptor;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

import java.sql.Timestamp;

public class AverageTimestampExample {
    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
                                return element.timestamp;
                            }
                        })
                );


        // 统计每个用户的点击频次，到达5次就输出统计结果
        stream.keyBy(data -> data.user)
                .flatMap(new AvgTsResult())
                .print();

        env.execute();
    }

    public static class AvgTsResult extends RichFlatMapFunction<Event, String>{
        // 定义聚合状态，用来计算平均时间戳
        AggregatingState<Event, Long> avgTsAggState;

        // 定义一个值状态，用来保存当前用户访问频次
        ValueState<Long> countState;

        @Override
        public void open(Configuration parameters) throws Exception {
            avgTsAggState = getRuntimeContext().getAggregatingState(new AggregatingStateDescriptor<Event, Tuple2<Long, Long>, Long>(
                    "avg-ts",
                    new AggregateFunction<Event, Tuple2<Long, Long>, Long>() {
                        @Override
                        public Tuple2<Long, Long> createAccumulator() {
                            return Tuple2.of(0L, 0L);
                        }

                        @Override
                        public Tuple2<Long, Long> add(Event value, Tuple2<Long, Long> accumulator) {
                            return Tuple2.of(accumulator.f0 + value.timestamp, accumulator.f1 + 1);
                        }

                        @Override
                        public Long getResult(Tuple2<Long, Long> accumulator) {
                            return accumulator.f0 / accumulator.f1;
                        }

                        @Override
                        public Tuple2<Long, Long> merge(Tuple2<Long, Long> a, Tuple2<Long, Long> b) {
                            return null;
                        }
                    },
                    Types.TUPLE(Types.LONG, Types.LONG)
            ));

            countState = getRuntimeContext().getState(new ValueStateDescriptor<Long>("count", Long.class));
        }

        @Override
        public void flatMap(Event value, Collector<String> out) throws Exception {
            Long count = countState.value();
            if (count == null){
                count = 1L;
            } else {
                count ++;
            }

            countState.update(count);
            avgTsAggState.add(value);

            // 达到5次就输出结果，并清空状态
            if (count == 5){
                out.collect(value.user + " 平均时间戳：" + new Timestamp(avgTsAggState.get()));
                countState.clear();
            }
        }
    }
}
```

### 9.2.4 状态生存时间（TTL）

  在实际应用中，很多状态会随着时间的推移逐渐增长，如果不加以限制，最终就会导致存储空间的耗尽。一个优化的思路是直接在代码中调用.clear()方法去清除状态，但是有时候我们的逻辑要求不能直接清除。这时就需要配置一个状态的“生存时间”（time-to-live，TTL），当状态在内存中存在的时间超出这个值时，就将它清除。

  具体实现上，如果用一个进程不停地扫描所有状态看是否过期，显然会占用大量资源做无用功。状态的失效其实不需要立即删除，所以我们可以给状态附加一个属性，也就是状态的“失效时间”。状态创建的时候，设置 失效时间 = 当前时间 + TTL；之后如果有对状态的访问和修改，我们可以再对失效时间进行更新；当设置的清除条件被触发时（比如，状态被访问的时候，或者每隔一段时间扫描一次失效状态），就可以判断状态是否失效、从而进行清除了。配置状态的 TTL 时，需要创建一个 StateTtlConfig 配置对象，然后调用状态描述器的.enableTimeToLive()方法启动 TTL 功能。

```
StateTtlConfig ttlConfig = StateTtlConfig
	 .newBuilder(Time.seconds(10))
 	 .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
 	 .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
	 .build();
ValueStateDescriptor<String> stateDescriptor = new ValueStateDescriptor<>("my
state", String.class);
stateDescriptor.enableTimeToLive(ttlConfig);
```

**这里用到了几个配置项：**
⚫ .newBuilder()
  状态 TTL 配置的构造器方法，必须调用，返回一个 Builder 之后再调用.build()方法就可以得到 StateTtlConfig 了。方法需要传入一个 Time 作为参数，==这就是设定的状态生存时间==。
⚫ .setUpdateType()
  设置更新类型。==更新类型指定了什么时候更新状态失效时间==，这里的 OnCreateAndWrite表示只有创建状态和更改状态（写操作）时更新失效时间。另一种类型 OnReadAndWrite 则表示无论读写操作都会更新失效时间，也就是只要对状态进行了访问，就表明它是活跃的，从而延长生存时间。这个配置默认为 OnCreateAndWrite。
⚫ .setStateVisibility()
  设置状态的可见性。所谓的“状态可见性”，是指因为清除操作并不是实时的，所以当状态过期之后还有可能基于存在，这时如果对它进行访问，能否正常读取到就是一个问题了。这里设置的 NeverReturnExpired 是默认行为，表示从不返回过期值，==也就是只要过期就认为它已经被清除了，应用不能继续读取==；这在处理会话或者隐私数据时比较重要。对应的另一种配置是 ReturnExpireDefNotCleanedUp，就是如果过期状态还存在，就返回它的值。

  除此之外，TTL 配置还可以设置在保存检查点（checkpoint）时触发清除操作，或者配置增量的清理（incremental cleanup），还可以针对 RocksDB 状态后端使用压缩过滤器（compaction filter）进行后台清理。关于检查点和状态后端的内容，我们会在后续章节继续讲解。
  这里需要注意，目前的 TTL 设置只支持处理时间。另外，所有集合类型的状态（例如ListState、MapState）在设置 TTL 时，都是针对每一项（per-entry）元素的。也就是说，一个列表状态中的每一个元素，都会以自己的失效时间来进行清理，而不是整个列表一起清理。

## 9.3 算子状态（Operator State）

  除按键分区状态（Keyed State）之外，另一大类受控状态就是算子状态（Operator State）。从某种意义上说，算子状态是更底层的状态类型，因为它只针对当前算子并行任务有效，不需要考虑不同 key 的隔离。算子状态功能不如按键分区状态丰富，应用场景较少，它的调用方法也会有一些区别。

### 9.3.1 基本概念和特点

  算子状态（Operator State）就是一个算子并行实例上定义的状态，==作用范围被限定为当前算子任务==。算子状态跟数据的 key 无关，所以不同 key 的数据只要被分发到同一个并行子任务，就会访问到同一个 Operator State。

  算子状态的实际应用场景不如 Keyed State 多，一般用在 Source 或 Sink 等与外部系统连接的算子上，或者完全没有 key 定义的场景。比如 Flink 的 Kafka 连接器中，就用到了算子状态。在我们给 Source 算子设置并行度后，Kafka 消费者的每一个并行实例，都会为对应的主题

  （topic）分区维护一个偏移量， 作为算子状态保存起来。这在保证 Flink 应用“精确一次”（exactly-once）状态一致性时非常有用。关于状态一致性的内容，我们会在第十章详细展开。当算子的并行度发生变化时，算子状态也支持在并行的算子任务实例之间做重组分配。根据状态的类型不同，重组分配的方案也会不同。

### 9.3.2 状态类型

算子状态也支持不同的结构类型，主要有三种：ListState、UnionListState 和 BroadcastState。

**1.列表状态（ListState）**

  与 Keyed State 中的 ListState 一样，将状态表示为一组数据的列表。

  与 Keyed State 中的列表状态的区别是：在算子状态的上下文中，不会按键（key）分别处理状态，所以每一个并行子任务上只会保留一个“列表”（list），也就是当前并行子任务上所有状态项的集合。列表中的状态项就是可以重新分配的最细粒度，彼此之间完全独立。

  当算子并行度进行缩放调整时，算子的列表状态中的所有元素项会被统一收集起来，相当于把多个分区的列表合并成了一个“大列表”，然后再均匀地分配给所有并行任务。这种“均匀分配”的具体方法就是“轮询”（round-robin），与之前介绍的 rebanlance 数据传输方式类似，是通过逐一“发牌”的方式将状态项平均分配的。这种方式也叫作“平均分割重组”（even-split redistribution）。

  算子状态中不会存在“键组”（key group）这样的结构，所以为了方便重组分配，就把它直接定义成了“列表”（list）。这也就解释了，为什么算子状态中没有最简单的值状态（ValueState）。

**2.联合列表状态（UnionListState）**

  与 ListState 类似，联合列表状态也会将状态表示为一个列表。它与常规列表状态的区别在于，算子并行度进行缩放调整时对于状态的分配方式不同。

  UnionListState 的重点就在于“联合”（union）。在并行度调整时，常规列表状态是轮询分配状态项，而联合列表状态的算子则会直接广播状态的完整列表。这样，并行度缩放之后的并行子任务就获取到了联合后完整的“大列表”，可以自行选择要使用的状态项和要丢弃的状态项。这种分配也叫作“联合重组”（union redistribution）。如果列表中状态项数量太多，为资源和效率考虑一般不建议使用联合重组的方式。

**3.广播状态（BroadcastState）**

  有时我们希望算子并行子任务都保持同一份“全局”状态，用来做统一的配置和规则设定。这时所有分区的所有数据都会访问到同一个状态，状态就像被“广播”到所有分区一样，这种特殊的算子状态，就叫作广播状态（BroadcastState）。

  因为广播状态在每个并行子任务上的实例都一样，所以在并行度调整的时候就比较简单，只要复制一份到新的并行任务就可以实现扩展；而对于并行度缩小的情况，可以将多余的并行子任务连同状态直接砍掉——因为状态都是复制出来的，并不会丢失。

  在底层，广播状态是以类似映射结构（map）的键值对（key-value）来保存的，必须基于一个“广播流”（BroadcastStream）来创建。关于广播流，我们在第八章“广播连接流”的讲解中已经做过介绍。

### 9.3.3 代码实现

  我们已经知道，状态从本质上来说就是算子并行子任务实例上的一个特殊本地变量。它的特殊之处就在于 Flink 会提供完整的管理机制，来保证它的持久化保存，以便发生故障时进行状态恢复；另外还可以针对不同的 key 保存独立的状态实例。按键分区状态（Keyed State）对这两个功能都要考虑；而算子状态（Operator State）并不考虑 key 的影响，所以主要任务就是要让 Flink 了解状态的信息、将状态数据持久化后保存到外部存储空间。

  看起来算子状态的使用应该更加简单才对。不过仔细思考又会发现一个问题：我们对状态进行持久化保存的目的是为了故障恢复；在发生故障、重启应用后，数据还会被发往之前分配的分区吗？显然不是，因为并行度可能发生了调整，不论是按键（key）的哈希值分区，还是直接轮询（round-robin）分区，数据分配到的分区都会发生变化。这很好理解，当打牌的人数从 3 个增加到 4 个时，即使牌的次序不变，轮流发到每个人手里的牌也会不同。数据分区发生变化，带来的问题就是，怎么保证原先的状态跟故障恢复后数据的对应关系呢？

  对于 Keyed State 这个问题很好解决：状态都是跟 key 相关的，而相同 key 的数据不管发往哪个分区，总是会全部进入一个分区的；于是只要将状态也按照 key 的哈希值计算出对应的分区，进行重组分配就可以了。恢复状态后继续处理数据，就总能按照 key 找到对应之前的状态，就保证了结果的一致性。所以 Flink 对 Keyed State 进行了非常完善的包装，我们不需实现任何接口就可以直接使用。

  而对于 Operator State 来说就会有所不同。因为不存在 key，所有数据发往哪个分区是不可预测的；也就是说，当发生故障重启之后，我们不能保证某个数据跟之前一样，进入到同一个并行子任务、访问同一个状态。所以 Flink 无法直接判断该怎样保存和恢复状态，而是提供了接口，让我们根据业务需求自行设计状态的快照保存（snapshot）和恢复（restore）逻辑。

**1.CheckpointedFunction 接口**

  在 Flink 中，对状态进行持久化保存的快照机制叫作“检查点”（Checkpoint）。于是使用算子状态时，就需要对检查点的相关操作进行定义，实现一个 CheckpointedFunction 接口。CheckpointedFunction 接口在源码中定义如下：

```java
public interface CheckpointedFunction {
	// 保存状态快照到检查点时，调用这个方法
	void snapshotState(FunctionSnapshotContext context) throws Exception
	// 初始化状态时调用这个方法，也会在恢复状态时调用
 	void initializeState(FunctionInitializationContext context) throws Exception;
}
```

  每次应用保存检查点做快照时，都会调用.snapshotState()方法，将状态进行外部持久化。而在算子任务进行初始化时，会调用. initializeState()方法。这又有两种情况：一种是整个应用第一次运行，这时状态会被初始化为一个默认值（default value）；另一种是应用重启时，从检查点（checkpoint）或者保存点（savepoint）中读取之前状态的快照，并赋给本地状态。所以，接口中的.snapshotState()方法定义了检查点的快照保存逻辑，而. initializeState()方法不仅定义了初始化逻辑，也定义了恢复逻辑。

  这里需要注意，CheckpointedFunction 接口中的两个方法，分别传入了一个上下文（context）作为参数。不同的是，.snapshotState()方法拿到的是快照的上下文 FunctionSnapshotContext，它可以提供检查点的相关信息，不过无法获取状态句柄；而. initializeState()方法拿到的是FunctionInitializationContext，这是函数类进行初始化时的上下文，是真正的“运行时上下文”。FunctionInitializationContext 中提供了“算子状态存储”（OperatorStateStore）和“按键分区状态存储（” KeyedStateStore），在这两个存储对象中可以非常方便地获取当前任务实例中的 OperatorState 和 Keyed State。例如：

```
ListStateDescriptor<String> descriptor =
 new ListStateDescriptor<>(
 "buffered-elements",
 Types.of(String));
ListState<String> checkpointedState = 
context.getOperatorStateStore().getListState(descriptor);
```

  我们看到，算子状态的注册和使用跟 Keyed State 非常类似，也是需要先定义一个状态描述器（StateDescriptor），告诉 Flink 当前状态的名称和类型，然后从上下文提供的算子状态存（OperatorStateStore）中获取对应的状态对象。如果想要从 KeyedStateStore 中获取 Keyed State也是一样的，前提是必须基于定义了 key 的 KeyedStream，这和富函数类中的方式并不矛盾。通过这里的描述可以发现，CheckpointedFunction 是 Flink 中非常底层的接口，它为有状态的流处理提供了灵活且丰富的应用。

**2.示例代码**

```java
import com.atguigu.chapter05.ClickSource;
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.api.common.state.ListState;
import org.apache.flink.api.common.state.ListStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.runtime.state.FunctionInitializationContext;
import org.apache.flink.runtime.state.FunctionSnapshotContext;
import org.apache.flink.runtime.state.storage.FileSystemCheckpointStorage;
import org.apache.flink.streaming.api.CheckpointingMode;
import org.apache.flink.streaming.api.checkpoint.CheckpointedFunction;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.CheckpointConfig;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.sink.SinkFunction;

import java.util.ArrayList;
import java.util.List;

public class BufferingSinkExample {

    public static void main(String[] args) throws Exception{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        env.enableCheckpointing(10000L);
//        env.setStateBackend(new EmbeddedRocksDBStateBackend());

//        env.getCheckpointConfig().setCheckpointStorage(new FileSystemCheckpointStorage(""));

        CheckpointConfig checkpointConfig = env.getCheckpointConfig();
        checkpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        checkpointConfig.setMinPauseBetweenCheckpoints(500);
        checkpointConfig.setCheckpointTimeout(60000);
        checkpointConfig.setMaxConcurrentCheckpoints(1);
        checkpointConfig.enableExternalizedCheckpoints(
                CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
        checkpointConfig.enableUnalignedCheckpoints();


        SingleOutputStreamOperator<Event> stream = env.addSource(new ClickSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forMonotonousTimestamps()
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event element, long recordTimestamp) {
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
        private transient ListState<Event> checkpointedState;
        private List<Event> bufferedElements;

        public BufferingSink(int threshold) {
            this.threshold = threshold;
            this.bufferedElements = new ArrayList<>();
        }

        @Override
        public void invoke(Event value, Context context) throws Exception {
            bufferedElements.add(value);
            if (bufferedElements.size() == threshold) {
                for (Event element: bufferedElements) {
                    // 输出到外部系统，这里用控制台打印模拟
                    System.out.println(element);
                }
                System.out.println("==========输出完毕=========");
                bufferedElements.clear();
            }
        }

        @Override
        public void snapshotState(FunctionSnapshotContext context) throws Exception {
            checkpointedState.clear();
            // 把当前局部变量中的所有元素写入到检查点中
            for (Event element : bufferedElements) {
                checkpointedState.add(element);
            }
        }

        @Override
        public void initializeState(FunctionInitializationContext context) throws Exception {
            ListStateDescriptor<Event> descriptor = new ListStateDescriptor<>(
                    "buffered-elements",
                    Types.POJO(Event.class));

            checkpointedState = context.getOperatorStateStore().getListState(descriptor);

            // 如果是从故障中恢复，就将ListState中的所有元素添加到局部变量中
            if (context.isRestored()) {
                for (Event element : checkpointedState.get()) {
                    bufferedElements.add(element);
                }
            }
        }
    }
}

```

![image-20221225102044103](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221225102044103.png)

  当初始化好状态对象后，我们可以通过调用. isRestored()方法判断是否是从故障中恢复。在代码中 BufferingSink 初始化时，恢复出的 ListState 的所有元素会添加到一个局部变量bufferedElements 中，以后进行检查点快照时就可以直接使用了。在调用.snapshotState()时，直接清空 ListState，然后把当前局部变量中的所有元素写入到检查点中。

  对于不同类型的算子状态，需要调用不同的获取状态对象的接口，对应地也就会使用不同的状态分配重组算法。比如获取列表状态时，调用.getListState() 会使用最简单的 平均分割重组（even-split redistribution）算法；而获取联合列表状态时，调用的是.getUnionListState() ，对应就会使用联合重组（union redistribution） 算法。

## 9.4 广播状态（Broadcast State）

  算子状态中有一类很特殊，就是广播状态（Broadcast State）。从概念和原理上讲，广播状态非常容易理解：状态广播出去，所有并行子任务的状态都是相同的；并行度调整时只要直接复制就可以了。然而在应用上，广播状态却与其他算子状态大不相同。本节我们就专门来讨论一下广播状态的使用。

### 9.4.1 基本用法

​		让所有并行子任务都持有同一份状态，也就意味着一旦状态有变化，所以子任务上的实例都要更新。什么时候会用到这样的广播状态呢？

​		一个最为普遍的应用，就是“动态配置”或者“动态规则”。我们在处理流数据时，有时会基于一些配置（configuration）或者规则（rule）。简单的配置当然可以直接读取配置文件，一次加载，永久有效；但数据流是连续不断的，如果这配置随着时间推移还会动态变化，那又该怎么办呢？

​		一个简单的想法是，定期扫描配置文件，发现改变就立即更新。但这样就需要另外启动一个扫描进程，如果扫描周期太长，配置更新不及时就会导致结果错误；如果扫描周期太短，又会耗费大量资源做无用功。解决的办法，还是流处理的“事件驱动”思路——我们可以将这动态的配置数据看作一条流，将这条流和本身要处理的数据流进行连接（connect），就可以实时地更新配置进行计算了。

​		由于配置或者规则数据是全局有效的，我们需要把它广播给所有的并行子任务。而子任务需要把它作为一个算子状态保存起来，以保证故障恢复后处理结果是一致的。这时的状态，就是一个典型的广播状态。我们知道，广播状态与其他算子状态的列表（list）结构不同，底层是以键值对（key-value）形式描述的，所以其实就是一个映射状态（MapState）。在代码上，可以直接调用 DataStream 的.broadcast()方法，传入一个“映射状态描述器”
（MapStateDescriptor）说明状态的名称和类型，就可以得到一个“广播流”（BroadcastStream）；进而将要处理的数据流与这条广播流进行连接（connect），就会得到“广播连接流”（BroadcastConnectedStream）。注意广播状态只能用在广播连接流中。

关于广播连接流，我们已经在 8.2.2 节做过介绍，这里可以复习一下：

```
MapStateDescriptor<String, Rule> ruleStateDescriptor = new 
MapStateDescriptor<>(...);
BroadcastStream<Rule> ruleBroadcastStream = ruleStream
 .broadcast(ruleStateDescriptor);
DataStream<String> output = stream
 .connect(ruleBroadcastStream)
 .process( new BroadcastProcessFunction<>() {...} );
```

​		这里我们定义了一个“规则流”ruleStream，里面的数据表示了数据流 stream 处理的规则，规则的数据类型定义为 Rule。于是需要先定义一个 MapStateDescriptor 来描述广播状态，然后传入 ruleStream.broadcast()得到广播流，接着用 stream 和广播流进行连接。这里状态描述器中的 key 类型为 String，就是为了区分不同的状态值而给定的 key 的名称。
​		对 于 广 播 连 接 流 调 用 .process() 方 法 ， 可 以 传 入 “ 广 播 处 理 函 数 ”KeyedBroadcastProcessFunction 或者 BroadcastProcessFunction 来进行处理计算。广播处理函数里面有两个方法.processElement()和.processBroadcastElement()，源码中定义如下：

```
public abstract class BroadcastProcessFunction<IN1, IN2, OUT> extends 
BaseBroadcastProcessFunction {
...
 public abstract void processElement(IN1 value, ReadOnlyContext ctx, 
Collector<OUT> out) throws Exception;
 public abstract void processBroadcastElement(IN2 value, Context ctx, 
Collector<OUT> out) throws Exception;
...
}
```

​		这里的.processElement()方法，处理的是正常数据流，第一个参数 value 就是当前到来的流数据；而.processBroadcastElement()方法就相当于是用来处理广播流的，它的第一个参数 value就是广播流中的规则或者配置数据。两个方法第二个参数都是一个上下文 ctx，都可以通过调用.getBroadcastState()方法获取到当前的广播状态；区别在于，.processElement()方法里的上下文 是 “ 只 读 ” 的 （ ReadOnly ）， 因 此 获 取 到 的 广 播 状 态 也 只 能 读 取 不 能 更 改 ；而.processBroadcastElement()方法里的 Context 则没有限制，可以根据当前广播流中的数据更新状态。

```
Rule rule = ctx.getBroadcastState( new MapStateDescriptor<>("rules", Types.String, 
Types.POJO(Rule.class))).get("my rule");
```

​		通过调用 ctx.getBroadcastState()方法，传入一个 MapStateDescriptor，就可以得到当前的叫作“rules”的广播状态；调用它的.get()方法，就可以取出其中“my rule”对应的值进行计算处理。

### 9.4.2 代码实例

  接下来我们举一个广播状态的应用案例。考虑在电商应用中，往往需要判断用户先后发生的行为的“组合模式”，比如“登录-下单”或者“登录-支付”，检测出这些连续的行为进行统计，就可以了解平台的运用状况以及用户的行为习惯。

```java

import org.apache.flink.api.common.state.BroadcastState;
import org.apache.flink.api.common.state.MapStateDescriptor;
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.datastream.BroadcastStream;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.co.KeyedBroadcastProcessFunction;
import org.apache.flink.util.Collector;

public class BroadcastStateExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 读取用户行为事件流
        DataStreamSource<Action> actionStream = env.fromElements(
                new Action("Alice", "login"),
                new Action("Alice", "pay"),
                new Action("Bob", "login"),
                new Action("Bob", "buy")
        );

        // 定义行为模式流，代表了要检测的标准
        DataStreamSource<Pattern> patternStream = env
                .fromElements(
                        new Pattern("login", "pay"),
                        new Pattern("login", "buy")
                );

        // 定义广播状态的描述器，创建广播流
        MapStateDescriptor<Void, Pattern> bcStateDescriptor = new MapStateDescriptor<>(
                "patterns", Types.VOID, Types.POJO(Pattern.class));
        BroadcastStream<Pattern> bcPatterns = patternStream.broadcast(bcStateDescriptor);

        // 将事件流和广播流连接起来，进行处理
        DataStream<Tuple2<String, Pattern>> matches = actionStream
                .keyBy(data -> data.userId)
                .connect(bcPatterns)
                .process(new PatternEvaluator());

        matches.print();

        env.execute();
    }

    public static class PatternEvaluator
            extends KeyedBroadcastProcessFunction<String, Action, Pattern, Tuple2<String, Pattern>> {

        // 定义一个值状态，保存上一次用户行为
        ValueState<String> prevActionState;

        @Override
        public void open(Configuration conf) {
            prevActionState = getRuntimeContext().getState(
                    new ValueStateDescriptor<>("lastAction", Types.STRING));
        }

        @Override
        public void processBroadcastElement(
                Pattern pattern,
                Context ctx,
                Collector<Tuple2<String, Pattern>> out) throws Exception {

            BroadcastState<Void, Pattern> bcState = ctx.getBroadcastState(
                    new MapStateDescriptor<>("patterns", Types.VOID, Types.POJO(Pattern.class)));

            // 将广播状态更新为当前的pattern
            bcState.put(null, pattern);
        }

        @Override
        public void processElement(Action action, ReadOnlyContext ctx,
                                   Collector<Tuple2<String, Pattern>> out) throws Exception {
            Pattern pattern = ctx.getBroadcastState(
                    new MapStateDescriptor<>("patterns", Types.VOID, Types.POJO(Pattern.class))).get(null);

            String prevAction = prevActionState.value();
            if (pattern != null && prevAction != null) {
                // 如果前后两次行为都符合模式定义，输出一组匹配
                if (pattern.action1.equals(prevAction) && pattern.action2.equals(action.action)) {
                    out.collect(new Tuple2<>(ctx.getCurrentKey(), pattern));
                }
            }
            // 更新状态
            prevActionState.update(action.action);
        }
    }

    // 定义用户行为事件POJO类
    public static class Action {
        public String userId;
        public String action;

        public Action() {
        }

        public Action(String userId, String action) {
            this.userId = userId;
            this.action = action;
        }

        @Override
        public String toString() {
            return "Action{" +
                    "userId=" + userId +
                    ", action='" + action + '\'' +
                    '}';
        }
    }

    // 定义行为模式POJO类，包含先后发生的两个行为
    public static class Pattern {
        public String action1;
        public String action2;

        public Pattern() {
        }

        public Pattern(String action1, String action2) {
            this.action1 = action1;
            this.action2 = action2;
        }

        @Override
        public String toString() {
            return "Pattern{" +
                    "action1='" + action1 + '\'' +
                    ", action2='" + action2 + '\'' +
                    '}';
        }
    }
}
```

![image-20221225104229471](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221225104229471.png)

## 9.5 状态持久化和状态后端

  在 Flink 的状态管理机制中，很重要的一个功能就是对状态进行持久化（persistence）保存，这样就可以在发生故障后进行重启恢复。Flink 对状态进行持久化的方式，就是将当前所有分布式状态进行“快照”保存，写入一个“检查点”（checkpoint）或者保存点（savepoint）保存到外部存储系统中。具体的存储介质，一般是分布式文件系统（distributed file system）。

### 9.5.1 检查点（Checkpoint）

  有状态流应用中的检查点（checkpoint），其实就是所有任务的状态在某个时间点的一个快照（一份拷贝）。简单来讲，就是一次“存盘”，让我们之前处理数据的进度不要丢掉。在一个流应用程序运行时，Flink 会定期保存检查点，在检查点中会记录每个算子的 id 和状态；如果发生故障，Flink 就会用最近一次成功保存的检查点来恢复应用的状态，重新启动处理流程，就如同“读档”一样。

  如果保存检查点之后又处理了一些数据，然后发生了故障，那么重启恢复状态之后这些数据带来的状态改变会丢失。为了让最终处理结果正确，我们还需要让源（Source）算子重新读取这些数据，再次处理一遍。这就需要流的数据源具有“数据重放”的能力，一个典型的例子就是 Kafka，我们可以通过保存消费数据的偏移量、故障重启后重新提交来实现数据的重放。这是对“至少一次”（at least once）状态一致性的保证，如果希望实现“精确一次”（exactly once）的一致性，还需要数据写入外部系统时的相关保证。关于这部分内容我们会在第 10 章继续讨
论。

  默认情况下，检查点是被禁用的，需要在代码中手动开启。直接调用执行环境的.enableCheckpointing()方法就可以开启检查点。

```
StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();
env.enableCheckpointing(1000);
```

  这里传入的参数是检查点的间隔时间，单位为毫秒。关于检查点的详细配置，可以参考第10 章的内容。

  除了检查点之外，Flink 还提供了“保存点”（savepoint）的功能。保存点在原理和形式上跟检查点完全一样，也是状态持久化保存的一个快照；区别在于，保存点是自定义的镜像保存，所以不会由 Flink 自动创建，而需要用户手动触发。这在有计划地停止、重启应用时非常有用。

### 9.5.2 状态后端（State Backends）

  检查点的保存离不开 JobManager 和 TaskManager，以及外部存储系统的协调。在应用进行检查点保存时，首先会由 JobManager 向所有 TaskManager 发出触发检查点的命令；TaskManger 收到之后，将当前任务的所有状态进行快照保存，持久化到远程的存储介质中；完成之后向 JobManager 返回确认信息。这个过程是分布式的，当 JobManger 收到所有TaskManager 的返回信息后，就会确认当前检查点成功保存。而这一切工作的协调，就需要一个“专职人员”来完成。

![image-20221225104724938](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221225104724938.png)

  在 Flink 中，状态的存储、访问以及维护，都是由一个可插拔的组件决定的，这个组件就叫作状态后端（state backend）。状态后端主要负责两件事：一是本地的状态管理，二是将检查点（checkpoint）写入远程的持久化存储。

# 第 10 章 容错机制(116-126)

## 10.1 检查点（Checkpoint）

### 10.1.1 检查点的保存

### 10.1.2 从检查点恢复状态

### 10.1.3 检查点算法

### 10.1.4 检查点配置

### 10.1.5 保存点（Savepoint）

## 10.2 状态一致性

### 10.2.1 一致性的概念和级别

### 10.2.2 端到端的状态一致性

## 10.3 端到端精确一次（end-to-end exactly-once）

### 10.3.1 输入端保证

### 10.3.2 输出端保证

### 10.3.3 Flink 和 Kafka 连接时的精确一次保证



# 第 11 章 Table API 和 SQL(127-156)

## 11.1 快速上手

### 11.1.1 需要引入的依赖

```
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
```

  这里的依赖是一个 Java 的“桥接器”（bridge），主要就是负责 Table API 和下层 DataStream API 的连接支持，按照不同的语言分为 Java 版和 Scala 版。如果我们希望在本地的集成开发环境（IDE）里运行 Table API 和 SQL，还需要引入以下依赖：

```
<dependency>
 <groupId>org.apache.flink</groupId>
<artifactId>flink-table-planner-blink_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
```

  这里主要添加的依赖是一个“计划器”（planner），它是 Table API 的核心组件，负责提供运行时环境，并生成程序的执行计划。这里我们用到的是新版的 blink planner。由于 Flink 安装包的 lib 目录下会自带 planner，所以在生产集群环境中提交的作业不需要打包这个依赖。而在 Table API 的内部实现上，部分相关的代码是用 Scala 实现的，所以还需要额外添加一个 Scala 版流处理的相关依赖。

  另外，如果想实现自定义的数据格式来做序列化，可以引入下面的依赖：

```
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-table-common</artifactId>
 <version>${flink.version}</version>
</dependency>
```

### 11.1.2 一个简单示例

```java
import com.atguigu.chapter05.Event;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import static org.apache.flink.table.api.Expressions.$;


public class SimpleTableExample {
    public static void main(String[] args) throws Exception {
        // 获取流执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 1. 读取数据源
        SingleOutputStreamOperator<Event> eventStream = env
                .fromElements(
                        new Event("Alice", "./home", 1000L),
                        new Event("Bob", "./cart", 1000L),
                        new Event("Alice", "./prod?id=1", 5 * 1000L),
                        new Event("Cary", "./home", 60 * 1000L),
                        new Event("Bob", "./prod?id=3", 90 * 1000L),
                        new Event("Alice", "./prod?id=7", 105 * 1000L)
                );

        // 2. 获取表环境
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 3. 将数据流转换成表
        Table eventTable = tableEnv.fromDataStream(eventStream);

        // 4. 用执行SQL 的方式提取数据
        Table resultTable1 = tableEnv.sqlQuery("select url, user from " + eventTable);

        // 5. 基于Table直接转换
        Table resultTable2 = eventTable.select($("user"), $("url"))
                .where($("user").isEqual("Alice"));

        // 6. 将表转换成数据流，打印输出
        tableEnv.toDataStream(resultTable1).print("result1");
        tableEnv.toDataStream(resultTable2).print("result2");

        // 执行程序
        env.execute();
    }
}

```

![image-20221227103650787](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227103650787.png)

  可以看到，我们将原始的 Event 数据转换成了(url，user)这样类似二元组的类型。每行输出前面有一个“+I”标志，这是表示每条数据都是“插入”（Insert）到表中的新增数据。Table 是 Table API 中的核心接口类，对应着我们熟悉的“表”的概念。基于 Table 我们也可以调用一系列查询方法直接进行转换，这就是所谓 Table API 的处理方式：

```
// 用 Table API 方式提取数据
Table clickTable2 = eventTable.select($("url"), $("user"));
```

  这里的$符号是 Table API 中定义的“表达式”类 Expressions 中的一个方法，传入一个字段名称，就可以指代数据中对应字段。将得到的表转换成流打印输出，会发现结果与直接执行SQL 完全一样。

## 11.2 基本 API

### 11.2.1 程序架构

  在 Flink 中，Table API 和 SQL 可以看作联结在一起的一套 API，这套 API 的核心概念就是“表”（Table）。在我们的程序中，输入数据可以定义成一张表；然后对这张表进行查询，就可以得到新的表，这相当于就是流数据的转换操作；最后还可以定义一张用于输出的表，负责将处理结果写入到外部系统。

  我们可以看到，程序的整体处理流程与 DataStream API 非常相似，也可以分为读取数据源（Source）、转换（Transform）、输出数据（Sink）三部分；只不过这里的输入输出操作不需要额外定义，只需要将用于输入和输出的表定义出来，然后进行转换查询就可以了。程序基本架构如下：

```
// 创建表环境
TableEnvironment tableEnv = ...;

// 创建输入表，连接外部系统读取数据
tableEnv.executeSql("CREATE TEMPORARY TABLE inputTable ... WITH ( 'connector' = ... )");

// 注册一个表，连接到外部系统，用于输出
tableEnv.executeSql("CREATE TEMPORARY TABLE outputTable ... WITH ( 'connector' = ... )");

// 执行 SQL 对表进行查询转换，得到一个新的表
Table table1 = tableEnv.sqlQuery("SELECT ... FROM inputTable... ");

// 使用 Table API 对表进行查询转换，得到一个新的表
Table table2 = tableEnv.from("inputTable").select(...);

// 将得到的结果写入输出表
TableResult tableResult = table1.executeInsert("outputTable");
```

  与上一节中不同，这里不是从一个 DataStream 转换成 Table，而是通过执行 DDL 来直接创建一个表。这里执行的 CREATE 语句中用 WITH 指定了外部系统的连接器，于是就可以连接外部系统读取数据了。这其实是更加一般化的程序架构，因为这样我们就可以完全抛开DataStream API，直接用 SQL 语句实现全部的流处理过程。

  而后面对于输出表的定义是完全一样的。可以发现，在创建表的过程中，其实并不区分“输入”还是“输出”，只需要将这个表“注册”进来、连接到外部系统就可以了；这里的 inputTable、outputTable 只是注册的表名，并不代表处理逻辑，可以随意更换。至于表的具体作用，则要等到执行后面的查询转换操作时才能明确。我们直接从 inputTable 中查询数据，那么 inputTable就是输入表；而 outputTable 会接收另外表的结果进行写入，那么就是输出表。

  在早期的版本中，有专门的用于输入输出的 TableSource 和 TableSink，这与流处理里的概念是一一对应的；不过这种方式与关系型表和 SQL 的使用习惯不符，所以已被弃用，不再区分 Source 和 Sink。

### 11.2.2 创建表环境

  对于 Flink 这样的流处理框架来说，数据流和表在结构上还是有所区别的。所以使用 Table API 和 SQL 需要一个特别的运行时环境，这就是所谓的“表环境”（TableEnvironment）。它主要负责：

（1）注册 Catalog 和表；
（2）执行 SQL 查询；
（3）注册用户自定义函数（UDF）；
（4）DataStream 和表之间的转换。

  这里的 Catalog 就是“目录”，与标准 SQL 中的概念是一致的，主要用来管理所有数据库（database）和表（table）的元数据（metadata）。通过 Catalog 可以方便地对数据库和表进行查询的管理，所以可以认为我们所定义的表都会“挂靠”在某个目录下，这样就可以快速检索。在表环境中可以由用户自定义 Catalog，并在其中注册表和自定义函数（UDF）。默认的 Catalog就叫作 default_catalog。

  每个表和 SQL 的执行，都必须绑定在一个表环境（TableEnvironment）中。TableEnvironment是 Table API 中提供的基本接口类，可以通过调用静态的 create()方法来创建一个表环境实例。方法需要传入一个环境的配置参数 EnvironmentSettings，它可以指定当前表环境的执行模式和计划器（planner）。执行模式有批处理和流处理两种选择，默认是流处理模式；计划器默认使用 blink planner。

```
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;
308
EnvironmentSettings settings = EnvironmentSettings
 .newInstance()
 .inStreamingMode() // 使用流处理模式
 .build();
 
TableEnvironment tableEnv = TableEnvironment.create(settings);
```

对于流处理场景，其实默认配置就完全够用了。所以我们也可以用另一种更加简单的方式来创建表环境：

```
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);
```

  这 里 我 们 引 入 了 一 个 “ 流 式 表 环 境 ”（ StreamTableEnvironment ）， 它 是 继 承 自TableEnvironment 的子接口。调用它的 create()方法，只需要直接将当前的流执行环境（StreamExecutionEnvironment）传入，就可以创建出对应的流式表环境了。这也正是我们在上一节简单示例中使用的方式。

### 11.2.3 创建表

  表（Table）是我们非常熟悉的一个概念，它是关系型数据库中数据存储的基本形式，也是 SQL 执行的基本对象。Flink 中的表概念也并不特殊，是由多个“行”数据构成的，每个行（Row）又可以有定义好的多个列（Column）字段；整体来看，表就是固定类型的数据组成的二维矩阵。

  为了方便地查询表，表环境中会维护一个目录（Catalog）和表的对应关系。所以表都是通过 Catalog 来进行注册创建的。表在环境中有一个唯一的 ID，由三部分组成：目录（catalog）名，数据库（database）名，以及表名。在默认情况下，目录名为 default_catalog，数据库名为default_database。所以如果我们直接创建一个叫作 MyTable 的表，它的 ID 就是：

```
default_catalog.default_database.MyTable
```

具体创建表的方式，有通过连接器（connector）和虚拟表（virtual tables）两种。

**1.连接器表（Connector Tables）**

  最直观的创建表的方式，就是通过连接器（connector）连接到一个外部系统，然后定义出对应的表结构。例如我们可以连接到 Kafka 或者文件系统，将存储在这些外部系统的数据以“表”的形式定义出来，这样对表的读写就可以通过连接器转换成对外部系统的读写了。当我们在表环境中读取这张表，连接器就会从外部系统读取数据并进行转换；而当我们向这张表写入数据，连接器就会将数据输出（Sink）到外部系统中。

  在代码中，我们可以调用表环境的 executeSql()方法，可以传入一个 DDL 作为参数执行SQL 操作。这里我们传入一个 CREATE 语句进行表的创建，并通过 WITH 关键字指定连接到外部系统的连接器：

```
tableEnv.executeSql("CREATE [TEMPORARY] TABLE MyTable ... WITH ( 'connector' = ... )");
```

  这里的 TEMPORARY 关键字可以省略。关于连接器的具体定义，我们会在 11.8 节中展开讲解。

  这里没有定义 Catalog 和 Database ， 所 以 都 是 默 认 的 ， 表 的 完 整 ID 就 是
default_catalog.default_database.MyTable。如果希望使用自定义的目录名和库名，可以在环境中
进行设置：

```
tEnv.useCatalog("custom_catalog");
tEnv.useDatabase("custom_database");
```

  这样我们创建的表完整 ID 就变成了 custom_catalog.custom_database.MyTable。之后在表
  环境中创建的所有表，ID 也会都以 custom_catalog.custom_database 作为前缀。

**2.虚拟表（Virtual Tables）**

在环境中注册之后，我们就可以在 SQL 中直接使用这张表进行查询转换了。

```
Table newTable = tableEnv.sqlQuery("SELECT ... FROM MyTable... ");
```

  这里调用了表环境的 sqlQuery()方法，直接传入一条 SQL 语句作为参数执行查询，得到的结果是一个 Table 对象。Table 是 Table API 中提供的核心接口类，就代表了一个 Java 中定义的表实例。

  得到的 newTable 是一个中间转换结果，如果之后又希望直接使用这个表执行 SQL，又该怎么做呢？由于 newTable 是一个 Table 对象，并没有在表环境中注册；所以我们还需要将这个中间结果表注册到环境中，才能在 SQL 中使用：

```
tableEnv.createTemporaryView("NewTable", newTable);
```

  我们发现，这里的注册其实是创建了一个“虚拟表”（Virtual Table）。这个概念与 SQL 语法中的视图（View）非常类似，所以调用的方法也叫作创建“虚拟视图”（createTemporaryView）。视图之所以是“虚拟”的，是因为我们并不会直接保存这个表的内容，并没有“实体”；只是在用到这张表的时候，会将它对应的查询语句嵌入到 SQL 中。
  注册为虚拟表之后，我们就又可以在 SQL 中直接使用 NewTable 进行查询转换了。不难看到，通过虚拟表可以非常方便地让 SQL 分步骤执行得到中间结果，这为代码编写提供了很大的便利。

  另外，虚拟表也可以让我们在 Table API 和 SQL 之间进行自由切换。一个 Java 中的 Table对象可以直接调用 Table API 中定义好的查询转换方法，得到一个中间结果表；这跟对注册好的表直接执行 SQL 结果是一样的。具体我们会在下一小节继续讲解。

### 11.2.4 表的查询

**1.执行 SQL 进行查询**

  基于表执行 SQL 语句，是我们最为熟悉的查询方式。Flink 基于 Apache Calcite 来提供对SQL 的支持，Calcite 是一个为不同的计算平台提供标准 SQL 查询的底层工具，很多大数据框架比如 Apache Hive、Apache Kylin 中的 SQL 支持都是通过集成 Calcite 来实现的。

  在代码中，我们只要调用表环境的 sqlQuery()方法，传入一个字符串形式的 SQL 查询语句就可以了。执行得到的结果，是一个 Table 对象。

```
// 创建表环境
TableEnvironment tableEnv = ...; 
// 创建表
tableEnv.executeSql("CREATE TABLE EventTable ... WITH ( 'connector' = ... )");
// 查询用户 Alice 的点击事件，并提取表中前两个字段
Table aliceVisitTable = tableEnv.sqlQuery(
 "SELECT user, url " +
 "FROM EventTable " +
 "WHERE user = 'Alice' "
 );
```

  目前 Flink 支持标准 SQL 中的绝大部分用法，并提供了丰富的计算函数。这样我们就可以把已有的技术迁移过来，像在 MySQL、Hive 中那样直接通过编写 SQL 实现自己的处理需求，从而大大降低了 Flink 上手的难度。

例如，我们也可以通过 GROUP BY 关键字定义分组聚合，调用 COUNT()、SUM()这样的函数来进行统计计算：

```
Table urlCountTable = tableEnv.sqlQuery(
 "SELECT user, COUNT(url) " +
 "FROM EventTable " +
 "GROUP BY user "
 );
```

  上面的例子得到的是一个新的 Table 对象，我们可以再次将它注册为虚拟表继续在 SQL中调用。另外，我们也可以直接将查询的结果写入到已经注册的表中，这需要调用表环境的executeSql()方法来执行 DDL，传入的是一个 INSERT 语句：

```
// 注册表
tableEnv.executeSql("CREATE TABLE EventTable ... WITH ( 'connector' = ... )");
tableEnv.executeSql("CREATE TABLE OutputTable ... WITH ( 'connector' = ... )");

// 将查询结果输出到 OutputTable 中
tableEnv.executeSql (
"INSERT INTO OutputTable " +
 "SELECT user, url " +
 "FROM EventTable " +
 "WHERE user = 'Alice' "
 );
```

**2.调用 Table API 进行查询**

  另外一种查询方式就是调用 Table API。这是嵌入在 Java 和 Scala 语言内的查询 API，核心就是 Table 接口类，通过一步步链式调用 Table 的方法，就可以定义出所有的查询转换操作。每一步方法调用的返回结果，都是一个 Table。

  由于Table API是基于Table的Java实例进行调用的，因此我们首先要得到表的Java对象。基于环境中已注册的表，可以通过表环境的 from()方法非常容易地得到一个 Table 对象：

```
Table eventTable = tableEnv.from("EventTable");
```

传入的参数就是注册好的表名。注意这里 eventTable 是一个 Table 对象，而 EventTable 是在环境中注册的表名。得到 Table 对象之后，就可以调用 API 进行各种转换操作了，得到的是一个新的 Table 对象：

```
Table maryClickTable = eventTable
 .where($("user").isEqual("Alice"))
 .select($("url"), $("user"));
```

  这里每个方法的参数都是一个“表达式”（Expression），用方法调用的形式直观地说明了想要表达的内容；“$”符号用来指定表中的一个字段。上面的代码和直接执行 SQL 是等效的。

  Table API 是嵌入编程语言中的 DSL，SQL 中的很多特性和功能必须要有对应的实现才可以使用，因此跟直接写 SQL 比起来肯定就要麻烦一些。目前 Table API 支持的功能相对更少，可以预见未来 Flink 社区也会以扩展 SQL 为主，为大家提供更加通用的接口方式；所以我们接下来也会以介绍 SQL 为主，简略地提及 Table API。

**3.两种 API 的结合使用**

  可以发现，无论是调用 Table API 还是执行 SQL，得到的结果都是一个 Table 对象；所以这两种 API 的查询可以很方便地结合在一起。

（1）无论是那种方式得到的 Table 对象，都可以继续调用 Table API 进行查询转换；
（2）如果想要对一个表执行 SQL 操作（用 FROM 关键字引用），必须先在环境中对它进行注册。所以我们可以通过创建虚拟表的方式实现两者的转换：

```
tableEnv.createTemporaryView("MyTable", myTable);
```

注意：这里的第一个参数"MyTable"是注册的表名，而第二个参数 myTable 是 Java 中的Table 对象。

另外要说明的是，在 11.1.2 小节的简单示例中，我们并没有将 Table 对象注册为虚拟表就直接在 SQL 中使用了：

```
Table clickTable = tableEnvironment.sqlQuery("select url, user from " + eventTable);
```

  这其实是一种简略的写法，我们将 Table 对象名 eventTable 直接以字符串拼接的形式添加到 SQL 语句中，在解析时会自动注册一个同名的虚拟表到环境中，这样就省略了创建虚拟视图的步骤。

  两种 API 殊途同归，实际应用中可以按照自己的习惯任意选择。不过由于结合使用容易引起混淆，而 Table API 功能相对较少、通用性较差，所以企业项目中往往会直接选择 SQL 的方式来实现需求。

### 11.2.5 输出表

  表的创建和查询，就对应着流处理中的读取数据源（Source）和转换（Transform）；而最后一个步骤 Sink，也就是将结果数据输出到外部系统，就对应着表的输出操作。

  在代码上，输出一张表最直接的方法，就是调用 Table 的方法 executeInsert()方法将一个Table 写入到注册过的表中，方法传入的参数就是注册的表名。

```
// 注册表，用于输出数据到外部系统
tableEnv.executeSql("CREATE TABLE OutputTable ... WITH ( 'connector' = ... )");
// 经过查询转换，得到结果表
Table result = ...
// 将结果表写入已注册的输出表中
result.executeInsert("OutputTable");
```

  在底层，表的输出是通过将数据写入到 TableSink 来实现的。TableSink 是 Table API 中提供的一个向外部系统写入数据的通用接口，可以支持不同的文件格式（比如 CSV、Parquet）、存储数据库（比如 JDBC、HBase、Elasticsearch）和消息队列（比如 Kafka）。它有些类似于DataStream API 中调用 addSink()方法时传入的 SinkFunction，有不同的连接器对它进行了实现。关于不同外部系统的连接器，我们会在 11.8 节展开介绍。

  这里可以发现，我们在环境中注册的“表”，其实在写入数据的时候就对应着一个 TableSink。

### 11.2.6 表和流的转换

**1.将表（Table）转换成流（DataStream）**

**（1）调用 toDataStream()方法**

  将一个 Table 对象转换成 DataStream 非常简单，只要直接调用表环境的方法 toDataStream()就可以了。例如，我们可以将 11.2.4 小节经查询转换得到的表 maryClickTable 转换成流打印输出，这代表了“Mary 点击的 url 列表”：

```
Table aliceVisitTable = tableEnv.sqlQuery(
 "SELECT user, url " +
 "FROM EventTable " +
 "WHERE user = 'Alice' "
 );
// 将表转换成数据流
tableEnv.toDataStream(aliceVisitTable).print();
```

这里需要将要转换的 Table 对象作为参数传入。

**（2）调用 toChangelogStream()方法**

  将 maryClickTable 转换成流打印输出是很简单的；然而，如果我们同样希望将“用户点击次数统计”表urlCountTable 进行打印输出，就会抛出一个 TableException 异常：

```
Exception in thread "main" org.apache.flink.table.api.TableException: Table sink 
'default_catalog.default_database.Unregistered_DataStream_Sink_1' doesn't 
support consuming update changes ...
```

这表示当前的 TableSink 并不支持表的更新（update）操作。这是什么意思呢？

  因为 print 本身也可以看作一个 Sink 操作，所以这个异常就是说打印输出的 Sink 操作不支持对数据进行更新。具体来说，urlCountTable 这个表中进行了分组聚合统计，所以表中的每一行是会“更新”的。也就是说，Alice 的第一个点击事件到来，表中会有一行(Alice, 1)；第二个点击事件到来，这一行就要更新为(Alice, 2)。但之前的(Alice, 1)已经打印输出了，“覆水难收”，我们怎么能对它进行更改呢？所以就会抛出异常。

  解决的思路是，对于这样有更新操作的表，我们不要试图直接把它转换成 DataStream 打印输出，而是记录一下它的“更新日志”（change log）。这样一来，对于表的所有更新操作，就变成了一条更新日志的流，我们就可以转换成流打印输出了。代码中需要调用的是表环境的 toChangelogStream()方法：

```
Table urlCountTable = tableEnv.sqlQuery(
 "SELECT user, COUNT(url) " +
 "FROM EventTable " +
 "GROUP BY user "
 );
// 将表转换成更新日志流
tableEnv.toDataStream(urlCountTable).print();
```

与“更新日志流”（Changelog Streams）对应的，是那些只做了简单转换、没有进行聚合统计的表，例如前面提到的 maryClickTable。它们的特点是数据只会插入、不会更新，所以也被叫作“仅插入流”（Insert-Only Streams）。

**2.将流（DataStream）转换成表（Table）**

**（1）调用 fromDataStream()方法**
  想要将一个 DataStream 转换成表也很简单，可以通过调用表环境的 fromDataStream()方法来实现，返回的就是一个 Table 对象。例如，我们可以直接将事件流 eventStream 转换成一个表：

```
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
// 获取表环境
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// 读取数据源
SingleOutputStreamOperator<Event> eventStream = env.addSource(...)

// 将数据流转换成表
Table eventTable = tableEnv.fromDataStream(eventStream);
```

  由于流中的数据本身就是定义好的 POJO 类型 Event，所以我们将流转换成表之后，每一行数据就对应着一个 Event，而表中的列名就对应着 Event 中的属性。

  另外，我们还可以在 fromDataStream()方法中增加参数，用来指定提取哪些属性作为表中的字段名，并可以任意指定位置：

```
// 提取 Event 中的 timestamp 和 url 作为表中的列
Table eventTable2 = tableEnv.fromDataStream(eventStream, $("timestamp"), 
$("url"));
```

  需要注意的是，timestamp 本身是 SQL 中的关键字，所以我们在定义表名、列名时要尽量避免。这时可以通过表达式的 as()方法对字段进行重命名：

```
// 将 timestamp 字段重命名为 ts
Table eventTable2 = tableEnv.fromDataStream(eventStream, $("timestamp").as("ts"), 
$("url"));
```

**（2）调用 createTemporaryView()方法**

  调用 fromDataStream()方法简单直观，可以直接实现 DataStream 到 Table 的转换；不过如果我们希望直接在 SQL 中引用这张表，就还需要调用表环境的 createTemporaryView()方法来创建虚拟视图了。

  对于这种场景，也有一种更简洁的调用方式。我们可以直接调用 createTemporaryView()方法创建虚拟表，传入的两个参数，第一个依然是注册的表名，而第二个可以直接就是DataStream。之后仍旧可以传入多个参数，用来指定表中的字段

```
tableEnv.createTemporaryView("EventTable", eventStream, 
$("timestamp").as("ts"),$("url"));
```

这样，我们接下来就可以直接在 SQL 中引用表 EventTable 了。

**（3）调用 fromChangelogStream ()方法**
  表环境还提供了一个方法 fromChangelogStream()，可以将一个更新日志流转换成表。这个方法要求流中的数据类型只能是 Row，而且每一个数据都需要指定当前行的更新类型（RowKind）；所以一般是由连接器帮我们实现的，直接应用比较少见，感兴趣的读者可以查看官网的文档说明。

**3.支持的数据类型**

  前面示例中的 DataStream，流中的数据类型都是定义好的 POJO 类。如果 DataStream 中的类型是简单的基本类型，还可以直接转换成表吗？这就涉及了 Table 中支持的数据类型。整体来看，DataStream 中支持的数据类型，Table 中也是都支持的，只不过在进行转换时需要注意一些细节。
**（1）原子类型**
  在 Flink 中，基础数据类型（Integer、Double、String）和通用数据类型（也就是不可再拆分的数据类型）统一称作“原子类型”。原子类型的 DataStream，转换之后就成了只有一列的Table，列字段（field）的数据类型可以由原子类型推断出。另外，还可以在 fromDataStream()方法里增加参数，用来重新命名列字段。

```
StreamTableEnvironment tableEnv = ...;

DataStream<Long> stream = ...;

// 将数据流转换成动态表，动态表只有一个字段，重命名为 myLong
Table table = tableEnv.fromDataStream(stream, $("myLong"));
```

**（2）Tuple 类型**
  当原子类型不做重命名时，默认的字段名就是“f0”，容易想到，这其实就是将原子类型看作了一元组 Tuple1 的处理结果。
  Table 支持 Flink 中定义的元组类型 Tuple，对应在表中字段名默认就是元组中元素的属性名 f0、f1、f2...。所有字段都可以被重新排序，也可以提取其中的一部分字段。字段还可以通过调用表达式的 as()方法来进行重命名。

```
StreamTableEnvironment tableEnv = ...;
DataStream<Tuple2<Long, Integer>> stream = ...;

// 将数据流转换成只包含 f1 字段的表
Table table = tableEnv.fromDataStream(stream, $("f1"));

// 将数据流转换成包含 f0 和 f1 字段的表，在表中 f0 和 f1 位置交换
Table table = tableEnv.fromDataStream(stream, $("f1"), $("f0"));

// 将 f1 字段命名为 myInt，f0 命名为 myLong
Table table = tableEnv.fromDataStream(stream, $("f1").as("myInt"), 
$("f0").as("myLong"));
```

**（3）POJO 类型**
  Flink 也支持多种数据类型组合成的“复合类型”，最典型的就是简单 Java 对象（POJO 类
型）。由于 POJO 中已经定义好了可读性强的字段名，这种类型的数据流转换成 Table 就显得
无比顺畅了。
  将 POJO 类型的 DataStream 转换成 Table，如果不指定字段名称，就会直接使用原始 POJO 
类型中的字段名称。POJO 中的字段同样可以被重新排序、提却和重命名，这在之前的例子中
已经有过体现。

```
StreamTableEnvironment tableEnv = ...;

DataStream<Event> stream = ...;

Table table = tableEnv.fromDataStream(stream);
Table table = tableEnv.fromDataStream(stream, $("user"));
Table table = tableEnv.fromDataStream(stream, $("user").as("myUser"), 
$("url").as("myUrl"));
```

**（4）Row 类型**
  Flink 中还定义了一个在关系型表中更加通用的数据类型——行（Row），它是 Table 中数据的基本组织形式。Row 类型也是一种复合类型，它的长度固定，而且无法直接推断出每个字段的类型，所以在使用时必须指明具体的类型信息；我们在创建 Table 时调用的 CREATE语句就会将所有的字段名称和类型指定，这在 Flink 中被称为表的“模式结构”（Schema）。除此之外，Row 类型还附加了一个属性 RowKind，用来表示当前行在更新操作中的类型。这样，Row 就可以用来表示更新日志流（changelog stream）中的数据，从而架起了 Flink 中流和表的
转换桥梁。
  所以在更新日志流中，元素的类型必须是 Row，而且需要调用 ofKind()方法来指定更新类型。下面是一个具体的例子：

```
DataStream<Row> dataStream =
 env.fromElements(
 Row.ofKind(RowKind.INSERT, "Alice", 12),
 Row.ofKind(RowKind.INSERT, "Bob", 5),
 Row.ofKind(RowKind.UPDATE_BEFORE, "Alice", 12),
 Row.ofKind(RowKind.UPDATE_AFTER, "Alice", 100));
 
// 将更新日志流转换为表
Table table = tableEnv.fromChangelogStream(dataStream);
```

**4.综合应用示例**

  现在，我们可以将介绍过的所有 API 整合起来，写出一段完整的代码。同样还是用户的一组点击事件，我们可以查询出某个用户（例如 Alice）点击的 url 列表，也可以统计出每个用户累计的点击次数，这可以用两句 SQL 来分别实现。具体代码如下：

```java
import com.atguigu.chapter05.Event;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;


public class TableToStreamExample {
    public static void main(String[] args) throws Exception {
        // 获取流环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 读取数据源
        SingleOutputStreamOperator<Event> eventStream = env
                .fromElements(
                        new Event("Alice", "./home", 1000L),
                        new Event("Bob", "./cart", 1000L),
                        new Event("Alice", "./prod?id=1", 5 * 1000L),
                        new Event("Cary", "./home", 60 * 1000L),
                        new Event("Bob", "./prod?id=3", 90 * 1000L),
                        new Event("Alice", "./prod?id=7", 105 * 1000L)
                );

        // 获取表环境
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 将数据流转换成表
        tableEnv.createTemporaryView("EventTable", eventStream);


        // 查询Alice的访问url列表
        Table aliceVisitTable = tableEnv.sqlQuery("SELECT url, user FROM EventTable WHERE user = 'Alice'");

        // 统计每个用户的点击次数
        Table urlCountTable = tableEnv.sqlQuery("SELECT user, COUNT(url) FROM EventTable GROUP BY user");

        // 将表转换成数据流，在控制台打印输出
        tableEnv.toDataStream(aliceVisitTable).print("alice visit");
        tableEnv.toChangelogStream(urlCountTable).print("count");

        // 执行程序
        env.execute();
    }
}


```

![image-20221227115420192](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227115420192.png)

  这里数据的前缀出现了+I、-U 和+U 三种 RowKind，分别表示 INSERT（插入）、UPDATE_BEFORE（更新前）和 UPDATE_AFTER（更新后）。当收到每个用户的第一次点击事件时，会在表中插入一条数据，例如+I[Alice, 1]、+I[Bob, 1]。而之后每当用户增加一次点击事件，就会带来一次更新操作，更新日志流（changelog stream）中对应会出现两条数据，分别表示之前数据的失效和新数据的生效；例如当 Alice 的第二条点击数据到来时，会出现一个-U[Alice, 1]和一个+U[Alice, 2]，表示 Alice 的点击个数从 1 变成了 2。

  这种表示更新日志的方式，有点像是声明“撤回”了之前的一条数据、再插入一条更新后的数据，所以也叫作“撤回流”（Retract Stream）。关于表到流转换过程的编码方式，我们会在下节进行更深入的讨论。

## 11.3 流处理中的表

  上一节中介绍了 Table API 和 SQL 的基本使用方法。我们会发现，在 Flink 中使用表和 SQL基本上跟其它场景是一样的；不过对于表和流的转换，却稍显复杂。当我们将一个 Table 转换成 DataStream 时，有“仅插入流”（Insert-Only Streams）和“更新日志流”（Changelog Streams）两种不同的方式，具体使用哪种方式取决于表中是否存在更新（update）操作。

  这种麻烦其实是不可避免的。我们知道，Table API 和 SQL 本质上都是基于关系型表的操作方式；而关系型表（Table）本身是有界的，更适合批处理的场景。所以在 MySQL、Hive这样的固定数据集中进行查询，使用 SQL 就会显得得心应手。而对于 Flink 这样的流处理框架来说，要处理的是源源不断到来的无界数据流，我们无法等到数据都到齐再做查询，每来一条数据就应该更新一次结果；这时如果一定要使用表和 SQL 进行处理，就会显得有些别扭了，需要引入一些特殊的概念。

### 11.3.1 动态表和持续查询

  流处理面对的数据是连续不断的，这导致了流处理中的“表”跟我们熟悉的关系型数据库中的表完全不同；而基于表执行的查询操作，也就有了新的含义。
  如果我们希望把流数据转换成表的形式，那么这表中的数据就会不断增长；如果进一步基于表执行 SQL 查询，那么得到的结果就不是一成不变的，而是会随着新数据的到来持续更新。

**1.动态表（Dynamic Tables）**

  当流中有新数据到来，初始的表中会插入一行；而基于这个表定义的 SQL 查询，就应该在之前的基础上更新结果。这样得到的表就会不断地动态变化，被称为“动态表”（Dynamic Tables）。
动态表是Flink在Table API和SQL中的核心概念，它为流数据处理提供了表和SQL支持。我们所熟悉的表一般用来做批处理，面向的是固定的数据集，可以认为是“静态表”；而动态表则完全不同，它里面的数据会随时间变化。
  其实动态表的概念，我们在传统的关系型数据库中已经有所接触。数据库中的表，其实是一系列 INSERT、UPDATE 和 DELETE 语句执行的结果；在关系型数据库中，我们一般把它称为更新日志流（changelog stream）。如果我们保存了表在某一时刻的快照（snapshot），那么接下来只要读取更新日志流，就可以得到表之后的变化过程和最终结果了。在很多高级关系型数据库（比如 Oracle、DB2）中都有“物化视图”（Materialized Views）的概念，可以用来缓存 SQL 查询的结果；它的更新其实就是不停地处理更新日志流的过程。
Flink 中的动态表，就借鉴了物化视图的思想。

**2.持续查询（Continuous Query）**

  动态表可以像静态的批处理表一样进行查询操作。由于数据在不断变化，因此基于它定义的 SQL 查询也不可能执行一次就得到最终结果。这样一来，我们对动态表的查询也就永远不会停止，一直在随着新数据的到来而继续执行。这样的查询就被称作“持续查询”（Continuous Query）。对动态表定义的查询操作，都是持续查询；而持续查询的结果也会是一个动态表。

  由于每次数据到来都会触发查询操作，因此可以认为一次查询面对的数据集，就是当前输入动态表中收到的所有数据。这相当于是对输入动态表做了一个“快照”（snapshot），当作有限数据集进行批处理；流式数据的到来会触发连续不断的快照查询，像动画一样连贯起来，就构成了“持续查询”。

![image-20221227152757354](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227152757354.png)

**持续查询的步骤如下：**

（1）流（stream）被转换为动态表（dynamic table）；

（2）对动态表进行持续查询（continuous query），生成新的动态表；

（3）生成的动态表被转换成流。

这样，只要 API 将流和动态表的转换封装起来，我们就可以直接在数据流上执行 SQL 查询，用处理表的方式来做流处理了。

### 11.3.2 将流转换成动态表

  为了能够使用 SQL 来做流处理，我们必须先把流（stream）转换成动态表。当然，之前在讲解基本 API 时，已经介绍过代码中的 DataStream 和 Table 如何转换；现在我们则要抛开具体的数据类型，从原理上理解流和动态表的转换过程。

  如果把流看作一张表，那么流中每个数据的到来，都应该看作是对表的一次插入（Insert）操作，会在表的末尾添加一行数据。因为流是连续不断的，而且之前的输出结果无法改变、只能在后面追加；所以我们其实是通过一个只有插入操作（insert-only）的更新日志（changelog）流，来构建一个表。

```java
// 获取流环境
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(1);
// 读取数据源
SingleOutputStreamOperator<Event> eventStream = env
 .fromElements(
     new Event("Alice", "./home", 1000L),
     new Event("Bob", "./cart", 1000L),
     new Event("Alice", "./prod?id=1", 5 * 1000L),
     new Event("Cary", "./home", 60 * 1000L),
     new Event("Bob", "./prod?id=3", 90 * 1000L),
     new Event("Alice", "./prod?id=7", 105 * 1000L)
 );
 
// 获取表环境
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// 将数据流转换成表
tableEnv.createTemporaryView("EventTable", eventStream, $("user"), $("url"), 
$("timestamp").as("ts"));
 
// 统计每个用户的点击次数
Table urlCountTable = tableEnv.sqlQuery("SELECT user, COUNT(url) as cnt FROM EventTable GROUP BY user");

// 将表转换成数据流，在控制台打印输出
tableEnv.toChangelogStream(urlCountTable).print("count");
 
// 执行程序
env.execute();
```

&emsp;&emsp;当用户点击事件到来时，就对应着动态表中的一次插入（Insert）操作，每条数据就是表中的一行；随着插入更多的点击事件，得到的动态表将不断增长。

![image-20221227153305493](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227153305493.png)

### 11.3.2 用 SQL 持续查询

**1.更新（Update）查询**

我们在代码中定义了一个 SQL 查询。

```
Table urlCountTable = tableEnv.sqlQuery("SELECT user, COUNT(url) as cnt FROM EventTable GROUP BY user");
```

  这个查询很简单，主要是分组聚合统计每个用户的点击次数。我们把原始的动态表注册为EventTable，经过查询转换后得到 urlCountTable；这个结果动态表中包含两个字段，具体定义

如下：

```
[
 user: VARCHAR, // 用户名
 cnt: BIGINT // 用户访问 url 的次数
]
```

  当原始动态表不停地插入新的数据时，查询得到的 urlCountTable 会持续地进行更改。由于 count 数量可能会叠加增长，因此这里的更改操作可以是简单的插入（Insert），也可以是对之前数据的更新（Update）。换句话说，用来定义结果表的更新日志（chanzgelog）流中，包含了 INSERT 和 UPDATE 两种操作。这种持续查询被称为更新查询（Update Query），更新查询得到的结果表如果想要转换成 DataStream，必须调用 toChangelogStream()方法。

![image-20221227153631631](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227153631631.png)

具体步骤解释如下：
（1）当查询启动时，原始动态表 EventTable 为空；

（2）当第一行 Alice 的点击数据插入 EventTable 表时，查询开始计算结果表，urlCountTable中插入一行数据[Alice，1]。

（3）当第二行 Bob 点击数据插入 EventTable 表时，查询将更新结果表并插入新行[Bob，1]。

（4）第三行数据到来，同样是 Alice 的点击事件，这时不会插入新行，而是生成一个针对已有行的更新操作。这样，结果表中第一行[Alice，1]就更新为[Alice，2]。

（5）当第四行 Cary 的点击数据插入到 EventTable 表时，查询将第三行[Cary，1]插入到结果表中。

**2.追加（Append）查询**

  上面的例子中，查询过程用到了分组聚合，结果表中就会产生更新操作。如果我们执行一个简单的条件查询，结果表中就会像原始表 EventTable 一样，只有插入（Insert）操作了。

```
Table aliceVisitTable = tableEnv.sqlQuery("SELECT url, user FROM EventTable WHERE user = 'Cary'");
```

  这样的持续查询，就被称为追加查询（Append Query），它定义的结果表的更新日志（changelog）流中只有 INSERT 操作。追加查询得到的结果表，转换成 DataStream 调用方法没有限制，可以直接用 toDataStream()，也可以像更新查询一样调用 toChangelogStream()。

  这样看来，我们似乎可以总结一个规律：只要用到了聚合，在之前的结果上有叠加，就会产生更新操作，就是一个更新查询。但事实上，更新查询的判断标准是结果表中的数据是否会有 UPDATE 操作，如果聚合的结果不再改变，那么同样也不是更新查询。

  什么时候聚合的结果会保持不变呢？一个典型的例子就是窗口聚合。

  我们考虑开一个滚动窗口，统计每一小时内所有用户的点击次数，并在结果表中增加一个endT 字段，表示当前统计窗口的结束时间。这时结果表的字段定义如下：

```
[
 user: VARCHAR, // 用户名
 endT: TIMESTAMP, // 窗口结束时间
 cnt: BIGINT // 用户访问 url 的次数
]
```

  如图 11-5 所示，与之前的分组聚合一样，当原始动态表不停地插入新的数据时，查询得到的结果 result 会持续地进行更改。比如时间戳在 12:00:00 到 12:59:59 之间的有四条数据，其中 Alice 三次点击、Bob 一次点击；所以当水位线达到 13:00:00 时窗口关闭，输出到结果表中的就是新增两条数据[Alice, 13:00:00, 3]和[Bob, 13:00:00, 1]。同理，当下一小时的窗口关闭时，也会将统计结果追加到 result 表后面，而不会更新之前的数据。

![image-20221227154300758](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227154300758.png)

  所以我们发现，由于窗口的统计结果是一次性写入结果表的，所以结果表的更新日志流中只会包含插入 INSERT 操作，而没有更新 UPDATE 操作。所以这里的持续查询，依然是一个追加（Append）查询。结果表 result 如果转换成 DataStream，可以直接调用 toDataStream()方法。

  需要注意的是，由于涉及时间窗口，我们还需要为事件时间提取时间戳和生成水位线。完整代码如下：

```java
import com.atguigu.chapter05.Event;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

import static org.apache.flink.table.api.Expressions.$;

public class CumulateWindowExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 读取数据源，并分配时间戳、生成水位线
        SingleOutputStreamOperator<Event> eventStream = env
                .fromElements(
                        new Event("Alice", "./home", 1000L),
                        new Event("Bob", "./cart", 1000L),
                        new Event("Alice", "./prod?id=1", 25 * 60 * 1000L),
                        new Event("Alice", "./prod?id=4", 55 * 60 * 1000L),
                        new Event("Bob", "./prod?id=5", 3600 * 1000L + 60 * 1000L),
                        new Event("Cary", "./home", 3600 * 1000L + 30 * 60 * 1000L),
                        new Event("Cary", "./prod?id=7", 3600 * 1000L + 59 * 60 * 1000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<Event>forMonotonousTimestamps()
                                .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                                    @Override
                                    public long extractTimestamp(Event element, long recordTimestamp) {
                                        return element.timestamp;
                                    }
                                })
                );

        // 创建表环境
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 将数据流转换成表，并指定时间属性
        Table eventTable = tableEnv.fromDataStream(
                eventStream,
                $("user"),
                $("url"),
                $("timestamp").rowtime().as("ts")
        );

        // 为方便在SQL中引用，在环境中注册表EventTable
        tableEnv.createTemporaryView("EventTable", eventTable);

        // 设置累积窗口，执行SQL统计查询
        Table result = tableEnv
                .sqlQuery(
                        "SELECT " +
                                "user, " +
                                "window_end AS endT, " +
                                "COUNT(url) AS cnt " +
                                "FROM TABLE( " +
                                "CUMULATE( TABLE EventTable, " +    // 定义累积窗口
                                "DESCRIPTOR(ts), " +
                                "INTERVAL '30' MINUTE, " +
                                "INTERVAL '1' HOUR)) " +
                                "GROUP BY user, window_start, window_end "
                );

        tableEnv.toDataStream(result).print();

        env.execute();
    }
}
```

![image-20221227154637939](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227154637939.png)

  可以看到，所有输出结果都以+I 为前缀，表示都是以 INSERT 操作追加到结果表中的；这是一个追加查询，所以我们直接使用 toDataStream()转换成流是没有问题的。这里输出的window_end 是一个 TIMESTAMP 类型；由于我们直接以一个长整型数作为事件发生的时间戳，所以可以看到对应的都是 1970 年 1 月 1 日的时间。

**3.查询限制**

  在实际应用中，有些持续查询会因为计算代价太高而受到限制。所谓的“代价太高”，可能是由于需要维护的状态持续增长，也可能是由于更新数据的计算太复杂。
⚫ 状态大小
  用持续查询做流处理，往往会运行至少几周到几个月；所以持续查询处理的数据总量可能非常大。例如我们之前举的更新查询的例子，需要记录每个用户访问 url 的次数。如果随着时间的推移用户数越来越大，那么要维护的状态也将逐渐增长，最终可能会耗尽存储空间导致查询失败。

```
SELECT user, COUNT(url)
FROM clicks
GROUP BY user;
```

⚫ 更新计算
  对于有些查询来说，更新计算的复杂度可能很高。每来一条新的数据，更新结果的时候可能需要全部重新计算，并且对很多已经输出的行进行更新。一个典型的例子就是 RANK()函数，它会基于一组数据计算当前值的排名。例如下面的 SQL 查询，会根据用户最后一次点击的时间为每个用户计算一个排名。当我们收到一个新的数据，用户的最后一次点击时间（lastAction）就会更新，进而所有用户必须重新排序计算一个新的排名。当一个用户的排名发生改变时，被他超过的那些用户的排名也会改变；这样的更新操作无疑代价巨大，而且还会随着用户的增多越来越严重。

```
SELECT user, RANK() OVER (ORDER BY lastAction)
FROM (
 SELECT user, MAX(ts) AS lastAction FROM EventTable GROUP BY user
);
```

  这样的查询操作，就不太适合作为连续查询在流处理中执行。这里 RANK()的使用要配合一个 OVER 子句，这是所谓的“开窗聚合”，我们会在 11.5 节展开介绍。

### 11.3.3 将动态表转换为流

  与关系型数据库中的表一样，动态表也可以通过插入（Insert）、更新（Update）和删除（Delete）操作，进行持续的更改。将动态表转换为流或将其写入外部系统时，就需要对这些更改操作进行编码，通过发送编码消息的方式告诉外部系统要执行的操作。在 Flink 中，Table API 和 SQL支持三种编码方式：

⚫ 仅追加（Append-only）流
  仅通过插入（Insert）更改来修改的动态表，可以直接转换为“仅追加”流。这个流中发出的数据，其实就是动态表中新增的每一行。

⚫ 撤回（Retract）流
  撤回流是包含两类消息的流，添加（add）消息和撤回（retract）消息。具体的编码规则是：INSERT 插入操作编码为 add 消息；DELETE 删除操作编码为 retract消息；而 UPDATE 更新操作则编码为被更改行的 retract 消息，和更新后行（新行）的 add 消息。这样，我们可以通过编码后的消息指明所有的增删改操作，一个动态表就可以转换为撤回流了。

  可以看到，更新操作对于撤回流来说，对应着两个消息：之前数据的撤回（删除）和新数据的插入。显示了将动态表转换为撤回流的过程。

![image-20221227172339254](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221227172339254.png)

  这里我们用+代表 add 消息（对应插入 INSERT 操作），用-代表 retract 消息（对应删除DELETE 操作）；当 Alice 的第一个点击事件到来时，结果表新增一条数据[Alice, 1]；而当 Alice的第二个点击事件到来时，结果表会将[Alice, 1]更新为[Alice, 2]，对应的编码就是删除[Alice, 1]、插入[Alice, 2]。这样当一个外部系统收到这样的两条消息时，就知道是要对 Alice 的点击统计次数进行更新了。

⚫ 更新插入（Upsert）流
  更新插入流中只包含两种类型的消息：更新插入（upsert）消息和删除（delete）消息。所谓的“upsert”其实是“update”和“insert”的合成词，所以对于更新插入流来说，INSERT 插入操作和UPDATE更新操作，统一被编码为upsert消息；而DELETE删除操作则被编码为delete消息。
  既然更新插入流中不区分插入（insert）和更新（update），那我们自然会想到一个问题：如果希望更新一行数据时，怎么保证最后做的操作不是插入呢？这就需要动态表中必须有唯一的键（key）。通过这个 key 进行查询，如果存在对应的数据就做更新（update），如果不存在就直接插入（insert）。这是一个动态表可以转换为更新插入流的必要条件。当然，收到这条流中数据的外部系统，也需要知道这唯一的键（key），这样才能正确地处理消息。
  如图 11-7 所示，显示了将动态表转换为更新插入流的过程。

![image-20221229105807279](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221229105807279.png)

  可以看到，更新插入流跟撤回流的主要区别在于，更新（update）操作由于有 key 的存在，
只需要用单条消息编码就可以，因此效率更高。

  需要注意的是，在代码里将动态表转换为 DataStream 时，只支持仅追加（append-only）和撤回（retract）流，我们调用 toChangelogStream()得到的其实就是撤回流；这也很好理解，DataStream 中并没有 key 的定义，所以只能通过两条消息一减一增来表示更新操作。而连接到外部系统时，则可以支持不同的编码方法，这取决于外部系统本身的特性。

## 11.4 时间属性和窗口

  基于时间的操作（比如时间窗口），需要定义相关的时间语义和时间数据来源的信息。在Table API 和 SQL 中，会给表单独提供一个逻辑上的时间字段，专门用来在表处理程序中指示时间。

  所以所谓的时间属性（time attributes），其实就是每个表模式结构（schema）的一部分。它可以在创建表的 DDL 里直接定义为一个字段，也可以在 DataStream 转换成表时定义。一旦定义了时间属性，它就可以作为一个普通字段引用，并且可以在基于时间的操作中使用。时间属性的数据类型为 TIMESTAMP，它的行为类似于常规时间戳，可以直接访问并且进行计算。

  按照时间语义的不同，我们可以把时间属性的定义分成事件时间（event time）和处理时间（processing time）两种情况。

### 11.4.1 事件时间

  我们在实际应用中，最常用的就是事件时间。在事件时间语义下，允许表处理程序根据每个数据中包含的时间戳（也就是事件发生的时间）来生成结果。

  事件时间语义最大的用途就是处理乱序事件或者延迟事件的场景。我们通过设置水位线（watermark）来表示事件时间的进展，而水位线可以根据数据的最大时间戳设置一个延迟时间。这样即使在出现乱序的情况下，对数据的处理也可以获得正确的结果。

  为了处理无序事件，并区分流中的迟到事件。Flink 需要从事件数据中提取时间戳，并生成水位线，用来推进事件时间的进展。

  事件时间属性可以在创建表 DDL 中定义，也可以在数据流和表的转换中定义。

**1.在创建表的 DDL 中定义**

  在创建表的 DDL（CREATE TABLE 语句）中，可以增加一个字段，通过 WATERMARK语句来定义事件时间属性。WATERMARK 语句主要用来定义水位线（watermark）的生成表达式，这个表达式会将带有事件时间戳的字段标记为事件时间属性，并在它基础上给出水位线的延迟时间。具体定义方式如下：

```
CREATE TABLE EventTable(
 user STRING,
 url STRING,
 ts TIMESTAMP(3),
 WATERMARK FOR ts AS ts - INTERVAL '5' SECOND
) WITH (
 ...
);
```

  这里我们把 ts 字段定义为事件时间属性，而且基于 ts 设置了 5 秒的水位线延迟。

```
INTERVAL '5' SECOND
```

  这里的数值必须用单引号引起来，而单位用 SECOND 和 SECONDS 是等效的。Flink 中支持的事件时间属性数据类型必须为 TIMESTAMP 或者 TIMESTAMP_LTZ。这里TIMESTAMP_LTZ 是指带有本地时区信息的时间戳（TIMESTAMP WITH LOCAL TIME ZONE）；一般情况下如果数据中的时间戳是“年-月-日-时-分-秒”的形式，那就是不带时区信息的，可以将事件时间属性定义为 TIMESTAMP 类型。

  而如果原始的时间戳就是一个长整型的毫秒数，这时就需要另外定义一个字段来表示事件时间属性，类型定义为 TIMESTAMP_LTZ 会更方便：

```
CREATE TABLE events (
 user STRING,
 url STRING,
 ts BIGINT,
 ts_ltz AS TO_TIMESTAMP_LTZ(ts, 3),
 WATERMARK FOR ts_ltz AS time_ltz - INTERVAL '5' SECOND
) WITH (
 ...
);
```

  这里我们另外定义了一个字段 ts_ltz，是把长整型的 ts 转换为 TIMESTAMP_LTZ 得到的；进而使用 WATERMARK 语句将它设为事件时间属性，并设置 5 秒的水位线延迟。

**2.在数据流转换为表时定义**

  事件时间属性也可以在将 DataStream 转换为表的时候来定义。我们调用 fromDataStream()方法创建表时，可以追加参数来定义表中的字段结构；这时可以给某个字段加上.rowtime() 后缀，就表示将当前字段指定为事件时间属性。这个字段可以是数据中本不存在、额外追加上去的“逻辑字段”，就像之前 DDL 中定义的第二种情况；也可以是本身固有的字段，那么这个字段就会被事件时间属性所覆盖，类型也会被转换为 TIMESTAMP。不论那种方式，时间属性字段中保存的都是事件的时间戳（TIMESTAMP 类型）。

  需要注意的是，这种方式只负责指定时间属性，而时间戳的提取和水位线的生成应该之前就在 DataStream 上定义好了。由于 DataStream 中没有时区概念，因此 Flink 会将事件时间属性解析成不带时区的 TIMESTAMP 类型，所有的时间值都被当作 UTC 标准时间。在代码中的定义方式如下：

```
// 方法一:
// 流中数据类型为二元组 Tuple2，包含两个字段；需要自定义提取时间戳并生成水位线
DataStream<Tuple2<String, String>> stream = 
inputStream.assignTimestampsAndWatermarks(...);
// 声明一个额外的逻辑字段作为事件时间属性
Table table = tEnv.fromDataStream(stream, $("user"), $("url"), 
$("ts").rowtime());

// 方法二:
// 流中数据类型为三元组 Tuple3，最后一个字段就是事件时间戳
DataStream<Tuple3<String, String, Long>> stream = 
inputStream.assignTimestampsAndWatermarks(...);
// 不再声明额外字段，直接用最后一个字段作为事件时间属性
Table table = tEnv.fromDataStream(stream, $("user"), $("url"), 
$("ts").rowtime());
```

### 11.4.2 处理时间

  相比之下处理时间就比较简单了，它就是我们的系统时间，使用时不需要提取时间戳（timestamp）和生成水位线（watermark）。因此在定义处理时间属性时，必须要额外声明一个字段，专门用来保存当前的处理时间。
  类似地，处理时间属性的定义也有两种方式：创建表 DDL 中定义，或者在数据流转换成
表时定义。

**1.在创建表的 DDL 中定义**

  在创建表的 DDL（CREATE TABLE 语句）中，可以增加一个额外的字段，通过调用系统内置的 PROCTIME()函数来指定当前的处理时间属性，返回的类型是 TIMESTAMP_LTZ。

```
CREATE TABLE EventTable(
 user STRING,
 url STRING,
 ts AS PROCTIME()
) WITH (
 ...
);
```

  这里的时间属性，其实是以“计算列”（computed column）的形式定义出来的。所谓的计算列是 Flink SQL 中引入的特殊概念，可以用一个 AS 语句来在表中产生数据中不存在的列，并且可以利用原有的列、各种运算符及内置函数。在前面事件时间属性的定义中，将 ts 字段转换成 TIMESTAMP_LTZ 类型的 ts_ltz，也是计算列的定义方式。

**2.在数据流转换为表时定义**

  处 理 时 间 属 性 同 样 可 以 在 将 DataStream 转 换 为 表 的 时 候 来 定 义 。 我 们 调 用fromDataStream()方法创建表时，可以用.proctime()后缀来指定处理时间属性字段。由于处理时间是系统时间，原始数据中并没有这个字段，所以处理时间属性一定不能定义在一个已有字段上，只能定义在表结构所有字段的最后，作为额外的逻辑字段出现。
代码中定义处理时间属性的方法如下：

```
DataStream<Tuple2<String, String>> stream = ...;
// 声明一个额外的字段作为处理时间属性字段
Table table = tEnv.fromDataStream(stream, $("user"), $("url"), 
$("ts").proctime());
```

### 11.4.3 窗口（Window）

  有了时间属性，接下来就可以定义窗口进行计算了。我们知道，窗口可以将无界流切割成大小有的“桶”（bucket）来做计算，通过截取有限数据集来处理无限的流数据。在 DataStream API 中提供了对不同类型的窗口进行定义和处理的接口，而在 Table API 和 SQL 中，类似的功能也都可以实现。

**1.分组窗口（Group Window，老版本）**

  在 Flink 1.12 之前的版本中，Table API 和 SQL 提供了一组“分组窗口”（Group Window）函数，常用的时间窗口如滚动窗口、滑动窗口、会话窗口都有对应的实现；具体在 SQL 中就是调用 TUMBLE()、HOP()、SESSION()，传入时间属性字段、窗口大小等参数就可以了。以滚动窗口为例：

```
TUMBLE(ts, INTERVAL '1' HOUR)
```

  这里的 ts 是定义好的时间属性字段，窗口大小用“时间间隔”INTERVAL 来定义。在进行窗口计算时，分组窗口是将窗口本身当作一个字段对数据进行分组的，可以对组内的数据进行聚合。基本使用方式如下：

```
Table result = tableEnv.sqlQuery(
 "SELECT " +
 "user, " +
"TUMBLE_END(ts, INTERVAL '1' HOUR) as endT, " +
 "COUNT(url) AS cnt " +
 "FROM EventTable " +
 "GROUP BY " + // 使用窗口和用户名进行分组
 "user, " +
 "TUMBLE(ts, INTERVAL '1' HOUR)" // 定义 1 小时滚动窗口
 );
```

  这里定义了 1 小时的滚动窗口，将窗口和用户 user 一起作为分组的字段。用聚合函数COUNT()对分组数据的个数进行了聚合统计，并将结果字段重命名为cnt；用TUPMBLE_END()函数获取滚动窗口的结束时间，重命名为 endT 提取出来。
  分组窗口的功能比较有限，只支持窗口聚合，所以目前已经处于弃用（deprecated）的状态。

**2.窗口表值函数（Windowing TVFs，新版本）**
  从 1.13 版本开始，Flink 开始使用窗口表值函数（Windowing table-valued functions，Windowing TVFs）来定义窗口。窗口表值函数是 Flink 定义的多态表函数（PTF），可以将表进行扩展后返回。表函数（table function）可以看作是返回一个表的函数，关于这部分内容，我们会在 11.6 节进行介绍。

**目前 Flink 提供了以下几个窗口 TVF：**
⚫ 滚动窗口（Tumbling Windows）；
⚫ 滑动窗口（Hop Windows，跳跃窗口）；
⚫ 累积窗口（Cumulate Windows）；
⚫ 会话窗口（Session Windows，目前尚未完全支持）。

  窗口表值函数可以完全替代传统的分组窗口函数。窗口 TVF 更符合 SQL 标准，性能得到了优化，拥有更强大的功能；可以支持基于窗口的复杂计算，例如窗口 Top-N、窗口联结（window join）等等。当然，目前窗口 TVF 的功能还不完善，会话窗口和很多高级功能还不支持，不过正在快速地更新完善。可以预见在未来的版本中，窗口 TVF 将越来越强大，将会是窗口处理的唯一入口。

  在窗口 TVF 的返回值中，除去原始表中的所有列，还增加了用来描述窗口的额外 3 个列：“窗口起始点”（window_start）、“窗口结束点”（window_end）、“窗口时间”（window_time）。起始点和结束点比较好理解，这里的“窗口时间”指的是窗口中的时间属性，它的值等于window_end - 1ms，所以相当于是窗口中能够包含数据的最大时间戳。

  在 SQL 中的声明方式，与以前的分组窗口是类似的，直接调用 TUMBLE()、HOP()、CUMULATE()就可以实现滚动、滑动和累积窗口，不过传入的参数会有所不同。下面我们就分别对这几种窗口 TVF 进行介绍。

**（1）滚动窗口（TUMBLE）**

  滚动窗口在 SQL 中的概念与 DataStream API 中的定义完全一样，是长度固定、时间对齐、无重叠的窗口，一般用于周期性的统计计算。在 SQL 中通过调用 TUMBLE()函数就可以声明一个滚动窗口，只有一个核心参数就是窗
口大小（size）。在 SQL 中不考虑计数窗口，所以滚动窗口就是滚动时间窗口，参数中还需要将当前的时间属性字段传入；另外，窗口 TVF 本质上是表函数，可以对表进行扩展，所以还应该把当前查询的表作为参数整体传入。具体声明如下：

```
TUMBLE(TABLE EventTable, DESCRIPTOR(ts), INTERVAL '1' HOUR)
```

  这里基于时间字段 ts，对表 EventTable 中的数据开了大小为 1 小时的滚动窗口。窗口会将表中的每一行数据，按照它们 ts 的值分配到一个指定的窗口中。

**（2）滑动窗口（HOP）**

  滑动窗口的使用与滚动窗口类似，可以通过设置滑动步长来控制统计输出的频率。在 SQL中通过调用 HOP()来声明滑动窗口；除了也要传入表名、时间属性外，还需要传入窗口大小（size）和滑动步长（slide）两个参数。

```
HOP(TABLE EventTable, DESCRIPTOR(ts), INTERVAL '5' MINUTES, INTERVAL '1' HOURS));
```

  这里我们基于时间属性 ts，在表 EventTable 上创建了大小为 1 小时的滑动窗口，每 5 分钟滑动一次。需要注意的是，紧跟在时间属性字段后面的第三个参数是步长（slide），第四个参数才是窗口大小（size）。

**（3）累积窗口（CUMULATE）**

  滚动窗口和滑动窗口，可以用来计算大多数周期性的统计指标。不过在实际应用中还会遇到这样一类需求：我们的统计周期可能较长，因此希望中间每隔一段时间就输出一次当前的统计值；与滑动窗口不同的是，在一个统计周期内，我们会多次输出统计值，它们应该是不断叠加累积的。

  例如，我们按天来统计网站的 PV（Page View，页面浏览量），如果用 1 天的滚动窗口，那需要到每天 24 点才会计算一次，输出频率太低；如果用滑动窗口，计算频率可以更高，但统计的就变成了“过去 24 小时的 PV”。所以我们真正希望的是，还是按照自然日统计每天的PV，不过需要每隔 1 小时就输出一次当天到目前为止的 PV 值。这种特殊的窗口就叫作“累积窗口”（Cumulate Window）。

![image-20221229112932566](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221229112932566.png)

  累积窗口是窗口 TVF 中新增的窗口功能，它会在一定的统计周期内进行累积计算。累积窗口中有两个核心的参数：最大窗口长度（max window size）和累积步长（step）。所谓的最大窗口长度其实就是我们所说的“统计周期”，最终目的就是统计这段时间内的数据。如图 11-8所示，开始时，创建的第一个窗口大小就是步长 step；之后的每个窗口都会在之前的基础上再扩展 step 的长度，直到达到最大窗口长度。在 SQL 中可以用 CUMULATE()函数来定义，具体如下：

```
CUMULATE(TABLE EventTable, DESCRIPTOR(ts), INTERVAL '1' HOURS, INTERVAL '1' DAYS))
```

  这里我们基于时间属性 ts，在表 EventTable 上定义了一个统计周期为 1 天、累积步长为 1小时的累积窗口。注意第三个参数为步长step，第四个参数则是最大窗口长度。上面所有的语句只是定义了窗口，类似于 DataStream API 中的窗口分配器；在 SQL 中窗口的完整调用，还需要配合聚合操作和其它操作。我们会在下一节详细讲解窗口的聚合。

## 11.5 聚合（Aggregation）查询

  在 SQL 中，一个很常见的功能就是对某一列的多条数据做一个合并统计，得到一个或多个结果值；比如求和、最大最小值、平均值等等，这种操作叫作聚合（Aggregation）查询。Flink 中的 SQL 是流处理与标准 SQL 结合的产物，所以聚合查询也可以分成两种：流处理中特有的聚合（主要指窗口聚合），以及 SQL 原生的聚合查询方式。

### 11.5.1 分组聚合



### 11.5.2 窗口聚合



### 11.5.3 开窗（Over）聚合



### 11.5.4 应用实例 —— Top N





## 11.6 联结（Join）查询

### 11.6.1 常规联结查询

### 11.6.2 间隔联结查询



## 11.7 函数

### 11.7.1 系统函数



### 11.7.2 自定义函数（UDF）



## 11.8 SQL 客户端

  有了 Table API 和 SQL，我们就可以使用熟悉的 SQL 来编写查询语句进行流处理了。不过，这种方式还是将 SQL 语句嵌入到 Java/Scala 代码中进行的；另外，写完的代码后想要提交作业还需要使用工具进行打包。这都给 Flink 的使用设置了门槛，如果不是 Java/Scala 程序员，即使是非常熟悉 SQL 的工程师恐怕也会望而生畏了。

  基于这样的考虑，Flink 为我们提供了一个工具来进行 Flink 程序的编写、测试和提交，这工具叫作“SQL 客户端”。SQL 客户端提供了一个命令行交互界面（CLI），我们可以在里面非常容易地编写 SQL 进行查询，就像使用 MySQL 一样；整个 Flink 应用编写、提交的过程全变成了写 SQL，不需要写一行 Java/Scala 代码。
具体使用流程如下：

**（1）首先启动本地集群**

```
./bin/start-cluster.sh
```

**（2）启动 Flink SQL 客户端**

```
./bin/sql-client.sh
```

SQL 客户端的启动脚本同样位于 Flink 的 bin 目录下。默认的启动模式是 embedded，也就是说客户端是一个嵌入在本地的进程，这是目前唯一支持的模式。未来会支持连接到远程 SQL客户端的模式。
**（3）设置运行模式**
  启动客户端后，就进入了命令行界面，这时就可以开始写 SQL 了。一般我们会在开始之前对环境做一些设置，比较重要的就是运行模式。

  首先是表环境的运行时模式，有流处理和批处理两个选项。默认为流处理：

```
Flink SQL> SET 'execution.runtime-mode' = 'streaming';
```

其次是 SQL 客户端的“执行结果模式”，主要有 table、changelog、tableau 三种，默认为table 模式：

```
Flink SQL> SET 'sql-client.execution.result-mode' = 'table';
```

  table 模式就是最普通的表处理模式，结果会以逗号分隔每个字段；changelog 则是更新日志模式，会在数据前加上“+”（表示插入）或“-”（表示撤回）的前缀；而 tableau 则是经典的可视化表模式，结果会是一个虚线框的表格。

  此外我们还可以做一些其它可选的设置，比如之前提到的空闲状态生存时间（TTL）：

```
Flink SQL> SET 'table.exec.state.ttl' = '1000';
```

  除了在命令行进行设置，我们也可以直接在 SQL 客户端的配置文件 sql-cli-defaults.yaml中进行各种配置，甚至还可以在这个 yaml 文件里预定义表、函数和 catalog。关于配置文件的更多用法，大家可以查阅官网的详细说明。
**（4）执行 SQL 查询**
  接下来就可以愉快的编写 SQL 语句了，这跟操作 MySQL、Oracle 等关系型数据库没什么区别。我们可以尝试把一开始举的简单聚合例子写一下：

> ```java
> Flink SQL> CREATE TABLE EventTable(
> user STRING,
> url STRING,
> `timestamp` BIGINT
> ) WITH (
> 'connector' = 'filesystem',
> 'path' = 'events.csv',
> 'format' = 'csv'
> );
> Flink SQL> CREATE TABLE ResultTable (
> user STRING,
> cnt BIGINT
> ) WITH (
> 'connector' = 'print'
> );
> 
> Flink SQL> INSERT INTO ResultTable SELECT user, COUNT(url) as cnt FROM EventTable
> GROUP BY user;
> ```

  这里我们直接用 DDL 创建两张表，注意需要有 WITH 定义的外部连接。一张表叫作EventTable，是从外部文件 events.csv 中读取数据的，这是输入数据表；另一张叫作 ResultTable，连接器为“print”，其实就是标准控制台打印，当然就是输出表了。所以接下来就可以直接执行 SQL 查询，并将查询结果 INSERT 写入结果表中了。

  在 SQL 客户端中，每定义一个 SQL 查询，就会把它作为一个 Flink 作业提交到集群上执行。所以通过这种方式，我们可以快速地对流处理程序进行开发测试。

# 第 12 章 Flink CEP(157-171)

## 12.1 基本概念

### 12.1.1 CEP 是什么

  所谓 CEP，其实就是“复杂事件处理（Complex Event Processing）”的缩写；而 Flink CEP，就是 Flink 实现的一个用于复杂事件处理的库（library）。
  那到底什么是“复杂事件处理”呢？就是可以在事件流里，检测到特定的事件组合并进行处理，比如说“连续登录失败”，或者“订单支付超时”等等。
  具体的处理过程是，把事件流中的一个个简单事件，通过一定的规则匹配组合起来，这就是“复杂事件”；然后基于这些满足规则的一组组复杂事件进行转换处理，得到想要的结果进行输出。

总结起来，复杂事件处理（CEP）的流程可以分成三个步骤：
（1）定义一个匹配规则
（2）将匹配规则应用到事件流上，检测满足规则的复杂事件
（3）对检测到的复杂事件进行处理，得到结果进行输出

![image-20221229114449329](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221229114449329.png)

  所以，CEP 是针对流处理而言的，分析的是低延迟、频繁产生的事件流。它的主要目的，就是在无界流中检测出特定的数据组合，让我们有机会掌握数据中重要的高阶特征。

### 12.1.2 模式（Pattern）

  CEP 的第一步所定义的匹配规则，我们可以把它叫作“模式”（Pattern）。模式的定义主要
就是两部分内容：

⚫ 每个简单事件的特征
⚫ 简单事件之间的组合关系
  当然，我们也可以进一步扩展模式的功能。比如，匹配检测的时间限制；每个简单事件是否可以重复出现；对于事件可重复出现的模式，遇到一个匹配后是否跳过后面的匹配；等等。所谓“事件之间的组合关系”，一般就是定义“谁后面接着是谁”，也就是事件发生的顺序。我们把它叫作“近邻关系”。可以定义严格的近邻关系，也就是两个事件之前不能有任何其他事件；也可以定义宽松的近邻关系，即只要前后顺序正确即可，中间可以有其他事件。另外，还可以反向定义，也就是“谁后面不能跟着谁”。

  CEP 做的事其实就是在流上进行模式匹配。根据模式的近邻关系条件不同，可以检测连续的事件或不连续但先后发生的事件；模式还可能有时间的限制，如果在设定时间范围内没有满足匹配条件，就会导致模式匹配超时（timeout）。

  Flink CEP 为我们提供了丰富的 API，可以实现上面关于模式的所有功能，这套 API 就叫作“模式 API”（Pattern API）。关于 Pattern API，我们会在后面的 12.3 节中详细介绍。

### 12.1.3 应用场景

  CEP 主要用于实时流数据的分析处理。CEP 可以帮助在复杂的、看似不相关的事件流中找出那些有意义的事件组合，进而可以接近实时地进行分析判断、输出通知信息或报警。这在企业项目的风控管理、用户画像和运维监控中，都有非常重要的应用。

⚫ 风险控制
  设定一些行为模式，可以对用户的异常行为进行实时检测。当一个用户行为符合了异常行为模式，比如短时间内频繁登录并失败、大量下单却不支付（刷单），就可以向用户发送通知信息，或是进行报警提示、由人工进一步判定用户是否有违规操作的嫌疑。这样就可以有效地控制用户个人和平台的风险。

⚫ 用户画像
  利用 CEP 可以用预先定义好的规则，对用户的行为轨迹进行实时跟踪，从而检测出具有特定行为习惯的一些用户，做出相应的用户画像。基于用户画像可以进行精准营销，即对行为匹配预定义规则的用户实时发送相应的营销推广；这与目前很多企业所做的精准推荐原理是一样的。

⚫ 运维监控
  对于企业服务的运维管理，可以利用 CEP 灵活配置多指标、多依赖来实现更复杂的监控模式。

  CEP 的应用场景非常丰富。很多大数据框架，如 Spark、Samza、Beam 等都提供了不同的CEP 解决方案，但没有专门的库（library）。而 Flink 提供了专门的 CEP 库用于复杂事件处理，可以说是目前 CEP 的最佳解决方案。

## 12.2 快速上手

### 12.2.1 需要引入的依赖

想要在代码中使用 Flink CEP，需要在项目的 pom 文件中添加相关依赖：

```
<dependency>
 <groupId>org.apache.flink</groupId>
 <artifactId>flink-cep_${scala.binary.version}</artifactId>
 <version>${flink.version}</version>
</dependency>
```

  为了精简和避免依赖冲突，Flink 会保持尽量少的核心依赖。所以核心依赖中并不包括任何的连接器（conncetor）和库，这里的库就包括了 SQL、CEP 以及 ML 等等。所以如果想要在 Flink 集群中提交运行 CEP 作业，应该向 Flink SQL 那样将依赖的 jar 包放在/lib 目录下。从这个角度来看，Flink CEP 和 Flink SQL 一样，都是最顶层的应用级 API。

### 12.2.2 一个简单实例  

  接下来我们考虑一个具体的需求：检测用户行为，如果连续三次登录失败，就输出报警信息。很显然，这是一个复杂事件的检测处理，我们可以使用 Flink CEP 来实现。
  我们首先定义数据的类型。这里的用户行为不再是之前的访问事件 Event 了，所以应该单独定义一个登录事件 POJO 类。具体实现如下：

```java
public class LoginEvent {
    public String userId;
    public String ipAddress;
    public String eventType;
    public Long timestamp;

    public LoginEvent(String userId, String ipAddress, String eventType, Long timestamp) {
        this.userId = userId;
        this.ipAddress = ipAddress;
        this.eventType = eventType;
        this.timestamp = timestamp;
    }

    public LoginEvent() {}

    @Override
    public String toString() {
        return "LoginEvent{" +
                "userId='" + userId + '\'' +
                ", ipAddress='" + ipAddress + '\'' +
                ", eventType='" + eventType + '\'' +
                ", timestamp=" + timestamp +
                '}';
    }
}
```

  接下来就是业务逻辑的编写。Flink CEP 在代码中主要通过 Pattern API 来实现。之前我们已经介绍过，CEP 的主要处理流程分为三步，对应到 Pattern API 中就是：
（1）定义一个模式（Pattern）；
（2）将Pattern应用到DataStream上，检测满足规则的复杂事件，得到一个PatternStream；
（3）对 PatternStream 进行转换处理，将检测到的复杂事件提取出来，包装成报警信息输出。
  具体代码实现如下：

```java
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.cep.CEP;
import org.apache.flink.cep.PatternSelectFunction;
import org.apache.flink.cep.PatternStream;
import org.apache.flink.cep.pattern.Pattern;
import org.apache.flink.cep.pattern.conditions.SimpleCondition;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.time.Duration;
import java.util.List;
import java.util.Map;

public class LoginFailDetectExample {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 1. 获取登录事件流，并提取时间戳、生成水位线
        KeyedStream<LoginEvent, String> stream = env
                .fromElements(
                        new LoginEvent("user_1", "192.168.0.1", "fail", 2000L),
                        new LoginEvent("user_1", "192.168.0.2", "fail", 3000L),
                        new LoginEvent("user_2", "192.168.1.29", "fail", 4000L),
                        new LoginEvent("user_1", "171.56.23.10", "fail", 5000L),
                        new LoginEvent("user_2", "192.168.1.29", "fail", 7000L),
                        new LoginEvent("user_2", "192.168.1.29", "fail", 8000L),
                        new LoginEvent("user_2", "192.168.1.29", "success", 6000L)
                )
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.<LoginEvent>forBoundedOutOfOrderness(Duration.ofSeconds(3))
                                .withTimestampAssigner(
                                        new SerializableTimestampAssigner<LoginEvent>() {
                                            @Override
                                            public long extractTimestamp(LoginEvent loginEvent, long l) {
                                                return loginEvent.timestamp;
                                            }
                                        }
                                )
                )
                .keyBy(r -> r.userId);

        // 2. 定义Pattern，连续的三个登录失败事件
        Pattern<LoginEvent, LoginEvent> pattern = Pattern.<LoginEvent>begin("first")    // 以第一个登录失败事件开始
                .where(new SimpleCondition<LoginEvent>() {
                    @Override
                    public boolean filter(LoginEvent loginEvent) throws Exception {
                        return loginEvent.eventType.equals("fail");
                    }
                })
                .next("second")    // 接着是第二个登录失败事件
                .where(new SimpleCondition<LoginEvent>() {
                    @Override
                    public boolean filter(LoginEvent loginEvent) throws Exception {
                        return loginEvent.eventType.equals("fail");
                    }
                })
                .next("third")     // 接着是第三个登录失败事件
                .where(new SimpleCondition<LoginEvent>() {
                    @Override
                    public boolean filter(LoginEvent loginEvent) throws Exception {
                        return loginEvent.eventType.equals("fail");
                    }
                });

        // 3. 将Pattern应用到流上，检测匹配的复杂事件，得到一个PatternStream
        PatternStream<LoginEvent> patternStream = CEP.pattern(stream, pattern);

        // 4. 将匹配到的复杂事件选择出来，然后包装成字符串报警信息输出
        patternStream
                .select(new PatternSelectFunction<LoginEvent, String>() {
                    @Override
                    public String select(Map<String, List<LoginEvent>> map) throws Exception {
                        LoginEvent first = map.get("first").get(0);
                        LoginEvent second = map.get("second").get(0);
                        LoginEvent third = map.get("third").get(0);
                        return first.userId + " 连续三次登录失败！登录时间：" + first.timestamp + ", " + second.timestamp + ", " + third.timestamp;
                    }
                })
                .print("warning");

        env.execute();
    }
}
```

![image-20221229115212371](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20221229115212371.png)

## 12.3 模式 API（Pattern API）

### 12.3.1 个体模式

  在 12.1.2 小节中我们已经知道，模式（Pattern）其实就是将一组简单事件组合成复杂事件的“匹配规则”。由于流中事件的匹配是有先后顺序的，因此一个匹配规则就可以表达成先后发生的一个个简单事件，按顺序串联组合在一起。

  这里的每一个简单事件并不是任意选取的，也需要有一定的条件规则；所以我们就把每个简单事件的匹配规则，叫作“个体模式”（Individual Pattern）

**1.基本形式**

在 12.2.2 小节中，每一个登录失败事件的选取规则，就都是一个个体模式。比如：

```
.<LoginEvent>begin("first") // 以第一个登录失败事件开始
 .where(new SimpleCondition<LoginEvent>() {
 @Override
 public boolean filter(LoginEvent loginEvent) throws Exception {
 return loginEvent.eventType.equals("fail");
 }
 })
```

或者后面的：

```
.next("second") // 接着是第二个登录失败事件
 .where(new SimpleCondition<LoginEvent>() {
 @Override
 public boolean filter(LoginEvent loginEvent) throws Exception {
 return loginEvent.eventType.equals("fail");
 }
  })
```

这些都是个体模式。个体模式一般都会匹配接收一个事件。
  每个个体模式都以一个“连接词”开始定义的，比如 begin、next 等等，这是 Pattern 对象的一个方法（begin 是 Pattern 类的静态方法），返回的还是一个 Pattern。这些“连接词”方法有一个 String 类型参数，这就是当前个体模式唯一的名字，比如这里的“first”、“second”。在之后检测到匹配事件时，就会以这个名字来指代匹配事件。
个体模式需要一个“过滤条件”，用来指定具体的匹配规则。这个条件一般是通过调用.where()方法来实现的，具体的过滤逻辑则通过传入的 SimpleCondition 内的.filter()方法来定义。

  另外，个体模式可以匹配接收一个事件，也可以接收多个事件。这听起来有点奇怪，一个单独的匹配规则可能匹配到多个事件吗？这是可能的，我们可以给个体模式增加一个“量词”（quantifier），就能够让它进行循环匹配，接收多个事件。接下来我们就对量词和条件（condition）进行展开说明。

**2.量词（Quantifiers ）**

  个体模式后面可以跟一个“量词”，用来指定循环的次数。从这个角度分类，个体模式可以包括“单例（singleton）模式”和“循环（looping）模式”。默认情况下，个体模式是单例模式，匹配接收一个事件；当定义了量词之后，就变成了循环模式，可以匹配接收多个事件。在循环模式中，对同样特征的事件可以匹配多次。比如我们定义个体模式为“匹配形状为三角形的事件”，再让它循环多次，就变成了“匹配连续多个三角形的事件”。注意这里的“连续”，只要保证前后顺序即可，中间可以有其他事件，所以是“宽松近邻”关系。

  在 Flink CEP 中，可以使用不同的方法指定循环模式，主要有：
⚫ .oneOrMore（）
  匹配事件出现一次或多次，假设 a 是一个个体模式，a.oneOrMore()表示可以匹配 1 个或多个 a 的事件组合。我们有时会用 a+来简单表示。

⚫ .times（times）
  匹配事件发生特定次数（times），例如 a.times(3)表示 aaa；

⚫ .times（fromTimes，toTimes）
  指定匹配事件出现的次数范围，最小次数为fromTimes，最大次数为toTimes。例如a.times(2, 4)可以匹配 aa，aaa 和 aaaa。

⚫ .greedy()
  只能用在循环模式后，使当前循环模式变得“贪心”（greedy），也就是总是尽可能多地去匹配。例如 a.times(2, 4).greedy()，如果出现了连续 4 个 a，那么会直接把 aaaa 检测出来进行处理，其他任意 2 个 a 是不算匹配事件的。

⚫ .optional()
  使当前模式成为可选的，也就是说可以满足这个匹配条件，也可以不满足。对于一个个体模式 pattern 来说，后面所有可以添加的量词如下：

```java
// 匹配事件出现 4 次
pattern.times(4);

// 匹配事件出现 4 次，或者不出现
pattern.times(4).optional();

// 匹配事件出现 2, 3 或者 4 次
pattern.times(2, 4);

// 匹配事件出现 2, 3 或者 4 次，并且尽可能多地匹配
pattern.times(2, 4).greedy();

// 匹配事件出现 2, 3, 4 次，或者不出现
pattern.times(2, 4).optional();

// 匹配事件出现 2, 3, 4 次，或者不出现；并且尽可能多地匹配
pattern.times(2, 4).optional().greedy();

// 匹配事件出现 1 次或多次
pattern.oneOrMore();

// 匹配事件出现 1 次或多次，并且尽可能多地匹配
pattern.oneOrMore().greedy();

// 匹配事件出现 1 次或多次，或者不出现
pattern.oneOrMore().optional();

// 匹配事件出现 1 次或多次，或者不出现；并且尽可能多地匹配
pattern.oneOrMore().optional().greedy();

// 匹配事件出现 2 次或多次
pattern.timesOrMore(2);

// 匹配事件出现 2 次或多次，并且尽可能多地匹配
pattern.timesOrMore(2).greedy();

// 匹配事件出现 2 次或多次，或者不出现
pattern.timesOrMore(2).optional()

// 匹配事件出现 2 次或多次，或者不出现；并且尽可能多地匹配
pattern.timesOrMore(2).optional().greedy();
```

  正是因为个体模式可以通过量词定义为循环模式，一个模式能够匹配到多个事件，所以之前代码中事件的检测接收才会用 Map 中的一个列表（List）来保存。而之前代码中没有定义量词，都是单例模式，所以只会匹配一个事件，每个 List 中也只有一个元素：

```
LoginEvent first = map.get("first").get(0);
```



### 12.3.2 组合模式



### 12.3.3 模式组



### 12.3.4 匹配后跳过策略



## 12.4 模式的检测处理

### 12.4.1 将模式应用到流上

  将模式应用到事件流上的代码非常简单，只要调用 CEP 类的静态方法.pattern()，将数据流（DataStream）和模式（Pattern）作为两个参数传入就可以了。最终得到的是一个 PatternStream：

```
DataStream<Event> inputStream = ...
Pattern<Event, ?> pattern = ...
PatternStream<Event> patternStream = CEP.pattern(inputStream, pattern);
```

  这里的 DataStream，也可以通过 keyBy 进行按键分区得到 KeyedStream，接下来对复杂事件的检测就会针对不同的 key 单独进行了。

  模式中定义的复杂事件，发生是有先后顺序的，这里“先后”的判断标准取决于具体的时间语义。默认情况下采用事件时间语义，那么事件会以各自的时间戳进行排序；如果是处理时间语义，那么所谓先后就是数据到达的顺序。对于时间戳相同或是同时到达的事件，我们还可以在 CEP.pattern()中传入一个比较器作为第三个参数，用来进行更精确的排序：

```
// 可选的事件比较器
EventComparator<Event> comparator = ... 
PatternStream<Event> patternStream = CEP.pattern(input, pattern, comparator);
```

  得到 PatternStream 后，接下来要做的就是对匹配事件的检测处理了。

### 12.4.2 处理匹配事件



### 12.4.3 处理超时事件



### 12.4.4 处理迟到数据

  CEP 主要处理的是先后发生的一组复杂事件，所以事件的顺序非常关键。前面已经说过，事件先后顺序的具体定义与时间语义有关。如果是处理时间语义，那比较简单，只要按照数据处理的系统时间算就可以了；而如果是事件时间语义，需要按照事件自身的时间戳来排序。这就有可能出现时间戳大的事件先到、时间戳小的事件后到的现象，也就是所谓的“乱序数据”或“迟到数据”。

  在 Flink CEP 中沿用了通过设置水位线（watermark）延迟来处理乱序数据的做法。当一个事件到来时，并不会立即做检测匹配处理，而是先放入一个缓冲区（buffer）。缓冲区内的数据，会按照时间戳由小到大排序；当一个水位线到来时，就会将缓冲区中所有时间戳小于水位线的事件依次取出，进行检测匹配。这样就保证了匹配事件的顺序和事件时间的进展一致，处理的顺序就一定是正确的。这里水位线的延迟时间，也就是事件在缓冲区等待的最大时间。这样又会带来另一个问题：水位线延迟时间不可能保证将所有乱序数据完美包括进来，总会有一些事件延迟比较大，以至于等它到来的时候水位线早已超过了它的时间戳。这时之前的数据都已处理完毕，这样的“迟到数据”就只能被直接丢弃了——这与窗口对迟到数据的默认处理一致。

  我们自然想到，如果不希望迟到数据丢掉，应该也可以借鉴窗口的做法。Flink CEP 同样提 供 了 将 迟 到 事 件 输 出 到 侧 输 出 流 的 方 式 ： 我 们 可 以 基 于 PatternStream 直接调用.sideOutputLateData()方法，传入一个 OutputTag，将迟到数据放入侧输出流另行处理。代码中调用方式如下：

```java
PatternStream<Event> patternStream = CEP.pattern(input, pattern);
// 定义一个侧输出流的标签
OutputTag<String> lateDataOutputTag = new OutputTag<String>("late-data"){};
SingleOutputStreamOperator<ComplexEvent> result = patternStream
 .sideOutputLateData(lateDataOutputTag) // 将迟到数据输出到侧输出流
 .select( 
// 处理正常匹配数据
 new PatternSelectFunction<Event, ComplexEvent>() {...}
 );
// 从结果中提取侧输出流
DataStream<String> lateData = result.getSideOutput(lateDataOutputTag);
```

  可以看到，整个处理流程与窗口非常相似。经处理匹配数据得到结果数据流之后，可以调用.getSideOutput()方法来提取侧输出流，捕获迟到数据进行额外处理。

## 12.5 CEP 的状态机实现





## 12.6 本章总结

  Flink CEP 是 Flink 对复杂事件处理提供的强大而高效的应用库。本章中我们从一个简单的应用实例出发，详细讲解了 CEP 的核心内容——Pattern API 和模式的检测处理，并以案例说明了对超时事件和迟到数据的处理。最后进行了深度扩展，举例讲解了 CEP 的状态机实现，这部分大家可以只做原理了解，不要求完全实现状态机的代码。

  CEP 在实际生产中有非常广泛的应用。对于大数据分析而言，应用场景主要可以分为统计分析和逻辑分析。企业的报表统计、商业决策都离不开统计分析，这部分需求在目前企业的分析指标中占了很大的比重，实时的流数据统计可以通过 Flink SQL 方便地实现；而逻辑分析可以进一步细分为风险控制、数据挖掘、用户画像、精准推荐等各个应用场景，如今对实时性要求也越来越高，Flink CEP 就可以作为对流数据进行逻辑分析、进行实时风控和推荐的有力工具。

  所以 DataStream API 和处理函数是 Flink 应用的基石，而 SQL 和 CEP 就是 Flink 大厦顶层扩展的两大工具。Flink SQL 也提供了与 CEP 相结合的“模式识别”（Pattern Recognition）语句——MATCH_RECOGNIZE，可以支持在 SQL 语句中进行复杂事件处理。尽管目前还不完善，不过相信随着 Flink 的进一步发展，Flink SQL 和 CEP 将对程序员更加友好，功能也将更加强大，全方位实现大数据实时流处理的各种应用需求。
