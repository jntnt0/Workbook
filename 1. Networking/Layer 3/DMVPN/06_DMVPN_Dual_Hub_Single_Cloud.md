


DMVPN_Dual_Hub_Single_Cloud.md

# DMVPN_Dual_Hub_Single_Cloud_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and hub/spoke behavior to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports mGRE, NHRP registration, tunnel IP to NBMA mapping, multicast mapping, and DMVPN phase behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Hub Redundancy | Supports multiple hubs in the same DMVPN cloud by adding multiple NHS mappings on spokes |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Tunnels | Supports `show dmvpn`, `show ip nhrp`, NHS verification, tunnel key, MTU/MSS, and Phase 3 behavior |

# DMVPN_Dual_Hub_Single_Cloud_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Dual-hub single-cloud | Two hubs exist inside one shared DMVPN cloud |
| Single cloud | One tunnel interface, one tunnel subnet, one NHRP domain, one logical DMVPN overlay |
| Hub role | Both hubs act as NHS devices for the same cloud |
| Spoke role | Each spoke registers to both hubs on the same tunnel interface |
| Redundancy model | If one hub fails, spokes can still use the other hub/NHS |
| Not dual-cloud | Do not create separate tunnel interfaces or separate tunnel subnets |
| Tunnel subnet | Hub 1, Hub 2, and all spokes share the same overlay tunnel network |
| NHRP network ID | Same value across the shared cloud for operational clarity |
| Tunnel key | Same key across all routers in the same cloud if a tunnel key is used |
| Routing domain | Both hubs and all spokes participate in the same overlay routing design |
| Phase 3 option | Both hubs use redirect and spokes use shortcut if this is Phase 3 |
| Hub preference | Routing metrics, BGP policy, or NHRP NHS priority can influence hub preference |
| Hard rule | If the design is single-cloud, every spoke should have one tunnel interface with two NHS entries |

