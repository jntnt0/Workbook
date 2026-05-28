

DMVPN_Phase_3_NHRP_Redirect_Shortcut.md

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN Phase 3 | Supports hub mGRE, spoke mGRE, NHRP redirect, NHRP shortcut, and spoke-to-spoke optimization |
| `All_combined_part3.md` | Book 6, Chapter 5, Lab 5-5 DMVPN Phase 3 | Supports Phase 3 lab validation using redirect, shortcut, NHRP, and traceroute checks |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Tunnels | Supports modern DMVPN verification with `show dmvpn`, `show ip nhrp`, tunnel state, and routing checks |

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Phase 3 | DMVPN model where hub and spokes all use mGRE, and NHRP optimizes spoke-to-spoke forwarding |
| Hub role | Hub acts as NHS and sends NHRP Redirect messages when traffic hairpins through the hub |
| Spoke role | Spoke registers to the hub, receives redirects, resolves the remote spoke, and installs shortcut forwarding |
| `ip nhrp redirect` | Hub command that tells a spoke a better direct path exists |
| `ip nhrp shortcut` | Spoke command that allows shortcut forwarding to be installed |
| Initial path | First spoke-to-spoke flow may go through the hub |
| Optimized path | Later traffic should go directly spoke to spoke |
| Phase 2 difference | Phase 2 requires routing to preserve the remote spoke next hop |
| Phase 3 advantage | Phase 3 works with hub summaries or default routes because NHRP shortcuts optimize forwarding |
| Summarization | Safe in Phase 3 when redirect and shortcut behavior are working |
| Default route | Spokes can point to the hub and still build direct forwarding after shortcut resolution |
| CEF rewrite | The spoke installs forwarding toward the remote spoke after NHRP resolution |
| Hard rule | If the first traceroute hits the hub and the second traceroute goes direct, Phase 3 is working |

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before DMVPN configuration |
| 2 | Confirm tunnel source interface state | Hub, Spokes | `show ip interface brief` | Tunnel source interface is `up/up` |
| 3 | Confirm transport route to remote NBMA | Hub, Spokes | `show ip route <remote-nbma-ip>` | Remote NBMA resolves through the underlay |
| 4 | Build Phase 3 hub tunnel | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub uses mGRE and participates in NHRP |
| 5 | Configure hub tunnel IP | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub has an overlay tunnel address |
| 6 | Configure hub tunnel source | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub sources GRE from the correct underlay address |
| 7 | Configure hub mGRE mode | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub can support multiple dynamic peers |
| 8 | Enable NHRP on hub | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub participates in the NHRP cloud |
| 9 | Enable dynamic multicast mapping on hub | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub supports multicast from registered spokes |
| 10 | Enable Phase 3 redirect on hub | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub can send NHRP Redirect messages |
| 11 | Configure optional hub MTU and MSS | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Tunnel accounts for GRE/IPsec overhead |
| 12 | Configure optional hub tunnel key | Hub | See `Hub_Phase3_mGRE_NHRP_Config` | Hub uses the expected tunnel key |
| 13 | Build Phase 3 spoke tunnel | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spoke uses mGRE, not point-to-point GRE |
| 14 | Configure spoke tunnel IP | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Each spoke has a unique overlay tunnel address |
| 15 | Configure spoke tunnel source | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spoke sources GRE from correct underlay address |
| 16 | Configure spoke mGRE mode | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spoke can support dynamic direct spoke-to-spoke tunnels |
| 17 | Enable NHRP on each spoke | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spoke participates in the NHRP cloud |
| 18 | Configure hub as NHS on each spoke | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spoke registers to the hub NHS |
| 19 | Configure spoke multicast mapping to hub | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Multicast-based routing protocols can reach the hub |
| 20 | Enable Phase 3 shortcut on spokes | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spokes can install NHRP shortcut entries |
| 21 | Configure optional spoke MTU and MSS | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Tunnel accounts for GRE/IPsec overhead |
| 22 | Configure optional spoke tunnel key | Spokes | See `Spoke_Phase3_mGRE_NHRP_Config` | Spoke tunnel key matches hub |
| 23 | Bring up tunnel interfaces | Hub, Spokes | `no shutdown` | Tunnel interfaces are administratively enabled |
| 24 | Verify tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 25 | Verify DMVPN peer state | Hub, Spokes | `show dmvpn` | Hub sees spokes, and spokes see hub |
| 26 | Verify NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spoke mappings and spokes have hub mapping |
| 27 | Verify redirect command on hub | Hub | `show running-config interface Tunnel<id>` | Hub has `ip nhrp redirect` |
| 28 | Verify shortcut command on spokes | Spokes | `show running-config interface Tunnel<id>` | Spokes have `ip nhrp shortcut` |
| 29 | Test spoke-to-hub tunnel reachability | Spokes | `ping <hub-tunnel-ip>` | Spokes can reach the hub tunnel IP |
| 30 | Test hub-to-spoke tunnel reachability | Hub | `ping <spoke-tunnel-ip>` | Hub can reach every spoke tunnel IP |
| 31 | Add routing overlay only after NHRP is stable | Hub, Spokes | Routing protocol config | Routes are exchanged across stable DMVPN |
| 32 | Configure hub route advertisement for Phase 3 | Hub | See `Phase3_Routing_Design_Reminder` | Hub can advertise summaries, defaults, or spoke routes |
| 33 | Confirm spoke route can point toward hub | Spokes | `show ip route <remote-spoke-lan-prefix>` | Route can point to hub, summary, or default |
| 34 | Trigger Phase 3 redirect and shortcut | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | First path may go through the hub |
| 35 | Verify shortcut NHRP entry appears | Spokes | `show ip nhrp` | Remote spoke or destination prefix appears dynamically |
| 36 | Verify CEF shortcut forwarding | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves toward the remote spoke path |
| 37 | Repeat traceroute after shortcut forms | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Later path should go directly to remote spoke |
| 38 | Confirm Phase 3 benefit over Phase 2 | Spokes | `show ip route` and `show ip nhrp` | Routing can point to hub while forwarding optimizes direct |
| 39 | Stop before adding IPsec or QoS if shortcuts are unstable | Hub, Spokes | `show dmvpn` and `show ip nhrp` | Phase 3 is stable before extra services are layered on |

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Skeleton

