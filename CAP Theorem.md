The theorem states that within a large-scale distributed data system, there are three requirements that have a relationship of sliding dependency:

#### Consistency
All database clients will read the same value for the same query, even given concurrent updates.

#### Availability
All database clients will always be able to read and write data.

#### Partition tolerance
The database can be split into multiple machines; it can continue functioning in the face of network segmentation breaks.
