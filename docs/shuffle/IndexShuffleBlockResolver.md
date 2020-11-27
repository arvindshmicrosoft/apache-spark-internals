= [[IndexShuffleBlockResolver]] IndexShuffleBlockResolver

*IndexShuffleBlockResolver* is a ShuffleBlockResolver.md[ShuffleBlockResolver] that manages shuffle block data and uses *shuffle index files* for faster shuffle data access.

IndexShuffleBlockResolver is <<creating-instance, created>> for SortShuffleManager.md#shuffleBlockResolver[SortShuffleManager] (for ShuffleManager.md#shuffleBlockResolver[retrieving shuffle block data]).

.IndexShuffleBlockResolver and SortShuffleManager
image::IndexShuffleBlockResolver-SortShuffleManager.png[align="center"]

IndexShuffleBlockResolver can <<writeIndexFileAndCommit, write>>, <<getBlockData, look up>> and <<removeDataByMap, remove>> shuffle block index and data files (given shuffle and
map IDs).

IndexShuffleBlockResolver is later used to create the SortShuffleManager.md#getWriter[ShuffleWriter] given a spark-shuffle-ShuffleHandle.md[ShuffleHandle].

== [[creating-instance]] Creating Instance

IndexShuffleBlockResolver takes the following to be created:

* [[conf]] SparkConf.md[SparkConf]
* [[_blockManager]][[blockManager]] storage:BlockManager.md[BlockManager]

IndexShuffleBlockResolver initializes the <<internal-properties, internal properties>>.

== [[writeIndexFileAndCommit]] Writing Shuffle Index and Data Files

[source, scala]
----
writeIndexFileAndCommit(
  shuffleId: Int,
  mapId: Int,
  lengths: Array[Long],
  dataTmp: File): Unit
----

Internally, `writeIndexFileAndCommit` first <<getIndexFile, finds the index file>> for the input `shuffleId` and `mapId`.

`writeIndexFileAndCommit` creates a temporary file for the index file (in the same directory) and writes offsets (as the moving sum of the input `lengths` starting from 0 to the final offset at the end for the end of the output file).

NOTE: The offsets are the sizes in the input `lengths` exactly.

.writeIndexFileAndCommit and offsets in a shuffle index file
image::IndexShuffleBlockResolver-writeIndexFileAndCommit.png[align="center"]

`writeIndexFileAndCommit` <<getDataFile, requests a shuffle block data file>> for the input `shuffleId` and `mapId`.

`writeIndexFileAndCommit` <<checkIndexAndDataFile, checks if the given index and data files match each other>> (aka _consistency check_).

If the consistency check fails, it means that another attempt for the same task has already written the map outputs successfully and so the input `dataTmp` and temporary index files are deleted (as no longer correct).

If the consistency check succeeds, the existing index and data files are deleted (if they exist) and the temporary index and data files become "official", i.e. renamed to their final names.

In case of any IO-related exception, `writeIndexFileAndCommit` throws a `IOException` with the messages:

```
fail to rename file [indexTmp] to [indexFile]
```

or

```
fail to rename file [dataTmp] to [dataFile]
```

NOTE: `writeIndexFileAndCommit` is used when ShuffleWriter.md[ShuffleWriters] are requested to write records to a shuffle system, i.e. shuffle:SortShuffleWriter.md#write[SortShuffleWriter], shuffle:BypassMergeSortShuffleWriter.md#write[BypassMergeSortShuffleWriter], and shuffle:UnsafeShuffleWriter.md#closeAndWriteOutput[UnsafeShuffleWriter].

== [[getBlockData]] Creating ManagedBuffer to Read Shuffle Block Data File -- `getBlockData` Method

[source, scala]
----
getBlockData(
  blockId: ShuffleBlockId): ManagedBuffer
----

NOTE: `getBlockData` is part of ShuffleBlockResolver.md#getBlockData[ShuffleBlockResolver] contract.

Internally, `getBlockData` <<getIndexFile, finds the index file>> for the input shuffle `blockId`.

NOTE: storage:BlockId.md#ShuffleBlockId[ShuffleBlockId] knows `shuffleId` and `mapId`.

`getBlockData` discards `blockId.reduceId` bytes of data from the index file.

NOTE: `getBlockData` uses Guava's ++https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#skipFully-java.io.InputStream-long-++[com.google.common.io.ByteStreams] to skip the bytes.

`getBlockData` reads the start and end offsets from the index file and then creates a `FileSegmentManagedBuffer` to read the <<getDataFile, data file>> for the offsets (using <<transportConf, transportConf>> internal property).

NOTE: The start and end offsets are the offset and the length of the file segment for the block data.

In the end, `getBlockData` closes the index file.

