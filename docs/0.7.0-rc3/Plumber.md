---
layout: doc_page
---

# Druid Plumbers
The plumber handles generated segments both while they are being generated and when they are "done". This is also technically a pluggable interface and there are multiple implementations. However, plumbers handle numerous complex details, and therefore an advanced understanding of Druid is recommended before implementing your own. 

|Field|Type|Description|Required|
|-----|----|-----------|--------|
|type|String|Specifies the type of plumber. Each value will have its own configuration schema. Plumbers packaged with Druid are described below. The default type is "realtime".|yes|

The following can be configured on the plumber:

* `windowPeriod` is the amount of lag time to allow events. This is configured with a 10 minute window, meaning that any event more than 10 minutes ago will be thrown away and not included in the segment generated by the realtime server.
* `basePersistDirectory` is the directory to put things that need persistence. The plumber is responsible for the actual intermediate persists and this tells it where to store those persists.
* `maxPendingPersists` is the maximum number of persists that can be pending, but not started. If this limit would be exceeded by a new intermediate persist, ingestion will block until the currently-running persist finishes.
* `segmentGranularity` specifies the granularity of the segment, or the amount of time a segment will represent.
* `rejectionPolicy` controls how data sets the data acceptance policy for creating and handing off segments. The following policies are available:
    * `serverTime` &ndash; The recommended policy for "current time" data, it is optimal for current data that is generated and ingested in real time. Uses `windowPeriod` to accept only those events that are inside the window looking forward and back.
    * `messageTime` &ndash; Can be used for non-"current time" as long as that data is relatively in sequence. Events are rejected if they are less than `windowPeriod` from the event with the latest timestamp. Hand off only occurs if an event is seen after the segmentGranularity and `windowPeriod`.
    * `none` &ndash; Never hands off data unless shutdown() is called on the configured firehose.
    * `test` &ndash; Useful for testing that handoff is working, *not useful in terms of data integrity*. It uses the sum of `segmentGranularity` plus `windowPeriod` as a window.

Available Plumbers
------------------

#### YeOldePlumber

This plumber creates single historical segments.

#### RealtimePlumber

This plumber creates real-time/mutable segments.
