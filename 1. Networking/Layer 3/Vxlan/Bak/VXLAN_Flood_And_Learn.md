
VXLAN_Flood_And_Learn_Configuration_Checklist

| Step | Task                                                   | Device      | Command                                                    | Expected Result                                                                  |                                                |
| ---: | ------------------------------------------------------ | ----------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------- | ---------------------------------------------- |
|    1 | Confirm NX-OS platform                                 | VTEP / Core | `show version`                                             | Nexus/NX-OS platform is confirmed                                                |                                                |
|    2 | Confirm this is flood-and-learn, not EVPN              | VTEP        | `show running-config                                       | include host-reachability`                                                       | No `host-reachability protocol bgp` is present |
|    3 | Verify physical underlay interfaces                    | VTEP / Core | `show ip interface brief`                                  | Routed underlay links show `up/up`                                               |                                                |
|    4 | Verify VTEP loopback exists                            | VTEP        | `show running-config interface loopback<id>`               | Loopback has the intended VTEP source IP                                         |                                                |
|    5 | Verify underlay route to remote VTEP loopback          | VTEP        | `show ip route <remote-vtep-loopback-ip>`                  | Remote VTEP loopback is reachable through underlay routing                       |                                                |
|    6 | Enable PIM feature                                     | VTEP / Core | `feature pim`                                              | PIM feature is enabled                                                           |                                                |
|    7 | Configure multicast RP                                 | VTEP / Core | `ip pim rp-address <rp-ip>`                                | Device knows the multicast RP                                                    |                                                |
|    8 | Configure Anycast RP if using redundant RPs            | Core        | `ip pim anycast-rp <anycast-rp-ip> <rp-member-ip>`         | RP redundancy is configured                                                      |                                                |
|    9 | Enable PIM on routed underlay links                    | VTEP / Core | `interface <underlay-interface>` then `ip pim sparse-mode` | PIM is enabled on underlay-facing routed links                                   |                                                |
|   10 | Enable PIM on loopback used for VTEP/RP reachability   | VTEP / Core | `interface loopback<id>` then `ip pim sparse-mode`         | Loopback participates in multicast routing                                       |                                                |
|   11 | Advertise loopback into underlay routing               | VTEP / Core | `ip router ospf <process-id> area <area-id>`               | Loopback is reachable through the underlay                                       |                                                |
|   12 | Verify PIM neighbors                                   | VTEP / Core | `show ip pim neighbor`                                     | PIM neighbors are formed on underlay links                                       |                                                |
|   13 | Verify RP state                                        | VTEP / Core | `show ip pim rp`                                           | Correct RP is listed for the multicast group range                               |                                                |
|   14 | Enable VLAN-to-VNI mapping                             | VTEP        | `feature vn-segment-vlan-based`                            | NX-OS can bind VLANs to VNIs                                                     |                                                |
|   15 | Enable VXLAN overlay feature                           | VTEP        | `feature nv overlay`                                       | NVE/VXLAN feature is enabled                                                     |                                                |
|   16 | Create tenant VLAN                                     | VTEP        | `vlan <vlan-id>`                                           | Tenant VLAN exists locally                                                       |                                                |
|   17 | Map tenant VLAN to VNI                                 | VTEP        | `vn-segment <vni-id>`                                      | VLAN is mapped to the VXLAN VNID                                                 |                                                |
|   18 | Repeat VLAN-to-VNI mapping for additional tenant VLANs | VTEP        | `vlan <vlan-id>` then `vn-segment <vni-id>`                | Each tenant VLAN has a matching VNI                                              |                                                |
|   19 | Enter NVE interface                                    | VTEP        | `interface nve1`                                           | NVE interface configuration mode is entered                                      |                                                |
|   20 | Set NVE source interface                               | VTEP        | `source-interface loopback<id>`                            | NVE uses the loopback as the VTEP source                                         |                                                |
|   21 | Map L2 VNI to multicast group                          | VTEP        | `member vni <vni-id> mcast-group <multicast-group>`        | VNI uses multicast for flood-and-learn BUM replication                           |                                                |
|   22 | Add additional VNIs to multicast groups                | VTEP        | `member vni <vni-id> mcast-group <multicast-group>`        | Additional VXLAN segments are enabled                                            |                                                |
|   23 | Enable NVE interface                                   | VTEP        | `no shutdown`                                              | NVE interface is administratively up                                             |                                                |
|   24 | Configure vPC VTEP secondary loopback if using vPC     | vPC VTEPs   | `ip address <anycast-vtep-ip>/32 secondary`                | Both vPC peers share the same secondary VTEP IP                                  |                                                |
|   25 | Verify NVE interface state                             | VTEP        | `show nve interface`                                       | `nve1` is up, encapsulation is VXLAN, source loopback is correct                 |                                                |
|   26 | Verify VLAN-to-VNI mapping                             | VTEP        | `show vxlan`                                               | VLANs map to expected VNIs                                                       |                                                |
|   27 | Verify VNI state                                       | VTEP        | `show nve vni`                                             | VNI state is `Up`, mode is `DP`, type is `L2`                                    |                                                |
|   28 | Verify multicast route for VXLAN group                 | VTEP / Core | `show ip mroute <multicast-group>`                         | Multicast tree exists for the VXLAN group                                        |                                                |
|   29 | Check NVE peers before traffic                         | VTEP        | `show nve peers`                                           | Peer table may be empty before traffic in flood-and-learn mode                   |                                                |
|   30 | Generate endpoint traffic                              | Host / VTEP | `ping <remote-host-ip>`                                    | Host traffic triggers flood-and-learn behavior                                   |                                                |
|   31 | Verify NVE peers after traffic                         | VTEP        | `show nve peers`                                           | Remote VTEP appears as `Up` with learn type `DP`                                 |                                                |
|   32 | Verify detailed NVE peer state                         | VTEP        | `show nve peers detail`                                    | Peer shows configured VNI and provision state `add-complete`                     |                                                |
|   33 | Verify detailed VNI state                              | VTEP        | `show nve vni <vni-id> detail`                             | VNI shows multicast group, state `Up`, mode `data-plane`, type `L2`              |                                                |
|   34 | Verify internal VNI readiness                          | VTEP        | `show nve internal vni <vni-id>`                           | Ready state shows `L2-vni-flood-learn-ready`                                     |                                                |
|   35 | Verify local and remote MAC learning                   | VTEP        | `show mac address-table dynamic`                           | Local MAC appears on access port and remote MAC appears behind `nve1(<peer-ip>)` |                                                |
|   36 | Verify core is only forwarding multicast               | Core        | `show ip mroute <multicast-group>`                         | Core shows multicast state, not NVE/VNI state                                    |                                                |
|   37 | Save configuration                                     | VTEP / Core | `copy running-config startup-config`                       | Configuration is saved                                                           |                                                |

