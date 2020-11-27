= EventLoggingListener

*EventLoggingListener* is a SparkListener.md[] that <<logEvent, writes out JSON-encoded events>> of a Spark application with event logging enabled (based on configuration-properties.md#spark.eventLog.enabled[spark.eventLog.enabled] configuration property).

EventLoggingListener supports custom spark-history-server:configuration-properties.md#EventLoggingListener[configuration properties].

EventLoggingListener writes out log files to a directory (based on configuration-properties.md#spark.eventLog.dir[spark.eventLog.dir] configuration property). All SparkListener.md[]s are logged (but SparkListener.md#SparkListenerBlockUpdated[SparkListenerBlockUpdated] and SparkListener.md#SparkListenerExecutorMetricsUpdate[SparkListenerExecutorMetricsUpdate]).

TIP: Use index.md[Spark History Server] to view the event logs in a browser (similarly to webui:index.md[web UI] of a Spark application).

[[inprogress-extension]][[IN_PROGRESS]]
EventLoggingListener uses *.inprogress* file extension for in-flight event log files of active Spark applications.

EventLoggingListener can compress events (based on configuration-properties.md#spark.eventLog.compress[spark.eventLog.compress] configuration property).

== [[creating-instance]] Creating Instance

EventLoggingListener takes the following to be created:

* [[appId]] Application ID
* [[appAttemptId]] Application Attempt ID (optional)
* [[logBaseDir]] Log Directory
* [[sparkConf]] SparkConf.md[SparkConf]
* [[hadoopConf]] Hadoop https://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/conf/Configuration.html[Configuration]

EventLoggingListener initializes the <<internal-properties, internal properties>>.

NOTE: When initialized with no <<hadoopConf, Hadoop Configuration>>, EventLoggingListener uses `SparkHadoopUtil` utility to spark-SparkHadoopUtil.md#newConfiguration[create a new one].

== [[logPath]] Event Log File

[source, scala]
----
logPath: String
----

`logPath` is...FIXME

NOTE: `logPath` is used when EventLoggingListener is requested to <<start, start>> and <<stop, stop>>.

== [[start]] Starting EventLoggingListener

[source, scala]
----
start(): Unit
----

`start` deletes the <<logPath, logPath>> with the <<IN_PROGRESS, .inprogress>> extension.

The log file's working name is created based on `appId` with or without the compression codec used and `appAttemptId`, i.e. `local-1461696754069`. It also uses `.inprogress` extension.

If <<spark_eventLog_overwrite, overwrite is enabled>>, you should see the WARN message:

```
Event log [path] already exists. Overwriting...
```

The working log `.inprogress` is attempted to be deleted. In case it could not be deleted, the following WARN message is printed out to the logs:

```
Error deleting [path]
```

The buffered output stream is created with metadata with Spark's version and `SparkListenerLogStart` class' name as the first line.

```
{"Event":"SparkListenerLogStart","Spark Version":"2.0.0-SNAPSHOT"}
```

At this point, EventLoggingListener is ready for event logging and you should see the following INFO message in the logs:

```
Logging events to [logPath]
```

`start` throws an `IllegalArgumentException` when the <<logBaseDir, logBaseDir>> is not a directory:

```text
Log directory [logBaseDir] is not a directory.
```

`start` is used when [SparkContext](../SparkContext.md) is created.

== [[logEvent]] Logging Event (In JSON Format)

[source, scala]
----
logEvent(
  event: SparkListenerEvent,
  flushLogger: Boolean = false): Unit
----

`logEvent` logs the given `event` as JSON.

CAUTION: FIXME

== [[stop]] Stopping EventLoggingListener

[source, scala]
----
stop(): Unit
----

`stop` closes `PrintWriter` for the log file and renames the file to be without `.inprogress` extension.

If the target log file exists (one without `.inprogress` extension), it overwrites the file if <<spark_eventLog_overwrite, spark.eventLog.overwrite>> is enabled. You should see the following WARN message in the logs:

```
Event log [target] already exists. Overwriting...
```

If the target log file exists and overwrite is disabled, an `java.io.IOException` is thrown with the following message:

```
Target log file already exists ([logPath])
```

NOTE: `stop` is executed while `SparkContext` is requested to [stop](../SparkContext.md#stop).

== [[getLogPath]] getLogPath Utility

[source, scala]
----
getLogPath(
  logBaseDir: URI,
  appId: String,
  appAttemptId: Option[String],
  compressionCodecName: Option[String] = None): String
----

`getLogPath`...FIXME

NOTE: `getLogPath` is used when EventLoggingListener is <<creating-instance, created>> (for the <<logPath, logPath>>).

== [[openEventLog]] openEventLog Utility

[source, scala]
----
openEventLog(
  log: Path,
  fs: FileSystem): InputStream
----

openEventLog...FIXME

openEventLog is used when...FIXME

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.scheduler.EventLoggingListener` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.scheduler.EventLoggingListener=ALL
----

Refer to spark-logging.md[Logging].

== [[internal-properties]] Internal Properties

=== [[hadoopDataStream]] FSDataOutputStream

Hadoop http://hadoop.apache.org/docs/r2.7.3/api/org/apache/hadoop/fs/FSDataOutputStream.html[FSDataOutputStream] (default: `None`)

Used when...FIXME

=== [[writer]] PrintWriter

{java-javadoc-url}/java/io/PrintWriter.html[java.io.PrintWriter] for <<logEvent, writing out events>> to the <<logPath, event log file>>.

Initialized when EventLoggingListener is requested to <<start, start>>

Used when EventLoggingListener is requested to <<logEvent, logEvent>>

Closed when EventLoggingListener is requested to <<stop, stop>>
