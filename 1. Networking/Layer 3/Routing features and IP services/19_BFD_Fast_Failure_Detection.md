BFD_Fast_Failure_Detection.md
# BFD_Fast_Failure_Detection

# BFD_Fast_Failure_Detection_Mental_Model

| Concept | Operational Meaning |
|---|---|
| BFD | Bidirectional Forwarding Detection provides fast failure detection between forwarding peers |
| Detection protocol | BFD detects path failure; it does not calculate routes or replace a routing protocol |
| BFD client | A routing protocol or feature such as OSPF, EIGRP, BGP, static tracking, or SD-WAN consumes BFD failure signals |
| Single-hop BFD | BFD session formed between directly connected Layer 3 neighbors |
| Multihop BFD | BFD session formed between non-directly connected peers, commonly used with multihop BGP designs |
| Control packets | BFD peers exchange lightweight packets to prove forwarding-path liveliness |
| Timer interval | How often the local router sends BFD packets |
| Minimum receive interval | Minimum interval the local router expects to receive BFD packets |
| Multiplier | Number of missed BFD packets required before declaring the session down |
| Detection time | Approximate failure detection time is receive interval multiplied by multiplier |
| Routing protocol independence | Routing hellos can stay conservative while BFD provides faster failure notification |
| Interface dependency | BFD timers are commonly configured on the interface participating in the adjacency |
| Client registration | BFD does nothing operationally until a protocol is told to use it |
| Fast failure boundary | BFD detects forwarding failure quickly, but routing convergence still depends on the client protocol and alternate path availability |
| Troubleshooting rule | Verify the underlay adjacency first, then BFD session state, then routing protocol client registration |

