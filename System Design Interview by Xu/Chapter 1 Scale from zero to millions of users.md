# How to build a system that can serve milions of user ?
- Before it, need to start from step 1: system for 1 user
- ![[Pasted image 20220709004724.png]]
		- Request flow for simple system 
- 
1. Users access websites through domain names, such as api.mysite.com. Usually, the Domain Name System (DNS) is a paid service provided by 3rd parties and not hosted by our servers.

2. Internet Protocol (IP) address is returned to the browser or mobile app. In the example, IP address 15.125.23.214 is returned.

3. Once the IP address is obtained, Hypertext Transfer Protocol (HTTP) [1] requests are sent directly to your web server.

4. The web server returns HTML pages or JSON response for rendering.

## Database
- Just 1 server is not enough -> need multiple servers, 1 for web/mobile, 1 for database. 
- Separating allows them to be scaled independently
-![[Pasted image 20220709005745.png]]
-> MVC model is like this

## Which database to use
There are 2 types of database:
- Traditional relational dabase
	- RDBMS: SQL -> Mysql, Oracle, PostgreSQL
	- Store data in tables and rows
	- Can perform join operations using Sql across different tables
- Non-relational database
	- Nosql database: CouchDB, Neo4j, Cassandra, HBase, Amazon DynamoDB
	- Grouped into 4 categories: 
		- key-value stores
		- graph stores
		- column stores
		- document stores
	- Join operations are generally not supported
### when to use non-relational db
Non-relational db might be the right choice if:
- your app requires super-low latency(độ trễ)
- your data unstructured, do not have any relational data
- you only need to serialize and deserialize data (json, xml, yaml)
- you need to store a massive amount of data
## Vertical scaling vs horizontal scaling
- Vertical scaling:
	- scale up: process of adding more power (cpu, ram) to your servers.
	- When traffic is low, vertical scaling is a great option, the simplicity is it's main advantage. But have serious limitations
		- Has a hard limit. Impossible to add unlimited cpu and memory to a single server
		- does not have failover and redundancy. If one server goes down, the app will goes down with it completely
- Horizontal scaling:
	- scale out: allow to scale by adding more servers into your pool of resources
	- More desirable for large scale applications due to the limitations of vertical scaling
-> when the single server have to face with a tons of request and reach its load limit -> it will go down 
-> that's the reason we need a load balancer, thats the best technique

# Load balancer
- ![[Pasted image 20220709011756.png]]
- User will connect to the public IP of the load balancer directly -> web servers are unreachable directly by clients. 
- For better security, private IPs are used for communication between servers. Load balancer connect with servers through private IPs
-> Web tier look good, but data tier is only one database, not support failover and redundancy
-> Database replication (replica) is a common technique

# Database replication
![[Pasted image 20220709012548.png]]
- Data replica is a type of master/slave relationship
	- Master db: 
		- only support write operations: insert, delete, update must be sent to master db
	- Slave db:
		- get copies of data from master db
		- only support read operations
	- Most applications require a much higher of ratio read and write: slave > master
### advantage of replica
- better performance: 
	- all writes and updates happen in master nodes, read options are distributed across slave nodes
	- -> improves performance because it allows more queries to be processed in parallel
- reliability: if one database is destroyed, data will not be loss because of the replicated multiple locations
- high availability: replicating data different locations, your website remains in operation even if a db is offline

Question
1. if only one slave is available and it's offline -> the read operations will be directed to master temporarily. new slave will replace the old one, read options can be redirected to other healthy slave db
2. if the master off, a slave will be promoted to be the new master, all the db operations will be temporarily executed on the new master, new slave will replace the old one for data replica

# Cache
- A temporary storage area that store the result of expensive responses or frequently
## Cache tier
![[Pasted image 20220711002341.png]]
- A temporary data store layer, much faster than the db. 
-> Read-through cache strategy
Usually with cache server
### Considerations for using cache
- Decide when to use cache
	- using cache if the data is read frequently but modified infrequently. data in cache is temporary, it can be delete if the cache server is out -> need to be save
- Expiration policy
	- mem cache should have expiration, not too long or too short
- Mitigating failures: a single cache server represents a potential single point of failure (SPOF): if a part of a system fail, it cause the whole system fail
	- ![[Pasted image 20220711220046.png]]
- Eviction policy: once the cache is full, any request to add items to the cache might cause existing item to be remove -> cache eviction.  Least-recently-used(LRU) is the most popular cache eviction policy. other policy is LFU, FIFO...
# Content delivery network (CDN)
- CDN is a network of geographically dispersed servers used to deliver static content. CDN server cache static content like video, images, CSS, Javascript files...
- ![[Pasted image 20220711230451.png]]
- CDN to cache static content
	- when a user visits a website, a cdn server closest to user will deliver static content.
