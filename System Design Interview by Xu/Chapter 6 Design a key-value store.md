A key-value store, also referred to as a key-value database, is a non-relational database. Each unique identifier is stored as a key with its associated value. This data pairing is known as a key-value pair
- In a key-value pair, the key must be unique, and the value associated with the key can be accessed through the key. Key can be plain text or hashed values.
	- Plain text key: last_logged_in_at
	- Hashed key: 253DDC3ASF
- THe value in a key-value pair can be strings, lists, objects,... The value is usually treated as an opaque object in key-value stores, such as Amazon dynamo, Memcached, Redis...
![[Pasted image 20220725230356.png]]

This chapter, you are asked to design a key-value store that support the following operations
- put(key, value) // insert "value" associated with "key"
- get(key) // get "value" associated with "key"
# Understand the problem and establish design scope
This chapter, we design a key-value store that comprises of the following characteristic
- The size of a key-value pair is small: less than 10kb
- Ability to store big data
- High availability: The system responds quickly, even during failures
- High scalability: The system can be scaled to support large dataset
- Automatic scaling: The addition/deletion of servers should be automatic based on traffic
- Turnable consistency
- Low latency

# Single server key-value store
Even though memory access is fast, fitting everything in memory may be impossible due to the space constraint. Two optimization can be done to fit more data in a single server
- Data compression
- Store only frequently used data in memory and the rest on disk

But these optimization maybe not enough -> distributed key-value store is required to support big data

# Distributed key-value store
- A distributed key-value store is also called a distributed hash-table, which distributes key-value pair across many servers. When designing a distributed system, it is important to understand CAP( Consistency, Availability, Partion Tolerance) theorem

## CAP theorem
- CAP theorem states it is impossible for a distributed system to simultaneously provide more than two of these three guarantees: consistency, availability, and partion tolerance
- Consistency: consistency means all clients see the same data at the same time no matter which node they connect to
- Availability: availability mean any client which requests data gets a response even if some of the nodes are down
- Partion Tolerance: a partion indicates a communication break between two nodes. Partion tolerance means the system continues to operate despite network partition

-> 1 of These 3 properties must be sacrifice to support 2 of the 3 properties
![[Pasted image 20220725232716.png]]


CA (consistency and availability) systems: a CA key-value store support consistency and availability while sacrificing partion tolerance. Since network failure is unavoiable, a distributed system must tolerate network partion. Thus, a CA system cannot exist in real-world application