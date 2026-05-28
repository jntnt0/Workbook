
DMVPN_IPv6_Over_IPv4_Transport.md

# DMVPN_IPv6_Over_IPv4_Transport_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 11, IPv6 | Supports IPv6 addressing, IPv6 routing, RIPng, OSPFv3, EIGRP for IPv6, and tunneling context |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports mGRE, NHRP mapping, tunnel IP to NBMA mapping, and DMVPN phase behavior |
| `All_combined_part3.md` | Book 6, Chapter 11, IPv6 | Supports IPv6 unicast routing, IPv6 interface addressing, IPv6 routing protocols, and IPv6 verification commands |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 19, IPv6 DMVPN Configuration | Supports IPv6 DMVPN command mapping, IPv6 tunnel protocol commands, IPv4 versus IPv6 transport command selection, and IPv6 DMVPN verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 19, IPv6 DMVPN display commands | Supports `show ipv6 nhrp`, `show dmvpn ipv6`, `show ipv6 nhrp traffic`, and `show ipv6 nhrp nhs detail` verification |

# DMVPN_IPv6_Over_IPv4_Transport_Mental_Model

| Concept | Operational Meaning |
|---|---|
| IPv6 over IPv4 DMVPN | IPv6 is the overlay payload, while IPv4 is the transport/NBMA underlay |
| IPv4 transport | Tunnel source, tunnel mode, and NBMA reachability use IPv4 |
| IPv6 overlay | Tunnel addressing, NHRP protocol mapping, and overlay routing use IPv6 |
| Command split | Use `ipv6 nhrp` commands for the tunneled protocol, but use IPv4 GRE transport commands |
| Correct tunnel mode | IPv6 over IPv4 uses `tunnel mode gre multipoint` |
| Wrong tunnel mode | `tunnel mode gre multipoint ipv6` is for IPv6 transport, not IPv4 transport |
| IPv6 NHS address | The NHS address is the hub IPv6 tunnel address |
| IPv4 NBMA address | The NBMA address is the hub IPv4 underlay address |
| Link-local address | Each tunnel interface needs a stable unique IPv6 link-local address |
| IPv6 unicast routing | Required before IPv6 overlay routing protocols can work |
| IPv6 NHRP | Resolves IPv6 tunnel/protocol addresses to IPv4 NBMA transport addresses |
| IPv6 Phase 3 | Hub uses `ipv6 nhrp redirect`; spokes use `ipv6 nhrp shortcut` |
| MTU/MSS | IPv6 payload plus GRE and possible IPsec overhead require careful MTU/MSS handling |
| Hard rule | If IPv4 NBMA reachability is broken, IPv6 overlay troubleshooting is a waste of time |

