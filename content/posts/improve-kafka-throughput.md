+++
title = "Improve Kafka throughput"
author = ["KK"]
date = 2021-05-28T00:57:00+08:00
lastmod = 2021-05-28T00:57:58+08:00
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

Kafka is a high-performance and scalable messaging system. Sometimes when handling big data. The default configuration may limit the maximum performance. In this article, I'll explain how messages are generate and saved in Kafka, and how to improve performance by changing configuration.


## How does Producer Send Messages? {#how-does-producer-send-messages}

In short, messages will assembled into batches (named `RecordBatch`) and send to broker.

The producer manages some internal queues, and each queue contains `RecordBatch` that will send to one broker. When calling `send` method, the producer will look into the internal queue and try to append this message to `RecordBatch` which is smaller than `batch.size` (default value is 16KB) or create new `RecordBatch`.

There is also a thread in producer which is responsible for turning `RecordBatch` into requests and send to broker.


## how are Records Saved? {#how-are-records-saved}

The details can be found [here](https://kafka.apache.org/documentation/#messageformat).

Here are some important properties in `RecordBatch` are: `batch_lenth`, `compresstion_type`, `CRC`, `timestamp` and, of course, the `List(Record)`.

Each `Record` consists of `length`, `timestamp_delta`, `key(byte)`, `value(byte)` etc.

When look into the kafka topic data directory, you may find files like this:

```nil
00000000000000000000.log
00000000000000000000.index
00000000000000000000.timeindex
00000000000000000035.log
00000000000000000035.index
00000000000000000035.timeindex
```

Kafka saves each partition as segments. When new record comes, it append to the active segment. If the segment's size limit is reached, a new segment is created as becomes the active segment. Segments are named by the offset of its first record, so the segments' names are incremental.

Furthermore, the segment divided into three kinds of file:  log file, index file and timeindex file.

-   The `log` file contains the actual data
-   The `index` file contains the record's relative offset and it's physical position in the log file. This makes the look up complexity for specific offset record to `O(1)`.
-   The `timeindex` file contains the records timestamp and its offset data.


## How does Consumer pull messages? {#how-does-consumer-pull-messages}

Consumer keeps reading data from broker, and decompress data if necessary. It will put data into a internal queue and return the target number of records to client.

`max.poll.records` (default values is 500) means the maximum number of records returned in a single call to poll()l.

`fetch.min.bytes` (default value is 1) means the minimum amount of data the broker should return from a fetch request. If insufficient data is available, the server will wait up to `fetch.max.wait.ms` ms and accumulate the data before answering the request.


## How to Improve Performance {#how-to-improve-performance}


### Increase Socket Buffer {#increase-socket-buffer}

The default socket buffer value in Java client is too small for high-throughput environment.
`socket.receive.buffer.bytes` and `socket.send.buffer.bytes` is the `SO_RCVBUFF` and `SO_SNDBUFF` buffer for socket connections respectively, which is default to 100KB. I recommend to set it to a bigger value or `-1` to use the OS default value.


### batch.size, linger.ms and buffer.memory {#batch-dot-size-linger-dot-ms-and-buffer-dot-memory}

As mentioned before, producer always send message as `RecordBatch`. Each batch should be smaller than `batch.size` (default value is 16KB). Increasing `batch.size` will not only reduce the TCP request to broker, but also lead to better compression ratio when compression is enabled.

The producer groups together any records that arrive in between request transmissions into a single batched request. If the system load is low, there will be only one message in the record batch. `linger.ms`'s default value is 0, this means broker will send message as quick as possible. If it's greater than 0, then producer will wait for more messages until the batch size is greater than `batch.size` or `current time - last transmissions > linger.ms`. Increasing this value to make sure each batch is close to `batch.size` and reducing the number of requests to be sent.

The `buffer.memory` (default value is 32MB) controls the total amount of memory available to the producer for buffering. If records are sent faster than they can be transmitted to the server then this buffer space will be exhausted. When the buffer space is exhausted additional send calls will block.


### Compression.type {#compression-dot-type}

As the throughput keep growing, bandwidth may become bottleneck. It's easy to tackle this by add `compresstion.type` param in producer. Once it is configured, the producer will compressed the `RecordBatch` before sending it to broker. If the records are texts, the compression ratio should be high and bandwidth usage will be significantly decreased.

There are two kind of `compresstion.type`, topic level and producer level.

If you set `compresstion.type` in producer, the producer will compress the records and send it to broker.

The topic also has a `compresstion.type` configuration. It can be set as `gizp`, `snappy`, `lz4`, `zstd`, `uncompressed`, and `producer`. The default value is `producer`, which means the broker will keep the original data send from the producer. Otherwise, the broker will convert the data to target `compresstion.type`.

How to choose compression type? According to cloudflare's test result in [Squeezing the firehose: getting the most from Kafka compression](https://blog.cloudflare.com/squeezing-the-firehose/):

| type   | CPU ration | Compression ratio |
|--------|------------|-------------------|
| None   | 1x         | 1x                |
| Gzip   | 10.14x     | 3.58x             |
| Snappy | 1.61x      | 2.35x             |
| Lz4    | 2.51x      | 1.81x             |

`Gzip` has best compression ratio but take lots of CPU time. `Snappy` keeps a balance between the CPU time and space. The new compression type `zstd` produce larger compression ratio than `Snappy` with the cost of a little more CPU time.

Ref:

1.  [Kafka message format](https://kafka.apache.org/documentation/#messageformat)
2.  [Kafka高性能探秘](https://juejin.cn/post/6844903632521920519)
3.  [Exploit Apache Kafka’s Message Format to Save Storage and Bandwidth](https://medium.com/swlh/exploit-apache-kafkas-message-format-to-save-storage-and-bandwidth-7e0c533edf26)
4.  [Consuming big messages from Kafka](https://ibmstreams.github.io/streamsx.kafka/docs/user/ConsumingBigMessages/)
5.  [How does max.poll.records affect the consumer poll](https://stackoverflow.com/questions/53308986/how-does-max-poll-records-affect-the-consumer-poll)
6.  [A Practical Introduction to Kafka Storage Internals](https://medium.com/@durgaswaroop/a-practical-introduction-to-kafka-storage-internals-d5b544f6925f)
7.  [Kafka message codec - compress and decompress](https://stackoverflow.com/questions/19890894/kafka-message-codec-compress-and-decompress)
8.  [20 Best Practices for Working With Apache Kafka at Scale](https://dzone.com/articles/20-best-practices-for-working-with-apache-kafka-at)
9.  [Kakfa Document](https://kafka.apache.org/documentation/#producerconfigs)
10. [kafka-python KafkaProducer](https://kafka-python.readthedocs.io/en/master/apidoc/KafkaProducer.html)
11. [Deep Dive Into Apache Kafka | Storage Internals](https://rohithsankepally.github.io/Kafka-Storage-Internals/)
