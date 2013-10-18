# Metrics in Titan

Starting in version 0.4.0, Titan supports [Metrics](http://metrics.codahale.com/).  Titan records histograms of time spent satisfying storage backend requests.  These histograms form a coarse-grained measure of the response time of the storage engine underlying Titan.

## Configuration

To enable Metrics, set the following in Titan's properties file:

```
# Required
metrics.enable-basic-metrics = true
```

Titan supports the following Metrics reporters:

* Console
* CSV
* Ganglia
* JMX
* Slf4j
* User-provided/Custom

Each reporter type is independent of and can coexist with the others.  For example, it's possible to configure Ganglia, JMX, and Slf4j Metrics reporters to operate simultaneously.  Just set all their respective configuration keys in titan.properties (and enable metrics as directed above).

### Console Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.console.interval | yes | Milliseconds to wait between dumping metrics to the console | null |

Example titan.properties snippet that prints metrics to the console once a minute:

```
# Required; specify logging interval in milliseconds
metrics.console.interval = 60000
```

### CSV File Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.csv.interval | yes | Milliseconds to wait between writing CSV lines | null |
| metrics.csv.dir | yes | Directory in which CSV files are written (will be created if it does not exist) | null |

Example titan.properties snippet that writes files once a minute to `./foo/bar`:

```
# Required; specify logging interval in milliseconds
metrics.csv.interval = 60000
metrics.csv.dir = foo/bar
```

### Ganglia Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.ganglia.host | yes | Unicast host or multicast group to which our Metrics are sent | null |
| metrics.ganglia.interval | yes | Milliseconds to wait between sending datagrams | null |
| metrics.ganglia.port | no | UDP port to which we send Metrics datagrams | 8649 |
| metrics.ganglia.addressing-mode | no | Must be "unicast" or "multicast" | unicast |
| metrics.ganglia.ttl | no | Multicast datagram TTL; ignore for unicast | 1 |
| metrics.ganglia.protocol-31 | no | Boolean; true to use Ganglia protocol 3.1, false to use 3.0 | true |
| metrics.ganglia.uuid | no | [Host UUID to report instead of IP:hostname](https://github.com/ganglia/monitor-core/wiki/UUIDSources) | null |
| metrics.ganglia.spoof | no | [Override IP:hostname reported to Ganglia](http://sourceforge.net/apps/trac/ganglia/wiki/gmetric_spoofing) | null |

Example titan.properties snippet that sends unicast UDP datagrams to localhost on the default port once every 30 seconds:

```
# Required; IP or hostname string
metrics.ganglia.host = 127.0.0.1 
# Required; specify logging interval in milliseconds
metrics.ganglia.interval = 30000
```

Example titan.properties snippet that sends unicast UDP datagrams to a non-default destination port and which also spoofs the IP and hostname reported to Ganglia:

```
# Required; IP or hostname string
metrics.ganglia.host = 1.2.3.4 
# Required; specify logging interval in milliseconds
metrics.ganglia.interval = 60000
# Optional
metrics.ganglia.port = 6789
metrics.ganglia.spoof = 10.0.0.1:zombo.com
```

### JMX Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.jmx.enabled | yes | Boolean | false |
| metrics.jmx.domain | no | Metrics will appear in this JMX domain | Metrics's own default |
| metrics.jmx.agentid | no | Metrics will be reported with this JMX agent ID | Metrics's own default |

Example titan.properties snippet:

```
# Required
metrics.jmx.enabled = true
# Optional; if omitted, then Metrics uses its default values
metrics.jmx.domain = foo
metrics.jmx.agentid = baz
```

### Slf4j Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.slf4j.interval | yes | Milliseconds to wait between dumping metrics to the logger | null |
| metrics.slf4j.logger | no | Slf4j logger name to use | "metrics" |

Example titan.properties snippet that logs metrics once a minute to the logger named `foo`:

```
# Required; specify logging interval in milliseconds
metrics.slf4j.interval = 60000
# Optional; uses Metrics default when unset
metrics.slf4j.logger = foo
```

### User-Provided/Custom Reporter Configuration

In case the Metrics reporter configuration options presented above don't meet your needs, Titan provides a utility method to access the single `MetricRegistry` instance which holds all of its measurements.

```java
com.codahale.metrics.MetricRegistry titanRegistry =
    com.thinkaurelius.titan.util.stats.MetricManager.INSTANCE.getRegistry();
```

You can attach arbitrary reporters to `titanRegistry`, including non-standard reporter types or standard reporter types with exotic configurations.  This approach is also useful if you need multiple differently-configured instances of one of Titan's supported reporter types.  For instance, you could use the `titanRegistry` to setup multiple unicast Graphite reporters whereas Titan only natively supports a single Graphite reporter configuration.

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

# JUnitBenchmarks in Titan

Starting in version 0.4.0, several of Titan's JUnit tests have been converted to [JUnitBenchmarks](http://labs.carrotsearch.com/junit-benchmarks.html).  To execute these tests in a clone of the Titan repository, invoke the following command:

```
mvn -Pperformance-test,memory-test clean install
```

This can be issued in the repository root to test all storage backends, or in a particular storage backend module, such as `titan-cassandra` or `titan-berkeleyje`, to test only that backend.  The results are written to console and to a series of files named `jub.<nanotime>.xml`, one file per test.

These tests are executed weekly on an AWS EC2 m1.medium instance and pushed to [Ducksboard](https://public.ducksboard.com/xWvWuJp2J0uMHm7TC_BM/).