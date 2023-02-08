![[Pasted image 20221018215550.png]]

A web crawler is used for many purposes:
- Search engine indexing: most common use case, collect web to create a local index for search engines, for example: googlebot
- Web archiving: process of collecting information from the web to preserve data for future uses. many national libraries run crawlers to archive web sites
- Web mining: the explosive growth of the web present an unprecedented opportunity for data mining. Web mining help to discover useful knowledge from the internet
- Web mornitoring: crawler help to monitor copyright and trademark infrigements over the Internet,
-> complexity of developing a web crawler depend on the scale we intend to support


# Step 1: Understand the problem and establish design scope
## Basic algorithm of a web crawler
- Given set of URLs, download all the web pages addressed by the URLs
- Extract URLs from these web pages
- Add new URLs to the list of URLs to be downloaded. Repeat these 3 steps

-> Define scope of this web crawler
- Main purpose: search engine indexing? data mining? sth else?
	- search engine indexing
- how many web page collect per month?
	- 1 bilion
- Content type included? HTML only? other content?
	- HTML only
- consider newly added or edited web pages
	- consider the newly added or edited web pages
- Need to store HTML pages?
	- up to 5 years
- how do we handle web pages with duplicate content
	- pages with duplicate content should be ignored


## Web scope
- Scalability: the web is very large. There are bilions of web pages out there. should be extremly efficient using paralleliztion
- Robustness: the web is full of traps. Bad HTML, unressponsive servers, crasheds... the crawler must handle all those edge cases
- Politeness: the cralwer should not make too many requests to a website within a short time interval
- Extensibility: the system is flexible so that minimal changes are needed to support new content types. For example, if we want to crawl image files in the future, we should not need to redesign the entire system

# Back of the envelop estimation
- Assume 1 bilion web pages are downloaded every mont
- QPS: 1,000,000,000 / 30 days / 24 hours / 3600 s = 400 pages per second
- Peak QPS = 2 * QPS = 800
- Assume the average web page size is 500k
- 1 bilion page x 500k = 500 TB storage per month
-> 5 years = 500TB x 12 x 5 = 30PB

# Step 2: Propose high-level design and get buy-in
![[Pasted image 20221018221749.png]]

Seed URLs
- Web crawler uses seed URLs as a starting point for the crawler process
- Choose seed urls can based on topics, shopping, sports, heathcare...
URL frontier
- Most modern web crawlers split the crawl into 2:  to be downloaded and already downloaded. The component that stores urls to be downloaded is called the url frontier
HTML downloader
- download web pages from the internet. those urls are provided by the url frontier
DNS resolver
- to download a web page, a URL must be translated into an IP address. the HTML Downloader calls the DNS resolver to get the corresponding for URL
Content Parser
- After a web page is downloaded, it must be parsed and validated because malformed web pages could provoke problems and waste storage space. Implementing a content parser in a crawl server will slow down the crawling process. THus, the content parser is a separate component
Content Seen?
- 29% of the web pages are duplicated contents. To compare 2 HTML documents, we can compare them character by character. But this method is slow and time-consuming, especially when billions of web pages are involved. An efficient way to accomplish this task is to compare the hash value of two web page
Content Storage
- Most of the content is stored on disk because the data set is too big to fit in memory
- Popular content is kept in memory to reduce latency
URL Extractor
- URL extractor parses and extract links from HTML pages
- ![[Pasted image 20221018224157.png]]

URL filter
- excludes certain content types, file extensions, error links, URL in blacklisted sites
URL seen
- a data structure that keep track of URLs that are visited before or already in the frontier. URL Seen help to avoid adding the same URL multiple times as this can increase server load and cause potential loops

![[Pasted image 20221018224739.png]]

# Step 3: Design deep dive

DFS and BFS
- BFS is commonly used by web crawlers and is implemented by FIFO queue. In a FIFO queue, URLs are dequeued, but this has problems
	- most link from the same webpage are linked back to same host -> the crawler tries to download web pages in parallel, the server will be flooded with requests -> this is called "impolite"
	- standard BFS does not take the priority of a URL into consideration. The web is large and not every page has the same level of quality and importance


