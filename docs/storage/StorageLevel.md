---
tags:
  - DeveloperApi
---

# StorageLevel

`StorageLevel` describes how an RDD is persisted (and addresses the following concerns):

* Does RDD use disk?
* Does RDD <<useMemory, use memory>> to store data?
* How much of RDD is in memory?
* Does RDD use off-heap memory?
* Should an RDD be serialized or <<deserialized, not>> (while storing the data)?
* How many replicas (default: `1`) to use (can only be less than `40`)?

There are the following `StorageLevel` (number `_2` in the name denotes 2 replicas):

* [[NONE]] `NONE` (default)
* `DISK_ONLY`
* `DISK_ONLY_2`
* [[MEMORY_ONLY]] `MEMORY_ONLY` (default for spark-rdd-caching.md#cache[`cache` operation] for RDDs)
* `MEMORY_ONLY_2`
* `MEMORY_ONLY_SER`
* `MEMORY_ONLY_SER_2`
* [[MEMORY_AND_DISK]] `MEMORY_AND_DISK`
* `MEMORY_AND_DISK_2`
* `MEMORY_AND_DISK_SER`
* `MEMORY_AND_DISK_SER_2`
* `OFF_HEAP`

You can check out the storage level using `getStorageLevel()` operation.

```
val lines = sc.textFile("README.md")

scala> lines.getStorageLevel
res0: org.apache.spark.storage.StorageLevel = StorageLevel(disk=false, memory=false, offheap=false, deserialized=false, replication=1)
```

[[useMemory]]
`StorageLevel` can indicate to use memory for data storage using `useMemory` flag.

[source, scala]
----
useMemory: Boolean
----

[[useDisk]]
`StorageLevel` can indicate to use disk for data storage using `useDisk` flag.

[source, scala]
----
useDisk: Boolean
----

[[deserialized]]
`StorageLevel` can indicate to store data in deserialized format using `deserialized` flag.

[source, scala]
----
deserialized: Boolean
----

[[replication]]
`StorageLevel` can indicate to replicate the data to other block managers using `replication` property.

[source, scala]
----
replication: Int
----