```text
# Hub_Phase3_mGRE_NHRP_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE3_HUB
 bandwidth <kbps>
 ip address <hub-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id <nhrp-id>
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source <hub-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Hub_Phase3_Optional_NHRP_Authentication
conf t
interface Tunnel<id>
 ip nhrp authentication <key-string>
end
```

```text
# Spoke_Phase3_mGRE_NHRP_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE3_SPOKE
 bandwidth <kbps>
 ip address <spoke-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source <spoke-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Spoke_Phase3_Separate_NHRP_Mapping_Alternative
conf t
interface Tunnel<id>
 ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 ip nhrp map multicast <hub-nbma-ip>
 ip nhrp nhs <hub-tunnel-ip>
end
```

```text
# Spoke_Phase3_Optional_NHRP_Authentication
conf t
interface Tunnel<id>
 ip nhrp authentication <key-string>
end
```

```text
# Phase3_Do_Not_Use_On_Spokes
! Do not configure this on Phase 3 spokes:
interface Tunnel<id>
 tunnel destination <hub-nbma-ip>
end
```

```text
# Phase3_Routing_Design_Reminder
! Phase 3 allows summarization or default routing from the hub.
! Unlike Phase 2, the spoke route does not need to preserve the remote spoke as next hop.
! Hub redirect and spoke shortcut handle the forwarding optimization after traffic is triggered.
```

```text
# Optional_EIGRP_Phase3_Hub_Adjustment
conf t
interface Tunnel<id>
 no ip split-horizon eigrp <asn>
 ip summary-address eigrp <asn> <summary-prefix> <summary-mask>
end
```

