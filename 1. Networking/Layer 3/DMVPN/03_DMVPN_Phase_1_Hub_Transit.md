# DMVPN_Phase_1_Hub_Transit_Mental_Model

| Concept               | Operational Meaning                                                                    |
| --------------------- | -------------------------------------------------------------------------------------- |
| Phase 1               | Hub-and-spoke DMVPN only                                                               |
| Hub role              | Hub uses mGRE and acts as the NHRP NHS                                                 |
| Spoke role            | Spokes use point-to-point GRE toward the hub NBMA address                              |
| Hub transit           | Spoke-to-spoke traffic must pass through the hub                                       |
| No spoke shortcut     | Spokes do not build direct dynamic tunnels to other spokes                             |
| No spoke mGRE         | Phase 1 spokes use `tunnel destination`, not `tunnel mode gre multipoint`              |
| Static mapping model  | Hub manually maps each spoke tunnel IP to each spoke NBMA IP                           |
| Dynamic mapping model | Spokes register to the hub with `ip nhrp nhs`                                          |
| Multicast mapping     | Optional for basic tunnel reachability, required for multicast-based routing protocols |
| Routing implication   | Spokes should learn remote spoke prefixes with the hub as the forwarding path          |
| Verification truth    | A spoke-to-spoke traceroute should show the hub as the intermediate tunnel hop         |
| Hard rule             | If a spoke bypasses the hub, you are not validating Phase 1 behavior                   |

# DMVPN_Phase_1_Hub_Transit_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Transport reachability works before tunnel configuration |
| 2 | Confirm tunnel source interface state | Hub, Spokes | `show ip interface brief` | Tunnel source interface is `up/up` |
| 3 | Confirm route to remote NBMA addresses | Hub, Spokes | `show ip route <remote-nbma-ip>` | Remote NBMA resolves through the underlay |
| 4 | Configure the Phase 1 hub tunnel | Hub | See `Hub_Phase1_mGRE_Base_Config` | Hub uses mGRE and has the tunnel IP |
| 5 | Enable NHRP on the hub | Hub | See `Hub_Phase1_mGRE_Base_Config` | Hub participates as the NHRP server-side device |
| 6 | Configure static spoke mappings if using static Phase 1 | Hub | See `Hub_Phase1_Static_NHRP_Mappings` | Hub has static tunnel-to-NBMA mappings for every spoke |
| 7 | Configure dynamic multicast mapping if using dynamic registration or multicast-based routing | Hub | See `Hub_Phase1_Dynamic_NHRP_Options` | Hub can learn/register spoke mappings and support multicast |
| 8 | Configure optional MTU and MSS on hub | Hub | See `Hub_Phase1_mGRE_Base_Config` | Tunnel accounts for GRE/IPsec overhead |
| 9 | Configure optional tunnel key on hub | Hub | See `Hub_Phase1_mGRE_Base_Config` | Hub uses the expected tunnel key |
| 10 | Configure each spoke as point-to-point GRE | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Spoke tunnel points directly to hub NBMA |
| 11 | Configure each spoke tunnel IP | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Each spoke has a unique overlay tunnel address |
| 12 | Configure each spoke tunnel source | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Spoke sources GRE from correct underlay interface or IP |
| 13 | Configure each spoke tunnel destination | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Spoke tunnel destination is the hub NBMA address |
| 14 | Enable NHRP on each spoke | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Spoke participates in NHRP |
| 15 | Configure spoke static hub mapping if using static NHRP | Spokes | See `Spoke_Phase1_Static_Hub_NHRP_Map` | Spoke maps hub tunnel IP to hub NBMA IP |
| 16 | Configure hub as NHS if using dynamic registration | Spokes | See `Spoke_Phase1_Dynamic_NHS_Config` | Spoke registers with the hub NHS |
| 17 | Configure optional spoke multicast mapping to hub | Spokes | See `Spoke_Phase1_Multicast_Option` | Multicast routing protocols can reach the hub |
| 18 | Configure optional MTU and MSS on spokes | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Tunnel accounts for GRE/IPsec overhead |
| 19 | Configure optional tunnel key on spokes | Spokes | See `Spoke_Phase1_P2P_GRE_Config` | Spoke tunnel key matches the hub |
| 20 | Bring up all tunnel interfaces | Hub, Spokes | `no shutdown` | Tunnel interfaces are administratively enabled |
| 21 | Verify tunnel interface state | Hub, Spokes | `show ip interface brief | include Tunnel` | Tunnel interfaces show `up/up` |
| 22 | Verify NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub and spokes show expected NHRP mappings |
| 23 | Verify DMVPN peer state | Hub, Spokes | `show dmvpn` | Hub sees spokes and spokes see hub |
| 24 | Test spoke-to-hub tunnel reachability | Spokes | `ping <hub-tunnel-ip>` | Spokes can reach the hub tunnel IP |
| 25 | Test hub-to-spoke tunnel reachability | Hub | `ping <spoke-tunnel-ip>` | Hub can reach every spoke tunnel IP |
| 26 | Test spoke-to-spoke tunnel reachability | Spokes | `ping <other-spoke-tunnel-ip>` | Spokes can reach each other through the hub |
| 27 | Prove spoke-to-spoke traffic transits hub | Spokes | `traceroute <other-spoke-tunnel-ip> numeric` | Hub tunnel IP appears as the intermediate hop |
| 28 | Stop before adding routing if Phase 1 behavior is wrong | Hub, Spokes | `show dmvpn` and `show ip nhrp` | Routing protocols are not added until Phase 1 behavior is stable |

