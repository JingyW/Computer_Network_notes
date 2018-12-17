# Chapter 5 Network Layer: Control Plane

### Two types of control plane
- Traditional(routing algorithm in each and every router)
  * Each router has a routing component to communicate with other routers
  * `OSPF` `BGP`
- Logically centralized controller(SDN, remote)
  * generalized match-plus-action
  * no direct interaction between two local control agents(CA)

### Routing Algorithm(slide 20)
- least cost path !=  shortest path
- A global routing algorithm is not necessarily centralized(LS is, OSPF not)
- Classification:
  * Centralized routing algorithm(Global information needed), link-state algorithm
  * Decentralized routing algorithm
    + calculation is carried out in an iterative, distributed manner by all the routers. DV(distance vector) algorithm, BGP.
  * Static: routes change slowly over time
  * Dynamic: change periodically or in response to the link cost changes.
- DV(Distance vector algorithm):
  * Bell Ford algorithm
- LS(Link state algorithm):
  * each node broadcasts to all other nodes in the network its link-state packets, containing the identities and costs of its attached links.
  * all nodes have an identical view of the network.
  * `Dijkstra's Algorithm` computed within each router respectively.
- Dijkstra's Algorithm:
  * construct shortest path tree by tracing predecessor nodes.
  * ties can exist (can be broken arbitrarily)
  * O(n^2) comparison, can be done in O(nlogn) if using heap
  * oscillations possible
    + Suppose link cost equals amount of carried traffic
    + So cost of a link may differ in the two directions
- Bell-Ford Algorithm:
  * Dynamic Programming: `dx(y) = min {c(x,v) + dv(y) }`
  * each router x maintains a set of numbers: `Dx = [Dx(y): y є N ]`
  * each node knows cost to each neighbor v + maintains its neighbors’ distance vectors `Dv = [Dv(y): y є N ]`
  * only neighbors share their distance vector information with each other periodically.
  * local iteration triggered by neighbor advertisements or local link cost changes.
  * distributive: each node notifies neighbors only when its DV changes(only when necessary).
  * "good news travels fast"
  * "count to infinity problem!"
    + possible solution: poisoned reverse. If z routes through y to get to destination x, then z will advertise to y that its distance to x is infinity, z will continue telling this little white lie to y as long as it routes to x via y.
    + poisoned reverse will not solve "count to infinity problem", involving three or more nodes.
- Diff vs LS and DV algorithm:
  * message complexity:
    + LS: with n nodes, E links, O(nE) msgs sent
    + DV: exchange between neighbors only
  * speed of convergence:
    + LS: O(n^2) algorithm, may have oscillations
    + DV: convergence time varies; routing loops and count-to-infinity problems
  * robustness: think about what propagates through(LS: wrong cost/DV: wrong path)

### AS(Administrative System):
- Two problems can be solved by administrative system(AS):
  + Scale: routers are way too many. can’t store all destinations in routing tables, and the exchange messages would swamp the links
  + Administrative Autonomy(AS):
    * the Internet is a network of ISPs, each ISP consisting of its own network of routers, and different policies would be wanted by each ISP.
- Administrative System(AS):
  * organizing routers into ASs, each AS consisting of a group of routers that are under the same administrative control.
  * one ISP can consist of one or more Ass.
  * each AS has a unique AS number(ASN).


### Intra-AS vs Inter-AS:
- Intra-AS for routers inside the same AS. All routers must have the same Intra-domain protocol.
- Inter-AS: gateways perform inter-domain routing (as well as intra-domain routing)
- gateway router: at “edge” of an own AS, has link(s) to router(s) in other ASs.
- forwarding table configured by both intra- and inter-AS routing algorithm

### OSPF(Open Shortest Path First)
- Intra-AS(Intra-autonomous System)
- one of LS, using Dijkstra's algorithm inside each local router.
- Set OSPF link weights
- Periodically broadcasts weights or broadcasts when there's change.
- Checks whether all links are operational by sending a hello message to its neighbors
- OSPF forms IP datagrams directly without transport protocol
- Embodied advances:
  * Security: exchange between two OSPF routers can be authenticated.
  * Multiple same-cost paths.
  * Support for hierarchy within a single AS(two-level hierarchy)
    + local area, backbone.
    + area border routers(to other areas within the as)/ backbone routers(run OSPF routing limited to backbone)/ boundary routers(to other as)

