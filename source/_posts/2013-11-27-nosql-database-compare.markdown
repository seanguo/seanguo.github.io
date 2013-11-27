---
layout: post
title: "NoSql database compare"
date: 2013-11-27 16:47:57 +0800
comments: true
categories: nosql
---
##Introduction

The goal of this article is trying compare the popular NoSql databases and choose one for one of our company's product. It was based on SQL database but suffering from low scalibility and performance. So we want to migrate to NoSQL based product. Since Most of the NoSQL database are of high scalibility and performance under some condition. 

Before that we need to know some of the backgournd knowlege. Currently there are two kind of data store model:</br>

###RDBMS:

A short definition of an RDBMS is: a DBMS in which data is stored in tables and the relationships among the data are also stored in tables. The data can be accessed or reassembled in many different ways without having to change the table forms.</br>

This one is the data store model currently used in the SIPoint product that is facilitated by the using of the Hibernate framework to get a uniform interface to manipulate all kinds of database. The Hibernate framework is doing the Object Relation Mapping to keep programmer from complex SQL language. </br>

But the disadvantage of the RDBMS is the low efficiency when get lots of concurrent write operation to the database and it is hard to scale the database to multiple servers.  And in the contrast here comes:

###NoSql:
In computing, NoSQL is a broad class of database management systems that differ from classic relational database management systems (RDBMSes) in some significant ways. These data stores may not require fixed table schemas, usually avoid join operations, and typically scale horizontally. Academia typically refers to these databases as structured storage, a term that would include classic relational databases as a subset.
NoSql sacrifices some not necessary consistence in some circumstances to get high performance and easy to scale up.</br>

##NoSQL vs. SQL databases

There are some concepts must be given a short introduction before we can do the compare.</br>

* **Relation Model**: The relational model's central idea is to describe a database as a collection of predicates over a finite set of predicate variables, describing constraints on the possible values and combinations of values. The content of the database at any given time is a finite (logical) model of the database, i.e. a set of relations, one per predicate variable, such that all predicates are satisfied. A request for information from the database (a database query) is also a predicate.
* **ACID**: ACID (atomicity, consistency, isolation, durability) is a set of properties that guarantee database transactions are processed reliably.
* **Atomicity** requires that database modifications must follow an "all or nothing" rule. Each transaction is said to be atomic. If one part of the transaction fails, the entire transaction fails and the database state is left unchanged.
* **Isolation** refers to the requirement that other operations cannot access data that has been modified during a transaction that has not yet completed.
* **Durability** is the ability of the DBMS to recover the committed transaction updates against any kind of system failure (hardware or software).
* **CAP**: The CAP theorem, also known as Brewer's theorem, states that there are three primary concerns you must balance when choosing a data management system: consistency, availability, and partition tolerance.</br>
* **Consistency** means that each client always has the same view of the data. </br>
* **Availability** means that all clients can always read and write.</br>
* **Partition** tolerance means that the system works well across physical network partitions.</br>
 According to the theorem, a distributed system can satisfy any two of these guarantees at the same time, but not all three.</br>
 
 
* **Data Model**:
* Relational systems are the databases we've been using for a while now. RDBMSs and systems that support ACIDity and joins are considered relational.</br>
* Key-value systems basically support get, put, and delete operations based on a primary key.</br>
* Column-oriented systems still use tables but have no joins (joins must be handled within your 
application). Obviously, they store data by column as opposed to traditional row-oriented databases. This makes aggregations much easier.</br>
* Document-oriented systems store structured "documents" such as JSON or XML but have no joins (joins must be handled within your application). It's very easy to map data from object-oriented software to these systems.</br>

* **MapReduce**: MapReduce is a framework for processing huge datasets on certain kinds of distributable problems using a large number of computers (nodes), collectively referred to as a cluster (if all nodes use the same hardware) or as a grid (if the nodes use different hardware). Computational processing can occur on data stored either in a filesystem (unstructured) or within a database (structured).
* "**Map**" step: The master node takes the input, partitions it up into smaller sub-problems, and distributes those to worker nodes. A worker node may do this again in turn, leading to a multi-level tree structure. The worker node processes that smaller problem, and passes the answer back to its master node.
* "**Reduce**" step: The master node then takes the answers to all the sub-problems and combines them in some way to get the output – the answer to the problem it was originally trying to solve.</br>
The advantage of MapReduce is that it allows for distributed processing of the map and reduction operations. Provided each mapping operation is independent of the others, all maps can be performed in parallel. This framework is often applied by the distributed NoSQL databases.</br>

