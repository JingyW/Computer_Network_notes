# Chapter 3 Transport Layer
- Application messages -> chunks, add transport layer header -> encapsulated within a datagram(network layer packets)
- routers only work on network-layer fields of the datagram

### diff trans vs network
- network provides logical-communication between hosts, transport provides between application-layer processes
- transport-layer protocol lives within the end systems; routers have nothing to do with it.
- transport-layer services are constrained by the network layer, but some services can be provided by transport-layer even though network layer can't: reliable data transfer.

### services not available:
- delay guarantees
- bandwidth guarantees

### UDP(User Datagram Protocol)
- unreliable, connectionless
- has integrity checking(error detection header)
- that's it! integrity checking and process-to-process delivery are the only two things that UDP provides. no delivery guarantee.
- process-to-process means that if it arrives, the UDP will deliver it to the correct process running on the host
- connectionless because there's no handshaking(connection establishment), and each UDP segment handled independently of others
- real-time delivery(video streaming)/DNS/SNMP uses UDP
- it is fast, no connection establishment, no congestion control, smaller header size(8 vs 20), no connection state.
- UDP can be reliable if more features are added on the application layer. e.g: Google QUIC protocol
- four headers: each takes two bytes. Length fields is header plus data.
- checksum: error detection:
  * one's complement of all 16-bit words sum(flip bits after adding), overflow wrapped around.
  * at sender, all 16-bits including the checksum should be added->result should be all 1s
  * it is necessary because no link-by-link reliability nor in-memory(in router) error detection guarantee.

### TCP(Transmission Control Protocol)
- reliable, connection-oriented
- has integrity checking(error detection header)
- reliable using flow control, timer, acknowledgements, sequence numbers, ensuring delivery, orderly, integrity
- congestion control: prevents swamping the links and routers with TCP connections
- TCP regulates the rate each sending sides can send traffic into the network, so each connection can have the same amount of bandwidth
- Three-way handshake to avoid half open connection situation in two-way handshake.
- Congestion Control:
  * end-to-end rather than network assisted since IP provides no feedback to the end systems regarding the network congestion(traffic situation).
  * solved by limiting the rate sending packets into the network:
  * congestion window{cwnd}, receive-window{rwnd}, min{cwnd, rwnd}
  * loss events(three dup/timeout) are hints of network congestion
  * self-clocking: TCP uses acknowledgments to trigger (or clock) its increase in congestion window size

### Multiplexing and Demultiplexing
- extending host-to-host delivery to process-to-process.
- has integrity checking(error detection header)
- deliver to an intermediary socket instead of the process directly; each socket has a unique identifier depending on the transport protocol(UDP/TCP). First of all, port numbers, a 16 bit number!
  * HTTP application process:80; FTP: 21
- The receiving host examines the headers to decide which socket to deliver to. Deliver to the right socket is called Demultiplexing.
- UDP is fully identifiable by a two-tuple: destination IP and destination port number
- TCP is slightly different since it provides connection-oriented service, four-tuple: destination IP, destination port number, source IP and source port number.
  * different source IP/port will be directed to a different socket
