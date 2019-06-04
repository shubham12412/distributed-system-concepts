https://learning.oreilly.com/library/view/cassandra-the-definitive/9781491933657/ch01.html

1) We encounter scalability problems when our relational applications become successful and usage goes up. Joins are inherent in any relatively normalized relational database of even modest size, and joins can be slow. The way that databases gain consistency is typically through the use of transactions, which require locking some portion of the database so it’s not available to other clients. This can become untenable under very heavy loads, as the locks mean that competing users start queuing up, waiting for their turn to read or write the data.


We typically address these problems in one or more of the following ways, sometimes in this order:

1.1) Throw hardware at the problem by adding more memory, adding faster processors, and upgrading disks. This is known as ***vertical scaling***. This can relieve you for a time.

1.2) When the problems arise again, the answer appears to be similar: ***now that one box is maxed out, you add hardware in the form of additional boxes in a database cluster.  Now you have the problem of ***data replication and consistency*** during regular usage and in failover scenarios. You didn’t have that problem before.

1.3)  Now we need to update the configuration of the database management system. This might mean optimizing the channels the database uses to write to the underlying filesystem. We turn off logging or journaling, which frequently is not a desirable (or, depending on your situation, legal) option.

1.4) Having put what attention we could into the database system, we turn to our application. ***We try to improve our indexes. We optimize the queries***. But presumably at this scale we weren’t wholly ignorant of index and query optimization, and already had them in pretty good shape. So this becomes a painful process of picking through the data access code to find any opportunities for fine-tuning. This might include ***reducing or reorganizing joins, throwing out resource-intensive features such as XML processing within a stored procedure***, and so forth. Of course, presumably we were doing that XML processing for a reason, so if we have to do it somewhere, we move that problem to the application layer, hoping to solve it there and crossing our fingers that we don’t break something else in the meantime.

1.5) We employ a ***caching layer***. For larger systems, this might include ***distributed caches such as memcached, Redis, Riak, EHCache***, or other related products. ***Now we have a consistency problem between updates in the cache and updates in the database***, which is exacerbated over a cluster.

1.6) We turn our attention to the database again and decide that, now that the application is built and we understand the primary query paths, we can duplicate some of the data to make it look more like the queries that access it. This process, called ***denormalization***, is antithetical to the five normal forms that characterize the relational model, and violates Codd’s 12 Rules for relational data. We remind ourselves that we live in this world, and not in some theoretical cloud, and then undertake to do what we must to make the application start responding at acceptable levels again, even if it’s no longer “pure.”

------------------------------------------------------------------------------------------------------------------------




