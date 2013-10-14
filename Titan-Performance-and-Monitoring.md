# Metrics in Titan

Starting in version 0.4.0, Titan supports [Metrics](http://metrics.codahale.com/).  Titan records histograms of time spent satisfying storage backend requests.  These histograms form a coarse-grained measure of the response time of the storage engine underlying Titan.

## Configuration

To enable Metrics, set the following in Titan's properties file:

```
# Required
metrics.enable-basic-metrics = true
```

Titan's currently supported Metrics reporters and template configurations for each one follow.

* JMX

  To enable JMX, set the following in Titan's properties file:
  ```
  # Required
  metrics.jmx.enabled = true
  # Optional; if omitted, then Metrics uses its default values
  metrics.jmx.domain = foo
  metrics.jmx.agentid = baz
  ```

  `metrics.jmx.domain` and `.agentid` are strings that override Metrics' internal defaults for the JMX reporting domain and JMX reporting agent ID, respectively.

* Slf4j

  ```
  # Required; specify logging interval in milliseconds
  metrics.slf4j.interval = 60000
  # Optional; uses Metrics default when unset
  metrics.slf4j.logger = foo
  ```

* Console

  ```
  # Required; specify logging interval in milliseconds
  metrics.console.interval = 60000
  ```

* CSV (writes one comma-separated values file per metric to a configurable directory)

  ```
  # Required; specify logging interval in milliseconds
  metrics.csv.interval = 60000
  metrics.csv.dir = foo/bar
  ```

  The output directory for CSV files will be created if it doesn't already exist.

## What's Measured

Titan times, counts, and histograms its method calls against the storage backend.  These measurements are organized by method name.  By default, measurements for the edge store, ID store, vertex index store, and edge index store are combined.  However, the configuration setting `metrics.merge-basic-metrics = false` splits the metrics for four different stores into separate datapoints instead of combining them.  The default metric names follow.

Index stores and caches are not yet covered by metrics.

### Default Names

These metric names are in effect when `metrics.merge-basic-metrics` is either absent from Titan's properties file or is set to true.

#### Counters

```
com.thinkaurelius.titan.stores.acquireLock.calls
com.thinkaurelius.titan.stores.acquireLock.exceptions
com.thinkaurelius.titan.stores.close.calls
com.thinkaurelius.titan.stores.close.exceptions
com.thinkaurelius.titan.stores.containsKey.calls
com.thinkaurelius.titan.stores.containsKey.exceptions
com.thinkaurelius.titan.stores.getKeys.calls
com.thinkaurelius.titan.stores.getKeys.exceptions
com.thinkaurelius.titan.stores.getLocalKeyPartition.calls
com.thinkaurelius.titan.stores.getLocalKeyPartition.exceptions
com.thinkaurelius.titan.stores.getName.calls
com.thinkaurelius.titan.stores.getName.exceptions
com.thinkaurelius.titan.stores.getSlice.calls
com.thinkaurelius.titan.stores.getSlice.entries-returned
com.thinkaurelius.titan.stores.getSlice.exceptions
com.thinkaurelius.titan.stores.mutate.calls
com.thinkaurelius.titan.stores.mutate.exceptions
```

Though most of the Metrics above follow the method-calls and method-exceptions pattern, there is one exception in `getSlice.entries-returned`.  This counts the aggregate number of column-value pairs processed by calls to the `getSlice` method in storage backends.  One method call may return one or very many such pairs, which is why those counters are separated.

#### Timers

```
com.thinkaurelius.titan.stores.acquireLock.time
com.thinkaurelius.titan.stores.close.time
com.thinkaurelius.titan.stores.containsKey.time
com.thinkaurelius.titan.stores.getKeys.time
com.thinkaurelius.titan.stores.getLocalKeyPartition.time
com.thinkaurelius.titan.stores.getName.time
com.thinkaurelius.titan.stores.getSlice.time
com.thinkaurelius.titan.stores.mutate.time
```

#### Histograms

```
com.thinkaurelius.titan.stores.getSlice.entries-histogram
```

This histograms the number of column-value entries returned per `getSlice` call.

### Per-Store Names

When `metrics.merge-basic-metrics = false` is set in Titan's properties file, the `stores` string in the metric names above is replaced by `idStore`, `edgeStore`, `vertexIndexStore`, or `edegIndexStore` according to the role of the backend instance handling the method call.