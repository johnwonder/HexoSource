title: kafka-consumergroup
date: 2016-10-11 20:45:59
tags: kafka consumergroup
---

## kafka 查看当前有多少group

```
kafka-run-class.bat kafka.admin.ConsumerGroupCommand --zookeeper localhost:2181 --list
```

## kafka 查看group情况

```
kafka-run-class.bat kafka.admin.ConsumerGroupCommand --zookeeper localhost:2181 --describe --group console-consumer-8047

```

[参考文档](http://kafka.apache.org/documentation#basic_ops_consumer_group)