# VXLAN_Flood_And_Learn_VTEP_Skeleton

```text
feature pim
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address <rp-ip>

vlan <vlan-id>
 vn-segment <vni-id>

interface loopback<id>
 ip address <vtep-loopback-ip>/32
 ip router ospf <process-id> area <area-id>
 ip pim sparse-mode

interface <underlay-interface>
 ip pim sparse-mode

interface nve1
 no shutdown
 source-interface loopback<id>
 member vni <vni-id> mcast-group <multicast-group>
```


# VXLAN_Flood_And_Learn_Multicast_Core_Skeleton

```text
feature pim

ip pim rp-address <rp-ip>

interface loopback<id>
 ip address <rp-or-router-loopback-ip>/32
 ip router ospf <process-id> area <area-id>
 ip pim sparse-mode

interface <underlay-interface>
 ip pim sparse-mode
```


# VXLAN_Flood_And_Learn_Anycast_RP_Skeleton

```text
feature pim

interface loopback<rp-loopback-id>
 ip address <anycast-rp-ip>/32
 ip router ospf <process-id> area <area-id>
 ip pim sparse-mode

ip pim rp-address <anycast-rp-ip>
ip pim anycast-rp <anycast-rp-ip> <rp-member-ip-1>
ip pim anycast-rp <anycast-rp-ip> <rp-member-ip-2>

interface <underlay-interface>
 ip pim sparse-mode
```


# VXLAN_Flood_And_Learn_vPC_VTEP_Skeleton

```text
feature vpc
feature pim
feature vn-segment-vlan-based
feature nv overlay

vpc domain <domain-id>
 peer-switch
 peer-keepalive destination <peer-keepalive-dst-ip> source <peer-keepalive-src-ip>
 peer-gateway

interface loopback<id>
 ip address <unique-vtep-primary-ip>/32
 ip address <shared-anycast-vtep-ip>/32 secondary
 ip router ospf <process-id> area <area-id>
 ip pim sparse-mode

interface nve1
 no shutdown
 source-interface loopback<id>
 member vni <vni-id> mcast-group <multicast-group>
```


# VXLAN_Flood_And_Learn_Verification_Commands

| Task | Command | Expected Result |
|---|---|---|
| Verify PIM neighbors | `show ip pim neighbor` | PIM neighbors exist on routed underlay links |
| Verify RP state | `show ip pim rp` | Correct RP is listed for multicast group range |
| Verify multicast tree | `show ip mroute <multicast-group>` | `*,G` and/or `S,G` entries exist for VXLAN group |
| Verify NVE interface | `show nve interface` | `nve1` is up and uses the correct source loopback |
| Verify VNI state | `show nve vni` | VNI is `Up`, mode is `DP`, type is `L2` |
| Verify detailed VNI | `show nve vni <vni-id> detail` | VNI shows multicast group and provision state `add-complete` |
| Verify internal VNI readiness | `show nve internal vni <vni-id>` | Ready state shows `L2-vni-flood-learn-ready` |
| Verify VLAN-to-VNI mapping | `show vxlan` | VLAN maps to expected VNID |
| Verify NVE peer after traffic | `show nve peers` | Remote VTEP appears as `Up`, learn type `DP` |
| Verify NVE peer details | `show nve peers detail` | Peer shows configured VNI and provision state `add-complete` |
| Verify MAC learning | `show mac address-table dynamic` | Remote MAC appears through `nve1(<remote-vtep-ip>)` |
| Verify endpoint reachability | `ping <remote-host-ip>` | Host-to-host reachability succeeds |
| Verify vPC state if used | `show vpc brief` | vPC peer adjacency and vPC member ports are up |
| Verify vPC VXLAN consistency if used | `show vpc consistency-parameters global` | NVE source, secondary IP, host reach mode, and VNI config match |
# VXLAN_Flood_And_Learn_BUM_Traffic_Handling

