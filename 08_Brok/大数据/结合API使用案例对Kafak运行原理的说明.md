# 结合Kafak API生成消费者—介绍Kafka运行原理

​		**本教程主要结合Kafka JavaAPI构建生产者和消费者完成对Kafka的相关使用总结，包含架构图等，希望对你有所帮助。**

如需获取更多源码，笔记，教程，请访问本人仓库，你的关注是我创作的动力！

链接：https://gitee.com/fanggaolei/learning-notes-warehouse

# 1.0Kafka运行原理

​       在消息发送的过程中，涉及到了两个线程——main 线程和 Sender 线程。在 main 线程中创建了一个双端队列 RecordAccumulator。main 线程将消息发送给 RecordAccumulator，Sender 线程不断从RecordAccumulator 中拉取消息发送到 Kafka Broker。

![1659099842289](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/1659099842289.png)

**图片来源:尚硅谷**

# 2.0业务流程

## 2.1构建存储区域

**先启动zookeeper再启动kafka**

启动完成后，通过命令行和配置文件创建kafka存储区

```
#创建一个分区为3，副本为2的kafka存储空间，命名为first
[fang@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --create --partitions 3 --replication-factor 2 --topic first
#查看是否创建成功
[fang@hadoop102 kafka]$ bin/kafka-topics.sh --bootstrap-server hadoop102:9092 --list
__consumer_offsets

calllog
ct
first
```

## 2.2使用Java API构建生产者

将数据插入到Kafka存储区中

```
package com.fang.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class CustomProducer {
    public static void main(String[] args) {

        //0 配置
        Properties properties = new Properties();

        //1.连接集群
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"hadoop102:9092,hadoop103:9092");

        //2.指定对应的value和key
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());

        //3.创建Kafka生产者对象
        KafkaProducer<String, String> KafkaProducer = new KafkaProducer<String, String>(properties);

        //4.发送数据
        for (int i = 0; i <5 ; i++) {
            KafkaProducer.send(new ProducerRecord<String, String>("first","fang"+i));
        }
        //5.关闭资源
        KafkaProducer.close();
    }
}

```

监控topic

```
bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic first
```

![image-20220823161001797](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220823161001797.png)

## 2.3使用Java API构建消费者

将数据从kafka存储区进行读取

```
package com.fang.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class CustomProducer {
    public static void main(String[] args) {

        //0 配置
        Properties properties = new Properties();

        //连接集群
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,"hadoop102:9092,hadoop103:9092");

        //指定对应的value和key
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,StringSerializer.class.getName());

        //1.创建Kafka生产者对象
        KafkaProducer<String, String> KafkaProducer = new KafkaProducer<String, String>(properties);

        //2.发送数据
        for (int i = 0; i <100 ; i++) {
            KafkaProducer.send(new ProducerRecord<String, String>("first","fang"+i));
        }
        //3.关闭资源
        KafkaProducer.close();
    }
}
```

**首先运行消费者**

**然后运行生产者**

查看控制台

![image-20220823161112940](https://pic-1313413291.cos.ap-nanjing.myqcloud.com/image-20220823161112940.png)
