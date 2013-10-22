# Metrics in Titan

Starting in version 0.4.0, Titan supports [Metrics](http://metrics.codahale.com/).  Titan can measure the following:

* The number of transactions begun, comitted, and rolled back
* The number of attempts and failures of each storage backend operation type
* The response time distribution of each storage backend operation type

## Configuring Metrics Collection

To enable Metrics collection, set the following in Titan's properties file:

```
# Required to enable Metrics in Titan
metrics.enable-basic-metrics = true
```

This setting makes Titan record measurements at runtime using Metrics classes like Timer, Counter, Histogram, etc.  To access these measurements, one or more Metrics reporters must be configured as described in the section "Configuring Metrics Reporting".

### Customizing the Default Metric Names

Titan prefixes all metric names with "com.thinkaurelius.titan".  This prefix can be changed through the `metrics.prefix` configuration property.  For example, to shorten the default "com.thinkaurelius.titan" prefix to just "titan":

```
# Optional
metrics.prefix = titan
```

### Transaction-Specific Metrics Names

Each Titan transaction can override the default Metrics name prefix with a custom value.  For example, the prefix could be changed to the name of the frontend application that opened the Titan transaction.  Note that Metrics maintains a ConcurrentHashMap of metric names and their associated objects in memory, so it's probably a good idea to keep the number of distinct metric prefixes small.

The method is `StandardTransactionBuilder.setMetricsPrefix(String)`:

```java
TitanGraph graph = ...;
TransactionBuilder tbuilder = graph.buildTransaction();
TitanTransaction tx = tbuilder.setMetricsPrefix("foobar").start();
```

### Merging Metrics for Backend Stores

Titan combines the Metrics for its various internal storage backend handles by default.  All Metrics appear under the name "stores", regardless of whether they come from the ID store, edge store, etc.  When `metrics.merge-basic-metrics = false` is set in Titan's properties file, the `stores` string in the metric names above is replaced by `idStore`, `edgeStore`, `vertexIndexStore`, or `edegIndexStore` according to the role of the backend instance triggering the Metric collection.

## Configuring Metrics Reporting

Titan supports the following Metrics reporters:

* Console
* CSV
* Ganglia
* Graphite
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
metrics.enable-basic-metrics = true
# Required; specify logging interval in milliseconds
metrics.console.interval = 60000
```

### CSV File Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.csv.interval | yes | Milliseconds to wait between writing CSV lines | null |
| metrics.csv.dir | yes | Directory in which CSV files are written (will be created if it does not exist) | null |

Example titan.properties snippet that writes CSV files once a minute to the directory `./foo/bar/` (relative to the process's working directory):

```
metrics.enable-basic-metrics = true
# Required; specify logging interval in milliseconds
metrics.csv.interval = 60000
metrics.csv.dir = foo/bar
```

### Ganglia Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.ganglia.hostname | yes | Unicast host or multicast group to which our Metrics are sent | null |
| metrics.ganglia.interval | yes | Milliseconds to wait between sending datagrams | null |
| metrics.ganglia.port | no | UDP port to which we send Metrics datagrams | 8649 |
| metrics.ganglia.addressing-mode | no | Must be "unicast" or "multicast" | unicast |
| metrics.ganglia.ttl | no | Multicast datagram TTL; ignore for unicast | 1 |
| metrics.ganglia.protocol-31 | no | Boolean; true to use Ganglia protocol 3.1, false to use 3.0 | true |
| metrics.ganglia.uuid | no | [Host UUID to report instead of IP:hostname](https://github.com/ganglia/monitor-core/wiki/UUIDSources) | null |
| metrics.ganglia.spoof | no | [Override IP:hostname reported to Ganglia](http://sourceforge.net/apps/trac/ganglia/wiki/gmetric_spoofing) | null |

Example titan.properties snippet that sends unicast UDP datagrams to localhost on the default port once every 30 seconds:

```
metrics.enable-basic-metrics = true
# Required; IP or hostname string
metrics.ganglia.hostname = 127.0.0.1 
# Required; specify logging interval in milliseconds
metrics.ganglia.interval = 30000
```

Example titan.properties snippet that sends unicast UDP datagrams to a non-default destination port and which also spoofs the IP and hostname reported to Ganglia:

```
metrics.enable-basic-metrics = true
# Required; IP or hostname string
metrics.ganglia.hostname = 1.2.3.4 
# Required; specify logging interval in milliseconds
metrics.ganglia.interval = 60000
# Optional
metrics.ganglia.port = 6789
metrics.ganglia.spoof = 10.0.0.1:zombo.com
```

### Graphite Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.graphite.hostname | yes | IP address or hostname to which [Graphite plaintext protocol](https://graphite.readthedocs.org/en/latest/feeding-carbon.html#the-plaintext-protocol) data are sent | null |
| metrics.graphite.interval | yes | Milliseconds to wait between pushing data to Graphite | null |
| metrics.graphite.port | no | Port to which Graphite plaintext protocol reports are sent | 2003 |
| metrics.graphite.prefix | no | Arbitrary string prepended to all metric names sent to Graphite | null |

Example titan.properties snippet that sends metrics to a Graphite server on 192.168.0.1 every minute:

```
metrics.enable-basic-metrics = true
# Required; IP or hostname string
metrics.graphite.hostname = 192.168.0.1
# Required; specify logging interval in milliseconds
metrics.graphite.interval = 60000
```

### JMX Reporter Configuration

| Config Key | Required? | Value | Default |
| ---------- | --------- | ----- | ------- |
| metrics.jmx.enabled | yes | Boolean | false |
| metrics.jmx.domain | no | Metrics will appear in this JMX domain | Metrics's own default |
| metrics.jmx.agentid | no | Metrics will be reported with this JMX agent ID | Metrics's own default |

Example titan.properties snippet:

```
metrics.enable-basic-metrics = true
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
metrics.enable-basic-metrics = true
# Required; specify logging interval in milliseconds
metrics.slf4j.interval = 60000
# Optional; uses Metrics default when unset
metrics.slf4j.logger = foo
```

### User-Provided/Custom Reporter Configuration

In case the Metrics reporter configuration options listed above are insufficient, Titan provides a utility method to access the single `MetricRegistry` instance which holds all of its measurements.

```java
com.codahale.metrics.MetricRegistry titanRegistry =
    com.thinkaurelius.titan.util.stats.MetricManager.INSTANCE.getRegistry();
```

Code that accesses `titanRegistry` this way can then attach non-standard reporter types or standard reporter types with exotic configurations to `titanRegistry`.  This approach is also useful if the surrounding application already has a framework for Metrics reporter configuration, or if the application needs multiple differently-configured instances of one of Titan's supported reporter types.  For instance, one could use this approach to setup multiple unicast Graphite reporters whereas Titan's properties configuration is limited to just one Graphite reporter.

# JUnitBenchmarks in Titan

Starting in version 0.4.0, several of Titan's JUnit tests have been converted to [JUnitBenchmarks](http://labs.carrotsearch.com/junit-benchmarks.html).  To execute these tests in a clone of the Titan repository, invoke the following command:

```
mvn -Pperformance-test,memory-test clean install
```

This can be issued in the repository root to test all storage backends, or in a particular storage backend module, such as `titan-cassandra` or `titan-berkeleyje`, to test only that backend.  The results are written to console and to a series of files named `jub.<nanotime>.xml`, one file per test.

These tests are executed weekly on an AWS EC2 m1.medium instance and pushed to [Ducksboard](https://public.ducksboard.com/xWvWuJp2J0uMHm7TC_BM/).