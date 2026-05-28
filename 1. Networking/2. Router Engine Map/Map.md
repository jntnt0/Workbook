Not a fixed number. **IOS XE routers are a stack of planes + feature engines**, and the exact engines depend on platform and enabled features.

For troubleshooting, think of **CEF as one forwarding engine/checkpoint**, not the whole router.

## Big IOS XE router “engine” map

|Area|What it does|Common checks|
|---|---|---|
|**RIB / Routing engine**|Builds the routing table from connected routes, statics, OSPF, EIGRP, BGP, etc.|`show ip route`, `show ip protocols`, `show bgp`, `show ip ospf neighbor`|
|**CEF / FIB engine**|Turns the RIB into the fast forwarding table.|`show ip cef`, `show ip cef detail`|
|**Adjacency engine**|Holds Layer 2 rewrite info: next-hop MAC, outgoing interface, ARP/ND result.|`show adjacency`, `show ip arp`, `show ipv6 neighbors`|
|**QFP / Datapath engine**|Platform forwarding processor on many IOS XE routers; handles packet processing, drops, services, IPsec, SD-WAN, etc.|`show platform hardware qfp active statistics drop`|
|**ACL / Policy engine**|Filters or classifies traffic.|`show access-lists`, `show ip access-lists`|
|**NAT engine**|Translates addresses/ports.|`show ip nat translations`, `show ip nat statistics`|
|**QoS engine**|Classifies, marks, queues, polices, or shapes traffic.|`show policy-map interface`|
|**Crypto / IPsec engine**|Encrypts/decrypts tunnel traffic.|`show crypto ipsec sa`, QFP IPsec drop commands|
|**SD-WAN engine**|Handles overlay control, tunnels, BFD, app-aware routing, policies.|`show sdwan control connections`, `show sdwan bfd sessions`|
|**Management / programmability engine**|NETCONF, RESTCONF, telemetry, APIs, automation, logging.|`show telemetry`, `show netconf-yang sessions`, logs|
|**Control-plane protection engine**|Protects the router CPU from punted traffic.|`show policy-map control-plane`|
|**Licensing / trust / secure boot services**|Smart licensing, SUDI, platform integrity, entitlement.|`show license summary`, `show platform`|

Cisco’s own platform material shows this split pretty clearly: Catalyst 8500 platforms have a **Quantum Flow Processor/QFP** for high-performance routing, IPsec, MACsec, SD-WAN services, plus a separate multicore x86 control processor for control-plane processes.

Cisco also describes IOS XE on Catalyst 8000/8500 as a feature-rich routing stack including **IP routing, IPsec, QoS, firewall, NAT, NBAR, Flexible NetFlow**, and more, which is basically the “feature engine” layer on top of the base forwarding architecture.

## The core packet path

For normal unicast forwarding, the mental model is:

```text
Interface receives packet
        ↓
Input features: ACL / QoS / NAT / firewall / SD-WAN / crypto if relevant
        ↓
Routing decision from RIB → CEF/FIB
        ↓
Adjacency lookup: next-hop MAC / rewrite info
        ↓
Output features: QoS / NAT / crypto / tunnel encapsulation
        ↓
Packet leaves interface
```

So for your table, the **must-know engines** are probably these five:

```text
RIB         = show ip route
CEF/FIB     = show ip cef
Adjacency   = show adjacency / show ip arp
QFP         = show platform hardware qfp active statistics drop
Policy      = ACL / NAT / QoS / crypto / SD-WAN checks
```

The important distinction:

```text
RIB says:       “I know the route.”
CEF says:       “I can forward it fast.”
Adjacency says: “I know how to rewrite the frame.”
QFP says:       “The platform datapath is actually processing or dropping it.”
Policy says:    “A feature may be allowing, modifying, or killing it.”
```

Cisco troubleshooting docs for SD-WAN/IOS XE explicitly collect **routing table**, **interface drops**, **QFP hardware drops**, **IPsec datapath drops**, **SD-WAN datapath stats**, **BFD datapath stats**, and packet-trace outputs when diagnosing application reachability. That is the practical proof that CEF is just one checkpoint inside a larger forwarding/datapath system.