---
title: local-queue 简介
date: 2025-01-30 22:48:22
tags:
  - local-queue
  - java
---

项目地址：https://github.com/wz2cool/local-queue

## 简介

一直以来我们都可以比较方便时用 jvm 队列比如 LinkedBlockingQueue 做一些缓存, 或者我们想削峰,远程调用使用的是 kafka 或者 rocketmq，这都是非常有用的做法，但是我们可能也会遇到一些问题，  
对于 jvm 队列

1. 如果内存队列是无界队列可能会有内存溢出(out-of-memory)的风险
2. 一但程序关闭，所有在内存中的数据都会丢失，如果没有及时持久化的话数据就没了
3. 没有位置信息，无法做消费位置移动。

对于 Kafka/rocketmq

1. 受到网络的影响，如果网络延迟可能会导致消息阻塞的和延迟
2. 对于特别小的及时消息对于 kafka 不友好，kafka 目标是大吞吐，希望是攒批消息
3. 受到目标中间件高可用的影响，一但中间件不可用可能会存在丢消息情况

基于上述情况，我们可以看一下 local-queue，本项目基于 Chronicle 框架封装， Chronicle 是一个微妙级别延迟的框架，并且底层使用的是 mmap 技术，所以如果了解 kafka/rocketmq 会非常熟悉，这也使得 local-queue 在延迟上和性能上得到了基础保障。

## 功能

1. 一写多读
2. 保留消费者消费位置信息，可以断点续读
3. 移动消费位点，从新的消费位点续读
4. IPC 进程间通信（同一台机器两个 jvm 应用）

## 案例

### 一写多读

多个消费者可以读取同一份消息，消费者之间不同使用的是 consumerId 的区分，默认情况下消费者都是从最新的位置开始读，如果从最头部开始读会导致瞬发的 IO

```java
@Test
public void testMultiConsumers() throws InterruptedException {
  try (SimpleQueue queue = new SimpleQueue(config)) {
      Thread consumer1Thread = new Thread(() -> {
          IConsumer consumer1 = queue.getConsumer("consumer1");
          try {
              // 阻塞式获取和linkBlockingQueue 一样
              QueueMessage take = consumer1.take();
              assertEquals("testContent", take.getContent());
              assertEquals("testMessageKey", take.getMessageKey());
              System.out.println("consumer1 take message");
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }
      });
      Thread consumer2Thread = new Thread(() -> {
          IConsumer consumer2 = queue.getConsumer("consumer2");
          try {
              // 阻塞式获取和linkBlockingQueue 一样
              QueueMessage take = consumer2.take();
              assertEquals("testContent", take.getContent());
              assertEquals("testMessageKey", take.getMessageKey());
              System.out.println("consumer2 take message");
          } catch (InterruptedException e) {
              Thread.currentThread().interrupt();
          }
      });
      consumer1Thread.start();
      consumer2Thread.start();
      // 确保两个消费者开始读准备好了
      Thread.sleep(2000);
      // 写入消息
      queue.offer("testMessageKey", "testContent");
      consumer1Thread.join();
      consumer2Thread.join();
  }
}
```

### 断点续读

local-queue 支持定时把消费位置存到本地，一般有个 position.dat 文件，如果下一次同一个 consumerId 再开始读的时候会从 position.dat 文件中读取位置，并且从这个位置续读

```java
@Test
public void testResumeConsumer() throws InterruptedException {
    try (SimpleQueue queue = new SimpleQueue(config)) {
        for (int i = 0; i < 10; i++) {
            String key = "key" + i;
            String content = "content" + i;
            queue.offer(key, content);
        }
        // 确保消息全部写入
        Thread.sleep(100);
        String consumerId = "consumer1";
        // 从最早的位置开始读，（默认是最新的，如果不用First 会读不到消息），
        // 这里我们是需要关闭所以用了try-with-resources, 一般不需要在queue 这释放
        try (IConsumer consumer1 = queue.getConsumer(consumerId, ConsumeFromWhere.FIRST)) {
            // 确保消息写入缓存
            Thread.sleep(100);
            List<QueueMessage> queueMessages = consumer1.batchTake(5);
            assertEquals(5, queueMessages.size());
            QueueMessage lastOne = queueMessages.get(queueMessages.size() - 1);
            assertEquals("key4", lastOne.getMessageKey());
            consumer1.ack(queueMessages);
            // 确保位置写入
            Thread.sleep(100);
        }
        // 上面的消费者关闭以后，新开一个同一个consumerId消费者断点续读
        try (IConsumer consumer1 = queue.getConsumer(consumerId, ConsumeFromWhere.FIRST)) {
            // 确保消息写入缓存
            Thread.sleep(100);
            List<QueueMessage> queueMessages = consumer1.batchTake(3);
            assertEquals(3, queueMessages.size());
            QueueMessage lastOne = queueMessages.get(queueMessages.size() - 1);
            assertEquals("key7", lastOne.getMessageKey());
            consumer1.ack(queueMessages);
            // 确保位置写入
            Thread.sleep(100);
        }
    }
}
```