### BGP(Border Gateway Protocol):
- Inter-AS(Inter-autonomous System) routing protocol
- decentralized and asynchronous protocol in the vein of DV routing
- Packets are routed to CIDRized prefix instead of a specific IP address.
- Provides each router the ability to:
  * Obtain prefix reachability information from neighboring ASs, allow advertise
  * Determine the “best” routes to the prefixes by reachability and configured BGP policies.
- Each router is either a gateway router or an internal router.
- Pairs of routers exchange routing information over semi-permanent TCP connections using port 179.
- Each TCP connection for advertising the information(BGP messages) is called  `BGP connection`
- eBGP: a BGP connection that spans two ASs/ iBGP: a BGP connection between two routers within the same AS.
  * iBGP is usually a full-mesh, meaning that it's not necessary that physical links exist between two routers.
- BGP attributes along with any BGP connection when advertising a prefix(two important ones):
  * AS-PATH: the AS list that the advertisement has passed through. Prevent looping advertisements.
  * NEXT-HOP: IP address of the router interface that begins the AS-PATH, it must be reachable without using BGP, e.g., it can use OSPF or they are directly connected.
    + there can be two versions of next-hop.
- Policy-based routing:
  * Gateway receiving route advertisement uses import policy to accept/decline path.
  * route selection policy to pick best path when multiple exist.
  * export policy determines which (if any) neighboring ASes to advertise best path to.
  * No Valley:
    + Don’t transit traffic for provider/peer to another provider/peer. Prevent from taking care of the traffic that nobody's paying for.
  * Prefer-Customer:
    + LOCAL-PREF: affects only within the AS
  * BGP Community:
    + optional labels that can be attached to any path(in&out), can be used to inform local-preference
    + 32 bit number
- Hot Potato Routing Algorithm:
  * choose the route that has the least cost to the NEXT-HOP router.
- Summary of ways to pick the best route in practice:
  * look at the local-pref
  * shortest AS-path
  * closest NEXT-HOP
  * BGP identifier

### SDN:
- Why a logically centralized control plane:
  * easier network management
  * table-based forwarding allows “programming” routers(OpenFlow API)
  * fast evolvement as a software, since you can have programmable control applications at the control plane now.
  * open implementation of control-plane.
- Four key characteristics:
  * Flow-based forwarding
  * Separation of data plane and control plane
  * Network control functions external to data switches:
    + controller + applications. controller for holding the network states, and passing these information to the applications for deciding.
  * A programmable network: through the applications.
- SDN helps to unbundle the network functionality.
- SDN controller, three-layer functionality:
  * A communication layer
  * A network-wide state-management layer: a distributed database
  * The interface to the network-control applications layer: abstractions API
- SDN is implemented as distributed system for performance, scalability, fault-tolerance, robustness

### Openflow API:
- For router, it matches the longest IP address prefix and perform the action.
- For switches, it matches the longest MAC address and perform the action
- For firewalls, it matches	IP	addresses	and	TCP/UDP	port numbers, either permit or deny.
- For NAT, it matches the IP address, port, and rewrite the address and port.
- Use TCP to exchange messages, so there can be optional encryption

### ODL(Open Daylight Controller)
- Network services applications: determine how data-plane forwarding and other services.
- Basic Network-services functions: at the heart of the controller
- Include OpenFlow and SNMP(Simple Network Management Protocol)

### ICMP(The Internet Control Message Protocol)
- used by hosts and routers to communicate network-layer information.
- error reporting is one of the most common usages.
- considered as part of an IP, but in fact, lies above it, inside the IP datagrams
- a type and a code field, a header and the first 8 bytes of the IP datagram, so that the sender can determine which datagram caused the error.
- PING program uses it. ICMP type 8 code 0 message to the specified host, dest host sends back a type 0 code.
- Traceroute uses ICMP as well:
  * Sends UDP included in an IP datagram, with unlikely UDP port number.
  * When the nth datagram arrives at the nth router, the nth router observes that the TTL of the datagram has just expired, the router discards the datagram and sends an ICMP warning message back to the source, which includes the destination name and IP.
  * When host unreachable gets passed back, the destination is reached.

### SNMP
- for network management
- key components:
  * managing server: application, running in NOC(network operations center), actions to control network are initiated, human network administrators are involved.
  * managed device: network equipment, lies within a managed network.
  * MIB(Management Information Base): where the pieces of information collected from the managed devices are stored. Information are visible to managing server.
  * network management agent: middle guy between managed device and managing server.
- two ways to convey MIB info, commands:
  * request/response mode: request made by the server, responded by the agent
  * trap mode: sent directly within a trap message from agent to manager: inform manager of exceptional event
