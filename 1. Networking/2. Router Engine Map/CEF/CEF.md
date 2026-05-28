CEF = **Cisco Express Forwarding**.

It is basically the router/switch’s **fast L3 forwarding engine**. The routing table decides _where traffic should go_. CEF is the optimized forwarding table the device actually uses to move packets quickly.

Think of it like this:

```text
Routing protocol / static route
        ↓
RIB: show ip route
        ↓
CEF/FIB: show ip cef
        ↓
Adjacency table: show adjacency
        ↓
Packet gets rewritten and forwarded out an interface
```

### The simple breakdown

|Piece|What it means|Command|
|---|---|---|
|**RIB**|Routing table / control-plane truth|`show ip route`|
|**FIB**|CEF forwarding table built from the RIB|`show ip cef`|
|**Adjacency table**|L2 rewrite info: next-hop MAC, outgoing interface, ARP/ND info|`show adjacency` / `show ip arp`|

CEF uses a prebuilt **FIB + adjacency table** so the router does not have to “think hard” for every packet. Your mechanism table describes CEF as hardware-accelerated L3 forwarding that uses a prebuilt FIB and adjacency table instead of slow process switching. It also notes that CEF depends on the routing table and ARP/ND adjacency information.

### Why you care during troubleshooting

`show ip route` answers:

> “Does the router know a route?”

`show ip cef` answers:

> “Is that route actually programmed for forwarding?”

So in your troubleshooting table:

```text
show ip route        = control-plane/RIB check
show ip cef          = data-plane/FIB check
show ip arp          = L2 next-hop resolution check
show adjacency       = L2 rewrite / forwarding adjacency check
```

Example failure chain:

```text
show ip route 10.10.50.4
% Subnet not in table

show ip cef 10.10.50.4
0.0.0.0/0
no route
```

That means the router has no usable route, so CEF also has nothing useful to forward with. One troubleshooting source shows exactly that pattern: after the missing route is added, forwarding works.

### What CEF does with equal-cost paths

CEF also decides which equal-cost path a flow uses. It usually hashes traffic based on fields like source/destination IP and sometimes ports. That is why two different flows to the same subnet may exit different interfaces. Cisco Press shows `show ip cef ... exact-route` being used to reveal which outgoing interface a specific source/destination pair will use.

Example:

```text
show ip cef exact-route 192.168.1.10 10.10.10.50
```

That tells you:

> “For traffic from this source to this destination, what path would CEF pick?”

### The big idea

CEF is **not routing protocol logic**.

CEF is:

```text
The forwarding-plane version of the routing table,
plus the Layer 2 rewrite info needed to actually send the packet.
```

So your table’s mental model should be:

```text
Interface up?        show ip interface brief
IP correct?          show run interface
Route exists?        show ip route
Next-hop resolved?   show ip arp
CEF programmed?      show ip cef
Path works?          ping / traceroute
```

In one sentence: **CEF is where “the route exists” becomes “the box can actually forward the packet at speed.”**