* **Eventual consistency**: This is a specific form of weak consistency; the storage system guarantees that if no new updates are made to the object, eventually all accesses will return the last updated value. If no failures occur, the maximum size of the inconsistency window can be determined based on factors such as communication delays, the load on the system, and the number of replicas involved in the replication scheme. The most popular system that implements eventual consistency is DNS (Domain Name System). Updates to a name are distributed according to a configured pattern and in combination with time-controlled caches; eventually, all clients will see the update.</br>
* **Multiversion concurrency control** (abbreviated MCC or MVCC), in the database field of computer science, is a concurrency control method commonly used by database management systems to provide concurrent access to the database and in programming languages to implement transactional memory.</br>

For instance, a database will implement updates not by deleting an old piece of data and overwriting it with a new one, but instead by marking the old data as obsolete and adding the newer "version." Thus there are multiple versions stored, but only one is the latest. This allows the database to avoid overhead of filling in holes in memory or disk structures but requires (generally) the system to periodically sweep through and delete the old, obsolete data objects.</br>

Below are the general compare between the two data store models:</br>

| Name  | Schema | ACID | CAP | Efficiency | Scalability | Programming |
| ----- | ------ | ---- | --- | ---------- | ----------- | ----------- |
| RDBMS | Fixed  | All | Hardly no P | Hard to scale | Hibernate to unify all DB |
| NoSQL | Not require fixed schema | Not All | CA or AP | High performance on IO, and can use MapReduce to distribute work to multiple machines | High scalability on design | Each database has its own API, but it’s simple |

</br>
###NoSql databases compare

Below we choose some popular NoSQL databases to do a comprehensive compare:
 
###Basic compare

