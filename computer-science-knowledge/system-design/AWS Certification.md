#### EC2 User Data
- Bootstrap our instances using EC2 User data script
- Launching commands when a machine starts
- Run only once at the instance first start
- Examples:
	- Install updates
	- Install software
	- Downloading common files from the internet
	- Anything you can think of
- Runs with the root user
![[Pasted image 20250323140125.png]]
![[Pasted image 20250323140206.png]]
![[Pasted image 20250323140238.png]]
![[Pasted image 20250323140308.png]]
#### Security Groups
- Security Groups control how traffic is allowed into our out of our EC2 instance
- Contain only allow rules
- Can reference by IP or by security group
- Acts as firewall on the EC2 instance
- Can be attached to multiple instances 
- Locked down to a region / VPC combination
- Is the layer outside the EC2 instance = traffic does not reach EC2
- If you application is not accessible (time out), then it is a security group issue
- If you application gives a connection refused error, then it is an application error or not launched
- All inbound traffic is blocked by default
- All outbound traffic is enabled by default

![[Pasted image 20250323141211.png]]
#### Classic Ports to know
- 22 = Secure Shell
- 21 = FTP
- 22 = SFTP
- 80 = HTTP
- 443 = HTTPS
- 3389 = RDP
#### EC2 Purchasing Options

![[Pasted image 20250323150406.png]]
#### Load Balancers

###### Why use Elastic Load Balancers?
- Managed load balancer
- High availability 
- Upgrades, maintenance taken care of
- Configuration available
###### 4 Kinds of load balancers
- Classic LoadBalancer - Deprecated
- Application Load Balancer (V2 - New Generation)
	- HTTP, HTTPS, WebSockets
- Network Load Balancer
	- TCP, TLS (Secure TCP), UDP
- Gateway Load Balancer
	- Operates at Layer 3 (Network Layer) - IP Protocol
#### Application Load Balancers
- Routing strategies
	- Routing based on path in the URL
	- Routing based on hostname in the URL
	- Routing based on query string, headers
- **Port Mapping feature to redirect to dynamic port in ECS**
- ALB does not provide static IP addresses. It mostly uses domain name for mapping
- If you need a static IP address then you will need a NLB in front of the ALB. The NLB's get a single static IP address per AZ.
###### Target Groups
- EC2 Instances - HTTP
- ECS Tasks - HTTP
- Lambda Functions - HTTP request is translated into a JSON event
- IP Addresses - Must be private IP
- **ALB can route to multiple target groups**
- **Health checks are at the target group level**
- Fixed HostName for the ELB
- Application Servers don't see the the IP of the client directly
	- The true IP of the client is inserted in the header *X-Forwarded-For*
	- We can also get the Port (X-Forwarded-Port) and proto (X-Forwarded-Proto)
#### Network Load Balancers (NLB)
- Forwards TCP and UDP traffic to your instances
- Handle millions of requests per second
- Ultra-low latency
- Has one static IP per AZ and supports assigning elastic IP
###### Target Groups
- EC2 instances
- IP Addresses - Must be Private IP
- Application Load Balancer
- Health Check support the TCP, HTTP and HTTPS protocols



