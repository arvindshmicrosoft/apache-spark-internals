# SparkListenerInterface

`SparkListenerInterface` is an abstraction of [event listeners](#implementations) (that `SparkListenerBus` notifies about [scheduling events](SparkListenerBus.md#doPostEvent)).

## <span id="onApplicationEnd"> onApplicationEnd

```scala
onApplicationEnd(
  applicationEnd: SparkListenerApplicationEnd): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerApplicationEnd event](SparkListenerBus.md#doPostEvent)

## <span id="onApplicationStart"> onApplicationStart

```scala
onApplicationStart(
  applicationStart: SparkListenerApplicationStart): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerApplicationStart event](SparkListenerBus.md#doPostEvent)

## <span id="onBlockManagerAdded"> onBlockManagerAdded

```scala
onBlockManagerAdded(
  blockManagerAdded: SparkListenerBlockManagerAdded): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerBlockManagerAdded event](SparkListenerBus.md#doPostEvent)

## <span id="onBlockManagerRemoved"> onBlockManagerRemoved

```scala
onBlockManagerRemoved(
  blockManagerRemoved: SparkListenerBlockManagerRemoved): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerBlockManagerRemoved event](SparkListenerBus.md#doPostEvent)

## <span id="onBlockUpdated"> onBlockUpdated

```scala
onBlockUpdated(
  blockUpdated: SparkListenerBlockUpdated): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerBlockUpdated event](SparkListenerBus.md#doPostEvent)

## <span id="onEnvironmentUpdate"> onEnvironmentUpdate

```scala
onEnvironmentUpdate(
  environmentUpdate: SparkListenerEnvironmentUpdate): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerEnvironmentUpdate event](SparkListenerBus.md#doPostEvent)

## <span id="onExecutorAdded"> onExecutorAdded

```scala
onExecutorAdded(
  executorAdded: SparkListenerExecutorAdded): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerExecutorAdded event](SparkListenerBus.md#doPostEvent)

## <span id="onExecutorBlacklisted"> onExecutorBlacklisted

```scala
onExecutorBlacklisted(
  executorBlacklisted: SparkListenerExecutorBlacklisted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerExecutorBlacklisted event](SparkListenerBus.md#doPostEvent)

## <span id="onExecutorBlacklistedForStage"> onExecutorBlacklistedForStage

```scala
onExecutorBlacklistedForStage(
  executorBlacklistedForStage: SparkListenerExecutorBlacklistedForStage): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerExecutorBlacklistedForStage event](SparkListenerBus.md#doPostEvent)

## <span id="onExecutorMetricsUpdate"> onExecutorMetricsUpdate

```scala
onExecutorMetricsUpdate(
  executorMetricsUpdate: SparkListenerExecutorMetricsUpdate): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerExecutorMetricsUpdate event](SparkListenerBus.md#doPostEvent)

## <span id="onExecutorRemoved"> onExecutorRemoved

```scala
onExecutorRemoved(
  executorRemoved: SparkListenerExecutorRemoved): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerExecutorRemoved event](SparkListenerBus.md#doPostEvent)

## <span id="onExecutorUnblacklisted"> onExecutorUnblacklisted

```scala
onExecutorUnblacklisted(
  executorUnblacklisted: SparkListenerExecutorUnblacklisted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerExecutorUnblacklisted event](SparkListenerBus.md#doPostEvent)

## <span id="onJobEnd"> onJobEnd

```scala
onJobEnd(
  jobEnd: SparkListenerJobEnd): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerJobEnd event](SparkListenerBus.md#doPostEvent)

## <span id="onJobStart"> onJobStart

```scala
onJobStart(
  jobStart: SparkListenerJobStart): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerJobStart event](SparkListenerBus.md#doPostEvent)

## <span id="onNodeBlacklisted"> onNodeBlacklisted

```scala
onNodeBlacklisted(
  nodeBlacklisted: SparkListenerNodeBlacklisted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerNodeBlacklisted event](SparkListenerBus.md#doPostEvent)

## <span id="onNodeBlacklistedForStage"> onNodeBlacklistedForStage

```scala
onNodeBlacklistedForStage(
  nodeBlacklistedForStage: SparkListenerNodeBlacklistedForStage): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerNodeBlacklistedForStage event](SparkListenerBus.md#doPostEvent)

## <span id="onNodeUnblacklisted"> onNodeUnblacklisted

```scala
onNodeUnblacklisted(
  nodeUnblacklisted: SparkListenerNodeUnblacklisted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerNodeUnblacklisted event](SparkListenerBus.md#doPostEvent)

## <span id="onOtherEvent"> onOtherEvent

```scala
onOtherEvent(
  event: SparkListenerEvent): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a custom SparkListenerEvent](SparkListenerBus.md#doPostEvent)

## <span id="onSpeculativeTaskSubmitted"> onSpeculativeTaskSubmitted

```scala
onSpeculativeTaskSubmitted(
  speculativeTask: SparkListenerSpeculativeTaskSubmitted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerSpeculativeTaskSubmitted event](SparkListenerBus.md#doPostEvent)

## <span id="onStageCompleted"> onStageCompleted

```scala
onStageCompleted(
  stageCompleted: SparkListenerStageCompleted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerStageCompleted event](SparkListenerBus.md#doPostEvent)

## <span id="onStageExecutorMetrics"> onStageExecutorMetrics

```scala
onStageExecutorMetrics(
  executorMetrics: SparkListenerStageExecutorMetrics): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerStageExecutorMetrics event](SparkListenerBus.md#doPostEvent)

## <span id="onStageSubmitted"> onStageSubmitted

```scala
onStageSubmitted(
  stageSubmitted: SparkListenerStageSubmitted): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerStageSubmitted event](SparkListenerBus.md#doPostEvent)

## <span id="onTaskEnd"> onTaskEnd

```scala
onTaskEnd(
  taskEnd: SparkListenerTaskEnd): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerTaskEnd event](SparkListenerBus.md#doPostEvent)

## <span id="onTaskGettingResult"> onTaskGettingResult

```scala
onTaskGettingResult(
  taskGettingResult: SparkListenerTaskGettingResult): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerTaskGettingResult event](SparkListenerBus.md#doPostEvent)

## <span id="onTaskStart"> onTaskStart

```scala
onTaskStart(
  taskStart: SparkListenerTaskStart): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerTaskStart event](SparkListenerBus.md#doPostEvent)

## <span id="onUnpersistRDD"> onUnpersistRDD

```scala
onUnpersistRDD(
  unpersistRDD: SparkListenerUnpersistRDD): Unit
```

Used when:

* `SparkListenerBus` is requested to [post a SparkListenerUnpersistRDD event](SparkListenerBus.md#doPostEvent)

## Implementations

* EventFilterBuilder
* SparkFirehoseListener
* [SparkListener](SparkListener.md)