# BFD_Fast_Failure_Detection_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the neighbor pair that needs fast failure detection | RTR/L3SW | `show ip interface brief` | Local interface, peer interface, and peer IP are known |
| 2 | Confirm the Layer 3 interface is up | RTR/L3SW | `show ip interface brief` | BFD-facing interface is `up/up` |
| 3 | Confirm Layer 3 forwarding is enabled on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 4 | Confirm direct peer reachability for single-hop BFD | RTR/L3SW | `ping <PEER_IP> source <LOCAL_INTERFACE_OR_IP>` | Peer responds over the intended interface |
| 5 | Confirm the routing protocol adjacency is already working before adding BFD | RTR/L3SW | `show ip ospf neighbor`<br>`show ip eigrp neighbors`<br>`show bgp ipv4 unicast summary` | Routing neighbor is established before BFD is attached |
| 6 | Confirm the current interface BFD configuration | RTR/L3SW | `show running-config interface <BFD_INTERFACE>` | Existing BFD timers are known before changes |
| 7 | Configure BFD timers on router A | RTR-A | `conf t`<br>`interface <BFD_INTERFACE>`<br>` bfd interval <TX_MSEC> min_rx <RX_MSEC> multiplier <MULTIPLIER>`<br>`end` | Router A sends and expects BFD packets using the configured timers |
| 8 | Configure BFD timers on router B | RTR-B | `conf t`<br>`interface <BFD_INTERFACE>`<br>` bfd interval <TX_MSEC> min_rx <RX_MSEC> multiplier <MULTIPLIER>`<br>`end` | Router B uses compatible BFD timers |
| 9 | Use a conservative baseline timer if the platform or lab is unstable | RTR/L3SW | `conf t`<br>`interface <BFD_INTERFACE>`<br>` bfd interval 300 min_rx 300 multiplier 3`<br>`end` | Detection target is roughly 900 ms |
| 10 | Use a faster timer only when the platform and link can support it | RTR/L3SW | `conf t`<br>`interface <BFD_INTERFACE>`<br>` bfd interval 100 min_rx 100 multiplier 3`<br>`end` | Detection target is roughly 300 ms |
| 11 | Attach BFD to OSPF on the interface | OSPF RTR | `conf t`<br>`interface <BFD_INTERFACE>`<br>` ip ospf bfd`<br>`end` | OSPF registers the interface adjacency with BFD |
| 12 | Optional: enable BFD for all OSPF interfaces in the process | OSPF RTR | `conf t`<br>`router ospf <PROCESS_ID>`<br>` bfd all-interfaces`<br>`end` | OSPF attempts to use BFD on all eligible OSPF interfaces |
| 13 | Optional: disable BFD on a specific OSPF interface when using global OSPF BFD | OSPF RTR | `conf t`<br>`interface <BFD_INTERFACE>`<br>` ip ospf bfd disable`<br>`end` | OSPF does not use BFD on that specific interface |
| 14 | Attach BFD to classic EIGRP for a specific interface | EIGRP RTR | `conf t`<br>`router eigrp <ASN>`<br>` bfd interface <BFD_INTERFACE>`<br>`end` | EIGRP registers the selected interface adjacency with BFD |
| 15 | Attach BFD to BGP single-hop neighbor | BGP RTR | `conf t`<br>`router bgp <ASN>`<br>` neighbor <PEER_IP> fall-over bfd`<br>`end` | BGP neighbor uses BFD failure notification |
| 16 | Attach BFD to BGP single-hop neighbor with explicit mode where supported | BGP RTR | `conf t`<br>`router bgp <ASN>`<br>` neighbor <PEER_IP> fall-over bfd single-hop`<br>`end` | BGP neighbor expects single-hop BFD behavior |
| 17 | Attach BFD to BGP multihop neighbor where supported and required | BGP RTR | `conf t`<br>`router bgp <ASN>`<br>` neighbor <PEER_IP> fall-over bfd multi-hop`<br>`end` | BGP neighbor expects multihop BFD behavior |
| 18 | Verify BFD session state | RTR/L3SW | `show bfd neighbors` | BFD neighbor appears and session state is `Up` |
| 19 | Verify detailed BFD session state | RTR/L3SW | `show bfd neighbors details` | Output shows peer, interface, timers, uptime, diagnostics, and client information |
| 20 | Verify BFD client registration | RTR/L3SW | `show bfd neighbors details` | BFD output shows the routing protocol client using the session |
| 21 | Verify routing protocol neighbor still remains established | RTR/L3SW | `show ip ospf neighbor`<br>`show ip eigrp neighbors`<br>`show bgp ipv4 unicast summary` | Routing adjacency remains up after BFD is added |
| 22 | Verify running configuration for the BFD interface | RTR/L3SW | `show running-config interface <BFD_INTERFACE>` | Interface contains the intended `bfd interval` command |
| 23 | Verify running configuration for the BFD client protocol | RTR/L3SW | `show running-config | section router ospf`<br>`show running-config | section router eigrp`<br>`show running-config | section router bgp` | Routing process contains the intended BFD client configuration |
| 24 | Test BFD failure in the lab by shutting the path | RTR-A | `conf t`<br>`interface <BFD_INTERFACE>`<br>` shutdown`<br>`end` | BFD session goes down quickly |
| 25 | Verify BFD failure detection | RTR-B | `show bfd neighbors details` | Session shows down state or recent diagnostic reason |
| 26 | Verify routing protocol reacts to BFD failure | RTR-B | `show ip route <PREFIX>`<br>`show ip ospf neighbor`<br>`show ip eigrp neighbors`<br>`show bgp ipv4 unicast summary` | Routing client withdraws or reconverges faster than default hello dead timers |
| 27 | Restore the BFD path | RTR-A | `conf t`<br>`interface <BFD_INTERFACE>`<br>` no shutdown`<br>`end` | Interface returns to `up/up` |
| 28 | Verify BFD session returns to up state | RTR-A/RTR-B | `show bfd neighbors` | BFD session returns to `Up` |
| 29 | Verify routing protocol adjacency recovers | RTR-A/RTR-B | `show ip ospf neighbor`<br>`show ip eigrp neighbors`<br>`show bgp ipv4 unicast summary` | Routing protocol adjacency returns to established state |
| 30 | Save the working configuration | RTR/L3SW | `copy running-config startup-config` | BFD timer and client configuration survive reload |

# BFD_Fast_Failure_Detection_Skeleton

conf t
!
ip routing
!
interface <BFD_INTERFACE>
 description BFD_FAST_FAILURE_DETECTION_LINK
 ip address <LOCAL_IP> <MASK>
 bfd interval <TX_MSEC> min_rx <RX_MSEC> multiplier <MULTIPLIER>
 no shutdown
!
end
write memory

# BFD_Conservative_Timer_Skeleton

conf t
!
interface <BFD_INTERFACE>
 bfd interval 300 min_rx 300 multiplier 3
!
end
write memory

# BFD_Fast_Timer_Skeleton

conf t
!
interface <BFD_INTERFACE>
 bfd interval 100 min_rx 100 multiplier 3
