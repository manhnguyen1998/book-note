- It would be nightmare if you get the question: designing a well-known product X?

# A 4-step for effective system design interview
## Step 1: Understand the problem and establish design scope
- Dont be shy if you answer slowly. Slow down and think deeply, ask question to clarify requirement and assumption.
- Engineer like to solve hard problem and jump into the final design.
	- -> however, this approach is likely lead you to design the wrong system. 
- One of the most important skill as a engineer is to ask the right question, make the proper assumption, and gather all the information needed to build a system. Do not afraid to ask questions
	- Write down assumption on the whiteboard or paper
### What kind of question to ask
- What specific features are we going to build
- How many users does the product have
- How fast does the company anticipate to scale up? What are the anticipated scales in 3 months, 6 months, and a year?
- What is the company's technology stack? What existing services you might leverage to simplify the design?

### Example: 
- Are asked to design a news feed system -> ask questions that help you clarify the requirements. 
- -> mobile app? web app? both?
- what is the most important feature?
- is the news feed sorted in reverse chronological order or a particular order? Each post is given a different weight -> post from your close friend are more important than posts from a group
- How many friends can a user have
- What is the traffic volumn
- Can feed contain media (img, video...)
## Step 2: Propose high-level design and get buy-in 
This step, we aim to develop a high-level design and reach an agreement with the interview on the design
- Come up with an initial blueprint for the design. Ask for the feedback. Treat your interviewer as a teamate and work together. Many good interviewers love to talk and get involved
- Draw box diagrams with key components on the whiteboard or paper. This might include clients (mobile/web). API, web servers, data stores, cache, CDN, message queue....
- Do back-of-the-envelop calculations to evaluate if your blueprint fits the scale constrain. THink out loud. Communicate with your interview if back-of-the-envelop is necessary before diving it

### Example: Design a news feed system
At the high level, the design is divied into two flows: feed publishing and news feed building
- Feed publishing: when a user publishes a post, corresponding data is written into cache/db, and the post will be populated into friend's news feed
- News feed building: the news feed is build by aggregating friend's posts in a reverse chronological order
- ![[Pasted image 20220717001317.png]]
![[Pasted image 20220717001905.png]]

# Step 3 Design deep dive
At this step, you and your interviewer should have already achived the following objectives:
- Agreed on the overall goals and feature scope
- Sketched out a high-level blueprint for the overall design
- Obtained feedback from your interviewer on the high-level design
- Had some initial ideas about areas to focus on in deep dive based on her feedback

In most cases, the interviewer may want you to dig into details of some system components. 
- For URL shortener, it is interesting to dive into the hash function design that converts a long URL to a short one
- For a chat system, how to reduce latency and how to support online/offline status are two interesting topics

Try not to get into unnecessary details


### Feed publishing
![[Pasted image 20220717003407.png]]

### News feed retrieval
![[Pasted image 20220717003626.png]]

### Step 4 Wrap up
In this final step, the interviewer might ask you a few follow-up question or give you the freedom to discuss other additional points. Here are a few directions to follow:
- Might want you to identify the system bottenecks and discuss potential improvement.
	- Never say your design is perfect and nothing can be improved. There is always sthing to improve upon
	- -> great opportunity to show your critical thinking and leave a good final impression
- It could be useful to give the interviewer a recap of your design
	- Refreshing your interviewer's memory can be helpful after a long session
- Error cases (server failure, network loss...) are interesting to talk about
- Operation issues are worth mentioning. How do you monitor metrics and errors logs. How to roll out the system
- How to handle the next scale curve is also an interesting topic. For example, if your current design supports 1 million users, what changes do you need to make to support 10 million users
- Propose other refinement you need if you had more time


#### Dos
- Always ask for clarification. Do not assume your assumption is correct
- Understand the requirements of the problem
- There is neither the right answer nor the best answer. A solution designed to solve the problems of a young startup is different from that of an established company with millions of users. Make sure you understand the requirements
- Let the interviewer know what you are thinking. Communicate with your interview.
- Suggest multiple approaches if possible
- Once you agree with your interviewer on the blue print, go into details on each component. Design the most critical components first
- Bounce ideas off the interviewer. A good interviewer works with you as a teamate.
- Never give up

### Donts
- Dont be unprepared for typical interview questions
- Dont jump into a solution without clarifying the requirements and assumptions
- Dont go into too much detail on a single component in the beginning. GIve the high-level design first then drills down
- If you get stuck, dont hestitate to ask for hint
- Again, communicate. Dont think in silence
- Dont think your interview is done once you give the design You are not done until your interviewer says your are done. Ask for feedback early and often

### TIme allocation on each step
45 minutes interview session

- Step 1: Understand the problem and establish design scope: 3 - 10 minutes
- Step 2: Propose high-level design and get buy-in: 10 - 15 minutes
- Step 3: Design deep dive: 10 - 25 minutes
- Step 4: Wrap: 3 - 5 minutes