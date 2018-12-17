# Chapter 6 Link Layer

### Link Layer Context:
- transportation mode = link layer protocol
- data-link layer has responsibility of transferring datagram from one node to `physically adjacent` node over a link.
- frame encapsulates datagrams:
  * data field, containing network layer datagram
  * header fields specified by the link-layer protocol
- `MAC` addresses used in frame headers to identify source, destination, which is different from IP address!
- functionalities:
  * flow control
  * error detection/error correction(without retransmission)(pp500)
    + more sophisticated than transport/network layer checksum mechanism.
    + not always get detected
  * half-duplex and full-duplex
- MAC protocol(Medium Access Control):
  * coordinate the frame transmissions of the many nodes.
- Reliability:
  * guarantees to move the network layer datagrams to the destination without error.
  * achieved by acknowledgements and retransmissions
  * used for links that are prone to high error rates(WIFI), so an error can be corrected locally(on the link).
  * many wired links don't provide such service, since it has low bit-error, the check is overhead/unnecessary.
- Implemented in each and every host.
  * For hosts, in a network adapter, a Network Interface Card(NIC), inside there's a link-layer controller.
    + attaches into host’s system buses, it is a combination of software(CPU), hardware(adapter), and firmware.
    + CPU has from link layer up to application layer.
    + network adapter has physical layer and link layer.
  * For routers, in a router's line card.

### Multiple Access links and protocols:
- Two types of network links:
  * point-to-point: PPP(point-to-point Protocol), HDLC(high-level data link protocol)
  * broadcast: old-fashioned ethernet, shared wire or medium(LANs)
- Multiple Access Problem: how to coordinate the access of multiple sending and receiving nodes to a shared broadcast channel:
  * Multiple Access Protocol: nodes regulate their accesses to the shared broadcast channels.
  * distributed algorithm/ communication regarding the channel must use channel itself, no out-of-band channel involved/ decentralized/ simple(inexpensive)/
  * Two desirable properties: when one node is active -- R bit rate/ R/M bit rate when M nodes are sending data averagely
  * Three categories of all multiple access protocols.
- Channel Partitioning Protocols:
  * divide channel into smaller “pieces” (time slots, frequency, code), and each piece for node is exclusive
  * TDMA(Time Division Multiple Access): each station gets fixed length slot (length = packet transmission time) in each round. Unused ones go idle -- waste!
  * FDMA(Frequency Division Multiple Access): each station assigned fixed frequency band. Unused ones go idle -- waste!
  * They are perfectly fair and totally eliminates collision.
  * Limited transmission rate and wasteful slots.
  * CDMA(Code Division Multiple Access): Each node has a unique code, its usage is tightly tied to Wireless Channels.
- Random Access Protocols:
  * Allow collisions and “recover” from collisions.
  * Nodes always sends at Full transmission rate R, a random access MAC protocol specifies how to detect and recover from an error(e.g. via retransmission):
    + Slotted ALOHA, ALOHA, CSMA, CSMA/CD, CSMA/CA
- Taking-turns Protocols:
  * nodes take turns, but nodes with more to send can take longer
turns


### Random Access Protocols:
- Slotted ALOHA:
  * Assumptions:
    * All frames have the same size L bits.
    * L/R(s) time slot
    * Nodes start to transmit frames only at the beginnings of slots.
    * nodes are synchronized so that when one slot is used, all nodes will know.
    * if collision, all the nodes detect the collision event before the slot ends.
  * Operations:
    * probability p: If there's collision, each node in that slot has a probability p to get retransmitted at the next available slot, and the retransmission keeps happening until it is successfully sent.
  * Pros:
    * Full rate(R)
    * Decentralized: nodes detect collision and make decision independently even though the slots are synchronized.
    * Simple
  * Cons:
    * waste slots when colliding
    * idle slots
    * synchronization
    * nodes may be able to detect collision in less than time to transmit packet -> synchronization would not be able to achieve.
  * Efficiency: for a slotted MAC Protocol, efficiency is defined to be  long-run fraction of successful slots under the assumption that many nodes, all with many frames to send
    * N nodes, probability p: N*p*(1-p)^(N-1)
    * at best: channel used for useful transmissions 37% of time!
- ALOHA:
  * Un-slotted, fully decentralized: when a frame arrives, immediately start transmitting into the broadcast channel; when collision happens, immediately start retransmitting with p%
  * Efficiency: half of the slotted ALOHA
