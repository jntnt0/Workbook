# VXLAN_BGP_EVPN_ARP_Suppression_Prerequisites

| Step | Task                                                    | Device         | Command                                               | Expected Result                                                                                     |                                                     |
| ---: | ------------------------------------------------------- | -------------- | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
|    1 | Confirm NX-OS platform and release                      | Leaf           | `show version`                                        | Platform and NX-OS release support VXLAN BGP EVPN ARP suppression                                   |                                                     |
|    2 | Confirm hardware platform if required                   | Leaf           | `show module`                                         | Line card/platform supports the intended VXLAN EVPN ARP suppression behavior                        |                                                     |
|    3 | Confirm baseline VXLAN BGP EVPN fabric is working       | Leaf           | `show bgp l2vpn evpn summary`                         | EVPN BGP neighbors are established                                                                  |                                                     |
|    4 | Confirm NVE interface is up                             | Leaf           | `show nve interface`                                  | `nve1` is up and uses the expected source loopback                                                  |                                                     |
|    5 | Confirm NVE peers are learned through control plane     | Leaf           | `show nve peers`                                      | Remote VTEPs show `Up` with learn type `CP`                                                         |                                                     |
|    6 | Confirm target L2 VNI is up                             | Leaf           | `show nve vni`                                        | Target L2 VNI shows `Up` and mode `CP`                                                              |                                                     |
|    7 | Confirm tenant VLAN maps to target L2 VNI               | Leaf           | `show vxlan`                                          | Tenant VLAN maps to the expected L2 VNI                                                             |                                                     |
|    8 | Confirm L2 VNI has BUM replication configured           | Leaf           | `show running-config interface nve1`                  | L2 VNI has `ingress-replication protocol bgp`, `mcast-group`, or is covered by `global mcast-group` |                                                     |
|    9 | Confirm anycast gateway MAC is configured               | Leaf           | `show running-config                                  | include anycast-gateway-mac`                                                                        | Common anycast gateway MAC exists across leaf VTEPs |
|   10 | Confirm tenant SVI exists                               | Leaf           | `show running-config interface vlan <tenant-vlan-id>` | Tenant SVI exists and is not shut                                                                   |                                                     |
|   11 | Confirm tenant SVI is in the correct VRF                | Leaf           | `show running-config interface vlan <tenant-vlan-id>` | SVI includes `vrf member <vrf-name>`                                                                |                                                     |
|   12 | Confirm tenant SVI has gateway IP                       | Leaf           | `show running-config interface vlan <tenant-vlan-id>` | SVI has `ip address <gateway-ip>/<prefix-length>`                                                   |                                                     |
|   13 | Confirm tenant SVI uses distributed anycast gateway     | Leaf           | `show running-config interface vlan <tenant-vlan-id>` | SVI includes `fabric forwarding mode anycast-gateway`                                               |                                                     |
|   14 | Confirm MAC routes are learned                          | Leaf           | `show l2route evpn mac all`                           | Local and remote MAC routes are present                                                             |                                                     |
|   15 | Confirm MAC/IP routes are learned                       | Leaf           | `show l2route evpn mac-ip all`                        | Local and remote MAC/IP bindings are present                                                        |                                                     |
|   16 | Confirm ARP table is sane before change                 | Leaf           | `show ip arp vrf <vrf-name>`                          | Tenant endpoint ARP entries exist or can be learned through traffic                                 |                                                     |
|   17 | Confirm suppression setting is consistent across fabric | All leaf VTEPs | `show running-config interface nve1`                  | Same L2 VNI is either consistently configured with ARP suppression or consistently not configured   |                                                     |
# VXLAN_BGP_EVPN_ARP_Suppression_Configuration_Delta

