References:
- https://aws.amazon.com/what-is/load-balancing/

Load balancing is the method of distributing network traffic equally across a pool of resources that support an application.
#### Algorithms
##### Static Load Balancing
Static load balancing algorithms follow fixed rules and are independent of the current server state.
###### Types
- Round Robin
- Weighted Round Robin
- IP Hash
##### Dynamic Load Balancing
Dynamic load balancing algorithms examine the current state of the servers before distributing traffic.
###### Types
- Least Connection Method
- Weighted Least Connection Method
- Least Response Time method
- Resource based method
#### Types of load balancing
- Application
- Network
- Global
	- Global server load balancing occurs across several geographically distributed servers. For example, companies can have servers in multiple data centers, in different countries, and in third-party cloud providers around the globe.
	- In this case, local load balancers manage the application load within a region or zone.
	-  They attempt to redirect traffic to a server destination that is geographically closer to the client.
	- They might redirect traffic to servers outside the client’s geographic zone only in case of server failure.
- DNS
	- In DNS load balancing, you configure your domain to route network requests across a pool of resources on your domain