!
end
write memory

# OSPF_BFD_Interface_Skeleton

conf t
!
interface <BFD_INTERFACE>
 bfd interval 100 min_rx 100 multiplier 3
 ip ospf bfd
!
end
write memory

# OSPF_BFD_All_Interfaces_Skeleton

conf t
!
router ospf <PROCESS_ID>
 bfd all-interfaces
!
end
write memory

# OSPF_BFD_Interface_Disable_Skeleton

conf t
!
interface <BFD_INTERFACE>
 ip ospf bfd disable
!
end
write memory

# EIGRP_BFD_Interface_Skeleton

conf t
!
interface <BFD_INTERFACE>
 bfd interval 100 min_rx 100 multiplier 3
!
router eigrp <ASN>
 bfd interface <BFD_INTERFACE>
!
end
write memory

# BGP_BFD_Single_Hop_Skeleton

conf t
!
interface <BFD_INTERFACE>
 bfd interval 100 min_rx 100 multiplier 3
!
router bgp <ASN>
 neighbor <PEER_IP> remote-as <PEER_ASN>
 neighbor <PEER_IP> fall-over bfd
!
end
write memory

# BGP_BFD_Explicit_Single_Hop_Skeleton

conf t
!
router bgp <ASN>
 neighbor <PEER_IP> fall-over bfd single-hop
!
end
write memory

# BGP_BFD_Multihop_Skeleton

conf t
!
router bgp <ASN>
 neighbor <PEER_IP> remote-as <PEER_ASN>
 neighbor <PEER_IP> ebgp-multihop <HOP_COUNT>
 neighbor <PEER_IP> fall-over bfd multi-hop
!
end
write memory

# BFD_Fast_Failure_Detection_Verification_Commands

show ip interface brief
show running-config | include ^ip routing
show running-config interface <BFD_INTERFACE>
show running-config | section router ospf
show running-config | section router eigrp
show running-config | section router bgp
show bfd neighbors
show bfd neighbors details
show bfd neighbors interface <BFD_INTERFACE>
show bfd neighbors interface <BFD_INTERFACE> details
show ip ospf neighbor
show ip eigrp neighbors
show bgp ipv4 unicast summary
show ip route
show ip route <PREFIX>
show ip cef <PEER_IP>
show ip cef exact-route <LOCAL_IP> <PEER_IP>
ping <PEER_IP> source <LOCAL_INTERFACE_OR_IP>
traceroute <PEER_IP> source <LOCAL_IP>
show interfaces <BFD_INTERFACE>
debug bfd events
debug bfd packets
show debugging
undebug all

# BFD_Failover_Test_Commands

conf t
interface <BFD_INTERFACE>
 shutdown
end

show bfd neighbors details
show ip ospf neighbor
show ip eigrp neighbors
show bgp ipv4 unicast summary
show ip route <PREFIX>

conf t
interface <BFD_INTERFACE>
 no shutdown
end

show bfd neighbors
show ip route <PREFIX>

# BFD_Fast_Failure_Detection_Rollback

conf t
!
interface <BFD_INTERFACE>
 no bfd interval
 no ip ospf bfd
 no ip ospf bfd disable
!
router ospf <PROCESS_ID>
 no bfd all-interfaces
!
router eigrp <ASN>
 no bfd interface <BFD_INTERFACE>
!
router bgp <ASN>
 no neighbor <PEER_IP> fall-over bfd
 no neighbor <PEER_IP> fall-over bfd single-hop
 no neighbor <PEER_IP> fall-over bfd multi-hop
!
end
write memory

# BFD_Failover_Test_Restore

conf t
!
interface <BFD_INTERFACE>
 no shutdown
!
end

