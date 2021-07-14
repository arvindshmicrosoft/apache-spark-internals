# SerializerInstance

*SerializerInstance* is an abstraction of instances of a serializer, for use by one thread at a time.

== [[serialize]] serialize Method

[source, scala]
----
serialize[T: ClassTag](
  t: T): ByteBuffer
----

serialize...FIXME

serialize is used when...FIXME

== [[deserialize]] deserialize Method

[source, scala]
----
deserialize[T: ClassTag](
  bytes: ByteBuffer): T
deserialize[T: ClassTag](
  bytes: ByteBuffer,
  loader: ClassLoader): T
----

deserialize...FIXME

deserialize is used when...FIXME

== [[serializeStream]] Serializing Output Stream

[source, scala]
----
serializeStream(
  s: OutputStream): SerializationStream
----

serializeStream...FIXME

serializeStream is used when...FIXME

== [[deserializeStream]] Deserializing Input Stream

[source, scala]
----
deserializeStream(
  s: InputStream): DeserializationStream
----

deserializeStream...FIXME

deserializeStream is used when...FIXME