# DMVPN_Dual_Hub_Single_Cloud_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm this is single-cloud, not dual-cloud | All Routers | Design check | One tunnel interface and one tunnel subnet are used |
| 2 | Confirm Hub 1 NBMA reachability | Spokes | `ping <hub1-nbma-ip>` | Spokes can reach Hub 1 over the underlay |
| 3 | Confirm Hub 2 NBMA reachability | Spokes | `ping <hub2-nbma-ip>` | Spokes can reach Hub 2 over the underlay |
| 4 | Confirm transport route to both hubs | Spokes | `show ip route <hub-nbma-ip>` | Hub NBMA routes point through the underlay |
| 5 | Confirm tunnel source interfaces are up | Hubs, Spokes | `show ip interface brief` | Tunnel source interfaces are `up/up` |
| 6 | Build Hub 1 shared-cloud tunnel | Hub 1 | See `Hub1_Single_Cloud_mGRE_NHRP_Config` | Hub 1 is an NHS-capable mGRE hub |
| 7 | Build Hub 2 shared-cloud tunnel | Hub 2 | See `Hub2_Single_Cloud_mGRE_NHRP_Config` | Hub 2 is an NHS-capable mGRE hub |
| 8 | Use same tunnel subnet on both hubs | Hub 1, Hub 2 | See hub config blocks | Both hubs are in the same overlay tunnel subnet |
| 9 | Use same NHRP network ID on both hubs | Hub 1, Hub 2 | See hub config blocks | Both hubs participate in the same NHRP cloud |
| 10 | Use same tunnel key on both hubs if configured | Hub 1, Hub 2 | See hub config blocks | Both hubs use the same shared-cloud tunnel key |
| 11 | Enable dynamic multicast mapping on both hubs | Hub 1, Hub 2 | See hub config blocks | Hubs support multicast from registered spokes |
| 12 | Enable Phase 3 redirect on both hubs if using Phase 3 | Hub 1, Hub 2 | See hub config blocks | Both hubs can send NHRP redirects |
| 13 | Configure optional inter-hub static NHRP if hubs need overlay adjacency | Hub 1, Hub 2 | See `Optional_InterHub_NHRP_Mapping` | Hubs can resolve each other across the same cloud |
| 14 | Build spoke shared-cloud tunnel | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Spoke uses one tunnel interface in the shared cloud |
| 15 | Configure spoke tunnel IP in shared subnet | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Spoke has one overlay address in the shared cloud |
| 16 | Configure spoke mGRE mode | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Spoke can use multipoint GRE |
| 17 | Configure Hub 1 as NHS | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Spoke registers to Hub 1 |
| 18 | Configure Hub 2 as NHS | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Spoke registers to Hub 2 |
| 19 | Configure spoke multicast mapping to both hubs | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Multicast-based routing can reach both hubs |
| 20 | Enable Phase 3 shortcut on spokes if using Phase 3 | Spokes | See `Spoke_Single_Cloud_Dual_NHS_Config` | Spokes can install shortcut forwarding |
| 21 | Configure optional NHS priority if supported | Spokes | See `Optional_NHS_Priority` | Spokes prefer the intended primary hub |
| 22 | Configure overlay routing on both hubs | Hub 1, Hub 2 | See `Overlay_Routing_Design_Reminder` | Both hubs participate in route exchange |
| 23 | Configure overlay routing on spokes | Spokes | See `Overlay_Routing_Design_Reminder` | Spokes learn routes through the shared cloud |
| 24 | Tune routing preference between hubs | Hubs, Spokes | Routing metric or BGP policy | Preferred hub path is deterministic |
| 25 | Verify spoke has both NHS entries | Spokes | `show ip nhrp nhs detail` | Hub 1 and Hub 2 appear as NHS devices |
| 26 | Verify Hub 1 learns spokes | Hub 1 | `show ip nhrp` | Hub 1 has dynamic spoke mappings |
| 27 | Verify Hub 2 learns spokes | Hub 2 | `show ip nhrp` | Hub 2 has dynamic spoke mappings |
| 28 | Verify DMVPN state on Hub 1 | Hub 1 | `show dmvpn` | Hub 1 sees spoke peers |
| 29 | Verify DMVPN state on Hub 2 | Hub 2 | `show dmvpn` | Hub 2 sees spoke peers |
| 30 | Verify spoke sees both hubs | Spokes | `show dmvpn` | Spoke sees both hubs in the same cloud |
| 31 | Test spoke reachability to Hub 1 tunnel IP | Spokes | `ping <hub1-tunnel-ip>` | Spoke can reach Hub 1 tunnel IP |
| 32 | Test spoke reachability to Hub 2 tunnel IP | Spokes | `ping <hub2-tunnel-ip>` | Spoke can reach Hub 2 tunnel IP |
| 33 | Verify routes through the preferred hub | Spokes | `show ip route <remote-prefix>` | Route uses intended hub or policy |
| 34 | Test Hub 1 failure behavior | Hub 1 | `shutdown` under tunnel | Spokes remain reachable through Hub 2 |
| 35 | Verify failover to Hub 2 | Spokes | `show dmvpn` and `show ip route` | Hub 2 remains active and routes are usable |
| 36 | Restore Hub 1 tunnel | Hub 1 | `no shutdown` under tunnel | Hub 1 returns to service |
| 37 | Test Hub 2 failure behavior | Hub 2 | `shutdown` under tunnel | Spokes remain reachable through Hub 1 |
| 38 | Restore Hub 2 tunnel | Hub 2 | `no shutdown` under tunnel | Hub 2 returns to service |
| 39 | Confirm final steady state | Hubs, Spokes | `show dmvpn`, `show ip nhrp`, `show ip route` | Both hubs and all spokes are stable |

# DMVPN_Dual_Hub_Single_Cloud_Skeleton

```text
# Hub1_Single_Cloud_mGRE_NHRP_Config
conf t
interface Tunnel<id>
 description DMVPN_SINGLE_CLOUD_HUB1
 bandwidth <kbps>
 ip address <hub1-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id <nhrp-id>
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source <hub1-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Hub2_Single_Cloud_mGRE_NHRP_Config
conf t
interface Tunnel<id>
 description DMVPN_SINGLE_CLOUD_HUB2
 bandwidth <kbps>
 ip address <hub2-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp map multicast dynamic
 ip nhrp network-id <nhrp-id>
 ip nhrp redirect
 ip tcp adjust-mss 1360
 tunnel source <hub2-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Hub_Optional_NHRP_Authentication
conf t
interface Tunnel<id>
 ip nhrp authentication <key-string>
end
```

```text
# Optional_InterHub_NHRP_Mapping
! Use only if the hubs need to resolve each other across the overlay.
! Not every lab needs this, but it is useful when hubs form overlay routing adjacencies.

! On Hub 1
conf t
interface Tunnel<id>
 ip nhrp map <hub2-tunnel-ip> <hub2-nbma-ip>
 ip nhrp map multicast <hub2-nbma-ip>
end

! On Hub 2
conf t
interface Tunnel<id>
 ip nhrp map <hub1-tunnel-ip> <hub1-nbma-ip>
 ip nhrp map multicast <hub1-nbma-ip>
end
```

