---
title: How to guarantee exactly once with Beam(on Flink) for side effects
categories:
- Frontend
- Web
tags:
- stream-processing
- Apache Beam
- Apache Flink
- exactly once
- Kafka
- failure recovery
author_profile: true
classes: wide
---

If you use [Apache Flink](https://flink.apache.org/flink-architecture.html) in your data stream architecture, you probably know about its exactly-once state consistency in case of failures. In this post, we will take a look at how Flink's exactly-once state consistency works, and see why it is not sufficient to provide end-to-end exactly-once guarantees even though the application state is exactly-once consistent.


# Exactly once state consistency

Flink relies on periodically and asynchronously checkpointing the local state to durable storage. First we need to know that the checkpointing mechanism in Flink requires the dat sources to be persistent and replayable such as Kafka.

When everything goes well, the input streams periodically emits checkpoint barriers and broadcast to all connected [parallel tasks](https://ci.apache.org/projects/flink/flink-docs-release-1.1/concepts/concepts.html#parallel-dataflows). You can consider it as a signal to tell operators to starting backup its state to the durable storage, once the state has been backed up, the operator forwards the barrier downstream, till all operators register its snapshot, then a global snapshot will be complete. Note that when saving checkpoint, the consumer offset on the message system such as Kafka is also recorded, this will also be used later in recovery.

Let's say after a global snapshot A has been successfully backed up, before the next global snapshot B can happen, some failure happened, Flink will use the latest (successful) checkpoint to recover, which includes two parts: 1) reverts all operator state to their respective state from the latest checkpoint(in this case from snapshot A), 2)restarts input stream from the latest barrier which there is a snapshot(you can think of it as consumer offset has been reset to when there was a successful snapshot).

This is where interesting thing happen, during above recovery process, Flink might read some message **twice**, after global snapshot A was successfully backed up, message x, y were processed, the failure happened while trying to process message z, to recover from the failure, the successful snapshot a will be used, the operator state and input stream are "reverted" to be like snapshot A, then the processing will continue, which means message x and y will be processed again. But it's important to know that even if message x and y has been processed twice, the mechanism still achieves **exactly-once state consistency**, because when applying lastest checkpoint, the operators are reset to a point where it had not seen message x or y yet.


# End to end exactly once
Now that we understand how exactly-once state consistency works, you might think what about side effects, such as sending out an email, or write to database. That is a valid concern, because Flink's recovery mechanism are not sufficient to provide end to end exactly once guarantees even though the application state is exactly once consistent, for example, if message x and y from above contains info and action to send out email, during failure recovery, these messages will be processed more than once which results in duplicated emails being sent. Therefore other techniques must used, such as idempotent writes, transaction writes etc. To avoid the duplicated email issue, we could save the key of the message that have been processed to a key-value data storage, and use that to check the key, however, since steam processing means unbounded message, it is tricky to manage this key-value checker with large throughput or unbounded time window by yourself. Here I will explain one approach to guarantee end to end exactly once such as only sending out email once with [Apache Beam](https://beam.apache.org/).

Apache Beam is a single programming model for both batch and streaming use cases that can run on multiple execution environments. In this case, Beam will be processing stream on Flink.

## Deduplicate with Beam
There is one API [DeduplicatePerKey](https://beam.apache.org/releases/pydoc/2.25.0/apache_beam.transforms.deduplicate.html#apache_beam.transforms.deduplicate.DeduplicatePerKey) in Beam can be used to deduplicate key value pair over a time domain and threshold. That sounds like a solution to the issues we saw earlier, for example, we can use the Kafka event message uuid as the 'key', then specify a reasonable time duration to apply `DeduplicatePerKey` so that a processed email sending related message won't be processed more than once. Here is the pseudo beam application code:
```
(
 messages
 | "map message to key value pair" >> [map message to key value pair each with an unique key]
 | "deduplicate on that unique key" >> DeduplicatePerKey([reasonable time duration])
 | "you may wanna reset to original message format" >> [use original message format]
 | "process message, send out email" >> [now you can use the message to send out emails etc without worrying about duplication]
)
```

Basically `DeduplicatePerKey` uses states to store seen message id and use a timer to clear state so that it won't grow unboundedly, in terms of how to choose the reasonable window duration, you need to consider your checkpointing interval since essentially in this case we want to deduplicate messages from last successful checkpoint till now.

Now you have end to end exactly once guarantee as well, hope this helps!
