The initial Titan import is tagged as _v0.1_ in the github repository. This initial import of Titan contains may of the research prototype implementations that were subsequently removed to prepare for the initial release:

* Query sending/forwarding and reference node implementations
* Range Queries which includes interface for getting keySlice in OrderedReadKeyColumnValueStore [including test methods]
* Removed support for Berkeley DB (standard C version with Java interface) [including test cases]
* Removed support for Cassandra direct interface (local JVM interface) [including test cases]

It is very likely that these features will re-emerge in a future version of Titan. At this point, it is very beneficial to consider the old codebase as a starting point. For the initial release, these features were removed to make Titan simpler. 