| Name | Type | Open Source  | Source Language | CAP | Managing shell | Query Method | API | Concurrency | Successful usecases   | Scalability |
|:----- | -------- | ----------- | --------------- | --- | -------------- | ------------ | --- | ----------- | 
| Cassadra | Wide column | Y | JAVA | AP, Configurable C | No shell  | MapReduce | Many Thrift client | Eventually consistence | Digg, Facebook, Twitter etc. | P2P, no single points of failure
| Voldemort | Key-value | Y | C++ | AP | Built-in |   | Java | Eventually Consistent | Linkedin | Dynamo
| Kyoto Cabinet & Kyoto Tycoon | Key-value | Y | C++ | AP | Command Line Utility `kchashmgr' | MapReduce | Many langs | Eventually consistency | mixi.jp | Asynchronous Master-slave or dual-master replication
| MongoDB | Document oriented | Y | C++ | CP | Built-in | Dynamic object-based language & MapReduce | BSON | Update in Place.  | The New York Times,SourceForge | Master-Slave Replication |
| HBase | Wide column | Y | JAVA | CP | Jruby | MapReduce Java / any exec | Java / any writer  |   |   | Hadoop
| Redis | Key-value | Y | C | CP | Built-in |  | Tons of languages | In memory and saves asynchronous disk after a defined time. Append only mode available. Different kinds of fsync policies. | github，Engine Yard | Master-slave replication
| CouchDB | Document oriented | Y | Erlang | AP | None | MapReduceR of JavaScript | Funcs REST/JSON | MVCC |  |  peer-based distributed

</br>
###Farther compare:

For those databases mentioned in the above, first we dropped CouchDB since it only has REST API. KC and Redis are the lightweight Key-value implementation and can scale up in a small cluster. And Cassandra, Voldemort and HBase are good solutions for large-scale clusters. And for MongoDB the only document oriented database we will describe it later. And we will regroup these databases and do a farther compare.

####Cassandra:

The Apache Cassandra Project develops a highly scalable second-generation distributed database, bringing together Dynamo's fully distributed design and Bigtable's ColumnFamily-based data model.
Cassandra was open sourced by Facebook in 2008, and is now developed by Apache committers and contributors from many companies.

#####Feature:

* Fault Tolerant 
Data is automatically replicated to multiple nodes for fault-tolerance. Replication across multiple data centers is supported. Failed nodes can be replaced with no downtime. 
* Decentralized
Every node in the cluster is identical. There are no network bottlenecks. There are no single points of failure. 
* You're in Control 
Choose between synchronous or asynchronous replication for each update. Highly available asynchronous operations are optimized with features like Hinted Handoff and Read Repair.
* Rich Data Model 
Allows efficient use for many applications beyond simple key/value. 
* Elastic 
Read and write throughput both increase linearly as new machines are added, with no downtime or interruption to applications. 
* Durable 
Cassandra is suitable for applications that can't afford to lose data, even when an entire data center goes down. 

#####Data Model:

Cassandra provides a structured key-value store with tunable consistency. Keys map to multiple values, which are grouped into column families. The column families are fixed when a Cassandra database is created, but columns can be added to a family at any time. Furthermore, columns are added only to specified keys, so different keys can have different numbers of columns in any given family.


The values from a column family for each key are stored together. This makes Cassandra a hybrid data management system between a column-oriented DBMS and a row-oriented store. Also, besides using the way of modeling of BigTable, it has properties like eventual consistency, the Gossip protocol, a master-master way of serving the read and write requests that are inspired by Amazon's Dynamo.
The basic concepts are:

* Cluster: the machines (nodes) in a logical Cassandra instance. Clusters can contain multiple keyspaces.
* Keyspace: a namespace for ColumnFamilies, typically one per application.
* ColumnFamilies contain multiple columns, each of which has a name, value, and a timestamp, and which are referenced by row keys.
* SuperColumns can be thought of as columns that themselves have subcolumns.

**Limitations**:

* All data for a single row must fit (on disk) on a single machine in the cluster. Because row keys alone are used to determine the nodes responsible for replicating their data, the amount of data associated with a single key has this upper bound.
* A single column value may not be larger than 2GB.
* The maximum of column per row is 2 billion.
* The key (and column names) must be under 64K bytes.

####HBase

HBase is the Hadoop database. Use it when you need random, realtime read/write access to your Big Data. This project's goal is the hosting of very large tables -- billions of rows X millions of columns -- atop clusters of commodity hardware.
HBase is an open-source, distributed, versioned, column-oriented store modeled after Google' Bigtable: A Distributed Storage System for Structured by Chang et al. Just as Bigtable leverages the distributed data storage provided by the Google File System, HBase provides Bigtable-like capabilities on top of Hadoop.

#####Feature:

* Convenient base classes for backing Hadoop MapReduce jobs with HBase tables including cascading, hive and pig source and sink modules 
* Query predicate push down via server side scan and get filters 
* Optimizations for real time queries 
* A Thrift gateway and a REST-ful Web service that supports XML, Protobuf, and binary data encoding options 
* Extensible jruby-based (JIRB) shell 
* Support for exporting metrics via the Hadoop metrics subsystem to files or Ganglia; or via JMX 

#####Data Model:

In short, applications store data into an HBase table. Tables are made of rows and columns. All columns in HBase belong to a particular column family. Table cells -- the intersection of row and column coordinates -- are versioned. A cell’s content is an uninterpreted array of bytes.

Table row keys are also byte arrays so almost anything can serve as a row key from strings to binary representations of longs or even serialized data structures. Rows in HBase tables are sorted by row key. The sort is byte-ordered. All table accesses are via the table row key -- its primary key.
Cassandra and HBase are both highly scalable column-oriented database. Cassandra wins for its simple configuration and flexibility to switch between CAP attributes, no single point of failure is also more scalable.

####Voldemort:

Voldemort is a distributed key-value storage system

#####Feature:

* Voldemort combines in memory caching with the storage system so that a separate caching tier is not required (instead the storage system itself is just fast)
* Unlike MySQL replication, both reads and writes scale horizontally
* Data portioning is transparent, and allows for cluster expansion without rebalancing all data
* Data replication and placement is decided by a simple API to be able to accommodate a wide range of application specific strategies
* The storage layer is completely mockable so development and unit testing can be done against a throw-away in-memory storage system without needing a real cluster (or even a real storage system) for simple testing 

#####Data Model:

The equivalent here is a "store", we don't use the word table since the data is not necessarily tabular (a value can contain lists and mappings which are not considered in a strict relational mapping). Each key is unique to a store, and each key can have at most one value.

Note that although we don't support one-many relations, we do support lists as values that accomplishes the same thing--so it is possible to store a reasonable number of values associated with a single key. This corresponds to a java.util.Map where the value is a java.util.List.

Serialization in Voldemort is pluggable so you can use one of the baked in serializers or easily write your own. At the lowest level the data format for Voldemort is just arrays of bytes for both keys and values. Higher-level data formats are a configuration option that are set for each Store--any format can be supported by implementing a Serializer class that handles the translation between bytes and objects. Doing this ensures the client serializes the bytes correctly.

**Operations**:

To enable high performance and availability we allow only very simple key-value data access. Both keys and values can be complex compound objects including lists or maps, but none-the-less the only supported queries are effectively the following: 
{% codeblock %}
value = store.get(key)
store.put(key, value)
store.delete(key)
{% endcodeblock %}

Voldemort supports hashtable semantics, so a single value can be modified at a time and retrieval is by primary key. This makes distribution across machines particularly easy since everything can be split by the primary key.

####Kyoto Cabinet:

Kyoto Cabinet is a library of routines for managing a database. The database is a simple data file containing records, each is a pair of a key and a value. Every key and value is serial bytes with variable length. Both binary data and character string can be used as a key and a value. Each key must be unique within a database. There is neither concept of data tables nor data types. Records are organized in hash table or B+ tree.

#####Feature

The database is a simple data file containing records, each is a pair of a key and a value. Every key and value is serial bytes with variable length. Both binary data and character string can be used as a key and a value. Each key must be unique within a database. There is neither concept of data tables nor data types. Records are organized in hash table or B+ tree.

#####Operations:

The following access methods are provided to the database: storing a record with a key and a value, deleting a record by a key, retrieving a record by a key. Moreover, traversal access to every key are provided.

####Redis

Redis is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.
You can run atomic operations on these types, like appending to a string; incrementing the value in a hash; pushing to a list; computing set intersection, union and difference; or getting the member with highest ranking in a sorted set.

#####Feature

Redis is an open source, advanced key-value store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets and sorted sets.
You can run atomic operations on these types, like appending to a string; incrementing the value in a hash; pushing to a list; computing set intersection, union and difference; or getting the member with highest ranking in a sorted set.

#####Data Model:

As you already probably know Redis is not a plain key-value store, actually it is a data structures server, supporting different kind of values. That is, you can't just set strings as values of keys. All the following data types are supported as values:

* Binary-safe strings.
* Lists of binary-safe strings.
* Sets of binary-safe strings, that are collection of unique unsorted elements. You can think at this as a Ruby hash where all the keys are set to the 'true' value.
* Sorted sets, similar to Sets but where every element is associated to a floating number score. The elements are taken sorted by score. You can think of this as Ruby hashes where the key is the element and the value is the score, but where elements are always taken in order without requiring a sorting operation.

####Limitations:

Redis meets the needs for Registrar very well since its support on list of value and its high performance on IO operation. But it’s written in C, it will require some work to make it start when delivering to users in the MOHO framework.</br>

  In the three Key-value type databases, Voldemort is a more distributed Key-value NoSQL database implementation than the other two, and it has a very simple API to manipulate the database.
  
##Conclusion:
MongoDB is document oriented and very good at large amount of data searching. It is a good option for document store.</br>
Redis will be good choice for K-V mapping don’t scale too large since it has very good performance on concurrent IO and support list type value. Voldemort will be good choice in a large-scale deployment for its high available design and simple API.</br>
But Redis and Voldemort are Key-value databases that do not have enough function for the future complicated data migrated from RDBMS requirement.</br>
So finally we choose Cassadra. It is a mixed type database, which has Dynamo’s distributed Key-value feature, and also have the rich function from the column feature like Google’s Bigtable. It is suitable for the situation both usecases in the our product. And it’s very mature on production, a open-source Java project and for its HA and HS in large scale. 

##Reference

1. SQL vs. NoSQL http://www.linuxjournal.com/article/10770?page=0,0</br>
2. NoSQL数据库探讨之一 － 为什么要用非关系数据库？ http://robbin.iteye.com/blog/524977</br>
3. NoSQL databases http://nosql-database.org/</br>
4. Visual Guide to NoSQL Systems http://blog.nahurst.com/visual-guide-to-nosql-systems</br>
5. Eventually consistence http://www.allthingsdistributed.com/2008/12/eventually_consistent.html</br>
6. Brewer's CAP Theorem http://www.julianbrowne.com/article/viewer/brewers-cap-theorem</br>
7. Cassandra http://cassandra.apache.org/</br>
8. Cassandra data model http://wiki.apache.org/cassandra/DataModel</br>
9. Cassandra Architecture http://wiki.apache.org/cassandra/ArchitectureOverview</br>
10. Cassandra Embedding http://wiki.apache.org/cassandra/Embedding</br>
11. Running Cassandra as an embedded service http://prettyprint.me/2010/02/14/running-cassandra-as-an-embedded-service/</br>
12. HBase http://hbase.apache.org/</br>
13. HBase vs. Cassandra NoSQL battle http://www.roadtofailure.com/2009/10/29/hbase-vs-cassandra-nosql-battle/</br>
14. HBase vs. Cassandra : why we moved http://ria101.wordpress.com/2010/02/24/hbase-vs-cassandra-why-we-moved/</br>
15. Voldemort http://project-voldemort.com/</br>
16. Voldemort design http://project-voldemort.com/design.php</br>
17. Dynamo: Amazon's Highly Available Key-Value Store http://s3.amazonaws.com/AllThingsDistributed/sosp/amazon-dynamo-sosp2007.pdf</br>
18. MongoDB http://www.mongodb.org/</br>
19. CouchDB http://couchdb.apache.org/</br>
20. Redis http://redis.io/</br>
21. Tokyo Cabinet http://fallabs.com/tokyocabinet/</br>
22. Fundamental Specifications of Kyoto Cabinet http://fallabs.com/kyotocabinet/spex.html#features</br>
23. MemcacheDB http://memcachedb.org/</br>
24. Database management system choices – beyond relational http://www.dbms2.com/2008/02/15/non-relational-database-management/</br>
25. NoSQL and RDBMS – choose your weapon http://www.servicestack.net/mythz_blog/?p=129</br>
26. Streamy Development blog http://devblog.streamy.com/</br>
27. Google Code University http://code.google.com/edu/parallel/mapreduce-tutorial.html</br>
28. Dynamo http://en.wikipedia.org/wiki/Dynamo_%28storage_system%29</br>
29. BigTable http://en.wikipedia.org/wiki/Bigtable</br>
30. Hadoop http://en.wikipedia.org/wiki/Hadoop</br>
31. Thrift http://thrift.apache.org/</br>

##Appendix NoSQL Database setup and programming guide

###Cassadra:

####Setup

Cassandra is meant to run on a cluster of nodes, but will run equally well on a single machine. Download a binary distribution. The distribution's sample configuration conf/cassandra.yaml contains reasonable defaults for single node operation, but you will need to make sure that the paths exist for data_file_directories, commitlog_directory, and saved_caches_directory. Additionally, take a minute now to look over the logging configuration in conf/log4j.properties and make sure that directories exist for the configured log file(s) as well. 

Some people running OS X have trouble getting Java 6 to work. If you've kept up with Apple's updates, Java 6 should already be installed (it comes in Mac OS X 10.5 Update 1). Unfortunately, Apple does not default to using it. What you have to do is change your JAVA_HOME environment setting to */System/Library/Frameworks/JavaVM.framework/Versions/1.6/Home* and add */System/Library/Frameworks/JavaVM.framework/Versions/1.6/Home/bin* to the beginning of your PATH. 

And now for the moment of truth, start up Cassandra by invoking bin/cassandra -f from the command line3. The service should start in the foreground and log gratuitously to standard-out. 

####Programming

The recommended way to communicate with Cassandra in your application is to use a higher-level client. These provide programming language specific APIs for talking to Cassandra in a variety of languages. The details will vary depending on programming language and client, but in general using a higher-level client will mean that you have to write less code and get several features for free that you would otherwise have to write yourself.  Using Hector for example:
{% codeblock lang:java%}
package com.riptano.cassandra.hector.example;


import me.prettyprint.cassandra.serializers.StringSerializer;
import me.prettyprint.hector.api.Cluster;
import me.prettyprint.hector.api.Keyspace;
import me.prettyprint.hector.api.beans.HColumn;
import me.prettyprint.hector.api.exceptions.HectorException;
import me.prettyprint.hector.api.factory.HFactory;
import me.prettyprint.hector.api.mutation.Mutator;
import me.prettyprint.hector.api.query.ColumnQuery;
import me.prettyprint.hector.api.query.QueryResult;

/**
 * Inserts the value "John" under the Column "first" for the 
 * key "jsmith" in the Standard1 ColumnFamily 
 * 
 * To run this example from maven: 
 * mvn -e exec:java -Dexec.mainClass="com.riptano.cassandra.hector.example.InsertSingleColumn"
 * 
 * @author zznate
 *
 */
public class InsertSingleColumn {
    
    private static StringSerializer stringSerializer = StringSerializer.get();
    
    public static void main(String[] args) throws Exception {
        Cluster cluster = HFactory.getOrCreateCluster("TestCluster", "localhost:9160");

        Keyspace keyspaceOperator = HFactory.createKeyspace("Keyspace1", cluster);
        try {
            Mutator<String> mutator = HFactory.createMutator(keyspaceOperator, StringSerializer.get());
            mutator.insert("jsmith", "Standard1", HFactory.createStringColumn("first", "John"));
            
            ColumnQuery<String, String, String> columnQuery = HFactory.createStringColumnQuery(keyspaceOperator);
            columnQuery.setColumnFamily("Standard1").setKey("jsmith").setName("first");
            QueryResult<HColumn<String, String>> result = columnQuery.execute();
            
            System.out.println("Read HColumn from cassandra: " + result.get());            
            System.out.println("Verify on CLI with:  get Keyspace1.Standard1['jsmith'] ");
            
        } catch (HectorException e) {
            e.printStackTrace();
        }
        cluster.getConnectionManager().shutdown();
    }
    
}
{% endcodeblock %}

###HBase


####Setup:

Download and unpack the latest stable release. Click on the folder named stable and then download the file that ends in .tar.gz to your local filesystem; e.g. hbase-0.91.0-SNAPSHOT.tar.gz. Decompress and untar your download and then change into the unpacked directory.
{% codeblock lang:bash%}
 $ tar xfz hbase-0.91.0-SNAPSHOT.tar.gz
 $ cd hbase-0.91.0-SNAPSHOT
{% endcodeblock %}
At this point, you are ready to start HBase. But before starting it, you might want to edit conf/hbase-site.xml and set the directory you want HBase to write to, hbase.rootdir. 
{% codeblock lang:xml%}
 <?xml version="1.0"?>
 <?xml-stylesheet type="text/xsl" href="configuration.xsl"?> 
<configuration>
   <property>
     <name>hbase.rootdir</name>
     <value>file:///DIRECTORY/hbase</value>
   </property>
 </configuration>  
{% endcodeblock %}

Replace DIRECTORY in the above with a path to a directory where you want HBase to store its data. By default, hbase.rootdir is set to /tmp/hbase-${user.name} which means you'll lose all your data whenever your server reboots (Most operating systems clear /tmp on restart).
$ ./bin/start-hbase.sh starting Master, logging to logs/hbase-user-master-example.org.out

You should now have a running standalone HBase instance. In standalone mode, HBase runs all daemons in the the one JVM; i.e. both the HBase and ZooKeeper daemons. HBase logs can be found in the logs subdirectory. Check them out especially if HBase had trouble starting.

####Programming

{% codeblock lang:java%}
import java.io.IOException;
import org.apache.hadoop.hbase.client.HTable;
import org.apache.hadoop.hbase.client.Scanner;
import org.apache.hadoop.hbase.io.BatchUpdate;
import org.apache.hadoop.hbase.io.Cell;
import org.apache.hadoop.hbase.io.RowResult;
public class MyClient {
public static void main(String args[]) throws IOException {
    // You need a configuration object to tell the client where to connect.
    // But don't worry, the defaults are pulled from the local config file.
    HBaseConfiguration config = new HBaseConfiguration();
    // This instantiates an HTable object that connects you to the "myTable"
    // table. 
    HTable table = new HTable(config, "myTable");
    // To do any sort of update on a row, you use an instance of the BatchUpdate
    // class. A BatchUpdate takes a row and optionally a timestamp which your
    // updates will affect. 
    BatchUpdate batchUpdate = new BatchUpdate("myRow");
    // The BatchUpdate#put method takes a Text that describes what cell you want
    // to put a value into, and a byte array that is the value you want to 
    // store. Note that if you want to store strings, you have to getBytes() 
    // from the string for HBase to understand how to store it. (The same goes
    // for primitives like ints and longs and user-defined classes - you must 
    // find a way to reduce it to bytes.)
    batchUpdate.put("myColumnFamily:columnQualifier1", "columnQualifier1 value!".getBytes());
    // Deletes are batch operations in HBase as well. 
    batchUpdate.delete("myColumnFamily:cellIWantDeleted");
    // Once you've done all the puts you want, you need to commit the results.
    // The HTable#commit method takes the BatchUpdate instance you've been 
    // building and pushes the batch of changes you made into HBase.
    table.commit(batchUpdate);
    // Now, to retrieve the data we just wrote. The values that come back are
    // Cell instances. A Cell is a combination of the value as a byte array and
    // the timestamp the value was stored with. If you happen to know that the 
    // value contained is a string and want an actual string, then you must 
    // convert it yourself.
    Cell cell = table.get("myRow", "myColumnFamily:columnQualifier1");
    String valueStr = new String(cell.getValue());
    
    // Sometimes, you won't know the row you're looking for. In this case, you
    // use a Scanner. This will give you cursor-like interface to the contents
    // of the table.
    Scanner scanner = 
      // we want to get back only "myColumnFamily:columnQualifier1" when we iterate
      table.getScanner(new String[]{"myColumnFamily:columnQualifier1"});
    
    
    // Scanners in HBase 0.2 return RowResult instances. A RowResult is like the
    // row key and the columns all wrapped up in a single interface. 
    // RowResult#getRow gives you the row key. RowResult also implements 
    // Map, so you can get to your column results easily. 
    
    // Now, for the actual iteration. One way is to use a while loop like so:
    RowResult rowResult = scanner.next();
    
    while(rowResult != null) {
      // print out the row we found and the columns we were looking for
      System.out.println("Found row: " + new String(rowResult.getRow()) + " with value: " +
       rowResult.get("myColumnFamily:columnQualifier1".getBytes()));
      
      rowResult = scanner.next();
    }
    
    // The other approach is to use a foreach loop. Scanners are iterable!
    for (RowResult result : scanner) {
      // print out the row we found and the columns we were looking for
      System.out.println("Found row: " + new String(result.getRow()) + " with value: " +
       result.get("myColumnFamily:columnQualifier1".getBytes()));
    }
    
    // Make sure you close your scanners when you are done!
    scanner.close();
}
}
{% endcodeblock %}

###Voldemort:

####Setup

* Step 1: Download the code
Download either a recent stable release or, for those who like to live more dangerously, the up-to-the-minute build from the build server. 

* Step 2: Start single node cluster
{% codeblock lang:bash%}
  > bin/voldemort-server.sh config/single_node_cluster > /tmp/voldemort.log & 
{% endcodeblock %}
* Step 3: Start commandline test client and do some operations

{% codeblock lang:bash%}
  > bin/voldemort-shell.sh test tcp://localhost:6666 
  Established connection to test via tcp://localhost:6666
  > put "hello" "world" 
  > get "hello"   version(0:1): "world"
  > delete "hello"
  > get "hello"   null
  > exit  k k thx bye. 
{% endcodeblock %}

####Programming

**Client**
Here is an example showing how to connect to a store as a client to do reads and writes from Java:
{% codeblock lang:java%}
String bootstrapUrl = "tcp://localhost:6666";
StoreClientFactory factory = new SocketStoreClientFactory(new ClientConfig().setBootstrapUrls(bootstrapUrl));
// create a client that executes operations on a single store
StoreClient client = factory.getStoreClient("my_store_name"); 
{% endcodeblock %}
After initializing the store client for every store once we can reuse it to run our queries as follows:
{% codeblock lang:java%}
// do some random pointless operations
  Versioned value = client.get("some_key");
  value.setObject("some_value");
  client.put("some_key", value); 
 {% endcodeblock %}
 
Note that StoreClient is just an interface, so for the purpose of unit testing we can completely mock the storage layer. This is something that is essentially impossible to do with a normal relational db since sql is the interface and it is vendor specific.

**Server**

There are three methods for using the server:
1. Start from the command line
You must first build the jar file using ant, as described above, then do the following: 
{% codeblock lang:bash%}
$ VOLDEMORT_HOME='/path/to/voldemort' 
$ cd $VOLDEMORT_HOME 
$ ./bin/voldemort-server.sh 
 {% endcodeblock %}

Alternately we can give VOLDEMORT_HOME on the command line and avoid having to set an environment variable
{% codeblock lang:bash%}
$ ./bin/voldemort-server.sh /path/to/voldemort
 {% endcodeblock %}
2. Embedded Server
You can instantiate the server directly in your code.
{% codeblock lang:java%}
VoldemortConfig config = VoldemortConfig.loadFromEnvironmentVariable(); 
VoldemortServer server = new VoldemortServer(config); 
server.start(); 
 {% endcodeblock %}
3. Deploy as a war
To do this build the war file using the 
{% codeblock lang:bash%}
ant war
 {% endcodeblock %}
target and deploy via whatever mechanism your servlet container supports.

####Kyoto Cabinet:

#####Setup

Install the latest version of Kyoto Cabinet beforehand and get the package of the Java binding of Kyoto Cabinet. JDK 6 or later is also required.
Enter the directory of the extracted package then perform installation.
{% codeblock lang:bash%}
./configure
 make
 make check
 su make install 
 {% endcodeblock %}
When a series of work finishes, the JAR file `kyotocabinet.jar' and the shared object files `libjkyotocabinet.so' and so on are installed under `/usr/local/lib'.
Let the class search path include `/usr/local/lib/kyotocabinet.jar' and let the library search path include `/usr/local/lib'.
{% codeblock lang:bash%}
CLASSPATH="$CLASSPATH:/usr/local/lib/kyotocabinet.jar" LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/lib" export CLASSPATH LD_LIBRARY_PATH 
{% endcodeblock %}
The above settings can be specified by options of the runtime command.
{% codeblock lang:bash%}
java -cp .:kyotocabinet.jar -Djava.library.path=.:/usr/local/lib FooBarBaz ... 
{% endcodeblock %}
All symbols of Kyoto Cabinet are defined in the package `kyotocabinet'. You can access them without any prefix by importing the package.
{% codeblock lang:java%}
import kyotocabinet.*; 
{% endcodeblock %}

#####Programming

The following code is a typical example to use a database. 
{% codeblock lang:java%}
import kyotocabinet.*;  
public class KCDBEX1 {   
public static void main(String[] args) {
      // create the object
     DB db = new DB();
      // open the database
     if (!db.open("casket.kch", DB.OWRITER | DB.OCREATE)){
       System.err.println("open error: " + db.error());
     }
      // store records
     if (!db.set("foo", "hop") ||
         !db.set("bar", "step") ||
         !db.set("baz", "jump")){
       System.err.println("set error: " + db.error());
     }
      // retrieve records
     String value = db.get("foo");
     if (value != null){
       System.out.println(value);
     } else {
       System.err.println("set error: " + db.error());
     }
      // traverse records
     Cursor cur = db.cursor();
     cur.jump();
     String[] rec;
     while ((rec = cur.get_str(true)) != null) {
       System.out.println(rec[0] + ":" + rec[1]);
     }
     cur.disable();
      // close the database
     if(!db.close()){
       System.err.println("close error: " + db.error());
     }
    }
 } 
{% endcodeblock %}

####Redis

#####Setup


Download, extract and compile Redis with:
{% codeblock lang:bash%}
 $ wget http://redis.googlecode.com/files/redis-2.2.12.tar.gz 
 $ tar xzf redis-2.2.12.tar.gz 
 $ cd redis-2.2.12
 $ make 
{% endcodeblock %}

#####Programming


Many Java clients, using Jedis for example:
{% codeblock lang:java%}
 Jedis jedis = new Jedis("localhost");
 jedis.set("foo", "bar");
 String value = jedis.get("foo");
{% endcodeblock %}

###MongoDB
####Setup
64-bit binaries
{% codeblock lang:bash%}
$ curl http://downloads.mongodb.org/osx/mongodb-osx-x86_64-x.y.z.tgz > mongo.tgz $ tar xzf mongo.tgz 
{% endcodeblock %}
Replace x.y.z with the current stable version.
Create a data directory
By default MongoDB will store data in /data/db, but it won't automatically create that directory. To create it, do:
{% codeblock lang:bash%}
$ mkdir -p /data/db 
{% endcodeblock %}
You can also tell MongoDB to use a different data directory, with the --dbpath option.
Run and connect to the server
First, start the MongoDB server in one terminal:
{% codeblock lang:bash%}
$ ./mongodb-xxxxxxx/bin/mongod
{% endcodeblock %}
In a separate terminal, start the shell, which will connect to localhost by default:
{% codeblock lang:bash%}
$ ./mongodb-xxxxxxx/bin/mongo > db.foo.save( { a : 1 } ) > db.foo.find() 
{% endcodeblock %}
Congratulations, you've just saved and retrieved your first document with MongoDB!

####Programming
{% codeblock lang:java%}
 import com.mongodb.Mongo;
 import com.mongodb.DB;
 import com.mongodb.DBCollection;
 import com.mongodb.BasicDBObject;
 import com.mongodb.DBObject;
 import com.mongodb.DBCursor;
  
 Mongo m = new Mongo();
 // or Mongo m = new Mongo( "localhost" );
 // or Mongo m = new Mongo( "localhost" , 27017 );
 DB db = m.getDB( "mydb" );
 BasicDBObject doc = new BasicDBObject();
 doc.put("name", "MongoDB");
 doc.put("type", "database");
 doc.put("count", 1);
 BasicDBObject info = new BasicDBObject();
 info.put("x", 203);
 info.put("y", 102);
 doc.put("info", info);
 coll.insert(doc);
{% endcodeblock %}

