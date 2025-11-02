#### Open Systems Interconnection (OSI Model)

Seven layers:
- Layer 7 - Application - HTTP/GRPC/WebRTC ------ Load Balancer
- Layer 6 - Presentation - Serialization, Encoding
- Layer 5 - Session - Connection establishment, TLS
- Layer 4 - Transport - TCP/UDP - Segments/Datagram ---- Firewall / Proxies
- Layer 3 - Network - IP - Packets ----- Routers
- Layer 2 - Data Link (MAC Addresses) - Frames ----- Switch
- Layer 1 - Physical layer such as electric signals

**TCP/IP Model Four layers:**
	- Application (Layer 5, 6, 7)
	- Transport (Layer 4)
	- Internet (Layer 3)
	- Data Link (Layer 2)
	
#### Host to Host Communication

Scenario: A needs to send a message to B
Assumptions:
- Each host will have a network interface card (NIC) that contains a unique Media Access Control (MAC) address. Example: 00:00:5e:00:53:af
Solution: 
- Broadcast the message with destination MAC address
- Every host will check if the message MAC address == MY MAC address. If it is true, accept the message.
Disadvantages:
- Millions of machines
- We need a better approach called *IP address*

**IP Address**
- Contains two parts: one part to identify network and host part to identify the host
- We still need MAC address at the lowest level

But, Address is not enough. We might have different applications running on the same host. **Ports solve the problem such as 80, 443, 22, 25.** 
#### IP Address

##### Network vs Host

- a.b.c.d/x, x is the network bits and remaining are host Example: 192.168.254.0/24
- The first three bytes (24 bits) are network and 1 byte (8 bits) are host group called subnets
##### Subnet Mask

- 192.168.254.0/24 is also called a subnet. Subnet has a mask 255.255.255.0
- Subnet mask is used to determine if the IP is in the same subnet
##### Default Gateway

- Most network consists of hosts and default gateway
- When host A wants to talk to host B, directly if they are part of the same subnet
- Otherwise it will send it to the default gateway
- The Gateway has an IP address too and every host should know the IP address of the gateway
##### Host 192.168.1.3 wants to talk to 192.168.1.2

255.255.255.0 & 192.168.1.3 = 192.168.1.0
255.255.255.0 & 192.168.1.2 = 192.168.1.0

Both are the same after applying bitwise & operation. So they will be in the same subnet and can talk to each other using MAC addresses.
##### Host 192.168.1.3 wants to talk to 192.168.2.2

255.255.255.0 & 192.168.1.3 = 192.168.1.0
255.255.255.0 & 192.168.2.2 = 192.168.2.0

Not the same subnet. The packet is then sent to default gateway at 192.168.1.100. 
#### IP Packet

- IP packet has headers and data sections
- Header = 20 Bytes (can go up to 60 bytes if options are enabled)
- Data = 65536 Bytes (64 KB)
##### Actual IP Packet

![[Screen Shot 2023-11-23 at 2.36.04 PM.png]]

![[Pasted image 20231123144628.png]]
#### Internet Control Message Protocol (ICMP)

- Designed for informational messages
	- Host unreachable, port unreachable, fragmentation needed
	- Packet expired (infinite loop in routers)
- Uses IP directly
- Since it is layer 3, does not require ports to be opened

**![[Pasted image 20231123152709.png]]

Some firewalls block ICMP for security reasons (need to understand why??). Disabling ICMP can cause real damage with connection establishment in case of fragmentation needed as the client will never realize why the packet is not being delivered. Also, called as TCP blackhole
#### Address Resolution Protocol (ARP)

- We need MAC address to send frames (layer 2)
- Many times we will know IP addresses because of DNS but not MAC addresses
- ARP tables is cached IP -> MAC mapping

Example 1: IP 10.0.0.2 wants to connect to 10.0.0.5 
- Host 2 checks if Host 5 is within the same subnet
- Host 2 checks the MAC address of Host 5
- Host 2 checks ARP table to understand if it has MAC address for Host 5
- Host 2 sends an ARP request broadcast to all machines in its network (Who has address 10.0.0.5?)
- Host 5 will reply with its MAC address
- Host 2 updates its ARP table

Example 2: Host 10.0.0.2 wants to connect to IP 1.2.3.4
- Host 2 checks if 1.2.3.4 is within its subnet, it is NOT
- Host 2 needs to talk to the gateway
- Gateway address will always be present with the machine
- Gateway will send its MAC address



#### User Datagram Protocol (UDP)

- Ability to address processes in a host using ports
- Prior communication is not required
- Stateless
- 8 byte header Datagram
##### Multiplexing and Demultiplexing

#### UDP Datagram

- UDP header is 8 bytes only
- Datagram slides into an IP packet as "data" with Protocol="UDP"
- Ports are 16 bit (0 to 65535 ports)

![[Pasted image 20231123172455.png]]
#### UDP Pros and Cons

##### Pros
- Simple protocol
- Header size is small so datagrams are small
- Uses less bandwidth
- Stateless
- Consumes less memory (no state stored in the server/client)
- Low latency - no handshake, order, retransmission or guaranteed delivery
##### Cons
- No acknowledgment
- No flow control
- No congestion control
- No guaranteed delivery
- Connection less
- No ordered packets
- Security - can be easily spoofed since no connection was established and the server has to process everything
#### Transmission Control Protocol

- Controls transmission
-  Connection based
- Requires handshake
- 20 byte headers (can go to 60 - similar to IP)
- Stateful
##### TCP Connection

- Connection is Layer 5 (session)
- Must create a connection to send data
- Connection is identified by 4 properties:
	- Source IP - Source Port
	- Destination IP - Destination Port
- Can't send data outside a connection
- Requires a 3 way handshake
- Segments are sequenced and ordered
- Segments are acknowledged
- Lost segments are retransmitted

![[Pasted image 20231123210103.png]]

![[Pasted image 20231123210240.png]]

SYN starts with an Initial Sequence Number (ISN). 
#### Flow Control

- TCP Packets arriving on receiver are stored in buffer that can get overwhelmed. Window size can be used to inform the client - 16 bit
- Sliding Window maintained by the client for sending packets to the receiver
- Window Scaling Factor can have value between (0-14). Window size = 2^16-1 * 2^14

