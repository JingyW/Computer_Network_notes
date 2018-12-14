# Chapter 4 Network Layer
it lies in each and every host and router in the network.

### IP(Internet-Protocol)
- provides best-effort delivery service, meaning no guarantees(delivery, integrity, orderly delivery); therefore, it is unreliable.
- every host has at least one network-layer address, a so-called IP address.

### diff: forwarding vs routing
- forwarding within each router, which input to which output.
- routing within the network, which router to which router.
- routing takes more times(typically seconds).

### Forwarding Table
- lies within each and every router.
- use datagram header to index into the forwarding table.
- determined by routing algorithms

### Service Model(p355)
- possible services:
  * guaranteed delivery
  * delivery with bounded delay
  * in order
  * minimal bandwidth
  * security


### Data Plane
- for forwarding(per-router functions)
- traditional IP forwarding(destination-based forwarding):
  * only the destination matters
- generalized forwarding(SDN)
  * a number of factors can be involved
- implemented using hardware, timescale(nanoseconds)
- forwarding table is copied from the routing processors to the input ports to make the forwarding decision locally
- the router uses the longest prefix matching rule! Looking at the longest matching prefix.


### Inside a router
- Input ports:
  * terminate an incoming physical link at a router.
  * interoperate with the link layer at the other side(link-layer protocol)
  * a lookup function and queuing
  * HOL(head of the line) blocking
- Switching fabrics:
  * between input and output ports. Completely contained within the router.
  * via memory:
    + two cannot be forwarded at the same time.
    + less than B/2 throughput(if B is the max # of packets can be written/read to/from the memory).
  * via bus:
    + no intervention of the routing processor, have a label to the packet indicating the local output port.
    + bus contention: switching speed limited by bus bandwidth, since only one packet can go at one time(every packet goes through a single bus).
  * via interconnection network(crossbar):
    + 2N buses(N input to N output)
    + non-blocking(multiple packets if no intersection)
- Output ports:
  * stores packets from switching fabrics and performs necessary link-layer and physical-layer functions.
  * needs buffer as well since it is possible to have transmission rate slower than datagrams arriving rate. where packet loss happens.
- Routing processors:
  * performs control-plane functions:
    + sdn: communicates with the remote controller
    + traditional: executes routing protocols, maintain routing table, attach link state information, computes forwarding table inside the router.

### Control Plane
- two approaches:
  * traditional(implemented in routers)
  * software-defined networking SDN (implemented in (remote) servers)
- Traditional:
  * Routing algorithm in each and every router interact in the control plane, and they determine the routing algorithm.
- SDN:
  * a physically separated remote controller computes and distributes the forwarding tables.
- typically implemented using software, timescale(ms/s)


### Switches vs routers(p82)
- link-layer switches; network-layer routers.
- both are packet switches
- routers can implement IP protocol; switches can't, therfore it can't recognize IP address.

### IPv4:
- IPv4:
  + 32 bits. checksum will be recomputed and stored again since ttl field will change.
  + IP header = 20B + TCP header = 40B along with the application-layer message
  + MTU(max transmission unit)
  + fragment when output link MTU is smaller than the input link datagram size
  + reassemble according to IP header bits for correct order.
- IPv4 Addressing:
  + the boundary between the host and the physical link is interface, and an IP address is related with the interface rather than the router.
  + subnet -- interconnecting one router interface and hosts interfaces.
    * can physically reach each other without intervening router
    * high order bits: subnet part
    * low order bits: network part
  + Classful Addressing:
    * 8-, 16-, 24-, class a, b, c, d different class can hold different sizes of ip addresses(hosts). Waste ip address
  + Classful vs Classless Trade-off:
    * easy to identify just by looking at the IP Address
    * Simper to build fast routers
    * Can support needs of different size networks
    * However, no mid-size subnet; waste of IP addresses
  + fix for classful:
    * short term -- CIDR(divide addresses on any bit boundary)
    * long term -- IPv6
  + broadcast address inside the each subnet: 255.255.255.255
  + network gets subnet part of the IP address by getting allocated portion of its provider ISP’s address space
  + Hierarchical Addressing(route aggregation) allows efficient advertisement of routing information; for the hosts not in the router subnet range, just announce a specific route, instead of having to move to the "right" subnet, which is costly.

### NAT(network address translation)
- Help sidestep IPv4 exhaustion; when all addresses within a subnet are used, and a new client comes in.
- all datagrams leaving local network have same single source NAT, different port numbers.
- range of addresses not needed from ISP: just one IP address for all devices within that subnet.
- can change addresses and/or devices in local network without notifying outside world.
- more secure since others don't need to know the actual IP address of that host.
- can change ISP without changing addresses of devices in local network
- Implementation:
  * replace source ip and port number for both outgoing and incoming
  * have a NAT translation table
- NAT is controversial:
  * Breaks global connectivity: not everyone can address everyone
  * Hard to run P2P apps
- NAT set up for TCP/ implicitly connection for UDP

### Host address:(DHCP)
- Can be configured manually or more typically, get allocated using the Dynamic Host Configuration Protocol(DHCP). Network administrator can configure so that one host gets one IP address constantly, or it gets a temporary one each time it connects to the network.
- DHCP allows the host to know more information: subnet mask, first-hop, local DNS server.
- DHCP( Dynamic Host Configuration Protocol) relay agent: router/ client: host/ DHCP server
- a new client arrives, and asks for new IP address
  * DHCP discovery message: inside a UDP packet, port number 67(dest port), 68(source port) within a IP datagram. Broadcast 255.255.255.255(dest ip) to everyone inside the subnet using 0.0.0.0(source ip).
  * DHCP offer message: DHCP server response, broadcast 255.255.255.255, containing lease-time/subnet mask/DNS server(IP and name)...
  * DHCP request: client pick one address and respond to its selected offer with the request message(to confirm and let everyone knows); 0.0.0.0(src ip) 255.255.255.255(dest ip)
  * DHCP ACK: the server respond the client, "you can use it now."
- Pros:
  * can renew its lease on address in use
  * dynamic allocation
  * allows reuse of addresses
  * support for mobile users who want to join network(short period of usage)
- Cons:
  * TCP connection cannot be consistent since the IP address is constantly changing from subnet to subnet

### IPv6
- changes in segment format:
  * traffic class: identify priority among datagrams in flow
  * flow Label: identify datagrams in same “flow.”
  * next header: identify upper layer protocol for data
  * A streamlined 40-byte header
- no checksum to reduce processing time at each hop
- `options` field moves to `next header`
- ICMPv6 -- additional message types
- From IPv4 to IPv6: tunneling: IPv6 datagram carried as payload in IPv4 datagram among IPv4 routers
  * protocol number:41, indicating the payload is IPv6 datagram
  * change the source ip and destination ip for the IPv4


### Generalized Forwarding && SDN
- Each router contains a flow table that is computed and distributed by a logically centralized routing controller.
- A `match-plus-action table` computed, updated, installed by a remote controller.
- OpenFlow: a successful standard for match-plus-action
  * header fields: entry
  * counters: the number of packets went through already and the time last time updated
  * action: what to do when match found
    + Forwarding: to one/more output ports
    + Dropping
    + Modifying fields
  * priority:	disambiguate overlapping	patterns
  * not all headers can be matched, so no over-burdening the abstraction.
- OpenFlow allows us to do match on three-layer headers(including transport layer, which principally should not be touched through router). total 12 values in book, recently 41.
  * ingress port: input port
  *