# URL frontier
- URL frontier help to address these problems. URL frontier is a data structure that store URLs to be downloaded
- URL frontier is an important component to ensure politeness, URL prioritization, and freshness

### Politeness
- A web crawler should avoid sending too many requests to the same hosting server within a short period. Sending too many request is considered as impolite, or even DDOS
-> enforcing politeness is to download one page at a time from the same host. A delay can be added between two download tasks. The politeness constrains is implemented by maintain a mapping from website hostname to download thread
Each downloader thread has a separate FIFO queue and only download URLs obtained from that queue.
![[Pasted image 20221027224119.png]]

![[Pasted image 20221027224620.png]]


## Priority
A random post from a discussion forum about Apple product carries very different weight than posts on the Apple homepage
-> Prioritize URLs based on usefulness, which can be measured by PageRank, website traffic, update frequency, 
Prioritizer is the component that handles URL prioritization
![[Pasted image 20221027225633.png]]

![[Pasted image 20221027234723.png]]


### Freshness
Web pages are constantly being added, deleted, and edited. A web crawler must periodically recrawl downloaded pages to keep our data set fresh
- Recrawl based on web pages's update history
- Prioritize URLs and recrawl important pages first and more frequently

### Storage for URL Frontier
The number of URLs in the frontier could be hundreds of millions
-> hybrid approach. Majority of URLs are stored on disk, so the storage space is not a problem. to reduce cost of reading from the disk and writing to the disk, we maintain buffers in memory for enqueue/dequeue operations. Data in the buffer is periodically written to the disk

### HTML Downloader

#### Robots.txt
Robots Exclusion Protocol, is a standard used by websites to communicate with crawlers. It specifies what pages crawler are allowed to download. Before crawl a page, it should check its corresponding robots.txt first and follow it rules
![[Pasted image 20221028001655.png]]

#### Performance optimization
1. Distributed crawl
TO achieve high performance, crawl jobs are distributed into multiple servers, and each server runs multiple thread. The URL space is partitioned into smaller pieces, so each downloader is responsible for a subset of URLs

![[Pasted image 20221028002346.png]]

2. Cache DNS Resolver
DNS Resolver is a bottleneck for crawler because DNS requests might take time due to synchronous nature of many DNS interface. DNS response time ranges from 10ms to 200ms. Once a request to DNS is carried out by a crawler thread, other threads are blocked until the first request is completed. 
3. Locality
Distributed crawl servers geographically. When crawl are closer to website hosts, crawlers experience faster download time. Design locality applies to most of the system components: crawl servers, cache, queue, storage
4. Short timeout
Some web servers respond slowly or may not respond at all. To avoid long wait time, a maximal wait time is specified. If a host does not respond within a predefined time, the crawler will stop the job and crawl some other pages

#### Robustness
Robustness is also an important consideration. 
- Consistent hashing: help to distribute load among downloader. New downloader server can be added or removed using consistent hashing
- Save crawl state and data: To guard agains failures, crawl states and data are written to a storage system. A disrupted crawl can be restarted easily by loading saved states and data
- Exception handling: Errors are inevitable and common in a large-scale system. The crawler must handle exception gracefully with out crashing the system
- Data validation: this is an important measure to prevent system errors
### Extensibility
Almost systems evolve, -> make the system flexibles to support the new content types

![[Pasted image 20221028003614.png]]

#### Detect and avoid problematic content
1. Redundant content
30% of the web pages are duplicate -> hash or checksums help to detect duplication
2. Spider traps
Spider trap is a web page that cause a crawler in an infinite loop
Example: www.spidertrapexample.com/foo/bar/foo/bar/foo/bar/...
spider trap can be avoided by setting maximal length for the urls
3. Data noise
Some of the content have little or no value, such as advertisement, code snippets, spam urls. Those content are not useful for crawler and should be exclude if possible