# Spark OOM Error

I spend the better part of yesterday afternoon debugging the following error coming from a simple spark/scala job running on my local machine.
Of course, now that I've solved the problem, I can't seem to reproduce the error even by going back in time through my commit history.
This leads me to believe that the environment variables I was setting to adjust the executor memory were the solution, but took a while to be picked up by sbt.

# Code and Commands
Sample [scala job](https://github.com/harterrt/telemetry-batch-view/blob/9a78ce7977e1e75d75391c78c144a85c9358d7df/src/main/scala/com/mozilla/telemetry/views/CrossSectionalView.scala).
(Note: main() should not be commented out.

Sample command:
```
sbt "run-main com.mozilla.telemetry.views.CrossSectionalView" &> local_err.txt
```

# Possible Solutions
If you see this error again, consider the following:
* Upgrade from spark 1.6.1 to 2.0.0 - Do the errors still persist?
* Add more memory to the executor with something like: `sparkConf.set("spark.executor.memory", "4g")`
* Don't use a HiveContext to read a local parquet dataset - instead use a SQLContext.

In general, keep better records and tag commits with what sort of error your getting.

# Error
The complete error is in this directory with name oom.txt.

```
16/08/15 10:21:21 INFO InternalParquetRecordReader: RecordReader initialized will read a total of 0 records.
16/08/15 10:21:21 INFO Executor: Finished task 4.0 in stage 1.0 (TID 8). 2543 bytes result sent to driver
16/08/15 10:21:21 INFO TaskSetManager: Finished task 4.0 in stage 1.0 (TID 8) in 2674 ms on localhost (2/5)
16/08/15 10:21:21 INFO GenerateSafeProjection: Code generated in 52.107492 ms
16/08/15 10:21:21 INFO GenerateUnsafeProjection: Code generated in 27.6363 ms
16/08/15 10:21:34 WARN TaskMemoryManager: leak 16.3 MB memory from org.apache.spark.unsafe.map.BytesToBytesMap@46a127aa
16/08/15 10:21:34 ERROR Executor: Managed memory leak detected; size = 17039360 bytes, TID = 7
16/08/15 10:21:34 ERROR Executor: Exception in task 3.0 in stage 1.0 (TID 7)
java.lang.OutOfMemoryError: Java heap space
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:45)
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:60)
	at org.apache.spark.sql.catalyst.expressions.codegen.UnsafeArrayWriter.initialize(UnsafeArrayWriter.java:48)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply26646_63$(Unknown Source)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRowConverter.currentRecord(CatalystRowConverter.scala:173)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:38)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:32)
	at org.apache.parquet.io.RecordReaderImplementation.read(RecordReaderImplementation.java:419)
	at org.apache.parquet.hadoop.InternalParquetRecordReader.nextKeyValue(InternalParquetRecordReader.java:209)
	at org.apache.parquet.hadoop.ParquetRecordReader.nextKeyValue(ParquetRecordReader.java:201)
	at org.apache.spark.rdd.SqlNewHadoopRDD$$anon$1.hasNext(SqlNewHadoopRDD.scala:194)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.processInputs(TungstenAggregationIterator.scala:504)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.<init>(TungstenAggregationIterator.scala:686)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:95)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:86)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:73)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:41)
	at org.apache.spark.scheduler.Task.run(Task.scala:89)
16/08/15 10:21:34 WARN TaskSetManager: Lost task 3.0 in stage 1.0 (TID 7, localhost): java.lang.OutOfMemoryError: Java heap space
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:45)
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:60)
	at org.apache.spark.sql.catalyst.expressions.codegen.UnsafeArrayWriter.initialize(UnsafeArrayWriter.java:48)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply26646_63$(Unknown Source)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRowConverter.currentRecord(CatalystRowConverter.scala:173)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:38)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:32)
	at org.apache.parquet.io.RecordReaderImplementation.read(RecordReaderImplementation.java:419)
	at org.apache.parquet.hadoop.InternalParquetRecordReader.nextKeyValue(InternalParquetRecordReader.java:209)
	at org.apache.parquet.hadoop.ParquetRecordReader.nextKeyValue(ParquetRecordReader.java:201)
	at org.apache.spark.rdd.SqlNewHadoopRDD$$anon$1.hasNext(SqlNewHadoopRDD.scala:194)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.processInputs(TungstenAggregationIterator.scala:504)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.<init>(TungstenAggregationIterator.scala:686)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:95)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:86)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:73)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:41)
	at org.apache.spark.scheduler.Task.run(Task.scala:89)

16/08/15 10:21:34 ERROR TaskSetManager: Task 3 in stage 1.0 failed 1 times; aborting job
16/08/15 10:21:34 ERROR SparkUncaughtExceptionHandler: Uncaught exception in thread Thread[Executor task launch worker-1,5,run-main-group-0]
java.lang.OutOfMemoryError: Java heap space
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:45)
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:60)
	at org.apache.spark.sql.catalyst.expressions.codegen.UnsafeArrayWriter.initialize(UnsafeArrayWriter.java:48)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply26646_63$(Unknown Source)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRowConverter.currentRecord(CatalystRowConverter.scala:173)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:38)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:32)
	at org.apache.parquet.io.RecordReaderImplementation.read(RecordReaderImplementation.java:419)
	at org.apache.parquet.hadoop.InternalParquetRecordReader.nextKeyValue(InternalParquetRecordReader.java:209)
	at org.apache.parquet.hadoop.ParquetRecordReader.nextKeyValue(ParquetRecordReader.java:201)
	at org.apache.spark.rdd.SqlNewHadoopRDD$$anon$1.hasNext(SqlNewHadoopRDD.scala:194)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.processInputs(TungstenAggregationIterator.scala:504)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.<init>(TungstenAggregationIterator.scala:686)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:95)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:86)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:73)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:41)
	at org.apache.spark.scheduler.Task.run(Task.scala:89)
16/08/15 10:21:34 INFO TaskSchedulerImpl: Cancelling stage 1
16/08/15 10:21:34 INFO Executor: Executor is trying to kill task 1.0 in stage 1.0 (TID 5)
16/08/15 10:21:34 INFO Executor: Executor is trying to kill task 0.0 in stage 1.0 (TID 4)
16/08/15 10:21:34 INFO TaskSchedulerImpl: Stage 1 was cancelled
16/08/15 10:21:34 INFO DAGScheduler: ShuffleMapStage 1 (count at CrossSectionalView.scala:82) failed in 24.931 s
16/08/15 10:21:34 INFO DAGScheduler: Job 1 failed: count at CrossSectionalView.scala:82, took 24.972275 s
[0m[[31merror[0m] [0m(run-main-0) org.apache.spark.SparkException: Job aborted due to stage failure: Task 3 in stage 1.0 failed 1 times, most recent failure: Lost task 3.0 in stage 1.0 (TID 7, localhost): java.lang.OutOfMemoryError: Java heap space[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:45)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:60)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.catalyst.expressions.codegen.UnsafeArrayWriter.initialize(UnsafeArrayWriter.java:48)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply26646_63$(Unknown Source)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.datasources.parquet.CatalystRowConverter.currentRecord(CatalystRowConverter.scala:173)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:38)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:32)[0m
[0m[[31merror[0m] [0m	at org.apache.parquet.io.RecordReaderImplementation.read(RecordReaderImplementation.java:419)[0m
[0m[[31merror[0m] [0m	at org.apache.parquet.hadoop.InternalParquetRecordReader.nextKeyValue(InternalParquetRecordReader.java:209)[0m
[0m[[31merror[0m] [0m	at org.apache.parquet.hadoop.ParquetRecordReader.nextKeyValue(ParquetRecordReader.java:201)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.SqlNewHadoopRDD$$anon$1.hasNext(SqlNewHadoopRDD.scala:194)[0m
[0m[[31merror[0m] [0m	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)[0m
[0m[[31merror[0m] [0m	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)[0m
[0m[[31merror[0m] [0m	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)[0m
[0m[[31merror[0m] [0m	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)[0m16/08/15 10:21:34 ERROR ContextCleaner: Error in cleaning thread
java.lang.InterruptedException
	at java.lang.Object.wait(Native Method)
	at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:135)
	at org.apache.spark.ContextCleaner$$anonfun$org$apache$spark$ContextCleaner$$keepCleaning$1.apply$mcV$sp(ContextCleaner.scala:176)
	at org.apache.spark.util.Utils$.tryOrStopSparkContext(Utils.scala:1180)
	at org.apache.spark.ContextCleaner.org$apache$spark$ContextCleaner$$keepCleaning(ContextCleaner.scala:173)
	at org.apache.spark.ContextCleaner$$anon$3.run(ContextCleaner.scala:68)

[0m[[31merror[0m] [0m	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.processInputs(TungstenAggregationIterator.scala:504)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.<init>(TungstenAggregationIterator.scala:686)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:95)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:86)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:73)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:41)[0m
[0m[[31merror[0m] [0m	at org.apache.spark.scheduler.Task.run(Task.scala:89)[0m
[0m[[31merror[0m] [0m[0m
[0m[[31merror[0m] [0mDriver stacktrace:[0m
16/08/15 10:21:34 ERROR Utils: uncaught error in thread SparkListenerBus, stopping SparkContext
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:996)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1303)
	at java.util.concurrent.Semaphore.acquire(Semaphore.java:317)
	at org.apache.spark.util.AsynchronousListenerBus$$anon$1$$anonfun$run$1$$anonfun$apply$mcV$sp$1.apply$mcV$sp(AsynchronousListenerBus.scala:66)
	at org.apache.spark.util.AsynchronousListenerBus$$anon$1$$anonfun$run$1$$anonfun$apply$mcV$sp$1.apply(AsynchronousListenerBus.scala:65)
	at org.apache.spark.util.AsynchronousListenerBus$$anon$1$$anonfun$run$1$$anonfun$apply$mcV$sp$1.apply(AsynchronousListenerBus.scala:65)
	at scala.util.DynamicVariable.withValue(DynamicVariable.scala:57)
	at org.apache.spark.util.AsynchronousListenerBus$$anon$1$$anonfun$run$1.apply$mcV$sp(AsynchronousListenerBus.scala:64)
	at org.apache.spark.util.Utils$.tryOrStopSparkContext(Utils.scala:1180)
	at org.apache.spark.util.AsynchronousListenerBus$$anon$1.run(AsynchronousListenerBus.scala:63)
org.apache.spark.SparkException: Job aborted due to stage failure: Task 3 in stage 1.0 failed 1 times, most recent failure: Lost task 3.0 in stage 1.0 (TID 7, localhost): java.lang.OutOfMemoryError: Java heap space
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:45)
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:60)
	at org.apache.spark.sql.catalyst.expressions.codegen.UnsafeArrayWriter.initialize(UnsafeArrayWriter.java:48)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply26646_63$(Unknown Source)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRowConverter.currentRecord(CatalystRowConverter.scala:173)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:38)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:32)
	at org.apache.parquet.io.RecordReaderImplementation.read(RecordReaderImplementation.java:419)
	at org.apache.parquet.hadoop.InternalParquetRecordReader.nextKeyValue(InternalParquetRecordReader.java:209)
	at org.apache.parquet.hadoop.ParquetRecordReader.nextKeyValue(ParquetRecordReader.java:201)
	at org.apache.spark.rdd.SqlNewHadoopRDD$$anon$1.hasNext(SqlNewHadoopRDD.scala:194)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.processInputs(TungstenAggregationIterator.scala:504)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.<init>(TungstenAggregationIterator.scala:686)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:95)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:86)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:73)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:41)
	at org.apache.spark.scheduler.Task.run(Task.scala:89)

Driver stacktrace:
	at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1431)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1419)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1418)
	at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
	at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:47)
	at org.apache.spark.scheduler.DAGScheduler.abortStage(DAGScheduler.scala:1418)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:799)
	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:799)
	at scala.Option.foreach(Option.scala:236)
	at org.apache.spark.scheduler.DAGScheduler.handleTaskSetFailed(DAGScheduler.scala:799)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.doOnReceive(DAGScheduler.scala:1640)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:1599)
	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:1588)
	at org.apache.spark.util.EventLoop$$anon$1.run(EventLoop.scala:48)
	at org.apache.spark.scheduler.DAGScheduler.runJob(DAGScheduler.scala:620)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1832)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1845)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1858)
	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1929)
	at org.apache.spark.rdd.RDD$$anonfun$collect$1.apply(RDD.scala:927)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:150)
	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:111)
	at org.apache.spark.rdd.RDD.withScope(RDD.scala:316)
	at org.apache.spark.rdd.RDD.collect(RDD.scala:926)
	at org.apache.spark.sql.execution.SparkPlan.executeCollect(SparkPlan.scala:166)
	at org.apache.spark.sql.execution.SparkPlan.executeCollectPublic(SparkPlan.scala:174)
	at org.apache.spark.sql.DataFrame$$anonfun$org$apache$spark$sql$DataFrame$$execute$1$1.apply(DataFrame.scala:1499)
	at org.apache.spark.sql.DataFrame$$anonfun$org$apache$spark$sql$DataFrame$$execute$1$1.apply(DataFrame.scala:1499)
	at org.apache.spark.sql.execution.SQLExecution$.withNewExecutionId(SQLExecution.scala:56)
	at org.apache.spark.sql.DataFrame.withNewExecutionId(DataFrame.scala:2086)
	at org.apache.spark.sql.DataFrame.org$apache$spark$sql$DataFrame$$execute$1(DataFrame.scala:1498)
	at org.apache.spark.sql.DataFrame.org$apache$spark$sql$DataFrame$$collect(DataFrame.scala:1505)
	at org.apache.spark.sql.DataFrame$$anonfun$count$1.apply(DataFrame.scala:1515)
	at org.apache.spark.sql.DataFrame$$anonfun$count$1.apply(DataFrame.scala:1514)
	at org.apache.spark.sql.DataFrame.withCallback(DataFrame.scala:2099)
	at org.apache.spark.sql.DataFrame.count(DataFrame.scala:1514)
	at org.apache.spark.sql.Dataset.count(Dataset.scala:176)
	at com.mozilla.telemetry.views.CrossSectionalView$.main(CrossSectionalView.scala:82)
	at com.mozilla.telemetry.views.CrossSectionalView.main(CrossSectionalView.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
Caused by: java.lang.OutOfMemoryError: Java heap space
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:45)
	at org.apache.spark.sql.catalyst.expressions.codegen.BufferHolder.grow(BufferHolder.java:60)
	at org.apache.spark.sql.catalyst.expressions.codegen.UnsafeArrayWriter.initialize(UnsafeArrayWriter.java:48)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply26646_63$(Unknown Source)
	at org.apache.spark.sql.catalyst.expressions.GeneratedClass$SpecificUnsafeProjection.apply(Unknown Source)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRowConverter.currentRecord(CatalystRowConverter.scala:173)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:38)
	at org.apache.spark.sql.execution.datasources.parquet.CatalystRecordMaterializer.getCurrentRecord(CatalystRecordMaterializer.scala:32)
	at org.apache.parquet.io.RecordReaderImplementation.read(RecordReaderImplementation.java:419)
	at org.apache.parquet.hadoop.InternalParquetRecordReader.nextKeyValue(InternalParquetRecordReader.java:209)
	at org.apache.parquet.hadoop.ParquetRecordReader.nextKeyValue(ParquetRecordReader.java:201)
	at org.apache.spark.rdd.SqlNewHadoopRDD$$anon$1.hasNext(SqlNewHadoopRDD.scala:194)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at scala.collection.Iterator$$anon$11.hasNext(Iterator.scala:327)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.processInputs(TungstenAggregationIterator.scala:504)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregationIterator.<init>(TungstenAggregationIterator.scala:686)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:95)
	at org.apache.spark.sql.execution.aggregate.TungstenAggregate$$anonfun$doExecute$1$$anonfun$2.apply(TungstenAggregate.scala:86)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.RDD$$anonfun$mapPartitions$1$$anonfun$apply$20.apply(RDD.scala:710)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.rdd.MapPartitionsRDD.compute(MapPartitionsRDD.scala:38)
	at org.apache.spark.rdd.RDD.computeOrReadCheckpoint(RDD.scala:306)
	at org.apache.spark.rdd.RDD.iterator(RDD.scala:270)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:73)
	at org.apache.spark.scheduler.ShuffleMapTask.runTask(ShuffleMapTask.scala:41)
	at org.apache.spark.scheduler.Task.run(Task.scala:89)
[0m[[31mtrace[0m] [0mStack trace suppressed: run [34mlast compile:runMain[0m for the full output.[0m
```
