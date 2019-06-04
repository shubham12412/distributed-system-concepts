The theorem states that within a large-scale distributed data system, there are three requirements that have a relationship of sliding dependency:

#### Consistency
All database clients will read the same value for the same query, even given concurrent updates.

#### Availability
All database clients will always be able to read and write data.

#### Partition tolerance
The database can be split into multiple machines; it can continue functioning in the face of network segmentation breaks.

Brewer’s theorem is that in any given system, you can strongly support only two of the three. This is analogous to the saying you may have heard in software development: “You can have it good, you can have it fast, you can have it cheap: pick two.”

We have to choose between them because of this sliding mutual dependency. The more consistency you demand from your system, for example, the less partition-tolerant you’re likely to be able to make it, unless you make some concessions around availability.