# DMVPN_Phase_1_Hub_Transit_Skeleton

```text
# Hub_Phase1_mGRE_Base_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE1_HUB
 bandwidth <kbps>
 ip address <hub-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip tcp adjust-mss 1360
 tunnel source <hub-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Hub_Phase1_Static_NHRP_Mappings
conf t
interface Tunnel<id>
 ip nhrp map <spoke1-tunnel-ip> <spoke1-nbma-ip>
 ip nhrp map <spoke2-tunnel-ip> <spoke2-nbma-ip>
 ip nhrp map <spoke3-tunnel-ip> <spoke3-nbma-ip>
end
```

```text
# Hub_Phase1_Static_NHRP_Multicast_Optional
conf t
interface Tunnel<id>
 ip nhrp map multicast <spoke1-nbma-ip>
 ip nhrp map multicast <spoke2-nbma-ip>
 ip nhrp map multicast <spoke3-nbma-ip>
end
```

```text
# Hub_Phase1_Dynamic_NHRP_Options
conf t
interface Tunnel<id>
 ip nhrp map multicast dynamic
end
```

```text
# Spoke_Phase1_P2P_GRE_Config
conf t
interface Tunnel<id>
 description DMVPN_PHASE1_SPOKE
 bandwidth <kbps>
 ip address <spoke-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip tcp adjust-mss 1360
 tunnel source <spoke-underlay-interface-or-nbma-ip>
 tunnel destination <hub-nbma-ip>
 tunnel key <key-id>
 no shutdown
end
```

```text
# Spoke_Phase1_Static_Hub_NHRP_Map
conf t
interface Tunnel<id>
 ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
end
```

```text
# Spoke_Phase1_Dynamic_NHS_Config
conf t
interface Tunnel<id>
 ip nhrp nhs <hub-tunnel-ip>
end
```

```text
# Spoke_Phase1_Multicast_Option
conf t
interface Tunnel<id>
 ip nhrp map multicast <hub-nbma-ip>
end
```

```text
# Phase1_Do_Not_Use_On_Spokes
! Do not configure this on Phase 1 spokes:
interface Tunnel<id>
 tunnel mode gre multipoint
end
```

# DMVPN_Phase_1_Hub_Transit_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| Transport route | Hub, Spokes | `show ip route <remote-nbma-ip>` | Route points through underlay |
| Tunnel state | Hub, Spokes | `show ip interface brief | include Tunnel` | Tunnel interface shows `up/up` |
| Hub tunnel mode | Hub | `show running-config interface Tunnel<id>` | Hub has `tunnel mode gre multipoint` |
| Spoke tunnel mode | Spokes | `show running-config interface Tunnel<id>` | Spoke has `tunnel destination <hub-nbma-ip>` |
| Spoke mGRE exclusion | Spokes | `show running-config interface Tunnel<id> \| include multipoint` | No mGRE tunnel mode appears on Phase 1 spoke |
| Hub NHRP cache | Hub | `show ip nhrp` | Hub has static or dynamic mappings for spokes |
| Spoke NHRP cache | Spokes | `show ip nhrp` | Spoke has mapping or NHS relationship to hub |
| DMVPN peer state | Hub, Spokes | `show dmvpn` | Hub-spoke peers show up |
| Hub-to-spoke tunnel ping | Hub | `ping <spoke-tunnel-ip>` | Ping succeeds |
| Spoke-to-hub tunnel ping | Spokes | `ping <hub-tunnel-ip>` | Ping succeeds |
| Spoke-to-spoke tunnel ping | Spokes | `ping <other-spoke-tunnel-ip>` | Ping succeeds through hub |
| Hub transit proof | Spokes | `traceroute <other-spoke-tunnel-ip> numeric` | Hub tunnel IP appears between spokes |
| Routing readiness | Hub, Spokes | `show dmvpn` and `show ip nhrp` | Phase 1 is stable before IGP or BGP is added |

