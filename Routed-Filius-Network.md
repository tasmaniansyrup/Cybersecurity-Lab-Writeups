Skills Demonstrated

TCP/IP networking fundamentals
ARP (Address Resolution Protocol) and MAC address resolution
Subnet identification and bitwise AND operations
Inter-subnet routing and default gateway configuration
OSI model Layer 2 / Layer 3 separation
Network simulation and analysis using Filius

Building a Routed Network with Filius
Course: CS 456 | Date: September 10, 2025
Overview
This project involved constructing a routed network topology in Filius to explore how devices communicate across subnets. The network consisted of two hosts on separate subnets (Host A on 192.168.1.x and Host C on 192.168.2.x), connected via a router, with Host B sharing Host A's subnet.

The Role of ARP
When Host A first pings Host B, Host A broadcasts an ARP request across the entire subnet asking which device holds Host B's IP address. Both Host B and the router's interface on that subnet receive the broadcast. Host B replies with its MAC address, and communication proceeds.
When Host A first pings Host C, the process differs because Host C exists on a separate subnet. Host A performs a bitwise AND of its own IP and subnet mask, then does the same with the destination IP, and determines that 192.168.1.x and 192.168.2.x are on different networks. Rather than ARPing for Host C directly, Host A sends an ARP request for its default gateway — the router interface on its local subnet. Once the router receives the packet, it issues its own ARP request on the 192.168.2.x subnet to resolve Host C's MAC address and forwards the packet accordingly.

The Default Gateway
Would a ping from Host A to Host B succeed without a configured default gateway?
Yes. Host B is on the same subnet as Host A, so no routing is required. The ARP request is sent as a local broadcast and Host B receives it directly. The default gateway is never consulted.
Would a ping from Host A to Host C succeed without a configured default gateway?
No. Host C lives on a different subnet, and Host A must forward the packet to its router to reach it. Without a default gateway configured, Host A has no way to identify which router interface to send the packet to, and the ping fails.

Packet Journey
When Host A sends a ping to Host C, the packet passes through both a switch and a router:

The Switch forwards the frame using the destination MAC address. It looks up this address in its MAC table and sends the frame out the appropriate port toward the router.
The Router forwards the packet using the destination IP address. It consults its routing table to determine which interface leads to the 192.168.2.x subnet and forwards the packet accordingly.


MAC Address Awareness
Host A's networking stack never learns Host C's MAC address. MAC addresses are only relevant within a single subnet — once traffic crosses a router boundary, the Layer 2 frame is stripped and rebuilt. Host A only ever ARPs for its default gateway's MAC address. The router handles all Layer 2 resolution on the destination subnet independently. This is a fundamental property of how the OSI model separates Layer 2 (Data Link) and Layer 3 (Network) responsibilities.

Screenshots
Network topology in Filius showing Host A (192.168.1.x), Host B (192.168.1.x), Host C (192.168.2.x), and the connecting router.
Successful ping from Host A to Host C, confirming correct routing and ARP resolution across subnets.
