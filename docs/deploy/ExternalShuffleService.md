= ExternalShuffleService
:navtitle: External Shuffle Service

*ExternalShuffleService* is a Spark service that can serve shuffle blocks from outside an executor:Executor.md[Executor] process. It runs as a standalone application and manages shuffle output files so they are available for executors at all time. As the shuffle output files are managed externally to the executors it offers an uninterrupted access to the shuffle output files regardless of executors being killed or down.

You start ExternalShuffleService using <<start-script, `start-shuffle-service.sh` shell script>> and enable its use by the driver and executors using configuration-properties.md#spark.shuffle.service.enabled[spark.shuffle.service.enabled].

There is a custom external shuffle service for Spark on YARN -- spark-on-yarn:spark-yarn-YarnShuffleService.md[YarnShuffleService].

== [[start-script]] start-shuffle-service.sh Shell Script

```
start-shuffle-service.sh
```

`start-shuffle-service.sh` shell script allows you to launch ExternalShuffleService. The script is under `sbin` directory.

When executed, it runs `sbin/spark-config.sh` and `bin/load-spark-env.sh` shell scripts. It then executes `sbin/spark-daemon.sh` with start command and the parameters: `org.apache.spark.deploy.ExternalShuffleService` and `1`.

[options="wrap"]
----
$ ./sbin/start-shuffle-service.sh
starting org.apache.spark.deploy.ExternalShuffleService, logging to ...logs/spark-jacek-org.apache.spark.deploy.ExternalShuffleService-1-japila.local.out

$ tail -f ...logs/spark-jacek-org.apache.spark.deploy.ExternalShuffleService-1-japila.local.out
Spark Command: /Library/Java/JavaVirtualMachines/Current/Contents/Home/bin/java -cp /Users/jacek/dev/oss/spark/conf/:/Users/jacek/dev/oss/spark/assembly/target/scala-2.11/jars/* -Xmx1g org.apache.spark.deploy.ExternalShuffleService
========================================
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
16/06/07 08:02:02 INFO ExternalShuffleService: Started daemon with process name: 42918@japila.local
16/06/07 08:02:03 INFO ExternalShuffleService: Starting shuffle service on port 7337 with useSasl = false
----

You can also use tools:spark-class.md[spark-class] to launch ExternalShuffleService.

[source,plaintext]
----
spark-class org.apache.spark.deploy.ExternalShuffleService
----

== [[main]] Launching ExternalShuffleService Standalone Application

When started, it executes `Utils.initDaemon(log)`.

CAUTION: FIXME `Utils.initDaemon(log)`? See spark-submit.

It loads default Spark properties and creates a `SecurityManager`.

It sets configuration-properties.md#spark.shuffle.service.enabled[spark.shuffle.service.enabled] to `true` (as <<create-instance, later it is checked whether it is enabled or not>>).

A ExternalShuffleService is <<create-instance, created>> and <<start, started>>.

A shutdown hook is registered so when ExternalShuffleService is shut down, it prints the following INFO message to the logs and the <<stop, stop>> method is executed.

[source,plaintext]
----
Shutting down shuffle service.
----

You should see the following INFO message in the logs:

[source,plaintext]
----
Registered executor [AppExecId] with [executorInfo]
----

You should also see the following messages when a `SparkContext` is closed:

[source,plaintext]
----
Application [appId] removed, cleanupLocalDirs = [cleanupLocalDirs]
Cleaning up executor [AppExecId]'s [executor.localDirs.length] local dirs
Successfully cleaned up directory: [localDir]
----

== [[creating-instance]] Creating Instance

ExternalShuffleService requires a SparkConf.md[SparkConf] and spark-security.md[SecurityManager].

When created, it reads configuration-properties.md#spark.shuffle.service.enabled[spark.shuffle.service.enabled] configuration property (disabled by default) and <<spark.shuffle.service.port, spark.shuffle.service.port>> (defaults to `7337`) configuration settings. It also checks whether authentication is enabled.

CAUTION: FIXME Review `securityManager.isAuthenticationEnabled()`

ExternalShuffleService creates a network:TransportConf.md[] (as `transportConf`).

It creates a deploy:ExternalShuffleBlockHandler.md[] (as `blockHandler`) and `TransportContext` (as `transportContext`).

CAUTION: FIXME TransportContext?

No internal `TransportServer` (as `server`) is created.

== [[start]] Starting ExternalShuffleService

[source, scala]
----
start(): Unit
----

start starts an ExternalShuffleService.

When start is executed, you should see the following INFO message in the logs:

[source,plaintext]
----
Starting shuffle service on port [port] with useSasl = [useSasl]
----

If `useSasl` is enabled, a `SaslServerBootstrap` is created.

CAUTION: FIXME SaslServerBootstrap?

The internal `server` reference (a `TransportServer`) is created (which will attempt to bind to `port`).

NOTE: `port` is <<creating-instance, set up by `spark.shuffle.service.port` or defaults to `7337` when ExternalShuffleService is created>>.

== [[stop]] Stopping ExternalShuffleService

[source, scala]
----
stop(): Unit
----

stop closes the internal `server` reference and clears it (i.e. sets it to `null`).

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.deploy.ExternalShuffleService` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.deploy.ExternalShuffleService=ALL
----

Refer to spark-logging.md[Logging].
