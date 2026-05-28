# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Phase 2 | DMVPN model where hub and spokes all use mGRE |
| Direct spoke-to-spoke | Spokes can build dynamic tunnels directly to other spokes |
| Hub role | Hub is the NHS and helps spokes resolve other spokes |
| Spoke role | Spokes register to the hub and dynamically resolve remote spokes |
| First packet behavior | Initial traffic may trigger NHRP resolution through the hub |
| Steady-state behavior | After NHRP resolution, traffic should flow directly spoke to spoke |
| Spoke tunnel mode | Phase 2 spokes use `tunnel mode gre multipoint` |
| No spoke tunnel destination | Phase 2 spokes do not use `tunnel destination <hub-nbma-ip>` |
| Routing dependency | Remote spoke routes must preserve the remote spoke tunnel IP as next hop |
| Summarization problem | Hub summaries or default-only routes break Phase 2 direct spoke-to-spoke behavior |
| Phase 3 exclusion | Phase 2 does not use `ip nhrp redirect` or `ip nhrp shortcut` |
| Hard rule | If the route points to the hub, Phase 2 will not give true direct spoke-to-spoke forwarding |

# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before DMVPN configuration |
| 2 | Confirm tunnel source interface state | Hub, Spokes | `show ip interface brief` | Tunnel source interface is `up/up` |
| 3 | Confirm transport route to remote NBMA | Hub, Spokes | `show ip route <remote-nbma-ip>` | Remote NBMA resolves through the underlay |
| 4 | Build Phase 2 hub tunnel | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub uses mGRE and participates in NHRP |
| 5 | Configure hub tunnel IP | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub has an overlay tunnel address |
| 6 | Configure hub tunnel source | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub sources GRE from the correct underlay address |
| 7 | Configure hub mGRE mode | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub can support multiple dynamic peers |
| 8 | Enable NHRP on hub | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub participates in the NHRP cloud |
| 9 | Enable dynamic multicast mapping on hub | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub supports multicast from registered spokes |
| 10 | Exclude Phase 3 redirect on hub | Hub | See `Hub_Phase2_No_Phase3_Config` | Hub does not send NHRP redirects |
| 11 | Configure optional hub MTU and MSS | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Tunnel accounts for GRE/IPsec overhead |
| 12 | Configure optional hub tunnel key | Hub | See `Hub_Phase2_mGRE_NHRP_Config` | Hub uses the expected tunnel key |
| 13 | Build Phase 2 spoke tunnel | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Spoke uses mGRE, not point-to-point GRE |
| 14 | Configure spoke tunnel IP | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Each spoke has a unique overlay tunnel address |
| 15 | Configure spoke tunnel source | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Spoke sources GRE from correct underlay address |
| 16 | Configure spoke mGRE mode | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Spoke can form dynamic direct tunnels |
| 17 | Enable NHRP on each spoke | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Spoke participates in the NHRP cloud |
| 18 | Configure hub as NHS on each spoke | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Spoke registers to the hub NHS |
| 19 | Configure spoke multicast mapping to hub | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Multicast-based routing protocols can reach the hub |
| 20 | Exclude Phase 3 shortcut on spokes | Spokes | See `Spoke_Phase2_No_Phase3_Config` | Spokes do not install Phase 3 shortcuts |
| 21 | Configure optional spoke MTU and MSS | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Tunnel accounts for GRE/IPsec overhead |
| 22 | Configure optional spoke tunnel key | Spokes | See `Spoke_Phase2_mGRE_NHRP_Config` | Spoke tunnel key matches hub |
| 23 | Bring up tunnel interfaces | Hub, Spokes | `no shutdown` | Tunnel interfaces are administratively enabled |
| 24 | Verify tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 25 | Verify DMVPN peer state | Hub, Spokes | `show dmvpn` | Hub sees spokes, and spokes see hub |
| 26 | Verify NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spoke mappings and spokes have hub mapping |
| 27 | Test spoke-to-hub tunnel reachability | Spokes | `ping <hub-tunnel-ip>` | Spokes can reach the hub tunnel IP |
| 28 | Test hub-to-spoke tunnel reachability | Hub | `ping <spoke-tunnel-ip>` | Hub can reach every spoke tunnel IP |
| 29 | Trigger spoke-to-spoke NHRP resolution | Spokes | `ping <other-spoke-tunnel-ip>` | Spoke learns dynamic mapping for remote spoke |
| 30 | Verify dynamic spoke-to-spoke NHRP entry | Spokes | `show ip nhrp` | Remote spoke tunnel IP maps to remote spoke NBMA IP |
| 31 | Verify direct spoke-to-spoke tunnel path | Spokes | `traceroute <other-spoke-tunnel-ip> numeric` | Path should resolve directly after NHRP resolution |
| 32 | Add routing overlay only after NHRP is stable | Hub, Spokes | Routing protocol config | Routes are exchanged across stable DMVPN |
| 33 | Verify route next hop for remote spoke LANs | Spokes | `show ip route <remote-spoke-lan-prefix>` | Next hop is remote spoke tunnel IP, not hub tunnel IP |
| 34 | Verify CEF forwarding toward remote spoke | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding resolves through remote spoke tunnel/NBMA path |
| 35 | Prove LAN spoke-to-spoke forwarding | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Path uses direct spoke-to-spoke forwarding |
| 36 | Stop if routes point to hub | Spokes | `show ip route <remote-spoke-lan-prefix>` | Routing policy must be fixed before blaming NHRP |

# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Skeleton

```text
# Hub_Phase2_mGRE_NHRP_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE2_HUB
 bandwidth <kbps>
 ip address <hub-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id <nhrp-id>
 ip tcp adjust-mss 1360
 tunnel source <hub-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Hub_Phase2_Optional_NHRP_Authentication
conf t
interface Tunnel<id>
 ip nhrp authentication <key-string>
end
```

```text
# Hub_Phase2_No_Phase3_Config
conf t
interface Tunnel<id>
 no ip nhrp redirect
end
```

```text
# Spoke_Phase2_mGRE_NHRP_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE2_SPOKE
 bandwidth <kbps>
 ip address <spoke-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 ip tcp adjust-mss 1360
 tunnel source <spoke-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Spoke_Phase2_Separate_NHRP_Mapping_Alternative
conf t
interface Tunnel<id>
 ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 ip nhrp map multicast <hub-nbma-ip>
 ip nhrp nhs <hub-tunnel-ip>
end
```

```text
# Spoke_Phase2_Optional_NHRP_Authentication
conf t
interface Tunnel<id>
 ip nhrp authentication <key-string>
end
```

```text
# Spoke_Phase2_No_Phase3_Config
conf t
interface Tunnel<id>
 no ip nhrp shortcut
end
```

```text
# Phase2_Do_Not_Use_On_Spokes
! Do not configure this on Phase 2 spokes:
interface Tunnel<id>
 tunnel destination <hub-nbma-ip>
end
```

```text
# Phase2_Routing_Design_Reminder
! Phase 2 requires the remote spoke route next hop to stay as the remote spoke tunnel IP.
! Do not summarize spoke LAN prefixes at the hub if that makes spokes point to the hub.
! Do not advertise only a default route from the hub if direct spoke-to-spoke forwarding is required.
```

# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| Transport route | Hub, Spokes | `show ip route <remote-nbma-ip>` | Route points through underlay |
| Tunnel state | Hub, Spokes | `show ip interface brief` | Tunnel interface shows `up/up` |
| Hub tunnel mode | Hub | `show running-config interface Tunnel<id>` | Hub has `tunnel mode gre multipoint` |
| Spoke tunnel mode | Spokes | `show running-config interface Tunnel<id>` | Spoke has `tunnel mode gre multipoint` |
| Spoke destination exclusion | Spokes | `show running-config interface Tunnel<id>` | No `tunnel destination` exists on Phase 2 spokes |
| Hub Phase 3 exclusion | Hub | `show running-config interface Tunnel<id>` | No `ip nhrp redirect` exists |
| Spoke Phase 3 exclusion | Spokes | `show running-config interface Tunnel<id>` | No `ip nhrp shortcut` exists |
| DMVPN peer state | Hub, Spokes | `show dmvpn` | Hub-spoke peers show up |
| NHRP cache | Hub, Spokes | `show ip nhrp` | Expected mappings exist |
| NHRP brief | Hub, Spokes | `show ip nhrp brief` | Hub has dynamic spokes and spokes have hub mapping |
| Spoke-to-hub ping | Spokes | `ping <hub-tunnel-ip>` | Ping succeeds |
| Hub-to-spoke ping | Hub | `ping <spoke-tunnel-ip>` | Ping succeeds |
| Spoke-to-spoke trigger | Spokes | `ping <other-spoke-tunnel-ip>` | Ping succeeds and triggers NHRP |
| Dynamic spoke cache | Spokes | `show ip nhrp` | Remote spoke appears as dynamic NHRP entry |
| Tunnel path proof | Spokes | `traceroute <other-spoke-tunnel-ip> numeric` | Path resolves directly after NHRP resolution |
| Remote LAN route next hop | Spokes | `show ip route <remote-spoke-lan-prefix>` | Next hop is remote spoke tunnel IP |
| CEF forwarding | Spokes | `show ip cef <remote-spoke-lan-ip>` | Forwarding points toward remote spoke path |
| LAN path proof | Spokes | `traceroute <remote-spoke-lan-ip> numeric` | Traffic goes directly to remote spoke after resolution |

# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Shut down spoke tunnels first | Spokes | `shutdown` under tunnel | Spokes stop registering and resolving peers |
| 2 | Remove spoke NHS compact command | Spokes | See `Rollback_Spoke_Phase2_NHRP` | Spoke no longer registers to hub |
| 3 | Remove separate NHRP mappings if used | Spokes | See `Rollback_Spoke_Phase2_NHRP` | Static hub mapping is removed |
| 4 | Remove spoke NHRP authentication if used | Spokes | See `Rollback_Spoke_Phase2_NHRP` | Authentication is removed |
| 5 | Remove spoke NHRP network ID | Spokes | See `Rollback_Spoke_Phase2_NHRP` | NHRP is disabled on spoke tunnel |
| 6 | Remove spoke mGRE mode | Spokes | See `Rollback_Spoke_Phase2_Tunnel` | Spoke no longer acts as mGRE endpoint |
| 7 | Remove spoke tunnel source and key | Spokes | See `Rollback_Spoke_Phase2_Tunnel` | Spoke tunnel source and key are cleared |
| 8 | Remove spoke tunnel IP | Spokes | See `Rollback_Spoke_Phase2_Tunnel` | Spoke overlay address is removed |
| 9 | Shut down hub tunnel | Hub | `shutdown` under tunnel | Hub stops accepting spoke registrations |
| 10 | Remove hub dynamic multicast mapping | Hub | See `Rollback_Hub_Phase2_NHRP` | Hub multicast mapping is removed |
| 11 | Remove hub NHRP authentication if used | Hub | See `Rollback_Hub_Phase2_NHRP` | Authentication is removed |
| 12 | Remove hub NHRP network ID | Hub | See `Rollback_Hub_Phase2_NHRP` | NHRP is disabled on hub tunnel |
| 13 | Remove hub mGRE mode | Hub | See `Rollback_Hub_Phase2_Tunnel` | Hub no longer acts as mGRE endpoint |
| 14 | Remove hub tunnel source and key | Hub | See `Rollback_Hub_Phase2_Tunnel` | Hub tunnel source and key are cleared |
| 15 | Remove hub tunnel IP | Hub | See `Rollback_Hub_Phase2_Tunnel` | Hub overlay address is removed |
| 16 | Delete tunnel interface if fully resetting | Hub, Spokes | `no interface Tunnel<id>` | Tunnel interface is removed |

```text
# Rollback_Spoke_Phase2_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
 no ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 no ip nhrp map multicast <hub-nbma-ip>
 no ip nhrp nhs <hub-tunnel-ip>
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Spoke_Phase2_Tunnel
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
# Rollback_Hub_Phase2_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp map multicast dynamic
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Hub_Phase2_Tunnel
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
# Clear_NHRP_For_Retest
clear ip nhrp
```

# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Spoke tunnel never reaches hub | Underlay or hub mapping is wrong | `ping <hub-nbma-ip>` | Fix underlay reachability and NHS mapping |
| Hub does not learn spokes | Spokes are not registering | `show ip nhrp` on hub | Fix spoke NHS, NHRP ID, authentication, or tunnel source |
| Spoke has no hub mapping | Missing NHS or static map | `show ip nhrp` | Add hub NHS or static hub mapping |
| Spoke has `tunnel destination` | Phase 1 spoke config was used | `show running-config interface Tunnel<id>` | Remove destination and use mGRE |
| Spoke-to-spoke ping fails | NHRP dynamic resolution is not working | `show ip nhrp` | Fix mGRE, NHS, and NBMA reachability |
| Spoke-to-spoke path hairpins through hub | Route next hop points to hub | `show ip route <remote-spoke-lan-prefix>` | Fix routing next-hop behavior |
| Route points to hub summary | Hub summarization broke Phase 2 | `show ip route <remote-spoke-lan-prefix>` | Remove summary or use Phase 3 |
| Default route points to hub | Default-only design is not Phase 2 friendly | `show ip route 0.0.0.0` | Advertise specific spoke routes or use Phase 3 |
| Phase 3 accidentally enabled | Redirect or shortcut exists | `show running-config interface Tunnel<id>` | Remove `ip nhrp redirect` and `ip nhrp shortcut` |
| Multicast routing protocol fails | Missing NHRP multicast mapping | `show running-config interface Tunnel<id>` | Add hub dynamic multicast and spoke multicast mapping |
| NHRP authentication fails | Key mismatch | `show running-config interface Tunnel<id>` | Match or remove NHRP authentication |
| Tunnel key mismatch | Peers fail despite underlay reachability | `show running-config interface Tunnel<id>` | Match or remove tunnel keys |
| Large packets fail | MTU or MSS issue | `ping <tunnel-ip> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames NHRP too early | Routing next hop is actually wrong | `show ip nhrp` and `show ip route <prefix>` | Separate tunnel resolution from routing policy |

##### Source_Basis
# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Mental_Model
# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Configuration_Checklist
# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Skeleton
# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Verification_Commands
# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Rollback
# DMVPN_Phase_2_Direct_Spoke_to_Spoke_Failure_Checks

