# Runet

The purpose of creating Runet is to learn the modern networking stack by building a fully functional web server with relatively low level programming interfaces. It documents the fundamental concepts in networking to introduce the core domain knowledge, and it demonstrates how network programming is done in practice. Rust is the chosen language for this project, but the focus is on implementing networking features rather than introducing Rust. Basic understanding of computing systems, OS, programming, and networking is assumed. All development is based off a Linux environment.

## Networking Fundamentals

### IP Routing
IP routing describes the process of forwarding network packets for the given IP addresses. IP addresses are typically written in CIDR notation (e.g. `192.168.122.5/27` for IPv4) where the prefix length (after `/`) indicates which bits form the network address, and the rest are the host bits. The network address itself (all host bits set to 0) cannot be a valid host address. Also, each network reserves a broadcast address used to send messages to all hosts, and this is computed by setting all host bits to 1.

There are broadly two classes of IP addresses; the ones that can be routed on the public Internet are called public IP addresses, and some other blocks that are meant to be used in private networks only are called private addresses (e.g. `10.0.0.0/8`). If a router on the Internet receives a packet destined to a private address, the packet will be dropped.

For each packet a router receives, it first gets the (destination) network prefix for the packet, and then it consults its internal routing table to decide whether the packet should be sent to one of its outgoing interfaces or dropped. The routing table can contain a *default route* where packets with unlearned destinations will be forwarded to. Each packet also has a Time to Live (TTL, hop count based, usually starts with 64) which will be decremented by routers whenever forwarded to prevent routing loops (packets are dropped when TTL reaches 0).

The routing table can be configured manually, but more commonly, the vast majority of the routes on the Internet are learned by routers running different routing protocols with their peers to share information. Routing protocols have two major types; the interior gateway protocols are used for routing within an Autonomous System (AS), and the exterior gateway protocols (e.g. BGP, the Border Gateway Protocol) are designed for routing between ASs.

### DNS
The Domain Name System is a fundamental mechanism for servers to discover IP addresses (from a human-readable domain name, e.g. `www.google.com`). An application typically invokes the system call `getaddrinfo` for address resolution. Each networking server will usually have a local DNS server configured in `/etc/resolv.conf` (first priority lookup), but for the vast majority of Internet domains, this file will point to another DNS server (e.g. ISP's DNS) which acts as a proxy.

Again, the next proxy DNS server may not be an authoritative server for the domain which means it will need to forward the request to a different server with better knowledge. There is a well-known list of root name servers maintained by ICANN, and these servers will have answers for the Top Level Domain (TLD) servers (e.g. `.com`). Once the proxy server learns the address of the TLD server, it will query that server to resolve the rest of the domain. In this example, the TLD server probably returns the server that knows about `google.com`, and the full resolution continues until the entire domain is resolved.

Eventually, the request reaches an authoritative server at Google, and it will respond with the IP address(es) and mark the records as *authoritative*. Once the local DNS server receives the answer, it sends the message to the OS which delivers to the application. Each DNS answer is cached and has a TTL (time based) indicating when the record will expire (after which point the resolution process occurs again if the domain is requested).

### TCP
The Internet Transmission Control Protocol is a connection-oriented service protocol that provides a *reliable* mechanism for network hosts to send and receive data. Before actual data transmission, the parties involved negotiates a virtual connection which is defined by a set of parameters (e.g. source and destination ports) that must be agreed upon. These parameters are included in the header section of the Protocol Data Unit (PDU) for TCP, commonly known as a *segment*.

The two parties that want to establish a connection will perform a three-way handshake (3 segments to be sent) which will finalize the sequence numbers and acknowledgement numbers to be expected during data exchange. These numbers are used to enforce the segment ordering and identify duplicate segments. TCP also allows parties to fine tune the rate of sending data, and it uses checksums to detect data corruption so that these segments can be retransmitted to maintain data integrity.

### UDP
The User Datagram Protocol is a connection-less service protocol which facilitates the transmission of messages that bear no defined relation to each other. Hosts communicating via UDP do not negotiate connections, and the ordering and reliability are not guaranteed. However, UDP segments still include checksums to ensure the correctness of messages.

UDP is much more lightweight and faster than TCP, and it is typically used in services where data reliability is not desired or when the overhead of connection maintenance too significant to satisfy the performance requirements. Video streaming is one such example. Protocols that run on top of UDP can also implement reliability if needed. IP routing and DNS are commonly connection-less services too.
