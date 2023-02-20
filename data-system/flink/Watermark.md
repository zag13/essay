# Watermark

## Event Time and Processing Time

- Processing Time: 处理时间，即执行相应操作时的机器系统时间
- Event Time: 事件时间，即事件在其生产设备发生的时间

<img src="../../assets/img/flink/event_processing_time.svg" width="92%">

## Event Time and Watermark

watermark本质上一种时间戳，通常会基于watermark机制触发window窗口计算，用于处理乱序事件或延迟数据。
watermark可以理解为全局进度指标，表示我们确信不会再有延迟事件到来的某个时间点。
当一个算子接收到时间为T的水位线，就可以认为不会再接收到任何时间戳小于或等于T的事件了。
而对于那些可能易于watermark的迟到事件，Flink中可以采取的机制有SideOutput、AllowedLateness或直接丢弃。

**有序的数据流watermark：**

在某些情况下，基于Event Time的数据流是有序的(相对event time)。
在有序流中，watermark就是一个简单的周期性标记。

<img src="../../assets/img/flink/stream_watermark_in_order.svg" width="92%">

**无序的数据流watermark：**

在更多场景下，基于Event Time的数据流是无序的(相对event time)。
在无序流中，它告诉operator比watermark更早(时间戳更小)的事件已经到达，可以触发window计算啦。

<img src="../../assets/img/flink/stream_watermark_out_of_order.svg" width="92%">

**并行流当中的watermark：**

在多并行度的情况下，watermark对齐会取所有channel最小的watermark。

<img src="../../assets/img/flink/parallel_streams_watermarks.svg" width="92%">

## Lateness

