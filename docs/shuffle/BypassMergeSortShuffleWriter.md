# BypassMergeSortShuffleWriter

`BypassMergeSortShuffleWriter` is a shuffle:ShuffleWriter.md[ShuffleWriter] for scheduler:ShuffleMapTask.md[ShuffleMapTasks] to <<write, write records into one single shuffle block data file>>.

.BypassMergeSortShuffleWriter and DiskBlockObjectWriters
image::BypassMergeSortShuffleWriter-write.png[align="center"]

## Creating Instance

BypassMergeSortShuffleWriter takes the following to be created:

* [[blockManager]] storage:BlockManager.md[BlockManager]
* <<shuffleBlockResolver, IndexShuffleBlockResolver>>
* [[handle]] shuffle:BypassMergeSortShuffleHandle.md[BypassMergeSortShuffleHandle<K, V>]
* [[mapId]] Map ID
* [[taskContext]] [TaskContext](../scheduler/TaskContext.md)
* [[conf]] ROOT:SparkConf.md[SparkConf]

BypassMergeSortShuffleWriter is created when SortShuffleManager is requested for a SortShuffleManager.md#getWriter[ShuffleWriter] (for a <<handle, BypassMergeSortShuffleHandle>>).

== [[partitionWriters]] Per-Partition DiskBlockObjectWriters

BypassMergeSortShuffleWriter uses storage:DiskBlockObjectWriter.md[DiskBlockObjectWriter] when...FIXME

== [[shuffleBlockResolver]] IndexShuffleBlockResolver

BypassMergeSortShuffleWriter is given a shuffle:IndexShuffleBlockResolver.md[IndexShuffleBlockResolver] to be created.

