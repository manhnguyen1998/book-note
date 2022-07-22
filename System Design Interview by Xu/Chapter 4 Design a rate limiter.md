In a network system, a rate limiter is used to control the rate of traffict sent by a client or a service

If the API request count exceeds the threshold defined by the rate limiter, all the excess call are blocked. A few examples
- A user can write no more than 2 posts per second
- You can create a maximum of 10 accounts per day from the same IP address

Benefit of using an API rate limiter
- Prevent resource starvation caused by DOS attack
	- Twitter limits the number of tweets: 300/ 3hrs
	- Google docs API: 300 per user per 60s
- Reduce cost
- Prevent servers from being overloaded

# Step 1 Understand the problem and establish design scope
## Requirement for the system
- Accurately limit excessive requests.

• Low latency. The rate limiter should not slow down HTTP response time.

• Use as little memory as possible.

• Distributed rate limiting. The rate limiter can be shared across multiple servers or processes.

• Exception handling. Show clear exceptions to users when their requests are throttled.

• High fault tolerance. If there are any problems with the rate limiter (for example, a cache server goes offline), it does not affect the entire system.

# Step 2 Propose high-level design and get buy-in

## Where to put the rate limiter
You can implement a rate limiter at either the client or server-side
- Client-side implement: Generally speaking, client is an unreliable place to enforce rate limiting because client request can easily be forged by malicious actors. 
	- more, we might not have control over the client implementation
- Server-side implementation: 
- ![[Pasted image 20220719233333.png]]

Beside the client and server-side implementation, we can put a middleware throttle request to your API 
![[Pasted image 20220719233740.png]]



### How the rate limiter work
![[Pasted image 20220719234320.png]]



### Cloud service
- Have become widely popular and rate limiting is usually implemented within a component called API gateway
- API gateway: is a fully managed service that support rate limiting, SSL termination, authentication, IP whitelisting, servicing static content/
	- But now, we only need to know that the API gateway is a middleware that support rate limiting

## While designing a rate limiter, important question to ask ourselves is where should the rater limiter be implemented?
### A few guideline
- Evaluate your current technology stack: Programming language, cache service... Make sure your current programming language is efficient to implement rate limiting on the server-side
- Identify the rate limiting algorithm that fits your business model
- If you have already used microservice architecture and inclued an API gateway in the design to perform authentication, IP whitelisting, etc... you may add a rate limiter to the API gateway
- Building your own rate limiting service takes time. If you do not have enough engineering resource to implement a rate limiter, a commercial API gateway is a better option

## Algorithm for rate limiting
- Popular algorithm
	- Token bucket
	- Leaking bucket
	- Fixed window counter
	- Sliding window log
	- Sliding window counter
-
### Token bucket algorithm
- Is widely used for rate limiting
- Amazon, Stripe use this algorithm to throttle their API request

#### How it work?
- A token bucket is a container that has pre-defined capacity. Once the bucket is full, no more tokens are added
- ![[Pasted image 20220720000408.png]]
- Each request consume one token. When a request arrives, we check if there are enough token in the bucket
	- If there are enough tokens, we take one token out for each request, and the request goes through
	- If there are not enough tokens, the request is dropped
![[Pasted image 20220720000746.png]]


![[Pasted image 20220720000826.png]]


The algorithm take two parameter
- Bucket size: the maximum number of tokens allowed in the bucket
- Refill rate: number of tokens put into the bucket every second

### How many bucket do we need?
- Depend on the rate-limiting rules
	- Necessary to have different bucket for different API endpoint
		- Example: if a user is allowed to make 1 post/second, add 150 friends per day, like 5 posts per second, 3 buckets are required for each user
		- If we need to throttle request based on IP addresses, each IP address requires a bucket
		- If the system allows a maximum of 10000 request per second, it make sense to have a global bucket shared by all request

#### Pros:
- The algorithm is easy to implement
- Memory efficient
- TOken bucket allows a burst of traffic for shord period. A request can go through as long as there are tokens left

#### Cons 
- Two parameters in the algorithm are bucket size and token refill rate. However, it might be challenging to tune them properly

### Leaking bucket algorithm
The leaking bucket algorithm is similar to the token bucket except that request are processed at a fixed rate. Usually be implemented with a FIFO queue
- When a request arrive, the system check if the queue is full. If it is not full, the request is added to the queue
- Otherwise, the request is dropped
- Request are pulled from the queue and processed at regular interval
- ![[Pasted image 20220720233652.png]]

Leaking bucket algorithm take the followint two parameter
- Bucket size: it is equal to the queue size. The queue hold the request to be processed at a fixed rate
- Outflow rate: it defines how many request can be processed at a fixed rate, usually in seconds

Shopify is using this algorithm

#### Pros
- Memory efficient given the limited queue size
- Request are processed at a fixed rate therefore it is suitable for use case that a stable outflow rate is needed

#### Cons
- A burst of traffic fill up the queue with old request, and if they are not processed in time, recent request will be rate limited
- There are two parameter in the algorithm. It might not be easy to tune them properly


### Fixed window counter algorithm
Fixed window counter algorithm works as follow
- The algorithm divide the timeline into fix-sized time windows and assign a counter for each window
- Each request increments the counter by one
- Once the counter reaches the pre-defined threshold, new requests are dropped until a new time window starts
- ![[Pasted image 20220720234500.png]]

### Pros
- Memory efficient
- Easy to understand
#### Cons
- Spike in traffic at the edges of a window could cause more request than the allowed quota go through


