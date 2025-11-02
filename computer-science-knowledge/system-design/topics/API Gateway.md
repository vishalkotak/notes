#### References
- https://www.hellointerview.com/learn/system-design/deep-dives/api-gateway
#### Core Responsibilities 
- request routing
- microservice architecture
##### Tracing a Request
![[Pasted image 20250411151335.png]]
1. Request Validation
2. Middleware
	1. Authentication JWT
	2. Rate limiting
	3. Terminating SSL
3. Routing
4. Backend communication
	1. If a client sends the request as HTTP, and the server only understands gRPC, the API gateway can convert the request to gRPC
5. Response transformation
6. Caching
#### Scaling an API gateway
- Horizontal scaling
	- API gateways are mostly stateless, so scaling them is not hard
- Global distribution
	- Regional distribution
	- GeoDNS - DNS based routing
	- Configuration synchronization
- 