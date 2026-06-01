# VXLAN_EVPN_BUM_Replication_Decision_Table

| Option | Command | Where Configured | Best Use | Dependency | Primary Verification |
|---|---|---|---|---|---|
| Ingress replication | `ingress-replication protocol bgp` | `interface nve1` then `member vni <l2-vni>` | Simpler EVPN fabric where BUM replication is handled by VTEPs using EVPN control-plane peer discovery | Working BGP EVPN control plane | `show nve peers`, `show nve vni`, `show bgp l2vpn evpn summary` |
| Multicast replication | `mcast-group <multicast-group>` | `interface nve1` then `member vni <l2-vni>` | Fabric where underlay multicast is intentionally deployed for BUM replication | Working PIM sparse-mode and RP in underlay | `show ip pim neighbor`, `show ip pim rp`, `show ip mroute <multicast-group>` |
| Global multicast group | `global mcast-group <multicast-group>` | `interface nve1` | Common multicast group applied globally to L2 VNIs unless overridden | Working PIM sparse-mode and RP in underlay | `show running-config interface nve1`, `show ip mroute <multicast-group>` |

| Rule                                           | Operational Meaning                                                                                       |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Pick one BUM replication method per L2 VNI     | Do not casually configure both `ingress-replication protocol bgp` and `mcast-group` under the same L2 VNI |
| Ingress replication depends on EVPN            | If BGP EVPN is down, ingress replication cannot correctly discover remote VTEPs                           |
| Multicast replication depends on PIM           | If PIM/RP/RPF is broken, multicast-based BUM replication fails                                            |
| BUM handling belongs in the baseline EVPN note | ARP suppression is separate and does not replace BUM replication                                          |

# VXLAN_EVPN_BUM_Traffic_Handling_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm baseline VXLAN EVPN fabric is being configured | Leaf | `show running-config interface nve1` | NVE interface exists or is planned |
| 2 | Confirm target L2 VNI exists under NVE | Leaf | `show running-config interface nve1` | `member vni <l2-vni>` is present or ready to be added |
| 3 | Confirm VLAN-to-VNI mapping | Leaf | `show vxlan` | Tenant VLAN maps to the intended L2 VNI |
| 4 | Confirm EVPN control plane if using ingress replication | Leaf / Spine | `show bgp l2vpn evpn summary` | EVPN BGP peers are established |
| 5 | Confirm remote VTEP discovery if using ingress replication | Leaf | `show nve peers` | Remote VTEPs appear after EVPN control plane is working |
| 6 | Confirm PIM feature if using multicast replication | Leaf / Spine | `show running-config | include feature pim` | `feature pim` is enabled |
| 7 | Confirm PIM neighbors if using multicast replication | Leaf / Spine | `show ip pim neighbor` | PIM neighbors are formed on underlay links |
| 8 | Confirm RP if using multicast replication | Leaf / Spine | `show ip pim rp` | Correct RP is shown for the multicast group range |
| 9 | Enter NVE interface | Leaf | `interface nve1` | NVE interface configuration mode is entered |
| 10 | Enter target L2 VNI context | Leaf | `member vni <l2-vni>` | L2 VNI submode is entered |
| 11 | Configure ingress replication option | Leaf | `ingress-replication protocol bgp` | BUM traffic for this L2 VNI uses EVPN-based ingress replication |
| 12 | Or configure multicast replication option | Leaf | `mcast-group <multicast-group>` | BUM traffic for this L2 VNI uses underlay multicast |
| 13 | Or configure global multicast group | Leaf | `global mcast-group <multicast-group>` | Common multicast group is applied under NVE |
| 14 | Save configuration | Leaf | `copy running-config startup-config` | BUM replication configuration is saved |
| 15 | Verify L2 VNI state | Leaf | `show nve vni` | L2 VNI shows `Up` |
| 16 | Verify NVE peers | Leaf | `show nve peers` | Remote VTEPs appear as up |
| 17 | Verify EVPN routes if using ingress replication | Leaf / Spine | `show bgp l2vpn evpn` | EVPN routes are present |
| 18 | Verify multicast route if using multicast replication | Leaf / Spine | `show ip mroute <multicast-group>` | Multicast route exists for the VXLAN group |
| 19 | Generate endpoint traffic | Host / Leaf | `ping <remote-host-ip>` | Host traffic succeeds across the VXLAN EVPN fabric |
| 20 | Verify MAC learning after traffic | Leaf | `show l2route evpn mac all` | Local and remote MAC routes are present |
# VXLAN_EVPN_BUM_Ingress_Replication_Skeleton

```text
interface nve1
 no shutdown
 source-interface loopback<nve-id>
 host-reachability protocol bgp
 member vni <l2-vni>
  ingress-replication protocol bgp
```

# VXLAN_EVPN_BUM_Multicast_Replication_Skeleton

```text
feature pim

ip pim rp-address <rp-ip>

interface <underlay-uplink-interface>
 ip pim sparse-mode

interface loopback<nve-id>
 ip pim sparse-mode

interface nve1
 no shutdown
 source-interface loopback<nve-id>
 host-reachability protocol bgp
 member vni <l2-vni>
  mcast-group <multicast-group>
```