```text
# Spoke_Single_Cloud_Dual_NHS_Config
conf t
interface Tunnel<id>
 description DMVPN_SINGLE_CLOUD_SPOKE_DUAL_NHS
 bandwidth <kbps>
 ip address <spoke-tunnel-ip> <mask>
 ip mtu 1400
 ip nhrp network-id <nhrp-id>
 ip nhrp nhs <hub1-tunnel-ip> nbma <hub1-nbma-ip> multicast
 ip nhrp nhs <hub2-tunnel-ip> nbma <hub2-nbma-ip> multicast
 ip nhrp shortcut
 ip tcp adjust-mss 1360
 tunnel source <spoke-underlay-interface-or-nbma-ip>
 tunnel mode gre multipoint
 tunnel key <key-id>
 no shutdown
end
```

```text
# Spoke_Separate_NHRP_Mapping_Alternative
conf t
interface Tunnel<id>
 ip nhrp map <hub1-tunnel-ip> <hub1-nbma-ip>
 ip nhrp map multicast <hub1-nbma-ip>
 ip nhrp nhs <hub1-tunnel-ip>
 ip nhrp map <hub2-tunnel-ip> <hub2-nbma-ip>
 ip nhrp map multicast <hub2-nbma-ip>
 ip nhrp nhs <hub2-tunnel-ip>
end
```

```text
# Optional_NHS_Priority
! Platform support varies.
! Use only if supported in the image/software version.

conf t
interface Tunnel<id>
 ip nhrp nhs <hub1-tunnel-ip> nbma <hub1-nbma-ip> multicast priority <preferred-value>
 ip nhrp nhs <hub2-tunnel-ip> nbma <hub2-nbma-ip> multicast priority <secondary-value>
end
```

```text
# Overlay_Routing_Design_Reminder
! Both hubs must participate in the overlay routing design.
! Spokes must have a usable path if either hub fails.
! Hub preference is usually controlled with:
! - EIGRP delay/bandwidth/summary
! - OSPF cost
! - BGP local preference, weight, MED, or route-map policy
! - Static route administrative distance
```

```text
# Optional_EIGRP_Hub_Adjustment
conf t
interface Tunnel<id>
 no ip split-horizon eigrp <asn>
end
```

```text
# Optional_Phase3_EIGRP_Summary_From_Hubs
conf t
interface Tunnel<id>
 ip summary-address eigrp <asn> <summary-prefix> <summary-mask>
end
```

# DMVPN_Dual_Hub_Single_Cloud_Verification_Commands

| Check                      | Device       | Command                                     | Good Output                                     |
| -------------------------- | ------------ | ------------------------------------------- | ----------------------------------------------- |
| Single-cloud structure     | All Routers  | `show ip interface brief \| include Tunnel` | One tunnel interface is used for this cloud     |
| Same tunnel subnet         | Hubs, Spokes | `show running-config interface Tunnel<id>`  | All tunnel IPs are in the same subnet           |
| Same NHRP ID               | Hubs, Spokes | `show running-config interface Tunnel<id>`  | Same `ip nhrp network-id` is used               |
| Same tunnel key            | Hubs, Spokes | `show running-config interface Tunnel<id>`  | Same `tunnel key` is used if configured         |
| Underlay to Hub 1          | Spokes       | `ping <hub1-nbma-ip>`                       | Ping succeeds                                   |
| Underlay to Hub 2          | Spokes       | `ping <hub2-nbma-ip>`                       | Ping succeeds                                   |
| Hub 1 tunnel state         | Hub 1        | `show ip interface brief \| include Tunnel` | Tunnel is `up/up`                               |
| Hub 2 tunnel state         | Hub 2        | `show ip interface brief \| include Tunnel` | Tunnel is `up/up`                               |
| Spoke tunnel state         | Spokes       | `show ip interface brief \| include Tunnel` | Tunnel is `up/up`                               |
| Spoke NHS list             | Spokes       | `show ip nhrp nhs detail`                   | Both hubs appear as NHS devices                 |
| Hub 1 NHRP cache           | Hub 1        | `show ip nhrp`                              | Spoke mappings appear dynamically               |
| Hub 2 NHRP cache           | Hub 2        | `show ip nhrp`                              | Spoke mappings appear dynamically               |
| Hub 1 DMVPN state          | Hub 1        | `show dmvpn`                                | Spoke peers show up                             |
| Hub 2 DMVPN state          | Hub 2        | `show dmvpn`                                | Spoke peers show up                             |
| Spoke DMVPN state          | Spokes       | `show dmvpn`                                | Both hubs appear in the same cloud              |
| Hub 1 tunnel ping          | Spokes       | `ping <hub1-tunnel-ip>`                     | Ping succeeds                                   |
| Hub 2 tunnel ping          | Spokes       | `ping <hub2-tunnel-ip>`                     | Ping succeeds                                   |
| Hub-to-spoke ping          | Hubs         | `ping <spoke-tunnel-ip>`                    | Both hubs can reach spokes                      |
| Overlay routing            | Hubs, Spokes | `show ip route`                             | Remote prefixes exist                           |
| Preferred hub path         | Spokes       | `show ip route <remote-prefix>`             | Route uses intended hub preference              |
| Phase 3 redirect on hubs   | Hubs         | `show running-config interface Tunnel<id>`  | Hubs have `ip nhrp redirect` if using Phase 3   |
| Phase 3 shortcut on spokes | Spokes       | `show running-config interface Tunnel<id>`  | Spokes have `ip nhrp shortcut` if using Phase 3 |
| Hub 1 failure test         | Spokes       | `show dmvpn` and `show ip route`            | Hub 2 remains usable after Hub 1 fails          |
| Hub 2 failure test         | Spokes       | `show dmvpn` and `show ip route`            | Hub 1 remains usable after Hub 2 fails          |

