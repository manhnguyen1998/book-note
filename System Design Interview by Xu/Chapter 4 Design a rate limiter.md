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