Notification system is very important feature, has 3 type format:
- Mobile push notification
- SMS messages
- Email
![[Pasted image 20221030222526.png]]


# Step 1 Understand the problem and establish design scope
- Scalable system that send outs million of notification a day is not an easy task. It requires a deep understanding of the notification ecosystem. Question is purposely desined to be open-ended and ambigous
Definition:
- Type of notification: push notification, sms, email
- Real-time: soft real-time. Want a user receive notifications as soon as possible, but a slight delay is acceptable
- Supported device: ios, android, laptop/desktop
- Triggers notification: trigger by client application, they can be also scheduled on the server-side
- User be able to opt-out? -> yes, user can choose to opt-out, will no longer receive notifications
- How many notifications are sent out each day? 10 million mobile push noti, 1m sms, 5m emails

# Step 2 Propose high-level design and get buy-in
High-level design that support various notification types: ios push notification, android, sms, email
- Different types of notifications
- contact info gathering flow
- notification sending/receiving flow
## Different types of notifications

### ios push notification
![[Pasted image 20221030230243.png]]

Primary need three components to send an ios push notification:
- Provider: a provider build and send notifcation requests to Apple Push Notification Sevice (APNS). provider provides the following data:
	- Device token: unique identifier used for sending push notifications
	- payload: this is a json dictionary that contain a notification's payload
- ![[Pasted image 20221030231153.png]]
- APNS: a remote service provided by apple to propagate push notification to ios device
- ios device: it is the end client, which receives push notifications
### android push notification
Android adopts a similar notification flow. Instead of using APNs, Firebase Cloud Messaging is commonly used to send push notifications to android device
![[Pasted image 20221030233259.png]]

![[Pasted image 20221030234954.png]]

### Contact info gathering flow
To send notifications, we need to gather mobile device tokens, phone numbers, email addresses
-> when a user installs our app or sign up for the first time, API servers collect user contact info and store it in the database
![[Pasted image 20221030235202.png]]

User can have multiple devices, indicating that a push notification can be sent to all the user devices
![[Pasted image 20221030235704.png]]

### Notification sending/receving flow
#### high levl design
![[Pasted image 20221030235816.png]]
- Service 1 to N: A service can be a micro-service, a cron job, or a distributed system that trigger notification sending events
- Notification system: the notification system is the centerpiece of sending/receiving notifications
- Third-party service: are responsible for delivering notifications to users. While interacting with third-party services, we need to pay extra attention to extensibility. Good extensibility mean a flexible system that can easily plugging or unplugging of a third-party service
- ios, android, sms, email: user receive notifications on theire devices. 3 problems
	- single point of failure: single notification server mean spof
	- hard to scale: the notification system handle everything related to push notification in one server. challenging is to scale databases, caches, different notification processing components independently
	- performance bottleneck: processing and sending notifications can be resource intensive
### high-level design 
Improve the design as listed below
- move the database and cache out of the notification server
- add more notification servers and set up automatic horizontal scaling
- introduce message queues to decouple the system component
![[Pasted image 20221031234803.png]]

- Service 1 to N: represent different services that send notification via API provided by notification servers
- Notification servers: provide the following functionalities
	- API for service to send notification. API are only accessible internally or by verified client to prevent spam
	- Carry out basic validations to verify emails, phone number
	- Query the database or cache or cache to fetch data needed to render a notification
	- Put noti data to message queues for parallel processing
- POST https://api.example.com/v/sms/send

Request body
 ![[Pasted image 20221031235544.png]]
![[Pasted image 20221031235756.png]]

## step 3 design deep dive
we will discuss different type of notifications, contact info gathering flow, and notification sending/receiving flow. We will explore the following deep dive
- Reliability
- Additional component and consideration: noti template, noti setting, rate limiting, retry mechanism, security in push noti, monitor queued noti, event tracking
- Updated design
## Reliability
Must answer a few important reliability question when designing a noti system in distributed environment
### How to prevent data loss?
- Noti can usually be delayed or re-ordered, but never lost
- -> the notification system persist noti data in a database and implement a retry mechanism
- ![[Pasted image 20221101221941.png]]
### Will recipient receive a notification exactly once ?
- No
- To reduce the duplication occurrence, we introduce a dedupe machanism and handle each failure case carefully
- When a noti event first arrives, we check if it is seen before by checking the event ID. If it is seen before, it is discarded. Otherwise, we will send out the noti. For interested readers to explore why we cannot have exactly once delivery, refer to the reference material 

#### Rate limiting
- to avoid overwhelming user with too many noti, we can limit the number of noti a user can receive. this is important because receiver could turn off noti completely if we send too often
#### Retry mechanism
When a third-party service fail to send a noti, the noti will be added to the message queue for retrying

#### Event tracking
Noti metric, such as open rate, click rate, engagement are important in understanding customer behavior. Analytics service implement event tracking. Intergrationg between the noti system and the analytics service is usually required
![[Pasted image 20221101223344.png]]



# Updated design
![[Pasted image 20221101223409.png]]
![[Pasted image 20221101223508.png]]