# DMVPN_Dual_Hub_Single_Cloud_Rollback

| Step | Task                                             | Device       | Command                                  | Expected Result                         |
| ---: | ------------------------------------------------ | ------------ | ---------------------------------------- | --------------------------------------- |
|    1 | Shut down spoke tunnels first                    | Spokes       | `shutdown` under tunnel                  | Spokes stop registering to hubs         |
|    2 | Remove Hub 2 NHS only if reverting to single hub | Spokes       | See `Rollback_Remove_Second_NHS`         | Spokes use only Hub 1                   |
|    3 | Remove both NHS entries if fully removing DMVPN  | Spokes       | See `Rollback_Spoke_Dual_NHS_NHRP`       | Spokes no longer register to either hub |
|    4 | Remove spoke shortcut if configured              | Spokes       | See `Rollback_Spoke_Dual_NHS_NHRP`       | Phase 3 shortcut is removed             |
|    5 | Remove spoke NHRP authentication if used         | Spokes       | See `Rollback_Spoke_Dual_NHS_NHRP`       | Authentication is removed               |
|    6 | Remove spoke NHRP network ID                     | Spokes       | See `Rollback_Spoke_Dual_NHS_NHRP`       | NHRP is disabled on spokes              |
|    7 | Remove spoke mGRE tunnel settings                | Spokes       | See `Rollback_Spoke_Single_Cloud_Tunnel` | Spoke tunnel core is cleared            |
|    8 | Shut down Hub 2 tunnel if removing second hub    | Hub 2        | `shutdown` under tunnel                  | Hub 2 stops accepting registrations     |
|    9 | Remove Hub 2 NHRP settings                       | Hub 2        | See `Rollback_Hub2_Single_Cloud`         | Hub 2 NHRP is removed                   |
|   10 | Remove Hub 2 tunnel settings                     | Hub 2        | See `Rollback_Hub2_Single_Cloud`         | Hub 2 tunnel core is cleared            |
|   11 | Shut down Hub 1 tunnel if fully removing DMVPN   | Hub 1        | `shutdown` under tunnel                  | Hub 1 stops accepting registrations     |
|   12 | Remove Hub 1 NHRP settings                       | Hub 1        | See `Rollback_Hub1_Single_Cloud`         | Hub 1 NHRP is removed                   |
|   13 | Remove Hub 1 tunnel settings                     | Hub 1        | See `Rollback_Hub1_Single_Cloud`         | Hub 1 tunnel core is cleared            |
|   14 | Delete tunnel interfaces if fully resetting      | Hubs, Spokes | `no interface Tunnel<id>`                | Tunnel interfaces are removed           |

```text
# Rollback_Remove_Second_NHS
conf t
interface Tunnel<id>
 no ip nhrp nhs <hub2-tunnel-ip> nbma <hub2-nbma-ip> multicast
 no ip nhrp map <hub2-tunnel-ip> <hub2-nbma-ip>
 no ip nhrp map multicast <hub2-nbma-ip>
 no ip nhrp nhs <hub2-tunnel-ip>
end
```