# DMVPN_IPv6_Over_IPv4_Transport_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm this is IPv6 over IPv4 | All Routers | Design check | IPv6 is overlay/tunneled protocol and IPv4 is transport |
| 2 | Confirm IPv4 NBMA reachability | Hub, Spokes | `ping <remote-ipv4-nbma-ip>` | IPv4 transport works before IPv6 DMVPN |
| 3 | Confirm IPv4 route to remote NBMA | Hub, Spokes | `show ip route <remote-ipv4-nbma-ip>` | Remote NBMA resolves through IPv4 underlay |
| 4 | Confirm tunnel source interface state | Hub, Spokes | `show ip interface brief` | IPv4 tunnel source interface is `up/up` |
| 5 | Enable IPv6 routing globally | Hub, Spokes | See `Global_IPv6_Routing_Enablement` | Router can forward IPv6 packets |
| 6 | Build hub IPv6-over-IPv4 tunnel | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub uses IPv6 overlay over IPv4 transport |
| 7 | Configure hub IPv6 tunnel address | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub has global IPv6 tunnel address |
| 8 | Configure hub unique link-local address | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub has stable IPv6 link-local address |
| 9 | Configure hub IPv6 MTU and MSS | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | IPv6 tunnel accounts for GRE/IPsec overhead |
| 10 | Enable IPv6 NHRP on hub | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub participates in IPv6 NHRP cloud |
| 11 | Enable IPv6 NHRP multicast mapping on hub | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub supports multicast from registered spokes |
| 12 | Enable IPv6 Phase 3 redirect on hub if using Phase 3 | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub can send IPv6 NHRP redirects |
| 13 | Configure hub IPv4 tunnel source | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub sources GRE from IPv4 underlay |
| 14 | Configure hub IPv4 transport mGRE mode | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub uses `tunnel mode gre multipoint` |
| 15 | Configure optional hub tunnel key | Hub | See `Hub_IPv6_Over_IPv4_DMVPN_Config` | Hub uses expected tunnel key |
| 16 | Build spoke IPv6-over-IPv4 tunnel | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke uses IPv6 overlay over IPv4 transport |
| 17 | Configure spoke IPv6 tunnel address | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke has global IPv6 tunnel address |
| 18 | Configure spoke unique link-local address | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke has stable IPv6 link-local address |
| 19 | Configure spoke IPv6 MTU and MSS | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | IPv6 tunnel accounts for GRE/IPsec overhead |
| 20 | Enable IPv6 NHRP on spokes | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spokes participate in IPv6 NHRP cloud |
| 21 | Configure hub as IPv6 NHS with IPv4 NBMA address | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke maps hub IPv6 tunnel address to hub IPv4 NBMA address |
| 22 | Configure spoke IPv6 multicast mapping to hub | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | IPv6 routing protocol multicast can cross DMVPN |
| 23 | Enable IPv6 Phase 3 shortcut on spokes if using Phase 3 | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spokes can install IPv6 NHRP shortcut entries |
| 24 | Configure spoke IPv4 tunnel source | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke sources GRE from IPv4 underlay |
| 25 | Configure spoke IPv4 transport mGRE mode | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke uses `tunnel mode gre multipoint` |
| 26 | Configure optional spoke tunnel key | Spokes | See `Spoke_IPv6_Over_IPv4_DMVPN_Config` | Spoke tunnel key matches hub |
| 27 | Optional: make tunnel state depend on NHRP | Spokes | See `Optional_NHRP_Interface_State_Control` | Tunnel line protocol follows NHRP state if supported |
| 28 | Verify tunnel mode is IPv4 transport mode | Hub, Spokes | `show running-config interface Tunnel<id>` | Output shows `tunnel mode gre multipoint`, not IPv6 transport mode |
| 29 | Verify IPv6 tunnel interface state | Hub, Spokes | `show ipv6 interface Tunnel<id>` | Tunnel has global and link-local IPv6 addresses |
| 30 | Verify IPv6 NHRP mappings | Hub, Spokes | `show ipv6 nhrp` | Hub has dynamic spoke mappings and spokes have hub mapping |
| 31 | Verify IPv6 NHRP NHS state | Spokes | `show ipv6 nhrp nhs detail` | Spokes list hub as NHS |
| 32 | Verify IPv6 DMVPN state | Hub, Spokes | `show dmvpn ipv6` | IPv6 DMVPN peers show up |
| 33 | Test spoke-to-hub IPv6 tunnel reachability | Spokes | `ping ipv6 <hub-tunnel-ipv6>` | Spokes reach hub IPv6 tunnel address |
| 34 | Test hub-to-spoke IPv6 tunnel reachability | Hub | `ping ipv6 <spoke-tunnel-ipv6>` | Hub reaches spoke IPv6 tunnel addresses |
| 35 | Configure IPv6 overlay routing protocol | Hub, Spokes | See `OSPFv3_Overlay_Config` or `EIGRP_IPv6_Overlay_Config` | IPv6 routes can be exchanged over DMVPN |
| 36 | Advertise IPv6 LAN or loopback prefixes | Hub, Spokes | See IPv6 routing blocks | Local IPv6 prefixes enter overlay routing |
| 37 | Verify IPv6 routing neighbors | Hub, Spokes | `show ospfv3 neighbor` or `show ipv6 eigrp neighbors` | IPv6 routing neighbors form over tunnel |
| 38 | Verify IPv6 overlay routes | Hub, Spokes | `show ipv6 route` | Remote IPv6 prefixes appear |
| 39 | Test IPv6 LAN reachability | Hub, Spokes | `ping ipv6 <remote-ipv6-lan-ip>` | IPv6 payload crosses IPv4 DMVPN transport |
| 40 | Verify IPv6 Phase 3 shortcut if used | Spokes | `show ipv6 nhrp` and `show ipv6 cef <remote-ipv6-lan-ip>` | IPv6 shortcut and forwarding entry appear |
| 41 | Save working IPv6-over-IPv4 DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_IPv6_Over_IPv4_Transport_Skeleton

```text
# Global_IPv6_Routing_Enablement
conf t
ipv6 unicast-routing
end
```