### Workflow
![[Pasted image 20220711230740.png]]


### Consideration of using CDN
-Cost: CDN run by third-party providers, if dont have benefits, should consider moving out of the CDN
- Setting an appropiate cache expiry: for time-sensitive content, setting a cache expiry time is important
- CDN fallback: should consider how your website/app copes with CDN failure. If there is a temporary CDN outage, client should be able to detect the problem and request resources from the origin
- Invalidate files: you can remove a file from the cdn before it expires by performing one of the following operations
	- invalidate the cdn object using APIs provided by CDN vendors
	- use object versioning
	-
- ![[Pasted image 20220712011638.png]]
1. Static assets(js, css, images...) are no longer served by web servers, they are fetched from the cdn for better performance
2. The database load is lightened by caching data
# Stateless web tier
![[Pasted image 20220712012009.png]]
#### Stateful and stateless server
- User A'session are store in server 1. To authenticate User A, Http requests must be routed to Server1. If a request is sent to other servers like server2, it would be failed -> That is stateful server
- ![[Pasted image 20220712012246.png]]
- In stateless server, http requests from users can be sent to any web servers, which fetch state data from shared data store. State data is stored in a shared data store and kept out of web servers. A stateless system is simpler, more robust, and scalable

-> The updated design with additional of stateless server
![[Pasted image 20220712012442.png]]
- The shared data store could be a relational db, Memcached/Redis, NoSql, etc...
	- NoSql is chosen as it is easy to scale. Autoscaling means adding or removing web servers automatically based on the traffic load
# Data centers
![[Pasted image 20220714000802.png]]
If Dc2 is off, then 100% is redirected to Dc1
![[Pasted image 20220714001126.png]]

# Message queue
- A message queue is a durable component, stored in memory, support asynchronous communication. 
	- Serve as a buffer and distributes asynchronous requests. 
## Architecture
- Input services, called producers/publisher, create message, and publish them to a message queue
- Other service or servers, called consumer/subcribes, connect to the queue, and perform actions defined by the messages
- ![[Pasted image 20220714002044.png]]
- Decoupling makes the message queue a prefered architecture for building a scalable and reliable application.
	- -> with the queue, producer can post a message to the queue when the consumer is unavailable to process it.
- The producer and consumer can be scaled independently. When the size of queue become larbe, more worker are added to reduce the processing time. if the queue is emty, number of workers can be reduced.
# Logging, metrics, automation
- A website that serves a large business, those tool are necessary
## Logging
- help to identify errors and problems in the system
## Metrics
- collecting different type of metrics help us to gain business insight and understand the heath status of the system. Some useful metrics
	- Host level metrics: CPU, memory, dis I/O....
	- Aggregated level metrics: performance of the entire database tier, cache tier,...
	- Key business metrics: daily active users, retention, revenue....

## Automation
- when a system get big and complex, we need to build or leverage automation tools to improve productivity
- -> Automating your build, test, deploy process, .... -> improve developer productivity significantly
## Adding message queues and different tool
![[Pasted image 20220714003849.png]]

# Database scaling
## Vertical scaling
- Scaling up: adding more cpu, ram, disk to existing machine

## Horizontal scaling
- Is known as sharding: adding more server
- ![[Pasted image 20220714004237.png]]
- Sharding separate large dbs into smaller, more easily managed parts called shards. Each shards shares the same schema, actual data on each shard is unique to the shard
- ![[Pasted image 20220714004423.png]]


### Sharding
- User data is allocated to a database server based on user IDs. anytime you access data, a hash function is used to find the corresponding shard. 
- For example, user_id%4 is used as the hash function, if the result equal to 0, shard 0 is used to store and fetch data. 
- The most important factor to consider when implementing a sharding strategy is the choice of the sharding key.
- The most important is choose a key that can evenly distributed data
- Sharding is great technique to scale the db but it is far from a perfect solution -> It introduce complexities and new challenges to the system
#### Resharding data
- is needed when
	- a single shard could no longer hold more data due to rapid growth
	- certain shard might experience shard exhaustion faster than other due to uneven data distribution. when shard exhaustion happen, it requires updating the sharding function and moving data around
		- Consistent hashing is common used to solve this problem
#### Celebrity problem
- called as hotspot key problem
- excessive access to a specific shard could cause server overload
- -> need to allocate a shard for each celebrity. might even require further partion
#### Join and de-normalization
- Once a database has been sharded across multiple servers, it is hard to perform join operation across database shard. 
- Some non-relation data is moved to no-sql
- ![[Pasted image 20220714011248.png]]


### how alex xu scale a system
![[Pasted image 20220714011532.png]]