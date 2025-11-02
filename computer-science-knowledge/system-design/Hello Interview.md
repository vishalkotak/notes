**Important Notes:**
- For infrastructure style questions, the functional requirements will not be exhaustive.
- Focus more on non-functional requirements

![[Pasted image 20250318220858.png]]

Topics to deep dive into:
- Count min sketch
#### Top K Youtube Videos 
- https://www.youtube.com/watch?v=1lfktgZ9Eeo

![[Pasted image 20250318213118.png]]

- https://www.youtube.com/watch?v=nQpkRONzEQI&t=116s

Functional Requirements
- All user facing operations should be as fast as possible
	- Publishing events
	- Reading the leaderboard
- Note that getting exact results can make fetching the leaderboard slow
	- We have a tradeoff between:
		- Fast Reads
		- Precise Results
- 

#### LeetCode (Online Coding Competition)
- https://www.youtube.com/watch?v=1xHADtekTNg

![[Pasted image 20250330105047.png]]
#### Design a Web Crawler
- https://www.youtube.com/watch?v=krsuaUp__pM

![[Pasted image 20250319163654.png]]

#### Design an Ad Click Aggregator
https://www.youtube.com/watch?v=Zcv_899yqhI&t=811s

![[Pasted image 20250321122502.png]]
#### Design Dropbox or Google Drive 
- https://www.youtube.com/watch?v=_UZ1ngy-kOI&t=1376s

![[Pasted image 20250321182300.png]]

![[Pasted image 20250321182356.png]]
#### Uber
- https://www.youtube.com/watch?v=lsKU38RKQSo

![[Pasted image 20250321212605.png]]

#### DynamoDB Dive Deep
- https://www.youtube.com/watch?v=2X2SO3Y-af8&t=41s

![[Pasted image 20250321214143.png]]
![[Pasted image 20250321214218.png]]
![[Pasted image 20250321214237.png]]
![[Pasted image 20250321214250.png]]
![[Pasted image 20250321214308.png]]
![[Pasted image 20250321214328.png]]
![[Pasted image 20250321214348.png]]
![[Pasted image 20250321214404.png]]
![[Pasted image 20250321214424.png]]
#### Redis Deep Dive


- In-Memory
- Single-Threaded
- Data Structure Store
- Key is used to find the hash slot
- If the redis client makes a request to the wrong node, the cluster will respond with a MOVED or ASK redirection. Client takes ownership of then sending it to the right node
- Redis nodes in a cluster use a gossip protocol
- In order to handle the hot key problem with redis, you can append a random number with the key and then generate the hash

![[Pasted image 20250321215525.png]]
Redis as a rate limiter
![[Pasted image 20250321220815.png]]
** Redis Stream



#### Design YouTube

![[Pasted image 20250322082513.png]]
#### Design WhatsApp
- https://www.youtube.com/watch?v=cr6p0n0N-VA&t=17s

![[Pasted image 20250322142948.png]]
#### Design Live Comments
- https://www.youtube.com/watch?v=LjLx0fCd1k8&t=1042s

![[Pasted image 20250322170152.png]]
#### BitLy

- https://www.youtube.com/watch?v=iUU4O1sWtJA&t=123s

![[Pasted image 20250322184402.png]]
#### Design Tinder
- https://www.youtube.com/watch?v=18Fg5Akhkqw&t=4199s
-
![[Pasted image 20250322215129.png]]

#### Design Distributed Task Scheduler

##### Requirements
- Schedule a task to be executed
- Schedule a task to be executed on a schedule
- 30 Seconds SLA
- Multi-Tenant = 2B Tasks / Day
- Each team = Tenant in the org
##### High Level Diagram

![[Pasted image 20250330174456.png]]
##### Deep Dive

![[Pasted image 20250330180819.png]]
- Poller service will also look at cron schedule and add it to jobs for next 10 runs

#####