[Allowed Lateness](https://nightlies.apache.org/flink/flink-docs-release-1.16/zh/docs/dev/datastream/operators/windows/#allowed-lateness)

## 代码分析

**Watermark为0时的输出：**

当Watermark（等于EventTime）大于WindowEndTime时，触发窗口计算，将窗口内的数据汇总输出。

| EventTime           | Watermark           | WindowStartTime     | WindowEndTime       | 备注                                                     |
|---------------------|---------------------|---------------------|---------------------|--------------------------------------------------------|
| 2023-01-01 00:00:00 | 2023-01-01 00:00:00 | 2023-01-01 00:00:00 | 2023-01-01 00:00:05 |                                                        |
| 2023-01-01 00:00:03 | 2023-01-01 00:00:03 |                     |                     |                                                        |
| 2023-01-01 00:00:05 | 2023-01-01 00:00:05 | 2023-01-01 00:00:05 | 2023-01-01 00:00:10 | Watermark >= WindowEndTime，触发第一个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:04 | 2023-01-01 00:00:05 |                     |                     | 迟到数据被丢弃，没有输出                                           |
| 2023-01-01 00:00:09 | 2023-01-01 00:00:09 |                     |                     |                                                        |
| 2023-01-01 00:00:12 | 2023-01-01 00:00:12 | 2023-01-01 00:00:10 | 2023-01-01 00:00:15 | Watermark >= WindowEndTime，触发第二个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:08 | 2023-01-01 00:00:12 |                     |                     | 迟到数据被丢弃，没有输出                                           |
| 2023-01-01 00:00:18 | 2023-01-01 00:00:18 | 2023-01-01 00:00:15 | 2023-01-01 00:00:20 | Watermark >= WindowEndTime，触发第三个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:20 | 2023-01-01 00:00:20 | 2023-01-01 00:00:20 | 2023-01-01 00:00:25 | Watermark >= WindowEndTime，触发第四个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:13 | 2023-01-01 00:00:20 |                     |                     | 迟到数据被丢弃，没有输出                                           |
| 2023-01-01 00:00:25 | 2023-01-01 00:00:25 | 2023-01-01 00:00:25 | 2023-01-01 00:00:30 | Watermark >= WindowEndTime，触发第五个窗口为5s的计算，将上方位于窗口内的数据汇总 |

最后一条数据的Watermark为`92278994-08-17 15:12:55`是`Long.MAX_VALUE`的时间戳，触发了第六个窗口计算。
具体原因未知！！！

**Watermark为5s时的输出：**

当Watermark（= EventTime - maxOutOfOrderness）大于WindowEndTime时，触发窗口计算，将窗口内的数据汇总输出。

| EventTime           | Watermark           | WindowStartTime     | WindowEndTime       | 备注                                                     |
|---------------------|---------------------|---------------------|---------------------|--------------------------------------------------------|
| 2023-01-01 00:00:00 | 2022-12-31 23:59:55 | 2023-01-01 00:00:00 | 2023-01-01 00:00:05 |                                                        |
| 2023-01-01 00:00:03 | 2022-12-31 23:59:58 |                     |                     |                                                        |
| 2023-01-01 00:00:05 | 2023-01-01 00:00:00 | 2023-01-01 00:00:05 | 2023-01-01 00:00:10 |                                                        |
| 2023-01-01 00:00:04 | 2023-01-01 00:00:00 |                     |                     | 迟到数据，没有被丢弃                                             |
| 2023-01-01 00:00:09 | 2023-01-01 00:00:04 |                     |                     |                                                        |
| 2023-01-01 00:00:12 | 2023-01-01 00:00:07 | 2023-01-01 00:00:10 | 2023-01-01 00:00:15 | Watermark >= WindowEndTime，触发第一个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:08 | 2023-01-01 00:00:07 |                     |                     | 迟到数据，没有被丢弃                                             |
| 2023-01-01 00:00:18 | 2023-01-01 00:00:13 | 2023-01-01 00:00:15 | 2023-01-01 00:00:20 | Watermark >= WindowEndTime，触发第二个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:20 | 2023-01-01 00:00:15 | 2023-01-01 00:00:20 | 2023-01-01 00:00:25 | Watermark >= WindowEndTime，触发第三个窗口为5s的计算，将上方位于窗口内的数据汇总 |
| 2023-01-01 00:00:13 | 2023-01-01 00:00:15 |                     |                     | 该数据的EventTime小于上个数据的Watermark，被丢弃，没有输出                 |
| 2023-01-01 00:00:25 | 2023-01-01 00:00:20 | 2023-01-01 00:00:25 | 2023-01-01 00:00:30 | Watermark >= WindowEndTime，触发第四个窗口为5s的计算，将上方位于窗口内的数据汇总 |

最后一条数据的Watermark为`92278994-08-17 15:12:55`是`Long.MAX_VALUE`的时间戳，会相继触发第五个和第六个窗口计算。

<details> <summary>code</summary>

```
package com.github.zag13.demo;

import org.apache.flink.api.common.eventtime.Watermark;
import org.apache.flink.api.common.eventtime.WatermarkGenerator;
import org.apache.flink.api.common.eventtime.WatermarkOutput;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import org.apache.flink.streaming.api.functions.windowing.ProcessWindowFunction;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.api.windowing.windows.TimeWindow;
import org.apache.flink.util.Collector;

import java.text.SimpleDateFormat;
import java.time.Duration;

public class WatermarkDemo {

    public static void main(String[] args) throws Exception {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        env.setParallelism(1);

        env.addSource(new MySource())
                .assignTimestampsAndWatermarks(
                        WatermarkStrategy.forGenerator((ctx) -> new MyWatermarkGenerator(Duration.ofSeconds(5)))
                                .withTimestampAssigner((event, timestamp) -> Long.parseLong(event.split(",")[1]))
                                .withIdleness(Duration.ofSeconds(30))
                )
                .keyBy(i -> i.split(",")[0])
                .window(TumblingEventTimeWindows.of(Time.seconds(5)))
                .process(new ProcessWindowFunction<String, String, String, TimeWindow>() {
                    @Override
                    public void process(String key, Context context, Iterable<String> elements, Collector<String> out) throws Exception {
                        System.out.println("--------------------  START  --------------------");
                        System.out.println("Watermark：" + sdf.format(context.currentWatermark()));
                        for (String element : elements) {
                            System.out.println("Key: " + key +
                                    ", EventTime: " + sdf.format(Long.parseLong(element.split(",")[1])) +
                                    ", WindowStart: " + sdf.format(context.window().getStart()) +
                                    ", WindowEnd: " + sdf.format(context.window().getEnd()));
                        }
                        System.out.println("--------------------   END   --------------------");
                    }
                });

        env.execute("Watermark Demo");
    }

    public static class MySource implements SourceFunction<String> {
        private final String[] simulateData = new String[]{
                "Hello zag13!,1672502400000",
                "Hello zag13!,1672502403000",
                "Hello zag13!,1672502405000",
                "Hello zag13!,1672502404000",
                "Hello zag13!,1672502409000",
                "Hello zag13!,1672502412000",
                "Hello zag13!,1672502408000",
                "Hello zag13!,1672502418000",
                "Hello zag13!,1672502420000",
                "Hello zag13!,1672502413000",
                "Hello zag13!,1672502425000",
        };
        private       boolean  isRunning    = true;
        long lastEventTs = 0;

        @Override
        public void run(SourceContext<String> sourceContext) throws Exception {
            for (String s : simulateData) {
                if (!isRunning) break;
                long currentEventTime = Long.parseLong(s.split(",")[1]);
                if (lastEventTs != 0 && lastEventTs < currentEventTime) {
                    Thread.sleep(currentEventTime - lastEventTs);
                } else {
                    Thread.sleep(1000);
                }
                lastEventTs = currentEventTime;
                sourceContext.collect(s);
            }
        }

        @Override
        public void cancel() {
            isRunning = false;
        }
    }

    public static class MyWatermarkGenerator implements WatermarkGenerator<String> {
        private       long maxTimestamp;
        private final long outOfOrdernessMillis;

        public MyWatermarkGenerator(Duration maxOutOfOrderness) {
            this.outOfOrdernessMillis = maxOutOfOrderness.toMillis();
            this.maxTimestamp = Long.MIN_VALUE + this.outOfOrdernessMillis + 1L;
        }

        @Override
        public void onEvent(String event, long eventTimestamp, WatermarkOutput output) {
            this.maxTimestamp = Math.max(this.maxTimestamp, eventTimestamp);

            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            long eventTime = Long.parseLong(event.split(",")[1]);
            long watermark = this.maxTimestamp - this.outOfOrdernessMillis;
            System.out.println("Input: " + event + ", EventTime: " + sdf.format(eventTime) + ", Watermark: " + sdf.format(watermark));
            output.emitWatermark(new Watermark(watermark));
        }

        @Override
        public void onPeriodicEmit(WatermarkOutput watermarkOutput) {
//            System.out.println("系统周期性的发射水印，CurrentTimestamp: " + System.currentTimeMillis());
        }
    }

}
```

</details>