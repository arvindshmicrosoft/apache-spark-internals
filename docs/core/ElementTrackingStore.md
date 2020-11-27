= ElementTrackingStore

*ElementTrackingStore* is a core:KVStore.md[].

== [[creating-instance]] Creating Instance

ElementTrackingStore takes the following to be created:

* [[store]] core:KVStore.md[]
* [[conf]] SparkConf.md[]

ElementTrackingStore is created when...FIXME

== [[write]] `write` Method

[source, scala]
----
write(value: Any, checkTriggers: Boolean): Unit
----

NOTE: `write` is part of LINK#write[HERE Contract] to...FIXME.

`write`...FIXME

NOTE: `write` is used when...FIXME

== [[Trigger]] Trigger

`Trigger` is...FIXME

== [[internal-properties]] Internal Properties

[cols="1,2",options="header",width="100%"]
|===
| Name
| Description

| `triggers`
| [[triggers]] <<Trigger, Triggers>> per class

Used when...FIXME

| `flushTriggers`
| [[flushTriggers]] Functions that...FIXME

Used when...FIXME

| `executor`
| [[executor]] Java `ExecutorService` that...FIXME

Used when...FIXME

| `stopped`
| [[stopped]] `stopped` flag to control...FIXME

Used when...FIXME
|===