BypassMergeSortShuffleWriter uses the IndexShuffleBlockResolver for <<write, writing records>> (to shuffle:IndexShuffleBlockResolver.md#writeIndexFileAndCommit[writeIndexFileAndCommit] and shuffle:IndexShuffleBlockResolver.md#getDataFile[getDataFile]).

== [[serializer]] Serializer

When created, BypassMergeSortShuffleWriter requests the shuffle:spark-shuffle-BaseShuffleHandle.md#dependency[ShuffleDependency] (of the given <<handle, BypassMergeSortShuffleHandle>>) for the [Serializer](../rdd/ShuffleDependency.md#serializer).

BypassMergeSortShuffleWriter creates a new instance of the Serializer for <<write, writing records>>.

== [[configuration-properties]] Configuration Properties

=== [[fileBufferSize]][[spark.shuffle.file.buffer]] spark.shuffle.file.buffer

BypassMergeSortShuffleWriter uses ROOT:configuration-properties.md#spark.shuffle.file.buffer[spark.shuffle.file.buffer] configuration property for...FIXME

=== [[transferToEnabled]][[spark.file.transferTo]] spark.file.transferTo

BypassMergeSortShuffleWriter uses ROOT:configuration-properties.md#spark.file.transferTo[spark.file.transferTo] configuration property to control whether to use Java New I/O while <<writePartitionedFile, writing to a partitioned file>>.

== [[write]] Writing Records to Shuffle File

[source, java]
----
void write(
  Iterator<Product2<K, V>> records)
----

write creates a new instance of the <<serializer, Serializer>>.

write initializes the <<partitionWriters, partitionWriters>> and <<partitionWriterSegments, partitionWriterSegments>> internal registries (for storage:DiskBlockObjectWriter.md[DiskBlockObjectWriters] and FileSegments for <<numPartitions, every partition>>, respectively).

write requests the <<blockManager, BlockManager>> for the storage:BlockManager.md#diskBlockManager[DiskBlockManager] and for <<numPartitions, every partition>> write requests it for a storage:DiskBlockManager.md#createTempShuffleBlock[shuffle block ID and the file]. write creates a storage:BlockManager.md#getDiskWriter[DiskBlockObjectWriter] for the shuffle block (using the <<blockManager, BlockManager>>). write stores the reference to DiskBlockObjectWriters in the <<partitionWriters, partitionWriters>> internal registry.

After all DiskBlockObjectWriters are created, write requests the <<writeMetrics, ShuffleWriteMetrics>> to executor:ShuffleWriteMetrics.md#incWriteTime[increment shuffle write time metric].

For every record (a key-value pair), write requests the <<partitioner, Partitioner>> for the rdd:Partitioner.md#getPartition[partition ID] for the key. The partition ID is then used as an index of the partition writer (among the <<partitionWriters, DiskBlockObjectWriters>>) to storage:DiskBlockObjectWriter.md#write[write the current record out to a block file].

Once all records have been writted out to their respective block files, write does the following for every <<partitionWriters, DiskBlockObjectWriter>>:

. Requests the DiskBlockObjectWriter to storage:DiskBlockObjectWriter.md#commitAndGet[commit and return a corresponding FileSegment of the shuffle block]

. Saves the (reference to) FileSegments in the <<partitionWriterSegments, partitionWriterSegments>> internal registry

. Requests the DiskBlockObjectWriter to storage:DiskBlockObjectWriter.md#close[close]

NOTE: At this point, all the records are in shuffle block files on a local disk. The records are split across block files by key.

write requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> for the shuffle:IndexShuffleBlockResolver.md#getDataFile[shuffle file] for the <<shuffleId, shuffle>> and the <<mapId, map IDs>>.

write creates a temporary file (based on the name of the shuffle file) and <<writePartitionedFile, writes all the per-partition shuffle files to it>>. The size of every per-partition shuffle files is saved as the <<partitionLengths, partitionLengths>> internal registry.

NOTE: At this point, all the per-partition shuffle block files are one single map shuffle data file.

write requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> to shuffle:IndexShuffleBlockResolver.md#writeIndexFileAndCommit[write shuffle index and data files] for the <<shuffleId, shuffle>> and the <<mapId, map IDs>> (with the <<partitionLengths, partitionLengths>> and the temporary shuffle output file).

write returns a scheduler:MapStatus.md[shuffle map output status] (with the storage:BlockManager.md#shuffleServerId[shuffle server ID] and the <<partitionLengths, partitionLengths>>).

write is part of shuffle:ShuffleWriter.md#write[ShuffleWriter] abstraction.

=== [[write-no-records]] No Records

When there is no records to write out, write initializes the <<partitionLengths, partitionLengths>> internal array (of <<numPartitions, numPartitions>> size) with all elements being 0.

write requests the <<shuffleBlockResolver, IndexShuffleBlockResolver>> to shuffle:IndexShuffleBlockResolver.md#writeIndexFileAndCommit[write shuffle index and data files], but the difference (compared to when there are records to write) is that the `dataTmp` argument is simply `null`.

write sets the internal `mapStatus` (with the address of storage:BlockManager.md[BlockManager] in use and <<partitionLengths, partitionLengths>>).

=== [[write-requirements]] Requirements

write requires that there are no <<partitionWriters, DiskBlockObjectWriters>>.

== [[writePartitionedFile]] Concatenating Per-Partition Files Into Single File

[source, java]
----
long[] writePartitionedFile(
  File outputFile)
----

writePartitionedFile creates a file output stream for the input `outputFile` in append mode.

writePartitionedFile starts tracking write time (as `writeStartTime`).

For every <<numPartitions, numPartitions>> partition, writePartitionedFile takes the file from the `FileSegment` (from <<partitionWriterSegments, partitionWriterSegments>>) and creates a file input stream to read raw bytes.

writePartitionedFile then <<copyStream, copies the raw bytes from each partition segment input stream to `outputFile`>> (possibly using Java New I/O per <<transferToEnabled, transferToEnabled>> flag set when <<creating-instance, BypassMergeSortShuffleWriter was created>>) and records the length of the shuffle data file (in `lengths` internal array).

In the end, writePartitionedFile executor:ShuffleWriteMetrics.md#incWriteTime[increments shuffle write time], clears <<partitionWriters, partitionWriters>> array and returns the lengths of the shuffle data files per partition.

writePartitionedFile is used when BypassMergeSortShuffleWriter is requested to <<write, write records>>.

== [[copyStream]] Copying Raw Bytes Between Input Streams

[source, scala]
----
copyStream(
  in: InputStream,
  out: OutputStream,
  closeStreams: Boolean = false,
  transferToEnabled: Boolean = false): Long
----

copyStream branches off depending on the type of `in` and `out` streams, i.e. whether they are both `FileInputStream` with `transferToEnabled` input flag is enabled.

If they are both `FileInputStream` with `transferToEnabled` enabled, copyStream gets their `FileChannels` and transfers bytes from the input file to the output file and counts the number of bytes, possibly zero, that were actually transferred.

NOTE: copyStream uses Java's {java-javadoc-url}/java/nio/channels/FileChannel.html[java.nio.channels.FileChannel] to manage file channels.

If either `in` and `out` input streams are not `FileInputStream` or `transferToEnabled` flag is disabled (default), copyStream reads data from `in` to write to `out` and counts the number of bytes written.

copyStream can optionally close `in` and `out` streams (depending on the input `closeStreams` -- disabled by default).

NOTE: `Utils.copyStream` is used when <<writePartitionedFile, BypassMergeSortShuffleWriter writes records into one single shuffle block data file>> (among other places).

TIP: Visit the official web site of https://jcp.org/jsr/detail/51.jsp[JSR 51: New I/O APIs for the Java Platform] and read up on {java-javadoc-url}/java/nio/package-summary.html[java.nio package].

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.shuffle.sort.BypassMergeSortShuffleWriter` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.shuffle.sort.BypassMergeSortShuffleWriter=ALL
----

Refer to ROOT:spark-logging.md[Logging].

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| numPartitions
| [[numPartitions]]

| partitionWriterSegments
| [[partitionWriterSegments]]

| mapStatus
| [[mapStatus]] scheduler:MapStatus.md[MapStatus] that <<stop, BypassMergeSortShuffleWriter returns when stopped>>

Initialized every time BypassMergeSortShuffleWriter <<write, writes records>>.

Used when <<stop, BypassMergeSortShuffleWriter stops>> (with `success` enabled) as a marker if <<write, any records were written>> and <<stop, returned if they did>>.

| partitionLengths
| [[partitionLengths]] Temporary array of partition lengths after records are <<write, written to a shuffle system>>.

Initialized every time BypassMergeSortShuffleWriter <<write, writes records>> before passing it in to IndexShuffleBlockResolver.md#writeIndexFileAndCommit[IndexShuffleBlockResolver]). After IndexShuffleBlockResolver finishes, it is used to initialize <<mapStatus, mapStatus>> internal property.

|===
