References:
- https://www.f5.com/company/blog/nginx/service-discovery-in-a-microservices-architecture

Consider a micro service architecture. We have services deployed handling different routes. These services will be deployed on instances that have an Elastic IP address (Dynamic IP allocation). We cannot have a configuration file mapping service names to their IP addresses as they can change frequently. During autoscaling, failures and upgrades (deployments), the mapping will change.

![[Pasted image 20250412184631.png]]
#### Patterns
- Client-Side discovery
- Server-Side discovery
##### Client Side discovery
Client queries a service registry, which is a database of available service instances. The client can then use a load balancing algorithm to select one of the available service instances and make a request.
![[Pasted image 20250412184921.png]]
##### Steps
- Service boots up and registers itself with the service registry
- Service sends regular heartbeats to the service registry
- Service registry will remove an instance due to lack of heartbeats and instance deletion
##### Examples
- Netflix Eureka Service Registry: https://github.com/Netflix/eureka
- Client working with the registry: https://github.com/Netflix/ribbon
##### Pros
- Simple 
- Service registry is the only component we need to maintain
- Clients can decide how they want to route requests
##### Cons
- Tight coupling between client and service registry
- If we have clients in multiple programming languages, we need to implement clients to connect to the service registry
##### Server-Side Discovery Pattern
Client makes a connection to the load balancer. The load balancer queries the service registry and routes each request to an available instance. 

![[Pasted image 20250412192031.png]]
###### Examples
- AWS Elastic Load Balancer
	- Use ELB to load balance traffic internal to VPC
	- Client makes connection request to ELB using DNS name
- Consul Service Registry with Consul Template (Investigate)
	- Periodically regenerates arbitrary configuration files from configuration data stored in the registry
	- Runs a shell command when the file holding the data changes
###### Kubernetes Service Discovery
Applications (services) are often broken down into multiple instances (containers/pods) running across many different physical or virtual machines (hosts/nodes). A client application (another service) needing to communicate with one of these services faces a challenge: How does it find a healthy, running instance of the service it needs _right now_? The IP addresses and ports of individual instances are dynamic and unreliable.

**Solution**
- The system runs a dedicated proxy process _on every single host_ within the cluster. Think of it as a local traffic manager or receptionist on each machine.
- The proxy doesn't just find _one_ instance; it typically knows about _all_ available, healthy instances for a given service.

**How it Works (Request Flow):**
- A client application (e.g., Service A) wants to send a request to another service (e.g., Service B).
- Instead of trying to find the specific IP of a Service B container, Service A sends its request to a _predictable, stable address_ managed by the proxy _on its own host_. This address is typically the host's own IP address (or `localhost`) combined with a _unique port assigned specifically to Service B_ by the orchestrator.
- The proxy running on Service A's host intercepts this request because it's listening on that specific `host_ip:service_b_port`.
- The proxy consults its internal routing table (which is kept up-to-date by the orchestrator like Kubernetes) to get the current list of _real_ IP addresses and ports for healthy instances of Service B running anywhere in the cluster.
- The proxy selects one of these real instances (using a load balancing algorithm like round-robin).
- The proxy then forwards the original request from Service A to the chosen Service B instance (`service_b_instance_ip:service_b_instance_port`).
- When Service B sends a response, it goes back through the proxy, which forwards it back to the original client (Service A).
###### Pros
- Details are abstracted away from the client
- No client side implementations required
###### Cons
- Until we get the load balancer we will have to maintain one which is a critical component in itself
#### Service Registry
- Replication needed for fault tolerance
- Clients can cache the data received from the service registry
##### How does Netflix achieve high availability of the service registry
- Run one registry instance i.e. Eureka servers in each EC2 availability zone
##### Examples
- etcd
- consul
- Zookeeper
#### Service Registration Options
Instances need to register with the service registry. 
##### Self-Registration Pattern
A service instance is responsible for registering and deregistering itself with the service registry. Also, if required, a service instance sends heartbeat requests to prevent its registration from expiring.

![[Pasted image 20250412192443.png]]
###### Examples
- Eureka client
- Spring Cloud Project
###### Pros
- Relatively simple
###### Cons
- Couples the service instances to the service registry
##### The Third‑Party Registration Pattern
Another system component known as the _service registrar_ handles the registration. The service registrar tracks changes to the set of running instances by either polling the deployment environment or subscribing to events. When it notices a newly available service instance it registers the instance with the service registry. The service registrar also deregisters terminated service instances. 

![[Pasted image 20250412192841.png]]
###### Examples
- Netflix Prana
	- Is a sidecar application that runs side by side with a service instance. Prana registers and deregisters the service instance with Netflix Eureka.
- EC2 instances created by an Autoscaling Group can be automatically registered with an ELB.
- Kubernetes services are automatically registered and made available for discovery.
###### Pro
- Services are decoupled from the service registry.
###### Cons
- Unless it’s built into the deployment environment, it is yet another highly available system component that you need to set up and manage.