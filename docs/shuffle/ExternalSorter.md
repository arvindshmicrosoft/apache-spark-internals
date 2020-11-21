# ExternalSorter

`ExternalSorter` is a shuffle:Spillable.md[Spillable] of WritablePartitionedPairCollection of pairs (of K keys and C values).

`ExternalSorter[K, V, C]` is a parameterized type of `K` keys, `V` values, and `C` combiner (partial) values.

## Creating Instance

ExternalSorter takes the following to be created:

* [[context]] [TaskContext](../scheduler/TaskContext.md)
* [[aggregator]] Optional rdd:Aggregator.md[Aggregator] (default: undefined)
* [[partitioner]] Optional rdd:Partitioner[Partitioner] (default: undefined)
* [[ordering]] Optional Scala's http://www.scala-lang.org/api/current/scala/math/Ordering.html[Ordering] for keys (default: undefined)
* [[serializer]] Optional serializer:Serializer.md[Serializer] (default: core:SparkEnv.md#serializer[system Serializer])

ExternalSorter is created when:

* SortShuffleWriter is requested to shuffle:SortShuffleWriter.md#write[write records] (as a `ExternalSorter[K, V, C]` or `ExternalSorter[K, V, V]` based on [Map-Size Partial Aggregation Flag](../rdd/ShuffleDependency.md#mapSideCombine))

* BlockStoreShuffleReader is requested to shuffle:BlockStoreShuffleReader.md#read[read records] (with sort ordering defined)

== [[in-memory-collection]][[buffer]][[map]] In-Memory Collections of Records

ExternalSorter uses PartitionedPairBuffers or PartitionedAppendOnlyMaps to store records in memory before spilling to disk. ExternalSorter uses PartitionedPairBuffers when created with no <<aggregator, Aggregator>> specified. Otherwise, ExternalSorter uses PartitionedAppendOnlyMaps.

ExternalSorter creates a PartitionedPairBuffer and a PartitionedAppendOnlyMap when created.

ExternalSorter inserts records to the collections when <<insertAll, insertAll>>.

ExternalSorter <<maybeSpillCollection, spills the in-memory collection to disk if needed>> and, shuffle:Spillable.md#maybeSpill[if so], creates a new collection.

ExternalSorter releases the collections (``null``s them) when requested to <<forceSpill, forceSpill>> and <<stop, stop>>. That is when the JVM garbage collector takes care of evicting them from memory completely.

== [[peakMemoryUsedBytes]][[_peakMemoryUsedBytes]] Peak Size of In-Memory Collection

ExternalSorter tracks the peak size (in bytes) of the <<in-memory-collection, in-memory collection>> whenever requested to <<maybeSpillCollection, spill the in-memory collection to disk if needed>>.

The peak size is used when:

* BlockStoreShuffleReader is requested to shuffle:BlockStoreShuffleReader.md#read[read combined records for a reduce task] (with a sort ordering defined)

* ExternalSorter is requested to <<writePartitionedFile, write all records into a partitioned file>>

== [[spills]] Spills Internal Registry

ExternalSorter manages spilled files.

== [[insertAll]] Inserting Records

[source, scala]
----
insertAll(
  records: Iterator[Product2[K, V]]): Unit
----

insertAll branches off per whether the optional <<aggregator, Aggregator>> was <<insertAll-shouldCombine, specified>> or <<insertAll-no-aggregator, not>> (to create the <<creating-instance, ExternalSorter>>).

insertAll takes all records eagerly and materializes the given records iterator.

=== [[insertAll-shouldCombine]] Map-Side Aggregator Specified

If there is an Aggregator specified, insertAll creates an update function based on the rdd:Aggregator.md#mergeValue[mergeValue] and rdd:Aggregator.md#createCombiner[createCombiner] functions of the Aggregator.

For every record, insertAll shuffle:Spillable.md#addElementsRead[increment internal read counter].

insertAll requests the <<map, PartitionedAppendOnlyMap>> to changeValue for the key (made up of the <<getPartition, partition>> of the key of the current record and the key itself, i.e. `(partition, key)`) with the update function.

In the end, insertAll <<maybeSpillCollection, spills the in-memory collection to disk if needed>> (with the usingMap flag on since the <<map, PartitionedAppendOnlyMap>> was updated).

=== [[insertAll-no-aggregator]] No Map-Side Aggregator Specified

With no Aggregator specified, insertAll iterates over all the records and uses the <<buffer, PartitionedPairBuffer>> instead.

For every record, insertAll shuffle:Spillable.md#addElementsRead[increment internal read counter].

insertAll requests the <<buffer, PartitionedPairBuffer>> to insert with the <<getPartition, partition>> of the key of the current record, the key itself and the value of the current record.

In the end, insertAll <<maybeSpillCollection, spills the in-memory collection to disk if needed>> (with the usingMap flag off since this time the <<buffer, PartitionedPairBuffer>> was updated, not the <<map, PartitionedAppendOnlyMap>>).

=== [[insertAll-usage]] Usage

insertAll is used when:

* SortShuffleWriter is requested to shuffle:SortShuffleWriter.md#write[write records] (as a `ExternalSorter[K, V, C]` or `ExternalSorter[K, V, V]` based on [Map-Size Partial Aggregation Flag](../rdd/ShuffleDependency.md#mapSideCombine))

* BlockStoreShuffleReader is requested to shuffle:BlockStoreShuffleReader.md#read[read records] (with sort ordering defined)

== [[writePartitionedFile]] Writing All Records Into Partitioned File

[source, scala]
----
writePartitionedFile(
  blockId: BlockId,
  outputFile: File): Array[Long]
----

writePartitionedFile...FIXME

writePartitionedFile is used when SortShuffleWriter is requested to shuffle:SortShuffleWriter.md#write[write records].

== [[stop]] Stopping ExternalSorter

[source, scala]
----
stop(): Unit
----

stop...FIXME

stop is used when:

* BlockStoreShuffleReader is requested to shuffle:BlockStoreShuffleReader.md#read[read records] (with sort ordering defined)

* SortShuffleWriter is requested to shuffle:SortShuffleWriter.md#stop[stop]

== [[spill]] Spilling Data to Disk

[source, scala]
----
spill(
  collection: WritablePartitionedPairCollection[K, C]): Unit
----

spill requests the given WritablePartitionedPairCollection for a destructive WritablePartitionedIterator.

spill <<spillMemoryIteratorToDisk, spillMemoryIteratorToDisk>> (with the destructive WritablePartitionedIterator) that creates a SpilledFile.

spill adds the SpilledFile to the <<spills, spills>> internal registry.

spill is part of the Spillable.md#spill[Spillable] abstraction.

== [[spillMemoryIteratorToDisk]] spillMemoryIteratorToDisk Method

[source, scala]
----
spillMemoryIteratorToDisk(
  inMemoryIterator: WritablePartitionedIterator): SpilledFile
----

spillMemoryIteratorToDisk...FIXME

spillMemoryIteratorToDisk is used when:

* ExternalSorter is requested to <<spill, spill>>

* SpillableIterator is requested to spill

== [[maybeSpillCollection]] Spilling In-Memory Collection to Disk

[source, scala]
----
maybeSpillCollection(
  usingMap: Boolean): Unit
----

maybeSpillCollection branches per the input usingMap flag (that is to determine which in-memory collection to use, the <<map, PartitionedAppendOnlyMap>> or the <<buffer, PartitionedPairBuffer>>).

maybeSpillCollection requests the collection to estimate size (in bytes) that is tracked as the <<peakMemoryUsedBytes, peakMemoryUsedBytes>> metric (for every size bigger than what is currently recorded).

maybeSpillCollection shuffle:Spillable.md#maybeSpill[spills the collection to disk if needed]. If spilled, maybeSpillCollection creates a new collection (a new PartitionedAppendOnlyMap or a new PartitionedPairBuffer).

maybeSpillCollection is used when ExternalSorter is requested to <<insertAll, insertAll>>.

== [[iterator]] iterator Method

[source, scala]
----
iterator: Iterator[Product2[K, C]]
----

iterator...FIXME

iterator is used when BlockStoreShuffleReader is requested to shuffle:BlockStoreShuffleReader.md#read[read combined records for a reduce task].

== [[partitionedIterator]] partitionedIterator Method

[source, scala]
----
partitionedIterator: Iterator[(Int, Iterator[Product2[K, C]])]
----

partitionedIterator...FIXME

partitionedIterator is used when ExternalSorter is requested for an <<iterator, iterator>> and to <<writePartitionedFile, write a partitioned file>>

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.util.collection.ExternalSorter` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.util.collection.ExternalSorter=ALL
----

Refer to ROOT:spark-logging.md[Logging].