# DMVPN_Phase_1_Hub_Transit_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Shut down spoke tunnels first | Spokes | `shutdown` under tunnel | Spokes stop forming tunnel state |
| 2 | Remove spoke NHS if used | Spokes | See `Rollback_Spoke_Phase1_NHRP` | Spoke no longer registers to hub |
| 3 | Remove spoke static hub mapping if used | Spokes | See `Rollback_Spoke_Phase1_NHRP` | Spoke no longer has static hub NHRP map |
| 4 | Remove spoke multicast mapping if used | Spokes | See `Rollback_Spoke_Phase1_NHRP` | Spoke no longer maps multicast to hub |
| 5 | Remove spoke point-to-point tunnel destination | Spokes | See `Rollback_Spoke_Phase1_Tunnel` | Spoke no longer points tunnel to hub NBMA |
| 6 | Remove spoke tunnel source and key | Spokes | See `Rollback_Spoke_Phase1_Tunnel` | Spoke tunnel source and key are cleared |
| 7 | Remove spoke tunnel IP and NHRP ID | Spokes | See `Rollback_Spoke_Phase1_Tunnel` | Spoke overlay tunnel config is cleared |
| 8 | Shut down hub tunnel | Hub | `shutdown` under tunnel | Hub stops accepting spoke tunnel traffic |
| 9 | Remove hub static spoke mappings if used | Hub | See `Rollback_Hub_Phase1_NHRP` | Hub static spoke mappings are removed |
| 10 | Remove hub dynamic multicast mapping if used | Hub | See `Rollback_Hub_Phase1_NHRP` | Hub dynamic multicast mapping is removed |
| 11 | Remove hub NHRP network ID | Hub | See `Rollback_Hub_Phase1_NHRP` | NHRP is disabled on hub tunnel |
| 12 | Remove hub mGRE mode | Hub | See `Rollback_Hub_Phase1_Tunnel` | Hub no longer acts as mGRE endpoint |
| 13 | Remove hub tunnel source and key | Hub | See `Rollback_Hub_Phase1_Tunnel` | Hub tunnel source and key are cleared |
| 14 | Remove hub tunnel IP | Hub | See `Rollback_Hub_Phase1_Tunnel` | Hub overlay tunnel address is removed |
| 15 | Delete tunnel interface if fully resetting | Hub, Spokes | `no interface Tunnel<id>` | Tunnel interface is removed |

```text
# Rollback_Spoke_Phase1_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp nhs <hub-tunnel-ip>
 no ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 no ip nhrp map multicast <hub-nbma-ip>
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Spoke_Phase1_Tunnel
conf t
interface Tunnel<id>
 no tunnel destination <hub-nbma-ip>
 no tunnel source <spoke-underlay-interface-or-nbma-ip>
 no tunnel key <key-id>
 no ip mtu 1400
 no ip tcp adjust-mss 1360
 no ip address
end
```

```text
# Rollback_Hub_Phase1_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp map <spoke1-tunnel-ip> <spoke1-nbma-ip>
 no ip nhrp map <spoke2-tunnel-ip> <spoke2-nbma-ip>
 no ip nhrp map <spoke3-tunnel-ip> <spoke3-nbma-ip>
 no ip nhrp map multicast <spoke1-nbma-ip>
 no ip nhrp map multicast <spoke2-nbma-ip>
 no ip nhrp map multicast <spoke3-nbma-ip>
 no ip nhrp map multicast dynamic
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Hub_Phase1_Tunnel
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

# DMVPN_Phase_1_Hub_Transit_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Spoke tunnel does not come up | Underlay cannot reach hub NBMA | `ping <hub-nbma-ip>` | Fix NBMA addressing, interface state, or routing |
| Hub shows no spoke NHRP entry | Static map missing or dynamic registration failing | `show ip nhrp` | Add static spoke map or fix spoke NHS |
| Spoke cannot reach hub tunnel IP | Missing hub mapping or wrong destination | `show running-config interface Tunnel<id>` | Correct tunnel destination or NHRP map |
| Hub cannot reach spoke tunnel IP | Hub lacks spoke tunnel-to-NBMA mapping | `show ip nhrp` | Add static mapping or fix spoke registration |
| Spoke has mGRE configured | Phase 2 or 3 spoke config was used by mistake | `show running-config interface Tunnel<id>` | Remove mGRE and configure `tunnel destination <hub-nbma-ip>` |
| Spoke-to-spoke path bypasses hub | Not behaving like Phase 1 | `traceroute <other-spoke-tunnel-ip> numeric` | Remove Phase 2/3 behavior and force hub transit |
| Routing protocol neighbors fail | Multicast mapping missing | `show running-config interface Tunnel<id>` | Add required NHRP multicast mapping |
| Dynamic registration fails | Spoke NHS missing or wrong hub tunnel IP | `show ip nhrp` | Configure `ip nhrp nhs <hub-tunnel-ip>` |
| Static mapping fails | Wrong tunnel IP to NBMA IP mapping | `show ip nhrp` | Correct static `ip nhrp map` entries |
| Tunnel key mismatch | Peers fail despite NBMA reachability | `show running-config interface Tunnel<id>` | Match or remove tunnel keys |
| Large packets fail | MTU or MSS issue | `ping <tunnel-ip> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer troubleshoots routing too early | Phase 1 tunnel/NHRP core is unstable | `show dmvpn` and `show ip nhrp` | Fix Phase 1 tunnel behavior first |

##### Source_Basis
# DMVPN_Phase_1_Hub_Transit_Mental_Model
# DMVPN_Phase_1_Hub_Transit_Configuration_Checklist
# DMVPN_Phase_1_Hub_Transit_Skeleton
# DMVPN_Phase_1_Hub_Transit_Verification_Commands
# DMVPN_Phase_1_Hub_Transit_Rollback
# DMVPN_Phase_1_Hub_Transit_Failure_Checks


