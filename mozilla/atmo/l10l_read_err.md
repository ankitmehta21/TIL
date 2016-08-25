# Problem
I keep running into the following error message, which has a non-intuitive fix. 
When running a telemetry analysis on an ATMO cluster, `sbt` will not have access to any of the columnar / derivative datasets by default.
In this case, it's easier to use spark-submit to run your job.

# Example
Here's an example using a 
[preliminary version of the Cross Sectional Dataset](https://github.com/harterrt/telemetry-batch-view/blob/candidate/src/main/scala/com/mozilla/telemetry/views/CrossSectionalView.scala)
from the telemetry-batch-view library:
```sh
sbt "run-main com.mozilla.telemetry.views.CrossSectionalView --outName=test"
```

Which will fail with something like:
```sh
...
16/08/25 01:06:01 INFO Datastore: The class "org.apache.hadoop.hive.metastore.model.MResourceUri" is tagged as "embedded-only" so does not have its own datastore table.
[error] (run-main-0) java.lang.RuntimeException: java.lang.RuntimeException: The root scratch dir: /tmp/hive on HDFS should be writable. Current permissions are: rwx------
java.lang.RuntimeException: java.lang.RuntimeException: The root scratch dir: /tmp/hive on HDFS should be writable. Current permissions are: rwx------
        at org.apache.hadoop.hive.ql.session.SessionState.start(SessionState.java:522)
        at org.apache.spark.sql.hive.client.ClientWrapper.<init>(ClientWrapper.scala:204)
        at org.apache.spark.sql.hive.client.IsolatedClientLoader.createClient(IsolatedClientLoader.scala:238)
        at org.apache.spark.sql.hive.HiveContext.executionHive$lzycompute(HiveContext.scala:218)
        at org.apache.spark.sql.hive.HiveContext.executionHive(HiveContext.scala:208)
        at org.apache.spark.sql.hive.HiveContext.functionRegistry$lzycompute(HiveContext.scala:462)
        at org.apache.spark.sql.hive.HiveContext.functionRegistry(HiveContext.scala:461)
        at org.apache.spark.sql.UDFRegistration.<init>(UDFRegistration.scala:40)
        at org.apache.spark.sql.SQLContext.<init>(SQLContext.scala:330)
        at org.apache.spark.sql.hive.HiveContext.<init>(HiveContext.scala:90)
        at org.apache.spark.sql.hive.HiveContext.<init>(HiveContext.scala:101)
        at com.mozilla.telemetry.views.CrossSectionalView$.main(CrossSectionalView.scala:72)
        at com.mozilla.telemetry.views.CrossSectionalView.main(CrossSectionalView.scala)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:606)
Caused by: java.lang.RuntimeException: The root scratch dir: /tmp/hive on HDFS should be writable. Current permissions are: rwx------
        at org.apache.hadoop.hive.ql.session.SessionState.createRootHDFSDir(SessionState.java:612)
        at org.apache.hadoop.hive.ql.session.SessionState.createSessionDirs(SessionState.java:554)
...
```

If you change those permissions, like I did, you'll get a new error message (included here because that's probably what you're searching Google with):
```sh
...
16/08/25 01:08:17 INFO SessionState: Created HDFS directory: /tmp/hive/hadoop/455f2125-8bba-4a11-87dd-46b4f46ad019
16/08/25 01:08:17 INFO SessionState: Created local directory: /mnt1/hadoop/455f2125-8bba-4a11-87dd-46b4f46ad019
16/08/25 01:08:17 INFO SessionState: Created HDFS directory: /tmp/hive/hadoop/455f2125-8bba-4a11-87dd-46b4f46ad019/_tmp_space.db
16/08/25 01:08:17 INFO ParseDriver: Parsing command: SELECT * FROM longitudinal
16/08/25 01:08:18 INFO ParseDriver: Parse Completed
16/08/25 01:08:18 INFO HiveMetaStore: 0: get_table : db=default tbl=longitudinal
16/08/25 01:08:18 INFO audit: ugi=hadoop        ip=unknown-ip-addr      cmd=get_table : db=default tbl=longitudinal
[error] (run-main-0) org.apache.spark.sql.AnalysisException: Table not found: longitudinal; line 1 pos 14
org.apache.spark.sql.AnalysisException: Table not found: longitudinal; line 1 pos 14
        at org.apache.spark.sql.catalyst.analysis.package$AnalysisErrorAt.failAnalysis(package.scala:42)
        at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.getTable(Analyzer.scala:306)
        at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$$anonfun$apply$9.applyOrElse(Analyzer.scala:315)
        at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$$anonfun$apply$9.applyOrElse(Analyzer.scala:310)
        at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan$$anonfun$resolveOperators$1.apply(LogicalPlan.scala:57)
        at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan$$anonfun$resolveOperators$1.apply(LogicalPlan.scala:57)
        at org.apache.spark.sql.catalyst.trees.CurrentOrigin$.withOrigin(TreeNode.scala:69)
        at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan.resolveOperators(LogicalPlan.scala:56)
        at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan$$anonfun$1.apply(LogicalPlan.scala:54)
        at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan$$anonfun$1.apply(LogicalPlan.scala:54)
        at org.apache.spark.sql.catalyst.trees.TreeNode$$anonfun$4.apply(TreeNode.scala:281)
        at scala.collection.Iterator$$anon$11.next(Iterator.scala:328)
        at scala.collection.Iterator$class.foreach(Iterator.scala:727)
        at scala.collection.AbstractIterator.foreach(Iterator.scala:1157)
        at scala.collection.generic.Growable$class.$plus$plus$eq(Growable.scala:48)
        at scala.collection.mutable.ArrayBuffer.$plus$plus$eq(ArrayBuffer.scala:103)
        at scala.collection.mutable.ArrayBuffer.$plus$plus$eq(ArrayBuffer.scala:47)
        at scala.collection.TraversableOnce$class.to(TraversableOnce.scala:273)
        at scala.collection.AbstractIterator.to(Iterator.scala:1157)
        at scala.collection.TraversableOnce$class.toBuffer(TraversableOnce.scala:265)
        at scala.collection.AbstractIterator.toBuffer(Iterator.scala:1157)
        at scala.collection.TraversableOnce$class.toArray(TraversableOnce.scala:252)
        at scala.collection.AbstractIterator.toArray(Iterator.scala:1157)
        at org.apache.spark.sql.catalyst.trees.TreeNode.transformChildren(TreeNode.scala:321)
        at org.apache.spark.sql.catalyst.plans.logical.LogicalPlan.resolveOperators(LogicalPlan.scala:54)
        at org.apache.spark.sql.catalyst.analysis.Analyzer$ResolveRelations$.apply(Analyzer.scala:310)
...

```

# The Fix
Instead, submit your job using spark-submit. In this case:
```sh
sbt assembly &&
spark-submit --master yarn-client \
  --class com.mozilla.telemetry.views.CrossSectionalView \
    target/scala-2.10/telemetry-batch-view-*.jar \
  --outName=test
```