### 重置位点

重置位点应该对 mq 是很重要的一个功能，一般当我们出问题了，希望回放一下数据就需要这个功能

#### 根据 position 重置 (推荐)

这个是性能非常高的一种重置位点方式，可以从 QueueMessage 获取 position, 以为很多操作都是异步的所以这里 demo 会使用 Thread.sleep 等一下

```java
@Test
public void testMoveToPosition() throws InterruptedException {
    try (SimpleQueue queue = new SimpleQueue(config)) {
        for (int i = 0; i < 10; i++) {
            String key = "key" + i;
            String content = "content" + i;
            queue.offer(key, content);
            Thread.sleep(100);
        }
        IConsumer consumer1 = queue.getConsumer("consumer1", ConsumeFromWhere.FIRST);
        // 确保消费者缓存填充完毕
        Thread.sleep(100);
        List<QueueMessage> queueMessages = consumer1.batchTake(10);
        // 尝试取第三位
        QueueMessage example = queueMessages.get(3);
        consumer1.ack(queueMessages);
        // 确保ack位点提交
        Thread.sleep(100);
        // 移动位置到第三个消息位置
        boolean moveToResult = consumer1.moveToPosition(example.getPosition());
        assertTrue(moveToResult);
        // 移动完成以后接着读
        QueueMessage takeMessage = consumer1.take();
        assertEquals("key3", takeMessage.getMessageKey());
        assertEquals(example.getPosition(), takeMessage.getPosition());
        assertEquals(example.getMessageKey(), takeMessage.getMessageKey());
    }
}

```

### 根据时间戳重置（探索阶段）

通过位置重置位点是一个非常高效的做法，但是往往我们有时候不知道位点，我们想重置到某个时间点上，因为原生不支持，所以先用遍历的方法找到位置，可能会性能上会有些延迟，性能上应该还可以优化

```java
@Test
public void testMoveToTimestamp() throws InterruptedException {
    try (SimpleQueue queue = new SimpleQueue(config)) {
        for (int i = 0; i < 10; i++) {
            String key = "key" + i;
            String content = "content" + i;
            queue.offer(key, content);
            Thread.sleep(100);
        }
        IConsumer consumer1 = queue.getConsumer("consumer1", ConsumeFromWhere.FIRST);
        // 确保消费者缓存填充完毕
        Thread.sleep(100);
        List<QueueMessage> queueMessages = consumer1.batchTake(10);
        // 尝试取第三位
        QueueMessage example = queueMessages.get(3);
        consumer1.ack(queueMessages);
        // 确保ack位点提交
        Thread.sleep(100);
        // 移动位置到第三个消息位置, 这次我们取的是时间
        long writeTime = example.getWriteTime();
        boolean moveToResult = consumer1.moveToTimestamp(writeTime);
        assertTrue(moveToResult);
        // 移动完成以后接着读
        QueueMessage takeMessage = consumer1.take();
        assertEquals("key3", takeMessage.getMessageKey());
        assertEquals(example.getPosition(), takeMessage.getPosition());
        assertEquals(example.getMessageKey(), takeMessage.getMessageKey());
    }
}
```

## 本地 IPC 通信

local-queue 不光支持多线程操作，而且支持本地多进程操作，我们知道 mmap 就是进程间通信的一种。这里我们不能再使用 SimpleQueue 要使用比较底层的封装 SimpleProducer 和 SimpleConsumer, 和 kafka/rocketmq 类似，一个进程生产数据，另外一个进程消费数据。

生产者程序

```java
public class Program {

    public static void main(String[] args) {
        SimpleProducerConfig config = new SimpleProducerConfig.Builder()
                .setDataDir(new File("./test"))
                // 保留7天数据
                .setKeepDays(7)
                .build();

        try (SimpleProducer producer = new SimpleProducer(config)) {
            while (true) {
                String formattedDateTime = LocalDateTime.now().format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);
                String message = "Produce message at: " + formattedDateTime;
                producer.offer(message);
            }
        }
    }
}
```

消费者程序

```java
public class Program {

    public static void main(String[] args) throws InterruptedException {
        SimpleConsumerConfig config = new SimpleConsumerConfig.Builder()
                .setDataDir(new File("./test"))
                // 自定义保存位置文件
                .setPositionFile(new File("./test/myPosition.dat"))
                .setConsumerId("consumerId")
                // 如果没有历史位置，一般从最新位置消费，从头消费会有瞬发消息，所以默认也是ConsumeFromWhere.LAST
                .setConsumeFromWhere(ConsumeFromWhere.LAST)
                .build();


        try (SimpleConsumer consumer = new SimpleConsumer(config)) {
            while (true) {
                // 阻塞式获取
                QueueMessage message = consumer.take();
                System.out.println("[receive:]" + message.getContent());
                consumer.ack(message);
            }
        }
    }
}
```

打印出来结果

```bash
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
[receive:]Produce message at: 2025-01-31T18:34:11.588
...
```

## 结尾

local-queue 刚才完成了一个初版，还有很多问题需要优化，大家如果有问题欢迎提出
