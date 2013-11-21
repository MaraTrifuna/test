Titan employs multiple layers of data caching to facilitate fast graph traversals:

h3. Transaction Level Caching

Within an open transaction, Titan maintains two caches:

* Vertex Cache: Caches accessed vertices and their adjacency list (or subsets thereof) so that subsequent access is significantly faster within the same transaction. Hence, this cache speeds up iterative traversals.
* Index Cache: Caches the results for index queries so that subsequent index calls can be served from the cache and do not requires invoking the indexing backend.

Read more about [[transaction level caching | Transaction Cache]].

h3. Database Level Caching

Caches retrieved adjacency lists (or subsets thereof) across multiple transactions and beyond the duration of a single transaction. The database level cache is shared by all transactions across a database. It is more space efficient than the transaction level caches but also slightly slower to access.

Read more about [[database level caching | Database Cache]].

h3. Storage Backend Caching

Each storage backend maintains its own data caching layer. These caches benefit from compression, data compactness, coordinated expiration and are often maintained off heap which means that large caches can be used without running in garbage collection issues.

The exact type of caching and its properties depends on the particular [[storage backend | Storage Backend Overview]]. Please refer to the respective documentation for more information about the caching infrastructure and how to optimize it.