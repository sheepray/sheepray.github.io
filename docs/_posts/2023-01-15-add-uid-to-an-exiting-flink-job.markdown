---
layout: post
title:  "Add uid to existing Flink job operators"
date:   2023-01-15 20:00:00 +0200
categories: Flink stateful operator uid
---

<style type="text/css">
  pre {
      white-space: pre-wrap;
  }
</style>

# Problem
I have a Flink application which consumes from 3 kafka sources (source **A**, **B**, **C**) and sinks to ClickHouse. Upon a feature request, I need to remove the kafka source **A** since it is no longer needed. So I have removed it from the code and tried to start the Flink job from the previous savepoint. But it has failed with the following error:
{% highlight scala %}
Cannot map checkpoint/savepoint state for operator 1cfeb06dbkil2d5556a9ajfdb2025f09 to the new program, because the operator is not available in the new program. If you want to allow to skip this, you can set the --allowNonRestoredState option on the CLI.
{% endhighlight %}
In the beginning, I thought it was caused by the removal of the FlinkKafkaConsumer of the source **A**. So I start the job again with `--allowNonRestoredState` set to `true`. This time the job starts fine.

After the job is started, one metric has caught my eye which was the event latency for the kafka consumer of the source **B**. It jumps from NRT to few days which is the retention period of that particular Kafka topic. Then I realize that the state of the FlinkKafkaConsumer for the source **B** was not restored, which contains the committed offset for each Kafka partitions. Since its state is not restored, it starts to consuming from the earliest (the default behaviour).

# Why
Why it is not restored? Because the stream graph has changed due to the removal of the source A and in turns has assigned a new operator id to the FlinkKafkaConsumer operator of the source B. How to avoid the issue in the future? Assign an uid to each stateful operators. But what if I want to make a change to the stream graph again in the future?

# Solution
Since adding uid now will still causes the same problem, we can use `.setUidHash(...)` function to explictly assign it with the auto-generated one. To get the auto-generated uid hash for all the stateful operators, I added `env.getStreamGraph().getJobGraph().getVertices()` before calling `env.execute()` and run the Flink job **locally** with a breakpoint over the line. Once it hits the breakpint, all the job vertices were shown in the debugger. Under each job vertice, it has operator id hashes as an array in the **reverse** order.

# Result
I managed to redeployed the updated Flink job in staging/preprod/prod envs with those operator states restored correctly.

# Conclusion
* when first creating the FLink job, set uid to all state operators.
* use `--allowNonRestoredState=true` cautiously.

# Note
```
-> StreamGraph (logical execution plan)
-> JobGraph (optimized execution plan)
-> ExecutionGraph (physical execution plan)
```

After optimization, each JobVertex contains an operator chain.