== [[checkIndexAndDataFile]] Checking Consistency of Shuffle Index and Data Files and Returning Block Lengths --  `checkIndexAndDataFile` Internal Method

[source, scala]
----
checkIndexAndDataFile(
  index: File,
  data: File,
  blocks: Int): Array[Long]
----

`checkIndexAndDataFile` first checks if the size of the input `index` file is exactly the input `blocks` multiplied by `8`.

`checkIndexAndDataFile` returns `null` when the numbers, and hence the shuffle index and data files, don't match.

`checkIndexAndDataFile` reads the shuffle `index` file and converts the offsets into lengths of each block.

`checkIndexAndDataFile` makes sure that the size of the input shuffle `data` file is exactly the sum of the block lengths.

`checkIndexAndDataFile` returns the block lengths if the numbers match, and `null` otherwise.

NOTE: `checkIndexAndDataFile` is used exclusively when IndexShuffleBlockResolver is requested to <<writeIndexFileAndCommit, write the shuffle index and data files>> (for shuffle and map IDs).

== [[removeDataByMap]] Removing Shuffle Index and Data Files (For Shuffle and Map IDs) -- `removeDataByMap` Method

[source, scala]
----
removeDataByMap(shuffleId: Int, mapId: Int): Unit
----

`removeDataByMap` <<getDataFile, finds>> and deletes the shuffle data for the input `shuffleId` and `mapId` first followed by <<getIndexFile, finding>> and deleting the shuffle data index file.

When `removeDataByMap` fails deleting the files, `removeDataByMap` prints out the following WARN message to the logs.

```
Error deleting data [path]
```

or

```
Error deleting index [path]
```

NOTE: `removeDataByMap` is used exclusively when `SortShuffleManager` is requested to SortShuffleManager.md#unregisterShuffle[unregister a shuffle] (remove a shuffle from a shuffle system).

== [[stop]] Stopping IndexShuffleBlockResolver -- `stop` Method

[source, scala]
----
stop(): Unit
----

NOTE: `stop` is part of ShuffleBlockResolver.md#stop[ShuffleBlockResolver contract].

`stop` is a noop operation, i.e. does nothing when called.

== [[getIndexFile]] Requesting Shuffle Block Index File (from DiskBlockManager)

[source, scala]
----
getIndexFile(
  shuffleId: Int,
  mapId: Int): File
----

`getIndexFile` requests the <<blockManager, BlockManager>> for the storage:BlockManager.md#diskBlockManager[DiskBlockManager] that is in turn requested for the storage:DiskBlockManager.md#getFile[shuffle index file] (with a new ShuffleIndexBlockId with the given shuffleId and mapId).

NOTE: `getIndexFile` is used when IndexShuffleBlockResolver <<writeIndexFileAndCommit, writes shuffle index and data files>>, <<getBlockData, creates a `ManagedBuffer` to read a shuffle block data file>>, and <<removeDataByMap, removes the shuffle index and data files>>.

== [[getDataFile]] Requesting Shuffle Block Data File

[source, scala]
----
getDataFile(
  shuffleId: Int,
  mapId: Int): File
----

`getDataFile` requests the <<blockManager, BlockManager>> for the storage:BlockManager.md#diskBlockManager[DiskBlockManager] that is in turn requested for the storage:DiskBlockManager.md#getFile[shuffle block data file] (for a storage:BlockId.md#ShuffleDataBlockId[ShuffleDataBlockId])

[NOTE]
====
`getDataFile` is used when:

* IndexShuffleBlockResolver is requested to <<getBlockData, get a ManagedBuffer for block data>>, <<removeDataByMap, removeDataByMap>>, and <<writeIndexFileAndCommit, write shuffle index and data files>>

* shuffle:BypassMergeSortShuffleWriter.md#write[BypassMergeSortShuffleWriter], shuffle:UnsafeShuffleWriter.md#closeAndWriteOutput[UnsafeShuffleWriter], and SortShuffleWriter.md#write[SortShuffleWriter] are requested to write records to a shuffle system
====

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.shuffle.IndexShuffleBlockResolver` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source]
----
log4j.logger.org.apache.spark.shuffle.IndexShuffleBlockResolver=ALL
----

Refer to spark-logging.md[Logging].

== [[internal-properties]] Internal Properties

[cols="30m,70",options="header",width="100%"]
|===
| Name
| Description

| transportConf
a| [[transportConf]] network:TransportConf.md[] for *shuffle* module

Created immediately when IndexShuffleBlockResolver is <<creating-instance, created>> by requesting `SparkTransportConf` object to network:TransportConf.md#SparkTransportConf-fromSparkConf[create one from SparkConf]

|===