### Sliding window log algorithm
The sliding window log algorithm fixed the issue of fixed window counter
- THe algorithm keep track of request timestamp. Timestamp data is usually kept in cache, such as sorted sets of Redis
- When a new request come in, remove all the outdated timestamp. Outdated timestamp are defined as those older than the start of the current time window
- Add timestamp of the new request to the log
- If the log size is the same or lower than the allowed count, a request is accepted. Otherwise, it is rejected
- ![[Pasted image 20220720235544.png]]
#### Pros
- Rate limiting implemented by this algorithm is very accurate. In any rolling window, request will not exceed the rate limit
#### Cons
- The algorithm consumes a lot of memory because even if a request is rejected, its timestamp might still be stored in memory


### Sliding window counter algorithm
This algorithm is a hybrid approach that combines the fixed window counter and sliding window log
![[Pasted image 20220721000154.png]]
The number of request in the rolling window is calculated using the following formula:
- Request in current window + request in the previous window * overlap percentage of the rolling window and previous window
- Using this formula, we get 3 + 5*0.7 = 6.5 requests. It can be rounded up and down: 6

#### Pros
- It smooth out spikes in traffic because the rate is based on the average of the previous window
- Memory efficient
#### Cons
- It only works for not-so-strict look back window. It is an approximation of the actual rate because it assumes requests in the previous window are evenly distributed. However, this problem may not be as bad as it seems. According to experiments done by Cloudflare, only 0.003% of request are wrongly allowed or rate limited among 400 million requests

## High-level architecture
At the high-level, we need a counter to keep track of how many requests are sent from the same user, IP address, If the counter is larger than the limit, the request is disallowed

- Where to store counter
	- Database is not good, should choose in-memory cache because it is fast, support time-based expiration strategy
- REdis: is a popular option to implement rate limiting. It is an in-memory store that offers two command: INCR and EXPIRE
	- INCR: It increase the stored counter by 1
	- EXPIRE: It sets a timeout for the counter. IF the timeout expires, the counter is automatically deleted
- ![[Pasted image 20220721001843.png]]

- The client send a request to rate limiting middleware
- Rate limiting middleware fetches the counter from the corresponding bucket in Redis and checks if the limit is reached or not
	- If the limit is reached, the request is rejected
	- If the limit is not reached, the request is sent to API servers. Meanwhile, the system increments the counter and saves it back to Redis

# Step 3 Design deep dive
The high-level design above does not answer the following questions
- How are rate limiting rules created? Where are the rules stored?
- How to handle requests that are rate limited?


## Rate limiting rules
Lyft's rate limiting rules:
![[Pasted image 20220721232144.png]]
-> allow a maximum of 5 marketing messages per day

## Exceeding the rate limit
In case a request is rate limited, APIs return a HTTP response code 429 (too many requests) to the client. Depending on the use cases, we may enqueue the rate-limited requests to be processed later. For example, if some orders are rate limited due to system overload, we may keep those orders to be processed later

### Rate limiter headers
How does a client know whether it is being throttled? And how does a client know the number of allowed remaining requests before throttled?
-> The answer lies in HTTP response headers
- X-Ratelimit-Remaining: The remaining number of allowed requests within the window
- X-Ratelimit-Limit: It indicates how many calls the client can make per time window
- X-Ratelimit-Retry-After: The number of seconds to wait until you can make a requests again without being throttled

When a user has sent too many requests, a 429 too many requests error and X-Ratelimit-Retry-After header are returned to the client

## Detailed design
![[Pasted image 20220721233841.png]]
- Rules are stored on the disk. Workers frequently pull rules from the disk and store them in the cache
- When a client send a request to the server, the request is sent to the rate limiter middleware first
- Rate limiter middlewares loads rules from the cache. It fetches counter and last request timestamp from Redis cache. Based on the response, the rate limter decides:
	- If the request is not rate llimited, it is forwarded to API server
	- If the request is rate limited, the rate limiter return 429 too many requests error to the client. In the meantime, the request is either dropped or forwarded to the queue

### Rate limiter in a distributed environment
Building a rate limiter that work in a single server is not difficult, but scaling the system to support multiple servers and concurrent threads is a different story

There are 2 challenges:
- Race condition
- Synchronization issue

#### Race condition
Rate limiter works as follow at the high-level
- Read the counter value from Redis
- Check if (counter +1) exceed the threshold
- If not, increment the counter value by 1 in Redis
- ![[Pasted image 20220722000336.png]]
2 request read the counter at the same time -> the counter is updated as 4, but the real value is 5

There are 2 strategies are commonly used
- Lua script
- Sorted sets data structure in Redis

#### Synchroniztion issue
Client can send request to a different rate limiter. But if no synchronization, rate limiter 1 does not contain any data about client 2
![[Pasted image 20220722000633.png]]

Solution: 
- A better approach is to use centralized data store like Redis
- ![[Pasted image 20220722000917.png]]

### Performance optimization
- Multi-data center setup is crucial for a rate limiter because latency is high for user located far away from the data center. Most cloud service providers build many edge server locations around the world.
- Synchronize data with an eventual consistency model

### Monitoring
After the rate limiter is put in the place, it is important to gather analytics data to check whether the rate limiter is effective. We want to make sure
- The rate limiting algorithm is effective
- The rate limiting rules are effective

# Step 4 Wrap up
Avoid being rate limited. Design your client with best practice
- Use client cache to avoid making frequent API call
- Understand the limit and do not send too many requests in a short time frame
- Include code to catch exceptions or errors so your client can gracefully recover from exceptions
- Add sufficient back off time to retry logic