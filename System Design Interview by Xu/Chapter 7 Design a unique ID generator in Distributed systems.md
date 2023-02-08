In this chapter, you are asked to design a unique ID generator in distributed systems. Your first thought might be to use a primary key with the auto_increment attribute in a traditional databasd. However, auto_increment does not work in a distributed environment because a single database server is not large enough and generating unique ID across multiple databases with minimal delay is challenging

![[Pasted image 20220908221339.png]]


# Step 1: Understand the problem and establish design scope
- Characteristic of unique ID
	- unique, sortable
	- icrement by time, not necessary by 1. IDs created in the evening are larger than those created in the morning on the same day
	- contain numerical values
	- lenght: 64bit
	- scale of system: should be able to generate 10,000 IDs per second

# Step 2: Propose high-level design and get buy-in
- There are multiple options:
	- Multi-master replication
	- Universally unique identifier (UUID)
	- Ticket server
	- Twitter snowflake approach

## Multi-master replication
![[Pasted image 20220908223024.png]]

- This approach use the database auto_increment feature
- Instead of increasing the next ID by 1, it increase it by k, where k is the number of database servers in use

### Drawback 
This strategy has some major drawback
- Hard to scale with multiple data centers
- IDs do not go up with time across multilple


# UUID
- UUID is another easy way to obtain unique IDs. UUID is a 128-bit number used to identify information in computer systems.
- Has very low probability of getting collision

Example of UUID: 09c93e62-50b4-468d-bf8a-c07e1040bfb2

## Desing of UUID
![[Pasted image 20220908225345.png]]
In this design, each web server contain an ID generator, and a web server is responsible for generating IDs independently

### Pros
- Generating UUID is simple. No coordition between servers is needed so there will not be any synchronious issues
- The system is easy to scale because each web server is responsible for generating IDs they consume. ID generator can easily scale with web servers
### Cons
- IDs are 128 bits, but our requirement is 64 bit
- IDs do not go up with time
- IDs could be non-numeric


# Ticket Server
Ticket server are another interesting way to generate unique IDs. Flicker developed ticket servers to generate distributed keys. It is worth mentioning how the system works

![[Pasted image 20220908225951.png]]

## Idea
Use a centralized auto_increment feature in a single database server (Ticket server)

### Pros
- Numeric IDs
- It is easy to implement, and it works for small to medium-scale application
### Cons
- Single point of failure. Single ticket server means if the ticket server goes down, all systems that depend on it will face issues. To avoid a single point of failure, we can set up multiple ticket servers. But this will introduce new challenges such as data synchronization

# Twitter snowflake approach
Twitter's unique ID generation system called "**snowflake**" is inspiring and can satisfy our requirements

Divide and conquer. Instead of generating an ID directly, we devide an ID into different sections

![[Pasted image 20220908231115.png]]

- Sign bit: 1 bit. will alway be 0. This is reserved for future uses. It can potentially be used to distinguish between signed and unsigned numbers
- Timestamp: 41 bits. Miliseconds since the epoch or custom epoch. Twitter snowflake default epoch 1288834974657, equivalent to Nov 04, 2010, 01:42:54 UTC.
- Datacenter ID: 5 bit, 2^5  centers
- Machine ID: 5 bits, which give us 2^5 = 32 machines per datacenter
- Sequence number: 12 bits. For every ID generated on that machine machine/process, the sequence number is incremented by 1. The number is reset to 0 every milisecond