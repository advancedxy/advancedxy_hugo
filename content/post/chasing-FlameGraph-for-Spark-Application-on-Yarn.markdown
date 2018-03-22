---
title: "Chasing FlameGraph for Spark Application on Yarn"
date: 2018-03-22T15:20:21+08:00
description: FlameGraph and Spark Application
tags: ["flamegraph", "spark"]
---

When profiling with distributed applications, old fashion profilers like `YourKit`, `VisualVM`, and `JProfiler` etc doesn't work well as they require a lot of user interaction. It's infeasible to interact many instances of one distributed application simultaneously. Hence we present a new approach enabled by [async-profiler](https://github.com/jvm-profiling-tools/async-profiler). 
 
## Introduction of FlameGraph
Let's talk a bit about FlameGraph first. [FlameGraph](http://www.brendangregg.com/flamegraphs.html) is a nifty debugging tool to determine where CPU time is being spent. The general idea of how FlameGraph works is well explained by [a post from Ihor Bobak](http://ihorbobak.com/index.php/2015/08/05/cluster-profiling/). The explanation is quoted as below:

Imagine you have some java class with a method A that called method B, B called C, C called D. We’ll denote a call by an arrow, so A->B->C->D. Every 10 milliseconds a special javaagent (embedded into JVM process) gets all stacktraces of all the threads of a process. For example, it may “see” that the main thread of our app is calling A->B->C (method C is executing now), at another moment of time it observes that A->B->C->D is the current stacktrace (method D is executing now), at the next moment of time it sees that it is in A->B->C->D again, and at the next moment we are in A->B (so that the code of B is executing).
If we put all stacktraces vertically, we can visualize it like this:

|        | D       | D       |         |
|--------|---------|---------|---------|
| C      | C       | C       |         |
| B      | B       | B       | B       |
| A      | k       | A       | A       |
| 0th ms | 10th ms | 20th ms | 30th ms |

The time goes from left to right column, and every stack trace is shown from bottom to top. Therefore, in the method D we spent 20 milliseconds, C was executing just 10 milliseconds on its own, and method B was executing 10 milliseconds. If we group the cells so that the same parent method call are joined, we get this:

<table>
<thead>
<tr>
<th></th>
<th style="background: #ed7d31;">D</th>
<th style="background: #ed7d31;">D</th>
<th></th>
</tr>
</thead>

<tbody>
<tr>
<td style="background: #f7caac;">C</td>
<td style="background: #f7caac;">C</td>
<td style="background: #f7caac;">C</td>
<td></td>
</tr>

<tr style="background: #ffd996;">
<td>B</td>
<td>B</td>
<td>B</td>
<td>B</td>
</tr>

<tr style="background: #fff2cc;">
<td>A</td>
<td>k</td>
<td>A</td>
<td>A</td>
</tr>

<tr style="background: #d9d9d9;">
<td>0th ms</td>
<td>10th ms</td>
<td>20th ms</td>
<td>30th ms</td>
</tr>
</tbody>
</table>

To summarize, the flame graph is an approximate visualization of what is going on: **which class/method is executing how much % of CPU time**. From the diagram above we can conclude that:

1. Method D was executing 50% of the whole CPU time;
2. Method C was executing 75% of CPU time, among which 25% of time it was executing its own code, and the rest 50% it spent in the call of D;
3. Method B was executing 100% of CPU time (same as A), but just 25% of time method B was executing itself, the rest of time it was calling other methods.

### Flame Graph Interpretation
![how to read FlameGraph](/imgs/how_to_read_flamegraph.jpg)

## Generating FlameGraph using async-profiler

##### Get async-profiler(latest release is 1.2, [releases](https://github.com/jvm-profiling-tools/async-profiler/releases))

```
mkdir -p async-profiler_workspace && cd async-profiler_workspace

wget https://github.com/jvm-profiling-tools/async-profiler/releases/download/v1.2/async-profiler-1.2-linux-x64.zip -O aysnc-profiler-1.2-linux-x64.zip
```

The directory structure is:
```
unzip -l async-profiler-1.2-linux-x64.zip
Archive:  async-profiler-1.2-linux-x64.zip
  Length     Date   Time    Name
 --------    ----   ----    ----
        0  03-06-18 01:56   build/
   136937  03-06-18 01:56   build/libasyncProfiler.so
     2489  03-06-18 01:56   build/async-profiler.jar
    16882  03-06-18 01:56   build/jattach
     5490  03-01-18 22:24   profiler.sh
    11357  12-11-17 03:48   LICENSE
     1259  03-06-18 01:08   CHANGELOG.md
    11419  03-06-18 01:26   README.md
 --------                   -------
   185833                   8 files
```

##### Prepare async-profiler as a yarn cachearchive

```
# $HADOOP_HOME should already be set
$HADOOP_HOME/bin/hadoop fs -put async-profiler-1.2-linux-x64.zip hdfs:///path/to/your/tooling/archives/

# setRepliation if there are many hosts in your cluster, num of replications could be 10 or larger depends on cluster size
$HADOOP_HOME/bin/hadoop fs -setRep 10 hdfs:///path/to/your/tooling/archives/async-profiler-1.2-linux-x64.zip 
```
##### Starting Application with async-profiler
1. Editing `conf/spark-defaults.conf`

    ```
    spark.yarn.dist.archives hdfs:///path/to/your/tooling/archives/async-profiler-1.2-linux-x64.zip#async-profiler-1.2
    spark.executor.extraJavaOptions -agentpath:./async-profiler-1.2/build/libasyncProfiler.so=start,interval=10000000,svg=samples,event=cpu,file=./app_flamegraph.svg
    ```
   Or you can specify archives and extraJavaOptions as arguments passed to `spark-submit`
   
2. Submit your Spark Application as normal

The FlameGraph of each executor will be generated in the yarn application's work_dir/app_flamegraph.svg

Some notes about using `async-profiler`:

1. You should refer the [code](https://github.com/jvm-profiling-tools/async-profiler/blob/master/src/arguments.cpp) about native agent options.
2. `async-profiler` supports multiple profile events: `cpu`, `alloc`, `lock` etc. You should try it yourself.
3. After launching spark application, `async-profiler` can be stopped, started or dumps profile result on demand. See [Basic Usage](https://github.com/jvm-profiling-tools/async-profiler#basic-usage) for help
4. If your Spark application loads other native library during execution, you can use its Java API and Spark's `SparkListener` callback to instruct `async-profiler` to reload after loading other libraries. Otherwise, `async-profiler` cannot identify symbols from these libraries.
5. Sometimes, you may want to increase profiling interval, see my [issue report](https://github.com/jvm-profiling-tools/async-profiler/issues/97) for the reason.


## Future work and limitations

### Limitations
1. The Java API of `async-profiler` is pretty limited: [issue#93](https://github.com/jvm-profiling-tools/async-profiler/issues/93)

### Future work
1. Improve the Java API with the author while satisfying our internal usage.
2. Since the `async-profiler` supports varies prof events, I plan to integrated it into Spark UI as an opt-in feature. 
   The final version should be similar with [brpc's status page](https://github.com/brpc/brpc/blob/master/docs/en/status.md)
   
## Conclusion and credits
I think it's the simplest approach to generate FlameGraph for distributed Java Applications by far. Thanks for [Andrei Pangin](https://github.com/apangin)'s great work. The approach I presented here should be general enough
for any yarn-based applications. It should be applicable to other distributed applications too.

During my usage of `async-profiler` and the writing of this post, I referred the following blogs/articles(many thanks to their work and writeup):

1. https://gist.github.com/kayousterhout/7008a8ebf2babeedc7ce6f8723fd1bf4
2. http://www.brendangregg.com/flamegraphs.html
3. https://www.paypal-engineering.com/2016/09/08/spark-in-flames-profiling-spark-applications-using-flame-graphs/
4. http://ihorbobak.com/index.php/2015/08/05/cluster-profiling/
5. https://medium.com/netflix-techblog/java-in-flames-e763b3d32166
