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
2) 

Another way to attempt to scale a relational database is to introduce sharding to your architecture. This has been used to good effect at large websites such as eBay, which supports billions of SQL queries a day, and in other modern web applications. The idea here is that you split the data so that instead of hosting all of it on a single server or replicating all of the data on all of the servers in a cluster, you divide up portions of the data horizontally and host them each separately.



For example, consider a large customer table in a relational database. The least disruptive thing (for the programming staff, anyway) is to vertically scale by adding CPU, adding memory, and getting faster hard drives, but if you continue to be successful and add more customers, at some point (perhaps into the tens of millions of rows), you’ll likely have to start thinking about how you can add more machines. When you do so, do you just copy the data so that all of the machines have it? Or do you instead divide up that single customer table so that each database has only some of the records, with their order preserved? Then, when clients execute queries, they put load only on the machine that has the record they’re looking for, with no load on the other machines.


It seems clear that in order to shard, you need to find a good key by which to order your records. For example, you could divide your customer records across 26 machines, one for each letter of the alphabet, with each hosting only the records for customers whose last names start with that particular letter. It’s likely this is not a good strategy, however—there probably aren’t many last names that begin with “Q” or “Z,” so those machines will sit idle while the “J,” “M,” and “S” machines spike. You could shard according to something numeric, like phone number, “member since” date, or the name of the customer’s state. It all depends on how your specific data is likely to be distributed.


There are three basic strategies for determining shard structure:

Feature-based shard or functional segmentation

This is the approach taken by Randy Shoup, Distinguished Architect at eBay, who in 2006 helped bring the site’s architecture into maturity to support many billions of queries per day.  Using this strategy, the data is split not by dividing records in a single table (as in the customer example discussed earlier), but rather by splitting into separate databases the features that don’t overlap with each other very much. For example, at eBay, the users are in one shard, and the items for sale are in another. At Flixster, movie ratings are in one shard and comments are in another. This approach depends on understanding your domain so that you can segment data cleanly.

Key-based sharding

In this approach, you find a key in your data that will evenly distribute it across shards. So instead of simply storing one letter of the alphabet for each server as in the (naive and improper) earlier example, you use a one-way hash on a key data element and distribute data across machines according to the hash. It is common in this strategy to find time-based or numeric keys to hash on.


Lookup table

In this approach, one of the nodes in the cluster acts as a “yellow pages” directory and looks up which node has the data you’re trying to access. This has two obvious disadvantages. The first is that you’ll take a performance hit every time you have to go through the lookup table as an additional hop. The second is that the lookup table not only becomes a bottleneck, but a single point of failure.


Sharding could be termed a kind of “shared-nothing” architecture that’s specific to databases. A shared-nothing architecture is one in which there is no centralized (shared) state, but each node in a distributed system is independent, so there is no client contention for shared resources.


-----------------------------------------------------------------------------------------------------------------
3.

If you take a look, you’ll see that many of the features of shared-nothing distributed data architecture, such as ease of high availability and the ability to scale to a very large number of machines, are the very things that Cassandra excels at.

MongoDB also provides auto-sharding capabilities to manage failover and node balancing. That many non-relational databases offer this automatically and out of the box is very handy; creating and maintaining custom data shards by hand is a wicked proposition. It’s good to understand sharding in terms of data architecture in general, but especially in terms of Cassandra more specifically, as it can take an approach similar to key-based sharding to distribute data across nodes, but does so automatically.

-------------------------------------------------------------------------------------------------------------------
4.

Web Scale

In summary, relational databases are very good at solving certain data storage problems, but because of their focus, they also can create problems of their own when it’s time to scale. Then, you often need to find a way to get rid of your joins, which means denormalizing the data, which means maintaining multiple copies of data and seriously disrupting your design, both in the database and in your application. Further, you almost certainly need to find a way around distributed transactions, which will quickly become a bottleneck. These compensatory actions are not directly supported in any but the most expensive RDBMSs. And even if you can write such a huge check, you still need to carefully choose partitioning keys to the point where you can never entirely ignore the limitation.



