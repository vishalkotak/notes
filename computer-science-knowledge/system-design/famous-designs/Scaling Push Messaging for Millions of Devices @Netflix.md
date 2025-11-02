Reference: https://www.youtube.com/watch?v=6w6E_B55p0E
![[Pasted image 20250323105100.png]]
![[Pasted image 20250323105714.png]]
![[Pasted image 20250323105923.png]]
![[Pasted image 20250323105938.png]]
![[Pasted image 20250323110002.png]]
![[Pasted image 20250323112235.png]]
![[Pasted image 20250323112329.png]]

![[Pasted image 20250323112104.png]]
#### Summary

The video discusses Zuul Push, Netflix's massively scalable push notification service, designed to handle millions of persistent, "always-on" connections from Netflix applications. Instead of relying on client devices to periodically poll the server for updates, Zuul Push proactively delivers new data, such as personalized movie recommendations, from the cloud directly to the devices. This approach significantly reduces data delivery latency and minimizes the cloud footprint by eliminating wasteful polling requests.

The presentation delves into the architecture of Zuul Push, highlighting its key components and their functions. The system utilizes open web protocols like WebSockets and Server-Sent Events (SSE) to ensure compatibility across a wide range of devices. A globally replicated client registry keeps track of which clients are connected to which Zuul Push server, enabling efficient message routing. Netflix microservices can send push notifications to connected clients through a simple API call, abstracting away the underlying infrastructure complexities. Message queues, implemented using Kafka, decouple message senders and receivers, allowing them to operate and scale independently. The Zuul Push server, built on top of the Zuul cloud gateway, handles a massive number of concurrent connections using non-blocking asynchronous I/O. Load balancing is achieved using Amazon Elastic Load Balancers (ELBs) in TCP mode. The video also touches upon operational aspects, emphasizing cost optimization and auto-scaling strategies based on the number of open connections.