| Step | Task                                                       | Device                             | Command                                               | Expected Result                                               |
| ---: | ---------------------------------------------------------- | ---------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------- |
|    1 | Confirm this is an add-on change, not base fabric build    | Leaf                               | `show nve peers`                                      | Baseline VXLAN BGP EVPN is already operational                |
|    2 | Confirm target L2 VNI is the intended VLAN extension       | Leaf                               | `show vxlan`                                          | Tenant VLAN maps to the target L2 VNI                         |
|    3 | Confirm tenant SVI is the distributed gateway              | Leaf                               | `show running-config interface vlan <tenant-vlan-id>` | SVI has `fabric forwarding mode anycast-gateway`              |
|    4 | Confirm MAC/IP learning exists before suppression          | Leaf                               | `show l2route evpn mac-ip all`                        | MAC/IP entries exist or populate after host traffic           |
|    5 | Enter global configuration mode                            | Leaf                               | `configure terminal`                                  | Device enters global configuration mode                       |
|    6 | Enter NVE interface                                        | Leaf                               | `interface nve1`                                      | NVE interface configuration mode is entered                   |
|    7 | Enter target L2 VNI context                                | Leaf                               | `member vni <l2-vni>`                                 | L2 VNI submode is entered                                     |
|    8 | Enable ARP suppression on the target L2 VNI                | Leaf                               | `suppress-arp`                                        | ARP suppression is enabled for this L2 VNI                    |
|    9 | Repeat per-VNI ARP suppression on every VTEP for same VNID | All leaf VTEPs hosting same L2 VNI | `member vni <l2-vni>` then `suppress-arp`             | ARP suppression is consistent across the fabric for that VNID |
|   10 | Save configuration                                         | Leaf                               | `copy running-config startup-config`                  | Configuration is saved                                        |
|   11 | Verify ARP suppression flag on VNI                         | Leaf                               | `show nve vni`                                        | Target VNI shows suppression flag such as `SA`                |
|   12 | Verify MAC/IP routes after change                          | Leaf                               | `show l2route evpn mac-ip all`                        | MAC/IP bindings remain present                                |
|   13 | Verify ARP suppression cache                               | Leaf                               | `show ip arp suppression-cache detail`                | Local and remote suppression entries appear                   |
|   14 | Test endpoint reachability                                 | Host / Leaf                        | `ping <remote-host-ip>`                               | Host-to-host reachability succeeds                            |
|   15 | Recheck suppression cache after traffic                    | Leaf                               | `show ip arp suppression-cache detail`                | Expected endpoint mappings appear in suppression cache        |
# VXLAN_BGP_EVPN_ARP_Suppression_Skeleton

```text
configure terminal
interface nve1
 member vni <l2-vni>
  suppress-arp
end
copy running-config startup-config
```

# VXLAN_BGP_EVPN_Global_ARP_Suppression_Skeleton

```text
configure terminal
interface nve1
 global suppress-arp
 member vni <l2-vni>
end
copy running-config startup-config
```

# VXLAN_BGP_EVPN_ARP_Suppression_Verification_Commands

| Task                               | Command                                               | Expected Result                                                                        |                                |
| ---------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------ |
| Verify EVPN BGP control plane      | `show bgp l2vpn evpn summary`                         | EVPN neighbors are established                                                         |                                |
| Verify NVE peers                   | `show nve peers`                                      | Remote VTEPs are up with control-plane learning                                        |                                |
| Verify target VNI state            | `show nve vni`                                        | Target L2 VNI is up and shows ARP suppression flag when enabled                        |                                |
| Verify NVE configuration           | `show running-config interface nve1`                  | Target L2 VNI includes `suppress-arp`, or global config includes `global suppress-arp` |                                |
| Verify global VXLAN overlay config | `show running-config nv overlay`                      | Global NVE and suppression configuration appears where expected                        |                                |
| Verify VLAN-to-VNI mapping         | `show vxlan`                                          | Tenant VLAN maps to the expected L2 VNI                                                |                                |
| Verify tenant SVI anycast gateway  | `show running-config interface vlan <tenant-vlan-id>` | SVI has `fabric forwarding mode anycast-gateway`                                       |                                |
| Verify global anycast gateway MAC  | `show running-config                                  | include anycast-gateway-mac`                                                           | Anycast gateway MAC is present |
| Verify MAC routes                  | `show l2route evpn mac all`                           | Local and remote MAC routes exist                                                      |                                |
| Verify MAC/IP routes               | `show l2route evpn mac-ip all`                        | MAC/IP bindings exist for ARP suppression to use                                       |                                |
| Verify detailed MAC/IP routes      | `show l2route evpn mac-ip all detail`                 | Remote MAC/IP next-hop details are visible                                             |                                |
| Verify ARP suppression cache       | `show ip arp suppression-cache detail`                | Local and remote ARP suppression entries appear                                        |                                |
| Verify ARP suppression summary     | `show ip arp suppression-cache summary`               | Suppression-cache totals are visible                                                   |                                |
| Verify suppression cache by VLAN   | `show ip arp suppression-cache vlan <tenant-vlan-id>` | Entries for the tenant VLAN are visible                                                |                                |
| Verify tenant ARP table            | `show ip arp vrf <vrf-name>`                          | Tenant endpoint ARP entries are present                                                |                                |
| Verify endpoint reachability       | `ping <remote-host-ip>`                               | Host-to-host traffic succeeds                                                          |                                |
# VXLAN_BGP_EVPN_ARP_Suppression_Rollback

