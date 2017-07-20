# Aerospike advantage over redis

```sh
1. Redis Application sharded Aerospike Auto sharded,Auto rebalancing and Auto Clustering
2. Redis Single Threaded Aerospike multi threaded
3. Redis support more datastructures centric and aerospike support key value centeric
4. 2^32 max keys per node but in aerospike you can keep 2^160 max keys per namespace
5. Caller App should know redis node Caller app need not know the aerospike node
6. Redis manual config of master-slave Aerospike auto assignment of master-slave
7. Async Replication in redis and Sync replication in aerospike
8. A node is only master or slave in redis but in aerospike node is both master and slave
```

# Aerospike defintion

1. Aerospike performs at a demonstrated per server transaction rate of at least 1M transactions per second (TPS). Beyond being highly performant, Aerospike is predictably performant, delivering high write throughput at low latencies, which enables enterprises to more easily build larger-scale applications at a lower cost.
2. Aerospike provides high availability and a demonstrated uptime of five 9s, enabled by unique cluster management and client technology, as well as local and remote replication.
3. Aerospike’s dynamic cluster management and unique in-memory technology enable our database to reliably handle millions of transactions per second, efficiently scaling to meet any data volume needs.
4. Aerospike is a distributed NoSQL database and key-value store architected for the performance needs of today’s web-scale applications; providing robustness and strong consistency with no downtime.
5. Aerospike can store data in DRAM, Flash (SSDs), and traditional rotating media. Indexes in RAM for high performance and parallelism Multiple data storage models Schema-less data model Built-in defragmentation Integrated continual data expiration Large block writes and small block reads for high performance.
6. Enables aggregations through programmatic indexed-MapReduce Aggregation system is implemented using User Defined Functions (UDFs) written in Lua
7. Provides value-based lookup through the use of bins (columns) and secondary indexes.

# Aerospike Basics : -

```sh
-> Aerospike uses an partition table
-> keys is hashed using hash function
-> 160 bits string and 12 bits of hash are used to identify partition id.
-> total 4096 partitions
-> structure like (key -> hash -> partition table)
-> partition table be like (partition id | master node | replica node)
```

# AeroSpike Terms : -
```sh
cluster - database service
node - instance of aerospike database
namespace - area of stroage realted to the media. Similiar to database
set - unstructed grouping of data like tables
record - key and values like rows
Bin - one part of data related to key like same bin in difference record have different type
```

# AeroSpike Hierarchy : -
```sh
Cluster -> group of nodes -> collection of namespaces -> collection of sets -> collection of records -> Record is collection of bins.
```

# Aerospike Cluster -

distribute request on different nodes
management of cluster is automated

# Nodes - 
traffic or data will be evenly balanced across the nodes

# Namespaces -

Associated with storage media
Hybrid - RAM+SSD
RAM+disk for presistance
RAM only

# Sets- 
similiar to tables
not pre defined
request of order results scan of set

# Records - 
similiar to row
all data for record will be stored on the same node
any change to record will result in a complete write of the entire record

# BINS - 
Values are typed.
-> Simple (integer,string)
->complex (list,map)
-> large datatype (LDTs)
any single bin can be updated by client

# How to Access Object :-
```sh
1. Client find master node from partition map
2. Client makes read request to master node
3. Master node find data location from index in RAM
4. Master node read object from SSD
5. Master node return value
```
# How to add object : -
```sh
1. client find master node
2. master node make write request to master
3. master node make an ram index and queues write in temp buffer
4. master node returns success to client
5. master node asynchronously writes data in blocks
6. index in ram point to that SSD data
```
# How Aerospike manage space - 
```sh
1. write data in large block in SSD
2. After some time when some data blocks deleted or updated
3. Remove that unused space by defragmentation which look 50% block occupied or not 

Old database user Database compaction compresses the database file by removing unused file sections created during updates
```

# Aerospike Data management - 

Every database has a maximum capcity.
so aerospike manage them using following operations : -
1. TTL or expiration
Aerospike has a mechanism for automtically expire data if it is not changed in a while In the config of namespace we can set ttl when data reached that ttl it will be automatically deleted from database.
dont want data to expire but want to keep all data set ttl value to 0.

# High Watermarks and Eviction : -
```sh
High watermark is therhold for ram or disk use. when water mark has been breached the server will begin to evict data. it will start by evicting data closest to its ttl.
For example: 
namespace has 90 day ttl.
server delete data automatically on 90th day.
but when server hits high waterwatermark it will begin to delete data that is 89 days then 88 days.
```
# stop writes :-
```sh
suppose data is being written/changed in database at high rate in which we can not rely on eviction then to prevent data loss we can keep stop write level like 100% then database will stop writing new data. then server will respond error to new request.
```
2 Namespace:-
1. RAM
2. SSD/Flash

Common parameter-
Replication factor 
watermarks
DRAM allocation
TTL

```sh
Best Replication factor - 2
1 - would not give databackup cause to loss of data
3 or more - require addition storage
Replication factor - copies of data in the database cluster
change dymacially no
```

# Storage watermark

-> high-water-memory-pct
-> high-water-disk-pct
-> stop-writes-pct
-> Dynamically change yes
-> watermark defines level of triggering action to act as safety valves
-> for proper defragementation keep the high-water-disk at 50

# RAM Allocation - 

All namespace require RAM for index
memory-size
change dynamically

# Common Config
```sh
namespace test_namespace {
replication-factor 2
high-water-memory-pct 60
high-water-disk-pct 50
stop-writes-pct 90
memory-size 4G
default-ttl 86400(1day)
}

storage-engine device {
file /opt/aerospike/data/test.data
filesize 32G
load-at-startup true
data-in-memory true
}
```

# aql(Aerospike query language) query
```sh
aql> set key_send true
aql> set output json
aql> set record_print_metadata true
aql> INSERT INTO <namespace>.<setname> (<primary-key>, <bin1>, <bin2>) VALUES ('vijay', 'abc', 123)
aql> select * from <namespace>.<setname> where <primary-key>='vijay'
aql> delete from <namespace>.<setname> where <primary-key>='vijay'
```
# aql terms
```sh
key	      Unique identifier. Records are addressable using a hash of its key, called the digest.
metadata	Record version information (generation) and the configured expiration, called the time-to-live (TTL).
bins	    Bins are equivalent to fields in RDBMS.
```