```text
# Hub_IPv6_Over_IPv4_DMVPN_Config
conf t
interface Tunnel<id>
 description DMVPN_IPV6_OVER_IPV4_HUB
 bandwidth <kbps>
 ipv6 address <hub-link-local-ipv6> link-local
 ipv6 address <hub-tunnel-ipv6>/<prefix-length>
 ipv6 mtu 1400
 ipv6 nhrp authentication <key-string>
 ipv6 nhrp map multicast dynamic
 ipv6 nhrp network-id <nhrp-id>
 ipv6 nhrp holdtime <seconds>
 ipv6 nhrp redirect
 ipv6 tcp adjust-mss 1360
 tunnel source <hub-ipv4-underlay-interface-or-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Spoke_IPv6_Over_IPv4_DMVPN_Config
conf t
interface Tunnel<id>
 description DMVPN_IPV6_OVER_IPV4_SPOKE
 bandwidth <kbps>
 ipv6 address <spoke-link-local-ipv6> link-local
 ipv6 address <spoke-tunnel-ipv6>/<prefix-length>
 ipv6 mtu 1400
 ipv6 nhrp authentication <key-string>
 ipv6 nhrp network-id <nhrp-id>
 ipv6 nhrp holdtime <seconds>
 ipv6 nhrp nhs <hub-tunnel-ipv6> nbma <hub-ipv4-nbma-ip> multicast
 ipv6 nhrp shortcut
 ipv6 tcp adjust-mss 1360
 tunnel source <spoke-ipv4-underlay-interface-or-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Spoke_Separate_IPv6_NHRP_Mapping_Alternative
conf t
interface Tunnel<id>
 ipv6 nhrp nhs <hub-tunnel-ipv6>
 ipv6 nhrp map <hub-tunnel-ipv6> <hub-ipv4-nbma-ip>
 ipv6 nhrp map multicast <hub-ipv4-nbma-ip>
end
```

```text
# Optional_NHRP_Interface_State_Control
! Platform support can vary.
! Useful when the tunnel should report down if NHRP registration is not healthy.

conf t
interface Tunnel<id>
 if-state nhrp
end
```

```text
# OSPFv3_Overlay_Config
conf t
router ospfv3 <process-id>
 router-id <router-id>
exit
!
interface Tunnel<id>
 ospfv3 <process-id> ipv6 area <area-id>
exit
!
interface <ipv6-lan-or-loopback-interface>
 ipv6 address <ipv6-prefix>/<prefix-length>
 ospfv3 <process-id> ipv6 area <area-id>
end
```

```text
# EIGRP_IPv6_Overlay_Config
conf t
ipv6 router eigrp <asn>
 eigrp router-id <router-id>
 no shutdown
exit
!
interface Tunnel<id>
 ipv6 eigrp <asn>
exit
!
interface <ipv6-lan-or-loopback-interface>
 ipv6 address <ipv6-prefix>/<prefix-length>
 ipv6 eigrp <asn>
end
```

```text
# RIPng_Overlay_Config
conf t
ipv6 router rip <ripng-name>
exit
!
interface Tunnel<id>
 ipv6 rip <ripng-name> enable
exit
!
interface <ipv6-lan-or-loopback-interface>
 ipv6 address <ipv6-prefix>/<prefix-length>
 ipv6 rip <ripng-name> enable
end
```

```text
# IPv6_Over_IPv4_Command_Rule
! IPv6 over IPv4 uses:
! - IPv6 tunneled protocol commands:
!   ipv6 address
!   ipv6 mtu
!   ipv6 tcp adjust-mss
!   ipv6 nhrp network-id
!   ipv6 nhrp nhs
!   ipv6 nhrp redirect
!   ipv6 nhrp shortcut
!
! - IPv4 transport command:
!   tunnel mode gre multipoint
!
! Do not use this unless the transport itself is IPv6:
!   tunnel mode gre multipoint ipv6
```

```text
# IPv4_Underlay_Reminder
! IPv4 NBMA reachability must stay outside the tunnel.
! Do not let overlay IPv6 routing distract from broken IPv4 transport.
! Always verify:
!   show ip route <remote-ipv4-nbma-ip>
!   ping <remote-ipv4-nbma-ip>
```