```text
# Optional_EIGRP_Phase3_Default_From_Hub
conf t
interface Tunnel<id>
 ip summary-address eigrp <asn> 0.0.0.0 0.0.0.0
end
```

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| Transport route | Hub, Spokes | `show ip route <remote-nbma-ip>` | Route points through underlay |
| Tunnel state | Hub, Spokes | `show ip interface brief` | Tunnel interface shows `up/up` |
| Hub tunnel mode | Hub | `show running-config interface Tunnel<id>` | Hub has `tunnel mode gre multipoint` |
| Spoke tunnel mode | Spokes | `show running-config interface Tunnel<id>` | Spoke has `tunnel mode gre multipoint` |
| Spoke destination exclusion | Spokes | `show running-config interface Tunnel<id>` | No `tunnel destination` exists on Phase 3 spokes |
| Hub redirect | Hub | `show running-config interface Tunnel<id>` | Hub has `ip nhrp redirect` |
| Spoke shortcut | Spokes | `show running-config interface Tunnel<id>` | Spokes have `ip nhrp shortcut` |
| DMVPN peer state | Hub, Spokes | `show dmvpn` | Hub-spoke peers show up |
| NHRP cache | Hub, Spokes | `show ip nhrp` | Expected mappings exist |
| NHRP brief | Hub, Spokes | `show ip nhrp brief` | Hub has dynamic spokes and spokes have hub mapping |
| Spoke-to-hub ping | Spokes | `ping <hub-tunnel-ip>` | Ping succeeds |
| Hub-to-spoke ping | Hub | `ping <spoke-tunnel-ip>` | Ping succeeds |
| Route toward remote LAN | Spokes | `show ip route <remote-spoke-lan-prefix>` | Route may point to hub, summary, or default |
| First path trigger | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | First path may go through hub |
| Shortcut NHRP entry | Spokes | `show ip nhrp` | Dynamic shortcut entry appears after traffic |
| CEF shortcut | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves toward remote spoke |
| Second path proof | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Later path goes directly to remote spoke |
| Summary/default proof | Spokes | `show ip route 0.0.0.0` | Default or summary can exist while shortcuts still work |
| Final stability | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN remains stable after shortcut testing |

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Shut down spoke tunnels first | Spokes | `shutdown` under tunnel | Spokes stop registering and forming shortcuts |
| 2 | Remove spoke shortcut | Spokes | See `Rollback_Spoke_Phase3_NHRP` | Spoke no longer installs Phase 3 shortcuts |
| 3 | Remove spoke NHS compact command | Spokes | See `Rollback_Spoke_Phase3_NHRP` | Spoke no longer registers to hub |
| 4 | Remove separate NHRP mappings if used | Spokes | See `Rollback_Spoke_Phase3_NHRP` | Static hub mapping is removed |
| 5 | Remove spoke NHRP authentication if used | Spokes | See `Rollback_Spoke_Phase3_NHRP` | Authentication is removed |
| 6 | Remove spoke NHRP network ID | Spokes | See `Rollback_Spoke_Phase3_NHRP` | NHRP is disabled on spoke tunnel |
| 7 | Remove spoke mGRE mode | Spokes | See `Rollback_Spoke_Phase3_Tunnel` | Spoke no longer acts as mGRE endpoint |
| 8 | Remove spoke tunnel source and key | Spokes | See `Rollback_Spoke_Phase3_Tunnel` | Spoke tunnel source and key are cleared |
| 9 | Remove spoke tunnel IP | Spokes | See `Rollback_Spoke_Phase3_Tunnel` | Spoke overlay address is removed |
| 10 | Remove Phase 3 summary/default if configured | Hub | See `Rollback_Phase3_Routing_Adjustments` | Hub stops advertising Phase 3 summary/default |
| 11 | Shut down hub tunnel | Hub | `shutdown` under tunnel | Hub stops accepting spoke registrations |
| 12 | Remove hub redirect | Hub | See `Rollback_Hub_Phase3_NHRP` | Hub no longer sends NHRP redirects |
| 13 | Remove hub dynamic multicast mapping | Hub | See `Rollback_Hub_Phase3_NHRP` | Hub multicast mapping is removed |
| 14 | Remove hub NHRP authentication if used | Hub | See `Rollback_Hub_Phase3_NHRP` | Authentication is removed |
| 15 | Remove hub NHRP network ID | Hub | See `Rollback_Hub_Phase3_NHRP` | NHRP is disabled on hub tunnel |
| 16 | Remove hub mGRE mode | Hub | See `Rollback_Hub_Phase3_Tunnel` | Hub no longer acts as mGRE endpoint |
| 17 | Remove hub tunnel source and key | Hub | See `Rollback_Hub_Phase3_Tunnel` | Hub tunnel source and key are cleared |
| 18 | Remove hub tunnel IP | Hub | See `Rollback_Hub_Phase3_Tunnel` | Hub overlay address is removed |
| 19 | Delete tunnel interface if fully resetting | Hub, Spokes | `no interface Tunnel<id>` | Tunnel interface is removed |