# BFD_Fast_Failure_Detection_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No BFD neighbor appears | Routing protocol client was not registered with BFD | `show running-config | section router` and `show bfd neighbors` | Attach BFD under OSPF, EIGRP, BGP, or the intended client feature |
| BFD neighbor stays down | Peer reachability is broken | `ping <PEER_IP> source <LOCAL_INTERFACE_OR_IP>` | Fix interface, addressing, VLAN, link, or routing to the peer |
| BFD session is missing on one side | BFD is configured only on one router or only one client is registered | `show running-config interface <BFD_INTERFACE>` on both routers | Configure compatible BFD timers and client registration on both sides |
| Routing adjacency is up but BFD is down | BFD packets are blocked or timers are incompatible | `show bfd neighbors details` | Check ACLs, CoPP, platform support, and timer values |
| BFD flaps repeatedly | Timers are too aggressive for the device, link, or CPU condition | `show bfd neighbors details` and `show processes cpu` | Use more conservative timers such as 300/300/3 or 500/500/3 |
| BFD detects failure but route does not change | No alternate route exists or routing protocol did not reconverge | `show ip route <PREFIX>` | Verify alternate path and routing protocol topology |
| BFD goes down when physical link stays up | Forwarding path failure, ACL, CoPP, or intermediate issue | `show interfaces <BFD_INTERFACE>` and `show access-lists` | Fix packet loss, filtering, or control-plane policing |
| BFD not used by OSPF | `ip ospf bfd` or `bfd all-interfaces` is missing | `show running-config interface <BFD_INTERFACE>` and `show running-config | section router ospf` | Configure interface or process-level OSPF BFD |
| OSPF BFD enabled globally but unwanted on one interface | `bfd all-interfaces` applies broadly | `show running-config | section router ospf` | Configure `ip ospf bfd disable` on the exception interface |
| BFD not used by EIGRP | EIGRP `bfd interface` command is missing | `show running-config | section router eigrp` | Configure `bfd interface <BFD_INTERFACE>` under EIGRP |
| BFD not used by BGP | Neighbor fall-over BFD is missing | `show running-config | section router bgp` | Configure `neighbor <PEER_IP> fall-over bfd` |
| BGP multihop BFD fails | Single-hop BFD configured for a multihop peer | `show running-config | section router bgp` | Use multihop BFD syntax where supported and confirm reachability |
| BGP session drops too quickly during transient loss | BFD timer values are too aggressive | `show bfd neighbors details` | Increase interval or multiplier |
| BFD command is rejected | Platform, license, image, or interface type does not support the command | `bfd ?` under interface and protocol mode | Use supported platform syntax or rely on routing protocol timers |
| Interface BFD timers configured but no session forms | BFD has no client requesting a session | `show bfd neighbors` | Register a routing protocol client with BFD |
| BFD session forms but no client is listed | Protocol has not attached to the BFD session | `show bfd neighbors details` | Fix OSPF, EIGRP, or BGP BFD attachment |
| ACL blocks BFD control traffic | Infrastructure ACL blocks UDP BFD packets | `show access-lists` | Permit required BFD control traffic between peers |
| CoPP drops BFD packets | Control-plane policy is too restrictive | CoPP policy counters | Permit or rate-limit BFD appropriately in control-plane policy |
| ECMP causes testing confusion | Tested source and destination flow follows a different path | `show ip cef exact-route <LOCAL_IP> <PEER_IP>` | Test the exact flow and interface carrying the BFD session |
| Subinterface BFD fails | BFD configured on wrong parent or subinterface | `show running-config interface <BFD_INTERFACE>` | Configure BFD on the actual Layer 3 interface forming the adjacency |
| Tunnel BFD is unstable | Tunnel underlay loss, MTU, or recursive routing issue | `show interfaces Tunnel<TUNNEL_ID>` and DF ping tests | Fix tunnel underlay, MTU, or recursion |
| Debug output is too noisy | BFD debug left running | `show debugging` | Run `undebug all` |
| Configuration disappears after reload | Configuration was not saved | `show startup-config interface <BFD_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |

# Index

BFD_Fast_Failure_Detection.md
BFD_Fast_Failure_Detection
BFD_Fast_Failure_Detection_Mental_Model
BFD_Fast_Failure_Detection_Configuration_Checklist
BFD_Fast_Failure_Detection_Skeleton
BFD_Conservative_Timer_Skeleton
BFD_Fast_Timer_Skeleton
OSPF_BFD_Interface_Skeleton
OSPF_BFD_All_Interfaces_Skeleton
OSPF_BFD_Interface_Disable_Skeleton
EIGRP_BFD_Interface_Skeleton
BGP_BFD_Single_Hop_Skeleton
BGP_BFD_Explicit_Single_Hop_Skeleton
BGP_BFD_Multihop_Skeleton
BFD_Fast_Failure_Detection_Verification_Commands
BFD_Failover_Test_Commands
BFD_Fast_Failure_Detection_Rollback
BFD_Failover_Test_Restore
BFD_Fast_Failure_Detection_Failure_Checks
