# BroadcastManager

![BroadcastManager, SparkEnv and BroadcastFactory](../images/core/BroadcastManager.png)

## Creating Instance

`BroadcastManager` takes the following to be created:

* <span id="isDriver"> `isDriver` flag
* <span id="conf"> [SparkConf](../SparkConf.md)
* <span id="securityManager"> `SecurityManager`

While being created, `BroadcastManager` is requested to [initialize](#initialize).

`BroadcastManager` is created when:

* `SparkEnv` utility is used to [create a base SparkEnv](../SparkEnv.md#create) (for the driver and executors)

### <span id="initialize"> Initializing

```scala
initialize(): Unit
```

Unless initialized already, `initialize` creates a [TorrentBroadcastFactory](#broadcastFactory) and requests it to [initialize itself](TorrentBroadcastFactory.md#initialize).

## <span id="broadcastFactory"> TorrentBroadcastFactory

`BroadcastManager` manages a [BroadcastFactory](BroadcastFactory.md):

* Creates and initializes it when [created](#creating-instance) (and requested to [initialize](#initialize))

* Stops it when stopped

`BroadcastManager` uses the `BroadcastFactory` in [newBroadcast](#newBroadcast) and [unbroadcast](#unbroadcast).

## <span id="newBroadcast"> Creating Broadcast Variable

```scala
newBroadcast[T: ClassTag](
  value_ : T,
  isLocal: Boolean): Broadcast[T]
```

`newBroadcast` requests the [BroadcastFactory](#broadcastFactory) for a [new broadcast variable](BroadcastFactory.md#newBroadcast) (with the [next available broadcast ID](#nextBroadcastId)).

`newBroadcast` is used when:

* `SparkContext` is requested for a [new broadcast variable](../SparkContext.md#broadcast)
* `MapOutputTracker` utility is used to [serializeMapStatuses](../scheduler/MapOutputTracker.md#serializeMapStatuses)

## <span id="nextBroadcastId"> Unique Identifiers of Broadcast Variables

`BroadcastManager` tracks [broadcast variables](#newBroadcast) and assigns unique and continuous identifiers.

## <span id="MapOutputTrackerMaster"> MapOutputTrackerMaster

`BroadcastManager` is used to create a [MapOutputTrackerMaster](../scheduler/MapOutputTrackerMaster.md#BroadcastManager)