# DMVPN_IPv6_Over_IPv4_Transport_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| IPv4 underlay reachability | Hub, Spokes | `ping <remote-ipv4-nbma-ip>` | IPv4 NBMA pings succeed |
| IPv4 route to NBMA | Hub, Spokes | `show ip route <remote-ipv4-nbma-ip>` | Route points through IPv4 underlay |
| IPv6 global routing | Hub, Spokes | `show running-config \| include ipv6 unicast-routing` | IPv6 unicast routing is enabled |
| Tunnel IPv6 addressing | Hub, Spokes | `show ipv6 interface Tunnel<id>` | Global and link-local IPv6 addresses exist |
| Unique link-local | Hub, Spokes | `show ipv6 interface Tunnel<id>` | Link-local address is unique per router |
| Tunnel source | Hub, Spokes | `show running-config interface Tunnel<id>` | Tunnel source is IPv4 underlay interface or IP |
| Tunnel mode | Hub, Spokes | `show running-config interface Tunnel<id>` | Shows `tunnel mode gre multipoint` |
| Wrong tunnel mode exclusion | Hub, Spokes | `show running-config interface Tunnel<id>` | Does not show `tunnel mode gre multipoint ipv6` |
| IPv6 NHRP cache | Hub, Spokes | `show ipv6 nhrp` | Expected IPv6 tunnel to IPv4 NBMA mappings exist |
| IPv6 NHRP brief | Hub, Spokes | `show ipv6 nhrp brief` | Hub sees dynamic spokes and spokes see hub |
| IPv6 NHS state | Spokes | `show ipv6 nhrp nhs detail` | Hub is listed as NHS |
| IPv6 NHRP traffic | Hub, Spokes | `show ipv6 nhrp traffic` | NHRP counters increment without repeated errors |
| IPv6 DMVPN state | Hub, Spokes | `show dmvpn ipv6` | IPv6 DMVPN peers show up |
| IPv6 DMVPN detail | Hub, Spokes | `show dmvpn ipv6 detail` | Shows IPv6 tunnel protocol with IPv4 NBMA transport mapping |
| IPv6 tunnel ping | Hub, Spokes | `ping ipv6 <remote-tunnel-ipv6>` | IPv6 tunnel pings succeed |
| OSPFv3 neighbor | Hub, Spokes | `show ospfv3 neighbor` | IPv6 OSPFv3 neighbors are full |
| EIGRP for IPv6 neighbor | Hub, Spokes | `show ipv6 eigrp neighbors` | IPv6 EIGRP neighbors are present |
| RIPng routes if used | Hub, Spokes | `show ipv6 route rip` | RIPng routes are present |
| IPv6 routes | Hub, Spokes | `show ipv6 route` | Remote IPv6 prefixes appear |
| IPv6 CEF path | Hub, Spokes | `show ipv6 cef <remote-ipv6-destination>` | Forwarding resolves through expected tunnel path |
| IPv6 LAN reachability | Hub, Spokes | `ping ipv6 <remote-ipv6-lan-ip>` | IPv6 payload crosses IPv4 DMVPN transport |
| IPv6 path proof | Hub, Spokes | `traceroute ipv6 <remote-ipv6-lan-ip>` | Path matches Phase 1, Phase 2, or Phase 3 design |

