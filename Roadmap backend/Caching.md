# What is caching
- Caching is a process or method of storing a copy of the continuously asked data in a temporary storage location or cache to be quickly accessed when needed
- A cache is a memory reserved for storing temporary file of data from apps, servers, websites, browsers to help load faster when requested. Images, videos, animation, gifs....

# Website caching
- Sever-side caching
- Client-side caching
- Remote caching

# Serverside caching
Sever side caching temporarily store web files and data on the origin server to reuse later

![[Pasted image 20220906223318.png]]

![[Pasted image 20220906223401.png]]

## Type of Server-side caching

### Object Caching: 
In object caching, we store database queries. As a result, it is easier to access the next time the request is made
- ![[Pasted image 20220906223519.png]]

### Obcode Caching
- Opcode caching is a performance booster for PHP that compiles human-readable PHP to bytecode understood by webserver
![[Pasted image 20220906223636.png]]

### CDN Caching
CDN is a group of servers around the globe to provide content delivery visitors
![[Pasted image 20220906223731.png]]


# Client-side caching
- Client-side caching temporarily stores web files and data in the browser
![[Pasted image 20220906224605.png]]


## Type of Client-side Caching
- Browser Request Caching: the most widely used and oldest form of caching. It is built into the HTTP protocol standard
- Javascript Ajax caching: modifies the websites to display changes made to dynamic content in real-time without refreshing the entire page
- HTML 5 caching: we try to cache images and scripts and HTML content. Since HTML take too long to load, it will delay other processe


# Remote caching
- Is similar to Server side caching, but it can also run an application to serialize and deserialize the data. The different is that you control the remote server, and someone else does not operate it