```text
# Rollback_Spoke_Phase3_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp shortcut
 no ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 no ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 no ip nhrp map multicast <hub-nbma-ip>
 no ip nhrp nhs <hub-tunnel-ip>
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Spoke_Phase3_Tunnel
conf t
interface Tunnel<id>
 no tunnel mode gre multipoint
 no tunnel source <spoke-underlay-interface-or-nbma-ip>
 no tunnel key <key-id>
 no ip mtu 1400
 no ip tcp adjust-mss 1360
 no ip address
end
```

```text
# Rollback_Hub_Phase3_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp redirect
 no ip nhrp map multicast dynamic
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Hub_Phase3_Tunnel
conf t
interface Tunnel<id>
 no tunnel mode gre multipoint
 no tunnel source <hub-underlay-interface-or-nbma-ip>
 no tunnel key <key-id>
 no ip mtu 1400
 no ip tcp adjust-mss 1360
 no ip address
end
```

```text
# Rollback_Phase3_Routing_Adjustments
conf t
interface Tunnel<id>
 no ip summary-address eigrp <asn> <summary-prefix> <summary-mask>
 no ip summary-address eigrp <asn> 0.0.0.0 0.0.0.0
end
```

```text
# Clear_NHRP_For_Retest
clear ip nhrp
```

# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Spoke tunnel never reaches hub | Underlay or hub mapping is wrong | `ping <hub-nbma-ip>` | Fix underlay reachability and NHS mapping |
| Hub does not learn spokes | Spokes are not registering | `show ip nhrp` on hub | Fix spoke NHS, NHRP ID, authentication, or tunnel source |
| Spoke has no hub mapping | Missing NHS or static map | `show ip nhrp` | Add hub NHS or static hub mapping |
| Spoke has `tunnel destination` | Phase 1 spoke config was used | `show running-config interface Tunnel<id>` | Remove destination and use mGRE |
| Hub lacks redirect | Phase 3 command missing | `show running-config interface Tunnel<id>` | Add `ip nhrp redirect` on hub |
| Spoke lacks shortcut | Phase 3 command missing | `show running-config interface Tunnel<id>` | Add `ip nhrp shortcut` on spokes |
| First traceroute hits hub forever | Shortcut is not being installed | `show ip nhrp` and `show ip cef <destination>` | Fix redirect, shortcut, NHRP, or routing trigger |
| Route exists but shortcut fails | NHRP resolution or CEF rewrite problem | `show ip nhrp` and `show ip cef <destination>` | Clear NHRP and verify shortcut behavior again |
| Summary route exists but traffic fails | Underlay or NHRP shortcut is broken | `show ip route` and `show ip nhrp` | Fix DMVPN Phase 3 state before blaming summary |
| Default route breaks NBMA reachability | Transport NBMA route points into tunnel | `show ip route <remote-nbma-ip>` | Add specific underlay routes for NBMA addresses |
| Phase 2 behavior mistaken for Phase 3 | Routing preserves remote spoke next hop | `show ip route <remote-spoke-lan-prefix>` | Validate redirect/shortcut using summary or default route |
| Routing protocol fails | Missing NHRP multicast mapping | `show running-config interface Tunnel<id>` | Add hub dynamic multicast and spoke multicast mapping |
| NHRP authentication fails | Key mismatch | `show running-config interface Tunnel<id>` | Match or remove NHRP authentication |
| Tunnel key mismatch | Peers fail despite underlay reachability | `show running-config interface Tunnel<id>` | Match or remove tunnel keys |
| Large packets fail | MTU or MSS issue | `ping <tunnel-ip> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames routing too early | Phase 3 NHRP core is unstable | `show dmvpn` and `show ip nhrp` | Fix DMVPN redirect/shortcut behavior first |

##### Source_Basis
# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Mental_Model
# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Configuration_Checklist
# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Skeleton
# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Verification_Commands
# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Rollback
# DMVPN_Phase_3_NHRP_Redirect_Shortcut_Failure_Checks