| Traffic Type          | How Flood-and-Learn Handles It                                | Required Mechanism                                  | Verification                                       |
| --------------------- | ------------------------------------------------------------- | --------------------------------------------------- | -------------------------------------------------- |
| Broadcast             | Encapsulated to the VNI multicast group                       | `member vni <vni-id> mcast-group <multicast-group>` | `show ip mroute <multicast-group>`                 |
| Unknown unicast       | Flooded through the multicast group until MAC learning occurs | PIM sparse-mode and RP in the underlay              | `show mac address-table dynamic`                   |
| Multicast             | Transported through multicast tree in underlay                | PIM sparse-mode on VTEP and core links              | `show ip pim neighbor`, `show ip mroute`           |
| Remote MAC learning   | Learned from data-plane traffic                               | Flood-and-learn data plane                          | `show nve peers`, `show mac address-table dynamic` |
| Remote VTEP discovery | Learned after traffic flows                                   | Data-plane traffic through multicast group          | `show nve peers` shows learn type `DP`             |

# VXLAN_Flood_And_Learn_Rollback

| Step | Task                                                       | Device      | Command                                                                      | Expected Result                                     |
| ---: | ---------------------------------------------------------- | ----------- | ---------------------------------------------------------------------------- | --------------------------------------------------- |
|    1 | Remove VNI from NVE                                        | VTEP        | `interface nve1` then `no member vni <vni-id> mcast-group <multicast-group>` | VNI is removed from NVE                             |
|    2 | Shut NVE interface if removing VXLAN entirely              | VTEP        | `interface nve1` then `shutdown`                                             | NVE stops forwarding VXLAN traffic                  |
|    3 | Remove VLAN-to-VNI mapping                                 | VTEP        | `vlan <vlan-id>` then `no vn-segment <vni-id>`                               | VLAN is no longer mapped to VXLAN                   |
|    4 | Remove tenant VLAN if no longer needed                     | VTEP        | `no vlan <vlan-id>`                                                          | VLAN is removed                                     |
|    5 | Remove PIM from underlay interface if no longer used       | VTEP / Core | `interface <underlay-interface>` then `no ip pim sparse-mode`                | Interface no longer participates in PIM             |
|    6 | Remove RP config if no longer used                         | VTEP / Core | `no ip pim rp-address <rp-ip>`                                               | RP config is removed                                |
|    7 | Remove VXLAN features only after all dependencies are gone | VTEP        | `no feature nv overlay` then `no feature vn-segment-vlan-based`              | VXLAN overlay and VLAN-to-VNI features are disabled |
|    8 | Save rollback config                                       | VTEP / Core | `copy running-config startup-config`                                         | Rollback state is saved                             |

# VXLAN_Flood_And_Learn_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| VNI is not up | `show nve vni` | Missing `feature nv overlay`, missing `interface nve1`, NVE shut, wrong source interface, or VNI not configured |
| VLAN is not mapped to VNI | `show vxlan` | Missing `vn-segment <vni-id>` under the tenant VLAN |
| NVE interface is down | `show nve interface` | `interface nve1` is shut or source loopback is missing/down |
| No NVE peers before traffic | `show nve peers` | Normal in flood-and-learn before endpoint traffic starts |
| No NVE peers after traffic | `show nve peers` | Multicast tree failure, VTEP loopback reachability issue, or VNI/multicast mismatch |
| NVE peer learn type is not DP | `show nve peers` | You are not looking at flood-and-learn behavior, or EVPN/control-plane config exists |
| Multicast group missing | `show ip mroute <multicast-group>` | PIM not enabled, RP missing, RPF failure, or VTEPs not joining the group |
| PIM neighbors missing | `show ip pim neighbor` | PIM not enabled on routed underlay links or physical underlay adjacency is down |
| RP missing or wrong | `show ip pim rp` | RP not configured consistently across VTEPs and core |
| Remote MAC not learned | `show mac address-table dynamic` | No traffic, VNI mismatch, multicast failure, or access VLAN issue |
| Ping fails but VNI is up | `show mac address-table dynamic` | Host VLAN, access port, MAC learning, or endpoint addressing issue |
| Core switch has no NVE state | `show nve vni` | Expected behavior. Core only needs multicast routing, not VXLAN/NVE features |
| vPC VTEP mismatch | `show vpc consistency-parameters global` | VLAN-to-VNI map, NVE source, shared secondary VTEP IP, or VNI config mismatch |