- CSMA(Carrier Sense Multiple Access):
  * listen before transmit. Only sends frame when it senses an empty channel, else defer.
  * Collision is still possible(needs time to propagate the usage of channel to all hosts):
    + propagation delay means two nodes may not hear each other’s transmission.
    + If collision: entire packet transmission time wasted.
    + distance & propagation delay play role in in determining collision probability
  * CSMA/CD(Collision detection):
    + detected within a short period of time.
    + colliding transmissions aborted, reducing channel wastage
    + easy in wired LANs: measure signal strengths, compare transmitted, received signals
    + difficult in wireless LANs: received signal strength overwhelmed by local transmission strength
  * Ethernet CSMA/CD Algorithm:
    + NIC gets IP datagram packets from network layer, encapsulates them inside a frame
    + NIC senses idle, start transmitting; otherwise, wait for idle channel
    + If collision detects, aborts and sends jam signal.
    + After aborting, NIC enters binary (exponential) backoff: after mth collision, NIC chooses K at random from {0,1,2, …, 2m-1}. NIC waits K·512 bit times, then start sensing again. More collision turns, more waiting time(backoff interval).
  * Efficiency:
    + tprop = max prop delay between 2 nodes in LAN
    + ttrans = time to transmit max-size frame
    + efficiency = 1/(1+ 5tprop/trans)
    + trans->infinity, tprop->0, efficiency higher(-> 1)
  * Better than ALOHA: efficiency. Also cheap and decentralized.

### "Taking turns" MAC Protocols:
- For the two desirable properties, above protocols only satisfy the first one(R bps when one node is active), but not the second(R/M bps)
- Polling Protocol:
  * One node has to be the master, others are "slaves".
  * Master node tells the slaves when to send and how many frames they can send in a cyclic manner(round-robin manner)
  * Pros:
    + Eliminates collision and empty slots.
  * ConsL
    + polling delay
    + "centralized": if master dies...
- Token-passing protocol:
  * control token passed from one node to next sequentially, with no master device.
  * node holds on the token only when it has frames to transmit. It can transmit up to max number of frames.
  * Pros:
    + highly efficient, decentralized
  * Cons:
    + one node dies...
    + one node fails to pass on the token.
    + therefore, some recovery procedure is necessary.

### DOCSIS(Data-Over-Cable Service Interface Specifications)
- The Link-Layer Protocol for Cable Internet Access: specifies the cable data architecture and protocol.
- Cable Access Network typically connects several thousand residential cable modems to a cable modem termination system (CMTS) at the cable network headend.
- Uses FDM to divide upstream(CMTS to modem) and downstream(modem to CMTS). Both upstream and downstream channels are broadcast ones.
- All modems at the end of the downstream would receive the frames. No Multiple Access Problem since there's only one CMTS transmitting to the downstream channel.
- Collisions are possible for upstream channel:
  * Upstream channel is divided into intervals of time, each containing minislots that the model can use to transmit frame: some slots assigned, some have contention
  * For the slots assignment: downstream MAP frame(control message): assigns upstream slots
  * request for upstream slots (and data) transmitted random access (binary backoff when collision is detected) in selected slots so that CMTS can know which modem needs slot for sending data.
  * The mini-slot requests are transmitted in a random-access fashion, so there might be collision. Modem detects collision or error if it does not receive any response.
- Cable Access network has FDM, TDM, Random access and centrally allocated time slots within this one network.

### Switched Local Area Networks
- MAC address(Link layer addressing)(LAN address/ Physical address):
  * 6 bytes long, designed to be permanent, but can be changed using software.
  * Unique, assigned by IEEE.
  * flat structure, therefore doesn't change based on location; as opposed to IP address, which is hierarchical structure(change as the network changes.)
  * Having MAC address, so datagrams can go through 2-layer switches, which don't recognize any network layer/transport layer identifiers(IP addresses). Additionally, it keeps the layers independent from each other.
  * Therefore, instead of thinking that hosts and routers have MAC addresses, but rather, think as the network adapters(interface) have the MAC addresses.
  * One router with multiple interfaces will have multiple MAC addresses, just like the IP address.
  * link-layer switches do not have link-layer addresses associated with their interfaces that connect to hosts and routers.
    + the job of the switches is to transfer datagram between routers and hosts, and they do the job transparently without them explicitly address the frame(intervening).
  * When sending sides want to broadcast, use 6F MAC address for broadcasting purpose.
- ARP(Address Resolution Protocol):
  * translate between IP and MAC address.
  * Difference between ARP and DNS:
    + DNS solves for hostname and IP address for everyone(in the internet), ARP only within its subnet.
  * Each host and router has an ARP table inside it(IP, MAC, TTL).
  * Process of requesting the MAC address for the destination:
    + sender constructs a special packet called an ARP packet to query all
the other hosts and routers on the subnet to determine the MAC address with broadcast mac: FF-FF-FF-FF-FF-FF.
    + Encapsulates this packet inside a link-layer frame and send it.
    + receiver checks with the ARP module, if matches, send back with the response ARP packet.
    + Sender adds this translation to its ARP table.
  * the query ARP message is sent within a broadcast frame, the response ARP message is sent within a standard frame.
  * ARP is plug-and-play. It gets built automatically.
  * ARP is probably best considered a protocol that straddles the boundary between the link and network layers.
    + it's encapsulated within the frame, but contains link-layer information.
  * Across two subnets, first use the router MAC address(ARP), then using the forwarding table, moving to the correct output interface for the destination port.

### Ethernet
- single chip, multiple speeds.
- first widely used LAN technology.
- Physical Topology: Bus/ Star
  * Bus: all nodes in same collision domain (can collide with each other)
  * Star: active switch in center, no collision