# DMVPN_IPv6_Over_IPv4_Transport_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove IPv6 routing protocol from tunnel | Hub, Spokes | See `Rollback_OSPFv3_Tunnel` or routing rollback block | Tunnel stops participating in IPv6 routing protocol |
| 2 | Remove IPv6 routing protocol from LAN or loopback | Hub, Spokes | See IPv6 routing rollback blocks | LAN or loopback stops advertising IPv6 prefix |
| 3 | Remove OSPFv3 process if fully reverting | Hub, Spokes | See `Rollback_OSPFv3_Process` | OSPFv3 process is removed |
| 4 | Remove EIGRP for IPv6 if fully reverting | Hub, Spokes | See `Rollback_EIGRP_IPv6_Process` | EIGRP for IPv6 process is removed |
| 5 | Remove RIPng if fully reverting | Hub, Spokes | See `Rollback_RIPng_Process` | RIPng process is removed |
| 6 | Shut down spoke tunnels first if resetting DMVPN | Spokes | `shutdown` under tunnel | Spokes stop registering |
| 7 | Remove spoke IPv6 NHS compact command | Spokes | See `Rollback_Spoke_IPv6_NHRP` | Spoke no longer uses hub as IPv6 NHS |
| 8 | Remove separate spoke IPv6 NHRP mappings if used | Spokes | See `Rollback_Spoke_IPv6_NHRP` | Static hub mapping is removed |
| 9 | Remove spoke IPv6 shortcut | Spokes | See `Rollback_Spoke_IPv6_NHRP` | Spoke no longer installs IPv6 shortcuts |
| 10 | Remove spoke IPv6 NHRP authentication | Spokes | See `Rollback_Spoke_IPv6_NHRP` | IPv6 NHRP authentication is removed |
| 11 | Remove spoke IPv6 NHRP network ID | Spokes | See `Rollback_Spoke_IPv6_NHRP` | IPv6 NHRP is disabled on spoke tunnel |
| 12 | Remove spoke IPv6 tunnel addresses | Spokes | See `Rollback_Spoke_IPv6_Tunnel_Addressing` | Spoke IPv6 tunnel addressing is removed |
| 13 | Shut down hub tunnel if fully resetting DMVPN | Hub | `shutdown` under tunnel | Hub stops accepting registrations |
| 14 | Remove hub IPv6 redirect | Hub | See `Rollback_Hub_IPv6_NHRP` | Hub stops sending IPv6 redirects |
| 15 | Remove hub IPv6 multicast mapping | Hub | See `Rollback_Hub_IPv6_NHRP` | Hub dynamic multicast mapping is removed |
| 16 | Remove hub IPv6 NHRP authentication | Hub | See `Rollback_Hub_IPv6_NHRP` | Hub IPv6 NHRP authentication is removed |
| 17 | Remove hub IPv6 NHRP network ID | Hub | See `Rollback_Hub_IPv6_NHRP` | IPv6 NHRP is disabled on hub tunnel |
| 18 | Remove hub IPv6 tunnel addresses | Hub | See `Rollback_Hub_IPv6_Tunnel_Addressing` | Hub IPv6 tunnel addressing is removed |
| 19 | Remove tunnel source and key if fully resetting | Hub, Spokes | See `Rollback_Common_IPv6_Over_IPv4_Tunnel` | Tunnel source and key are removed |
| 20 | Remove mGRE mode if fully resetting | Hub, Spokes | See `Rollback_Common_IPv6_Over_IPv4_Tunnel` | mGRE mode is removed |
| 21 | Delete tunnel interface if fully resetting | Hub, Spokes | `no interface Tunnel<id>` | Tunnel interface is removed |
| 22 | Disable IPv6 routing only if no other IPv6 lab needs it | Hub, Spokes | `no ipv6 unicast-routing` | Router stops IPv6 forwarding globally |

```text
# Rollback_OSPFv3_Tunnel
conf t
interface Tunnel<id>
 no ospfv3 <process-id> ipv6 area <area-id>
end
```

```text
# Rollback_OSPFv3_Process
conf t
no router ospfv3 <process-id>
end
```

```text
# Rollback_EIGRP_IPv6_Tunnel
conf t
interface Tunnel<id>
 no ipv6 eigrp <asn>
end
```

```text
# Rollback_EIGRP_IPv6_Process
conf t
no ipv6 router eigrp <asn>
end
```

```text
# Rollback_RIPng_Tunnel
conf t
interface Tunnel<id>
 no ipv6 rip <ripng-name> enable
end
```

```text
# Rollback_RIPng_Process
conf t
no ipv6 router rip <ripng-name>
end
```

```text
# Rollback_Spoke_IPv6_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ipv6 nhrp shortcut
 no ipv6 nhrp nhs <hub-tunnel-ipv6> nbma <hub-ipv4-nbma-ip> multicast
 no ipv6 nhrp nhs <hub-tunnel-ipv6>
 no ipv6 nhrp map <hub-tunnel-ipv6> <hub-ipv4-nbma-ip>
 no ipv6 nhrp map multicast <hub-ipv4-nbma-ip>
 no ipv6 nhrp authentication <key-string>
 no ipv6 nhrp network-id <nhrp-id>
 no ipv6 nhrp holdtime <seconds>
end
```

```text
# Rollback_Spoke_IPv6_Tunnel_Addressing
conf t
interface Tunnel<id>
 no ipv6 address <spoke-tunnel-ipv6>/<prefix-length>
 no ipv6 address <spoke-link-local-ipv6> link-local
 no ipv6 mtu 1400
 no ipv6 tcp adjust-mss 1360
end
```

