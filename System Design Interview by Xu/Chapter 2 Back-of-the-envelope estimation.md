# Back-of-the-envelop estimation 
- estimates you create using a combination of thought experiments and common performance numbers to get a good feel for which design will meet your requirements
	- power of two:
		- latency numbers every programmer should know
	- availability numbers
# Power of two
- Although data volumn can become enormous when dealing with distributed systems, calculation all boils down to the basis.
	- -> to obtain correct calculations, it is critical to know the data volumn unit using the power of 2
	- ![[Pasted image 20220715000206.png]]
# Latency numbers every programmer should know
- the length of typical computer operation 
- ![[Pasted image 20220715000325.png]]
![[Pasted image 20220715000455.png]]

By analyzing the number in figure 2.1, we get the following conclusion:
- Memory is fast but the disk is slow
- Avoid disk seeks if possible
- Simple compression algorithms are fast
- Compress data before sending it over the internet if possible
- Data centers are usually in different regions, and it takes time to send data between them

# Availability numbers
- High availability is the ability of a system to be continuously operational for a desirably long period of time. High availabitiy is mesured as a precentage,
	- 100%: service that has 0 downtime. most service fall between 100% and 99%
- Service level agreement(SLA) is a commonly used term for service providers
	- an agreement between you(service provider) and your customer. this agreement formally define the level of uptime your service will deliver
		- Amazon, Google, Microsoft set their SLA at 99.9%
		- ![[Pasted image 20220715001255.png]]
## Example: estimate Twitter QPS and storage requirements
### Asumption:
- 300 milion monthly active users
- 50% of users use Twitter daily
- User post 2 tweets perday on average
- 10% of tweet contain media
- Data is stored for 5 years

### Estimation
- Query per second (QPS) estimate:
	- Daily active users(DAU): 300m * 50% = 150m
	- Tweets QPS: 150m * 2 tweets * / 24h / 3600s = ~ 3500
	- Peek QPS = 2 * QPS  = ~7000
- estimate media storage
	- average tweet size:
		- tweet id 64bytes
		- text 140 bytes
		- media 1mb
	- media storage: 150m * 2 * 10% *1 MB = 30TB per day
	- 5-year media storage: 30TB * 365 * 5 = 55PB
# Tips
- Back-of-the-envelope estimation is all about the process
	- Solving problem is more important than obtaining results
- Tips
	- Rounding and Approximation. For example: there is no need to calculate exactly the result of 100000/123211, just round it to be fine
	- Write down your assumption: It is a good idea to write down your assumption to be referenced later
	- Label your unit: just 5 is going to make you confuse -> need to write 5mb, 5gb or ....
	- commonly asked back-of-the-envelop estimation: QPS, peak QPS, storage, cache, number of servers....