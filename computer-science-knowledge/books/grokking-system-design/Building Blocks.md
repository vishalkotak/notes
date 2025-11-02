#### Domain Name System

Domain Name System is the Internet's naming service that maps human friendly name to a machine-readable IP address. 

![[Pasted image 20231122093944.png]]

Name Servers: Servers that respond to user's request are called name servers.
Resource Records (RR): DNS database maintains domain name to IP mapping in form of resource records (RR). 
Caching: DNS uses caching at different layers to reduce request latency of the user.

**DNS Hierarchy**

![[Pasted image 20231122094406.png]]

- **DNS resolver:** Resolvers initiate the querying sequence and forward requests to other DNS name servers. 
- **Root Level Name Servers:** Inbound requests from local servers such as DNS resolver. They maintain a mapping of top level domain names to servers such as .io will have a mapping of servers that hold IP data for .io.
- **Top Level Domain (TLD) name servers:** These servers hold the IP address of the authoritative name servers. 
- **Authoritative Name Servers:** These are the organization's DNS name servers that provide the IP address of the web or application servers. 
##### Iterative vs Recursive Query Resolution

![[Pasted image 20231122095014.png]]

**Caching**

![[Pasted image 20231122095201.png]]

**Highly Scalable**
- Roughly 1000 replicas of 13 root-level servers
- Working is divided among TLD and Root Servers and authoritative servers are maintained by the organization
- Every layer has caching 
- Server replication done across the globe
- Protocol: UDP faster than TCP

**Consistent**
- DNS compromises on strong consistency to achieve high availability as reads are way more than writes. So it provides eventual consistency. 
- Since ISP's might cache the mapping, sometimes organizations might change the IP mapping and ISP might have stale data. Thus every record will have a TTL.
- Low TTL is preferred so that users are not pinging machines that might be down