| Step | Task                                                        | Device                             | Command                                     | Expected Result                                                             |
| ---: | ----------------------------------------------------------- | ---------------------------------- | ------------------------------------------- | --------------------------------------------------------------------------- |
|    1 | Enter global configuration mode                             | Leaf                               | `configure terminal`                        | Device enters global configuration mode                                     |
|    2 | Enter NVE interface                                         | Leaf                               | `interface nve1`                            | NVE interface config mode is entered                                        |
|    3 | Enter target L2 VNI                                         | Leaf                               | `member vni <l2-vni>`                       | L2 VNI submode is entered                                                   |
|    4 | Remove per-VNI ARP suppression                              | Leaf                               | `no suppress-arp`                           | ARP suppression is removed from the target VNI                              |
|    5 | Or disable global suppression for this VNI                  | Leaf                               | `suppress-arp disable`                      | Specific VNI overrides global suppression                                   |
|    6 | Remove global ARP suppression if no longer wanted           | Leaf                               | `no global suppress-arp`                    | Global ARP suppression is removed from NVE                                  |
|    7 | Repeat rollback consistently across all VTEPs for same VNID | All leaf VTEPs hosting same L2 VNI | `no suppress-arp` or `suppress-arp disable` | ARP suppression state matches across the fabric                             |
|    8 | Save rollback state                                         | Leaf                               | `copy running-config startup-config`        | Rollback is saved                                                           |
|    9 | Verify VNI state after rollback                             | Leaf                               | `show nve vni`                              | Target L2 VNI remains up                                                    |
|   10 | Verify ARP suppression config is gone or overridden         | Leaf                               | `show running-config interface nve1`        | Target VNI no longer has active ARP suppression unless intentionally global |
|   11 | Test host reachability after rollback                       | Host / Leaf                        | `ping <remote-host-ip>`                     | Host-to-host traffic still succeeds                                         |
# VXLAN_BGP_EVPN_ARP_Suppression_Failure_Checks

| Symptom                                                     | Command                                                        | What Usually Broke                                                                             |
| ----------------------------------------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| ARP suppression configured but unsupported behavior appears | `show running-config interface vlan <tenant-vlan-id>`          | Tenant SVI is missing `fabric forwarding mode anycast-gateway`                                 |
| ARP suppression missing from intended VNI                   | `show running-config interface nve1`                           | `suppress-arp` was not configured under the target L2 VNI                                      |
| Global suppression affects more VNIs than intended          | `show running-config interface nve1`                           | `global suppress-arp` was configured without `suppress-arp disable` on excluded VNIs           |
| Suppression is inconsistent across leafs                    | `show running-config interface nve1`                           | Same VNID has ARP suppression enabled on some VTEPs but not others                             |
| Target L2 VNI is down                                       | `show nve vni`                                                 | Baseline L2 VNI, NVE, VLAN-to-VNI mapping, or EVPN control plane is broken                     |
| NVE peer missing                                            | `show nve peers`                                               | VXLAN BGP EVPN baseline is not working                                                         |
| EVPN neighbor down                                          | `show bgp l2vpn evpn summary`                                  | BGP EVPN control plane is down                                                                 |
| MAC/IP bindings missing                                     | `show l2route evpn mac-ip all`                                 | Endpoint has not been learned, type-2 MAC/IP route is absent, or host traffic has not occurred |
| ARP suppression cache is empty                              | `show ip arp suppression-cache detail`                         | MAC/IP binding missing, no host traffic, or EVPN learning issue                                |
| Host ping fails after suppression enabled                   | `show ip arp vrf <vrf-name>`                                   | Wrong VLAN, wrong VRF, missing SVI gateway, or missing endpoint ARP entry                      |
| Tenant SVI exists but gateway behavior fails                | `show running-config interface vlan <tenant-vlan-id>`          | Missing VRF membership, missing gateway IP, or missing anycast gateway mode                    |
| VNI shows up but no ARP optimization                        | `show nve vni`                                                 | VNI lacks suppression flag such as `SA`                                                        |
| MAC routes exist but MAC/IP routes do not                   | `show l2route evpn mac all` and `show l2route evpn mac-ip all` | Host IP binding is not being advertised or learned                                             |
| BUM replication is broken and blamed on ARP suppression     | `show running-config interface nve1`                           | L2 VNI lacks `ingress-replication protocol bgp`, `mcast-group`, or `global mcast-group`        |
| Multicast BUM mode fails after suppression                  | `show ip mroute <multicast-group>`                             | PIM/RP/RPF issue, not ARP suppression                                                          |
| Ingress replication BUM mode fails after suppression        | `show l2route evpn imet all`                                   | EVPN remote VTEP discovery or IMET route exchange issue                                        |