```text
# Rollback_Spoke_Dual_NHS_NHRP
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp shortcut
 no ip nhrp nhs <hub1-tunnel-ip> nbma <hub1-nbma-ip> multicast
 no ip nhrp nhs <hub2-tunnel-ip> nbma <hub2-nbma-ip> multicast
 no ip nhrp map <hub1-tunnel-ip> <hub1-nbma-ip>
 no ip nhrp map multicast <hub1-nbma-ip>
 no ip nhrp nhs <hub1-tunnel-ip>
 no ip nhrp map <hub2-tunnel-ip> <hub2-nbma-ip>
 no ip nhrp map multicast <hub2-nbma-ip>
 no ip nhrp nhs <hub2-tunnel-ip>
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
end
```

```text
# Rollback_Spoke_Single_Cloud_Tunnel
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
# Rollback_Hub2_Single_Cloud
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp redirect
 no ip nhrp map multicast dynamic
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
 no tunnel mode gre multipoint
 no tunnel source <hub2-underlay-interface-or-nbma-ip>
 no tunnel key <key-id>
 no ip mtu 1400
 no ip tcp adjust-mss 1360
 no ip address
end
```

```text
# Rollback_Hub1_Single_Cloud
conf t
interface Tunnel<id>
 shutdown
 no ip nhrp redirect
 no ip nhrp map multicast dynamic
 no ip nhrp authentication <key-string>
 no ip nhrp network-id <nhrp-id>
 no tunnel mode gre multipoint
 no tunnel source <hub1-underlay-interface-or-nbma-ip>
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

# DMVPN_Dual_Hub_Single_Cloud_Failure_Checks

| Symptom                                 | Likely Cause                           | Check                                       | Fix                                                   |
| --------------------------------------- | -------------------------------------- | ------------------------------------------- | ----------------------------------------------------- |
| Spoke registers to only one hub         | Missing second NHS or underlay failure | `show ip nhrp nhs detail`                   | Add second NHS or fix NBMA reachability               |
| Hub 2 sees no spokes                    | Spokes are not registering to Hub 2    | `show ip nhrp` on Hub 2                     | Add Hub 2 as NHS on spokes                            |
| Spoke has two tunnels                   | Dual-cloud was built by mistake        | `show ip interface brief \| include Tunnel` | Use one tunnel interface for single-cloud             |
| Hubs use different tunnel subnets       | Not actually one shared cloud          | `show run interface Tunnel<id>`             | Put both hubs in same tunnel subnet                   |
| Hubs use different NHRP IDs             | Cloud consistency problem              | `show run interface Tunnel<id>`             | Match NHRP network ID                                 |
| Tunnel keys mismatch                    | Peers fail despite reachability        | `show run interface Tunnel<id>`             | Match or remove tunnel keys                           |
| NHRP authentication fails               | Authentication key mismatch            | `show run interface Tunnel<id>`             | Match or remove NHRP authentication                   |
| Hub failover fails                      | Routing has no alternate path          | `show ip route`                             | Fix routing metrics, policy, or advertisements        |
| Spoke loses transport reachability      | NBMA route points into tunnel          | `show ip route <hub-nbma-ip>`               | Add specific underlay route to hub NBMA               |
| Phase 3 shortcut fails                  | Missing redirect or shortcut           | `show run interface Tunnel<id>`             | Add redirect on hubs and shortcut on spokes           |
| Routing neighbors fail                  | Missing multicast mapping              | `show run interface Tunnel<id>`             | Add hub dynamic multicast and spoke multicast mapping |
| EIGRP routes do not pass between spokes | Hub split horizon still enabled        | `show run interface Tunnel<id>`             | Use `no ip split-horizon eigrp <asn>` on hubs         |
| Wrong hub is preferred                  | Routing preference not engineered      | `show ip route <prefix>`                    | Tune routing metric or BGP policy                     |
| Large packets fail                      | MTU or MSS issue                       | `ping <destination> size <size> df-bit`     | Configure tunnel MTU and TCP MSS                      |
| Engineer built dual-cloud accidentally  | Separate tunnels or subnets were used  | `show run \| section interface Tunnel`      | Collapse back to one shared tunnel cloud              |

##### Source_Basis
# DMVPN_Dual_Hub_Single_Cloud_Mental_Model
# DMVPN_Dual_Hub_Single_Cloud_Configuration_Checklist
# DMVPN_Dual_Hub_Single_Cloud_Skeleton
# DMVPN_Dual_Hub_Single_Cloud_Verification_Commands
# DMVPN_Dual_Hub_Single_Cloud_Rollback
# DMVPN_Dual_Hub_Single_Cloud_Failure_Checks