```text
# Rollback_Hub_IPv6_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ipv6 nhrp redirect
 no ipv6 nhrp map multicast dynamic
 no ipv6 nhrp authentication <key-string>
 no ipv6 nhrp network-id <nhrp-id>
 no ipv6 nhrp holdtime <seconds>
end
```

```text
# Rollback_Hub_IPv6_Tunnel_Addressing
conf t
interface Tunnel<id>
 no ipv6 address <hub-tunnel-ipv6>/<prefix-length>
 no ipv6 address <hub-link-local-ipv6> link-local
 no ipv6 mtu 1400
 no ipv6 tcp adjust-mss 1360
end
```

```text
# Rollback_Common_IPv6_Over_IPv4_Tunnel
conf t
interface Tunnel<id>
 no if-state nhrp
 no tunnel key <key-id>
 no tunnel source <ipv4-underlay-interface-or-ip>
 no tunnel mode gre multipoint
end
```

# DMVPN_IPv6_Over_IPv4_Transport_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| IPv6 DMVPN never comes up | IPv4 NBMA transport is broken | `ping <remote-ipv4-nbma-ip>` | Fix IPv4 underlay first |
| Tunnel mode is wrong | IPv6 transport command was used by mistake | `show running-config interface Tunnel<id>` | Use `tunnel mode gre multipoint` |
| Spoke cannot register with IPv6 NHRP | Wrong NHS IPv6 address, wrong IPv4 NBMA address, or auth mismatch | `show ipv6 nhrp` | Correct NHS, NBMA, and authentication |
| Hub sees no IPv6 NHRP entries | Spokes are not registering | `show ipv6 nhrp` on hub | Fix spoke NHS, IPv6 NHRP ID, authentication, or tunnel source |
| IPv6 routing protocol will not start | IPv6 unicast routing is disabled | `show running-config \| include ipv6 unicast-routing` | Configure `ipv6 unicast-routing` |
| OSPFv3 neighbor will not form | Missing link-local, passive tunnel, area mismatch, or multicast issue | `show ipv6 interface Tunnel<id>` and `show ospfv3 neighbor` | Add link-local, fix area/passive state, or fix NHRP multicast |
| EIGRP for IPv6 neighbor will not form | IPv6 EIGRP process shutdown or tunnel not enabled | `show ipv6 eigrp neighbors` | Enable EIGRP for IPv6 process and tunnel interface |
| RIPng routes missing | RIPng not enabled on tunnel or LAN | `show ipv6 route rip` | Enable RIPng on tunnel and required interfaces |
| IPv6 route exists but ping fails | NHRP or CEF resolution problem | `show ipv6 nhrp` and `show ipv6 cef <destination>` | Fix IPv6 NHRP and DMVPN phase behavior |
| Phase 3 IPv6 shortcut does not form | Missing IPv6 redirect or shortcut | `show running-config interface Tunnel<id>` | Add hub `ipv6 nhrp redirect` and spoke `ipv6 nhrp shortcut` |
| Link-local next hops look confusing | IPv6 routing protocols use link-local next hops | `show ipv6 route` | Verify neighbor state and global reachability |
| Duplicate link-local causes bad behavior | Same manual link-local reused | `show ipv6 interface Tunnel<id>` | Assign unique link-local addresses |
| IPv6 multicast routing protocol fails | IPv6 NHRP multicast mapping missing | `show running-config interface Tunnel<id>` | Add hub multicast dynamic and spoke multicast mapping |
| Large IPv6 packets fail | GRE/IPsec overhead or IPv6 MTU issue | `ping ipv6 <destination> size <size>` | Configure IPv6 MTU and IPv6 TCP MSS |
| IPv4 tunnel works but IPv6 overlay fails | Only IPv4 NHRP was configured | `show running-config interface Tunnel<id>` | Add IPv6 tunnel addressing and `ipv6 nhrp` commands |
| Engineer troubleshoots IPv6 before transport | IPv4 underlay route is missing | `show ip route <remote-ipv4-nbma-ip>` | Fix IPv4 NBMA reachability first |

##### Source_Basis
# DMVPN_IPv6_Over_IPv4_Transport_Mental_Model
# DMVPN_IPv6_Over_IPv4_Transport_Configuration_Checklist
# DMVPN_IPv6_Over_IPv4_Transport_Skeleton
# DMVPN_IPv6_Over_IPv4_Transport_Verification_Commands
# DMVPN_IPv6_Over_IPv4_Transport_Rollback
# DMVPN_IPv6_Over_IPv4_Transport_Failure_Checks

