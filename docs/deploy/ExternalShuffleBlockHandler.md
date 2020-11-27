= ExternalShuffleBlockHandler

*ExternalShuffleBlockHandler* is a network:RpcHandler.md[].

When created, ExternalShuffleBlockHandler requires a network:OneForOneStreamManager.md[] and network:TransportConf.md[] with a `registeredExecutorFile` to create a `ExternalShuffleBlockResolver`.

It <<handleMessage, handles two `BlockTransferMessage` messages>>: <<OpenBlocks, OpenBlocks>> and <<RegisterExecutor, RegisterExecutor>>.

== [[handleMessage]] Handling Messages

[source, java]
----
handleMessage(
  BlockTransferMessage msgObj,
  TransportClient client,
  RpcResponseCallback callback)
----

handleMessage handles two types of BlockTransferMessage messages:

* <<OpenBlocks, OpenBlocks>>
* <<RegisterExecutor, RegisterExecutor>>

For any other `BlockTransferMessage` message it throws a `UnsupportedOperationException`:

```
Unexpected message: [msgObj]
```

== [[OpenBlocks]] OpenBlocks

[source, java]
----
OpenBlocks(
  String appId,
  String execId,
  String[] blockIds)
----

When `OpenBlocks` is received, <<handleMessage, handleMessage>> authorizes the `client`.

CAUTION: FIXME `checkAuth`?

It then <<ExternalShuffleBlockResolver-getBlockData, gets block data>> for each block id in `blockIds` (using <<ExternalShuffleBlockResolver, ExternalShuffleBlockResolver>>).

Finally, it network:OneForOneStreamManager.md#registerStream[registers a stream] and does `callback.onSuccess` with a serialized byte buffer (for the `streamId` and the number of blocks in `msg`).

CAUTION: FIXME `callback.onSuccess`?

You should see the following TRACE message in the logs:

```
Registered streamId [streamId] with [length] buffers for client [clientId] from host [remoteAddress]
```

== [[RegisterExecutor]] RegisterExecutor

[source, java]
----
RegisterExecutor(
  String appId,
  String execId,
  ExecutorShuffleInfo executorInfo)
----

RegisterExecutor...FIXME

== [[logging]] Logging

Enable `ALL` logging level for `org.apache.spark.network.shuffle.ExternalShuffleBlockHandler` logger to see what happens inside.

Add the following line to `conf/log4j.properties`:

[source,plaintext]
----
log4j.logger.org.apache.spark.network.shuffle.ExternalShuffleBlockHandler=ALL
----

Refer to spark-logging.md[Logging].
