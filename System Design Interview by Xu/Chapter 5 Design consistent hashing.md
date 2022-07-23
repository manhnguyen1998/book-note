To archive horizontal scaling, it is important to distribute requests/data efficiently and evenly across servers. Consistent hashing is a commonly used technique to archive this goal. 

# The rehashing problem
- If you have n cache servers, a common way to balance the load is to use the following hash method
	- serverIndex = hash(key) % N, N is the size of the server pool
	- ![[Pasted image 20220723001150.png]]

To fetch the server where a key is stored, we perform the modular operation f(key) % 4

![[Pasted image 20220723001327.png]]

However, when new servers added, or existing servers are removed,
	if server 1 offline, server pool becomes 3

![[Pasted image 20220723003815.png]]

-> most keys are redistributed -> most cache clients will connect to the wrong servers to fetch data


## Consistent hashing
- Consistent hashing is a special kind of hashing such that when a hash table is re-sized and consistent hashing is used, only k/n keys need to be remapped on average, 
	- k is the number of keys
	- n is the number of slots
- In most traditional hash tables, a change in the number of array slots cause nearly all keys to be remapped

## Hash space and hash ring
Assume SHA-1 is used as the hash function f, output range is x0, x1, x2.... xn

## Hash servers
Using the same hash fucntion f, map servers based on server IP or name onto the ring

![[Pasted image 20220723010058.png]]

## Hash keys
cache keys are hashed onto the hash ring
![[Pasted image 20220723010537.png]]

## Server lookup
To determine which server a key is stored on, go clockwise(chiều kim đồng hồ) from the key position on the ring until a server is found
- Going clockwise, key0 is stored on server 0, key1 is stored on server 1.....
- ![[Pasted image 20220723010828.png]]

## Add a server
Using logic described above, adding a new server will only require redistribution of a fraction of keys
- After a new server 4 is added, only key0 need to be redistributed.
	- k1, k2, k3 remain on the same server
	- key0 will be stored on server 4
- ![[Pasted image 20220723011219.png]]

## Remove a server
When a server is removed, only a small fraction of keys require distribution with consistent hashing

![[Pasted image 20220723011356.png]]


# Two issues in the basic approach
Consistent hashing algorithm has basic steps
- Map servers and keys on to the ring using a uniformly distributed hash function
- To find out which server a key is mapped to, go clockwise from the key position until the first server on the ring is found

It has 2 problems
- It is impossible to keep the same size of partions on the rings for all servers considering a server can be added or removed
	- A partion is the hash space between adjacent servers
	- ![[Pasted image 20220723011756.png]]
	- s2's partion is twice as large as s0 and s3
- It is posibble to have a non-uniform key distribution on the ring. If servers are mapped to positions listed, most of the keys are stored on server2. server 1 and s3 have no data
-> Virtual nodes or replica is used to solve these problem

# Virtual nodes
Virtual node refers to the real node, and each server is represented by multiple virtual nodes on the ring
![[Pasted image 20220723224507.png]]

To find which server a key is stored on, we go clockwise from the key's location and find the first virtual node encountered on the ring.
- TO find out which server k0 is stored on, go clockwise from k0's location and find virtual node s1_1
-  ![[Pasted image 20220723224817.png]]
As the number of virtual node increase, the distribution of keys become more balanced. Because the standard deviation gets smaller with more virtual nodes, leading to balanced data distribution

## Find affected keys
When a server is added or removed, a fraction of data needs to be redistributed. How to find the affected range to redistributed the keys?
![[Pasted image 20220723225146.png]]

![[Pasted image 20220723225232.png]]