# VXLAN_EVPN_BUM_Global_Multicast_Group_Skeleton

```text
feature pim

ip pim rp-address <rp-ip>

interface <underlay-uplink-interface>
 ip pim sparse-mode

interface loopback<nve-id>
 ip pim sparse-mode

interface nve1
 no shutdown
 source-interface loopback<nve-id>
 host-reachability protocol bgp
 global mcast-group <multicast-group>
 member vni <l2-vni>
```

# VXLAN_EVPN_BUM_Ingress_Replication_Checklist

| Step | Task                                   | Device      | Command                              | Expected Result                                |                              |
| ---: | -------------------------------------- | ----------- | ------------------------------------ | ---------------------------------------------- | ---------------------------- |
|    1 | Confirm EVPN control plane is enabled  | Leaf        | `show running-config                 | include nv overlay evpn`                       | `nv overlay evpn` is present |
|    2 | Confirm BGP EVPN neighbor is up        | Leaf        | `show bgp l2vpn evpn summary`        | EVPN neighbor is established                   |                              |
|    3 | Confirm NVE uses BGP host reachability | Leaf        | `show running-config interface nve1` | `host-reachability protocol bgp` is present    |                              |
|    4 | Enter NVE interface                    | Leaf        | `interface nve1`                     | NVE interface config mode is entered           |                              |
|    5 | Enter target L2 VNI                    | Leaf        | `member vni <l2-vni>`                | L2 VNI submode is entered                      |                              |
|    6 | Configure ingress replication          | Leaf        | `ingress-replication protocol bgp`   | BUM replication uses EVPN-discovered VTEPs     |                              |
|    7 | Save configuration                     | Leaf        | `copy running-config startup-config` | Configuration is saved                         |                              |
|    8 | Verify VNI state                       | Leaf        | `show nve vni`                       | L2 VNI is up                                   |                              |
|    9 | Verify NVE peers                       | Leaf        | `show nve peers`                     | Remote VTEPs are learned through control plane |                              |
|   10 | Verify EVPN routes                     | Leaf        | `show bgp l2vpn evpn`                | EVPN route table is populated                  |                              |
|   11 | Test endpoint traffic                  | Host / Leaf | `ping <remote-host-ip>`              | Host-to-host traffic succeeds                  |                              |
# VXLAN_EVPN_BUM_Multicast_Replication_Checklist

| Step | Task                                 | Device       | Command                                                           | Expected Result                                       |
| ---: | ------------------------------------ | ------------ | ----------------------------------------------------------------- | ----------------------------------------------------- |
|    1 | Enable PIM feature                   | Leaf / Spine | `feature pim`                                                     | PIM feature is enabled                                |
|    2 | Configure multicast RP               | Leaf / Spine | `ip pim rp-address <rp-ip>`                                       | RP is configured                                      |
|    3 | Enable PIM on L3 uplinks             | Leaf / Spine | `interface <underlay-uplink-interface>` then `ip pim sparse-mode` | PIM is enabled on underlay links                      |
|    4 | Enable PIM on NVE source loopback    | Leaf         | `interface loopback<nve-id>` then `ip pim sparse-mode`            | NVE source loopback participates in multicast routing |
|    5 | Verify PIM neighbors                 | Leaf / Spine | `show ip pim neighbor`                                            | PIM neighbors are formed                              |
|    6 | Verify RP state                      | Leaf / Spine | `show ip pim rp`                                                  | Correct RP is visible                                 |
|    7 | Enter NVE interface                  | Leaf         | `interface nve1`                                                  | NVE interface config mode is entered                  |
|    8 | Enter target L2 VNI                  | Leaf         | `member vni <l2-vni>`                                             | L2 VNI submode is entered                             |
|    9 | Configure multicast group for L2 VNI | Leaf         | `mcast-group <multicast-group>`                                   | L2 VNI uses multicast replication                     |
|   10 | Save configuration                   | Leaf / Spine | `copy running-config startup-config`                              | Configuration is saved                                |
|   11 | Verify VNI state                     | Leaf         | `show nve vni`                                                    | L2 VNI shows `Up` and the multicast group             |
|   12 | Verify multicast route               | Leaf / Spine | `show ip mroute <multicast-group>`                                | `*,G` and/or `S,G` entries exist                      |
|   13 | Test endpoint traffic                | Host / Leaf  | `ping <remote-host-ip>`                                           | Host-to-host traffic succeeds                         |
|   14 | Verify MAC learning                  | Leaf         | `show l2route evpn mac all`                                       | Local and remote MAC routes exist                     |
# VXLAN_EVPN_BUM_Traffic_Handling_Verification_Commands

| Task                                  | Command                              | Expected Result                                                                                          |
| ------------------------------------- | ------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| Verify NVE config                     | `show running-config interface nve1` | Each L2 VNI has `ingress-replication protocol bgp`, `mcast-group`, or is covered by `global mcast-group` |
| Verify NVE interface                  | `show nve interface`                 | NVE interface is up                                                                                      |
| Verify VNI state                      | `show nve vni`                       | L2 VNI shows `Up`                                                                                        |
| Verify NVE peers                      | `show nve peers`                     | Remote VTEPs are visible                                                                                 |
| Verify EVPN session                   | `show bgp l2vpn evpn summary`        | EVPN BGP peers are established                                                                           |
| Verify EVPN route table               | `show bgp l2vpn evpn`                | EVPN routes are present                                                                                  |
| Verify MAC learning                   | `show l2route evpn mac all`          | Local and remote MAC routes exist                                                                        |
| Verify MAC/IP learning                | `show l2route evpn mac-ip all`       | MAC/IP bindings are present after host traffic                                                           |
| Verify VLAN-to-VNI mapping            | `show vxlan`                         | Tenant VLAN maps to the expected VNI                                                                     |
| Verify PIM neighbor, multicast only   | `show ip pim neighbor`               | PIM neighbors are formed                                                                                 |
| Verify RP, multicast only             | `show ip pim rp`                     | Correct RP is shown                                                                                      |
| Verify multicast tree, multicast only | `show ip mroute <multicast-group>`   | Multicast state exists for the VXLAN group                                                               |
| Verify endpoint reachability          | `ping <remote-host-ip>`              | Host-to-host traffic succeeds                                                                            |
# VXLAN_EVPN_BUM_Traffic_Handling_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove ingress replication from L2 VNI | Leaf | `interface nve1` then `member vni <l2-vni>` then `no ingress-replication protocol bgp` | Ingress replication is removed from the L2 VNI |
| 2 | Remove multicast group from L2 VNI | Leaf | `interface nve1` then `member vni <l2-vni>` then `no mcast-group <multicast-group>` | Multicast replication is removed from the L2 VNI |
| 3 | Remove global multicast group | Leaf | `interface nve1` then `no global mcast-group <multicast-group>` | Global multicast replication setting is removed |
| 4 | Remove PIM from NVE loopback if no longer needed | Leaf | `interface loopback<nve-id>` then `no ip pim sparse-mode` | Loopback no longer participates in PIM |
| 5 | Remove PIM from uplink if no longer needed | Leaf / Spine | `interface <underlay-uplink-interface>` then `no ip pim sparse-mode` | Uplink no longer participates in PIM |
| 6 | Remove RP if no longer used | Leaf / Spine | `no ip pim rp-address <rp-ip>` | RP configuration is removed |
| 7 | Disable PIM feature only if unused elsewhere | Leaf / Spine | `no feature pim` | PIM feature is disabled |
| 8 | Save rollback state | Leaf / Spine | `copy running-config startup-config` | Rollback is saved |
# VXLAN_EVPN_BUM_Traffic_Handling_Failure_Checks

| Symptom                                        | Command                                                     | What Usually Broke                                                                      |
| ---------------------------------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| L2 VNI has no replication method               | `show running-config interface nve1`                        | Missing `ingress-replication protocol bgp`, `mcast-group`, or `global mcast-group`      |
| Ingress replication fails                      | `show bgp l2vpn evpn summary`                               | EVPN BGP neighbor is down                                                               |
| Ingress replication has no remote VTEPs        | `show nve peers`                                            | EVPN control plane did not learn remote VTEPs                                           |
| EVPN routes missing                            | `show bgp l2vpn evpn`                                       | Missing EVPN address family, route-target issue, or route reflector issue               |
| Multicast replication fails                    | `show ip mroute <multicast-group>`                          | PIM, RP, RPF, or multicast group configuration is broken                                |
| PIM neighbors missing                          | `show ip pim neighbor`                                      | PIM not enabled on L3 underlay uplinks or physical adjacency is down                    |
| RP missing                                     | `show ip pim rp`                                            | RP not configured or not consistently reachable                                         |
| VNI down                                       | `show nve vni`                                              | Missing VLAN-to-VNI mapping, missing NVE member VNI, NVE shut, or source loopback issue |
| NVE peer missing                               | `show nve peers`                                            | BUM replication cannot reach or discover remote VTEPs                                   |
| MAC routes missing                             | `show l2route evpn mac all`                                 | Endpoint not learned, EVPN route not received, or no host traffic                       |
| MAC/IP routes missing                          | `show l2route evpn mac-ip all`                              | Host ARP not learned or endpoint traffic has not occurred                               |
| Host ping fails but control plane is up        | `show vxlan` and `show ip arp vrf <vrf-name>`               | Wrong VLAN-to-VNI mapping, wrong VRF, SVI issue, or host-side problem                   |
| Multicast configured but no PIM on uplinks     | `show running-config interface <underlay-uplink-interface>` | Missing `ip pim sparse-mode` on L3 uplinks                                              |
| Both replication methods appear under same VNI | `show running-config interface nve1`                        | Design is muddy; choose ingress replication or multicast replication intentionally      |