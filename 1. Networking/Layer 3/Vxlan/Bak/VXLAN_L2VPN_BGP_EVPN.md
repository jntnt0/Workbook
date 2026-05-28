# VXLAN_L2VPN_BGP_EVPN_Prerequisites

| Step | Task                                                          | Device       | Command                                            | Expected Result                                                               |
| ---: | ------------------------------------------------------------- | ------------ | -------------------------------------------------- | ----------------------------------------------------------------------------- |
|    1 | Confirm NX-OS platform                                        | Leaf / Spine | `show version`                                     | Nexus/NX-OS platform is confirmed                                             |
|    2 | Verify feature licensing/support                              | Leaf / Spine | `show license usage`                               | Required NX-OS features are available                                         |
|    3 | Verify underlay interfaces                                    | Leaf / Spine | `show ip interface brief`                          | Routed underlay links and loopbacks show `up/up`                              |
|    4 | Verify underlay routing                                       | Leaf / Spine | `show ip route <remote-loopback-ip>`               | Remote loopbacks are reachable                                                |
|    5 | Verify BGP loopback is separate from NVE loopback             | Leaf         | `show running-config interface loopback<id>`       | NVE source loopback is not the same loopback used for BGP updates             |
|    6 | Verify NVE source loopback is in default VRF                  | Leaf         | `show running-config interface loopback<nve-id>`   | NVE loopback is not placed inside a tenant VRF                                |
|    7 | Verify L3 uplinks are in default VRF                          | Leaf         | `show running-config interface <uplink-interface>` | VXLAN underlay uplinks are routed L3 interfaces in the default VRF            |
|    8 | Verify no multi-access ARP issue on L3 uplinks                | Leaf         | `show ip arp interface <uplink-interface>`         | L3 uplink has expected point-to-point neighbor behavior                       |
|    9 | Verify baseline BGP reachability path                         | Leaf / Spine | `ping <peer-loopback-ip> source loopback<bgp-id>`  | Loopback-sourced reachability succeeds                                        |
|   10 | Verify no flood-and-learn-only config remains unintentionally | Leaf         | `show running-config interface nve1`               | L2VNI uses either EVPN ingress replication or multicast replication by design |
# VXLAN_L2VPN_BGP_EVPN_Configuration_Checklist

| Step | Task                                             | Device       | Command                                                 | Expected Result                                            |
| ---: | ------------------------------------------------ | ------------ | ------------------------------------------------------- | ---------------------------------------------------------- |
|    1 | Enter global configuration mode                  | Leaf / Spine | `configure terminal`                                    | Device enters global configuration mode                    |
|    2 | Enable EVPN overlay control plane                | Leaf / Spine | `nv overlay evpn`                                       | EVPN control plane is enabled                              |
|    3 | Enable BGP feature                               | Leaf / Spine | `feature bgp`                                           | BGP feature is enabled                                     |
|    4 | Enable VXLAN overlay feature                     | Leaf         | `feature nv overlay`                                    | NVE/VXLAN overlay support is enabled                       |
|    5 | Enable VLAN-to-VNI mapping                       | Leaf         | `feature vn-segment-vlan-based`                         | VLANs can be mapped to VNIs                                |
|    6 | Enable SVI support                               | Leaf         | `feature interface-vlan`                                | VLAN interfaces can be configured                          |
|    7 | Configure tenant VLAN                            | Leaf         | `vlan <tenant-vlan-id>`                                 | Tenant VLAN exists locally                                 |
|    8 | Map tenant VLAN to L2 VNI                        | Leaf         | `vn-segment <l2-vni>`                                   | Tenant VLAN is bound to the L2 VNI                         |
|    9 | Configure L3VNI VLAN                             | Leaf         | `vlan <l3vni-vlan-id>`                                  | L3VNI VLAN exists locally                                  |
|   10 | Map L3VNI VLAN to L3 VNI                         | Leaf         | `vn-segment <l3-vni>`                                   | L3VNI VLAN is bound to the L3 VNI                          |
|   11 | Create tenant VRF                                | Leaf         | `vrf context <vrf-name>`                                | Tenant VRF is created                                      |
|   12 | Associate tenant VRF with L3 VNI                 | Leaf         | `vni <l3-vni>`                                          | Tenant VRF is tied to the L3 VNI                           |
|   13 | Configure VRF route distinguisher                | Leaf         | `rd auto`                                               | RD is automatically derived                                |
|   14 | Enter IPv4 VRF address family                    | Leaf         | `address-family ipv4 unicast`                           | IPv4 VRF address-family mode is entered                    |
|   15 | Configure IPv4 route targets                     | Leaf         | `route-target both auto`                                | IPv4 import/export route target is configured              |
|   16 | Configure EVPN route targets                     | Leaf         | `route-target both auto evpn`                           | EVPN import/export route target is configured              |
|   17 | Configure L3VNI SVI                              | Leaf         | `interface vlan <l3vni-vlan-id>`                        | L3VNI SVI configuration mode is entered                    |
|   18 | Bind L3VNI SVI to tenant VRF                     | Leaf         | `vrf member <vrf-name>`                                 | L3VNI SVI belongs to tenant VRF                            |
|   19 | Enable L3VNI forwarding                          | Leaf         | `ip forward`                                            | SVI forwards for the L3 VNI                                |
|   20 | Disable redirects on L3VNI SVI                   | Leaf         | `no ip redirects`                                       | IP redirects are disabled                                  |
|   21 | Configure tenant gateway SVI                     | Leaf         | `interface vlan <tenant-vlan-id>`                       | Tenant SVI configuration mode is entered                   |
|   22 | Bind tenant SVI to tenant VRF                    | Leaf         | `vrf member <vrf-name>`                                 | Tenant SVI belongs to tenant VRF                           |
|   23 | Configure tenant gateway IP                      | Leaf         | `ip address <gateway-ip>/<prefix-length>`               | Tenant SVI has the default gateway IP                      |
|   24 | Configure distributed anycast gateway MAC        | Leaf         | `fabric forwarding anycast-gateway-mac <mac-address>`   | Common anycast gateway MAC is configured across leaf VTEPs |
|   25 | Enable distributed gateway on tenant SVI         | Leaf         | `fabric forwarding mode anycast-gateway`                | Leaf acts as the tenant distributed gateway                |
|   26 | Enter NVE interface                              | Leaf         | `interface nve1`                                        | NVE interface configuration mode is entered                |
|   27 | Enable NVE interface                             | Leaf         | `no shutdown`                                           | NVE interface is administratively up                       |
|   28 | Set NVE source interface                         | Leaf         | `source-interface loopback<nve-id>`                     | NVE uses the VTEP loopback as source                       |
|   29 | Enable EVPN host reachability                    | Leaf         | `host-reachability protocol bgp`                        | MAC/IP reachability uses BGP EVPN                          |
|   30 | Add L3 VNI to NVE                                | Leaf         | `member vni <l3-vni> associate-vrf`                     | L3 VNI is associated with the tenant VRF                   |
|   31 | Add L2 VNI to NVE                                | Leaf         | `member vni <l2-vni>`                                   | L2 VNI context is entered under NVE                        |
|   32 | Choose ingress replication for BUM traffic       | Leaf         | `ingress-replication protocol bgp`                      | BUM replication uses BGP EVPN ingress replication          |
|   33 | Or choose multicast replication for BUM traffic  | Leaf         | `mcast-group <multicast-group>`                         | BUM replication uses multicast in the underlay             |
|   34 | Configure BGP process                            | Leaf / Spine | `router bgp <asn>`                                      | BGP process is created                                     |
|   35 | Configure BGP router ID                          | Leaf / Spine | `router-id <loopback-ip>`                               | Stable BGP router ID is set                                |
|   36 | Configure EVPN neighbor                          | Leaf / Spine | `neighbor <peer-loopback-ip> remote-as <asn>`           | BGP EVPN neighbor is defined                               |
|   37 | Source BGP from loopback                         | Leaf / Spine | `update-source loopback<bgp-id>`                        | BGP session uses loopback as source                        |
|   38 | Enter EVPN neighbor address family               | Leaf / Spine | `address-family l2vpn evpn`                             | EVPN address-family mode is entered                        |
|   39 | Send standard and extended communities           | Leaf / Spine | `send-community both`                                   | EVPN route-target and community attributes are exchanged   |
|   40 | Configure route reflector client on iBGP spine   | Spine        | `route-reflector-client`                                | Spine reflects EVPN routes to leafs                        |
|   41 | Retain route targets on eBGP spine if used       | Spine        | `retain route-target all`                               | Spine retains EVPN routes without local VNI membership     |
|   42 | Preserve next-hop behavior for eBGP EVPN if used | Spine        | `route-map <name> out` with `set ip next-hop unchanged` | Spine does not break EVPN next-hop behavior                |
|   43 | Configure tenant VRF BGP context                 | Leaf         | `vrf <vrf-name>`                                        | Tenant VRF BGP mode is entered                             |
|   44 | Enter tenant IPv4 address family                 | Leaf         | `address-family ipv4 unicast`                           | Tenant IPv4 BGP address-family is entered                  |
|   45 | Redistribute direct tenant routes                | Leaf         | `redistribute direct route-map <route-map-name>`        | Direct tenant routes are advertised into EVPN              |
|   46 | Save configuration                               | Leaf / Spine | `copy running-config startup-config`                    | Configuration is saved                                     |

# VXLAN_L2VPN_BGP_EVPN_BUM_Replication_Options

| Option | Command | Where | Use Case | Verification |
|---|---|---|---|---|
| Ingress replication | `ingress-replication protocol bgp` | Under `interface nve1` then `member vni <l2-vni>` | BUM traffic is replicated by the VTEPs using EVPN-learned remote VTEPs | `show nve vni`, `show nve peers`, `show l2route evpn imet all` |
| Multicast replication | `mcast-group <multicast-group>` | Under `interface nve1` then `member vni <l2-vni>` | BUM traffic is replicated through multicast in the underlay | `show ip pim neighbor`, `show ip pim rp`, `show ip mroute <multicast-group>` |
| Global multicast group | `global mcast-group <multicast-group>` | Under `interface nve1` | Common multicast group is used for L2VNIs unless overridden | `show running-config interface nve1`, `show ip mroute <multicast-group>` |

| Rule                                                                                        | Operational Meaning                                                              |
| ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Do not configure ingress replication and multicast replication casually for the same L2 VNI | Pick one BUM replication model unless the design specifically requires otherwise |
| If using multicast replication                                                              | PIM sparse-mode and RP must work in the underlay                                 |
| If using ingress replication                                                                | EVPN control plane must discover the remote VTEPs                                |
| If no replication method exists                                                             | L2 VNI may come up incorrectly or fail BUM traffic behavior                      |

# VXLAN_L2VPN_BGP_EVPN_Leaf_Skeleton_Ingress_Replication

```text
nv overlay evpn
feature bgp
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

fabric forwarding anycast-gateway-mac <anycast-gateway-mac>

vlan <tenant-vlan-id>
 vn-segment <l2-vni>

vlan <l3vni-vlan-id>
 vn-segment <l3-vni>

vrf context <vrf-name>
 vni <l3-vni>
 rd auto
 address-family ipv4 unicast
  route-target both auto
  route-target both auto evpn

interface vlan <l3vni-vlan-id>
 no shutdown
 vrf member <vrf-name>
 ip forward
 no ip redirects

interface vlan <tenant-vlan-id>
 no shutdown
 vrf member <vrf-name>
 ip address <gateway-ip>/<prefix-length>
 fabric forwarding mode anycast-gateway

interface nve1
 no shutdown
 source-interface loopback<nve-id>
 host-reachability protocol bgp
 member vni <l3-vni> associate-vrf
 member vni <l2-vni>
  ingress-replication protocol bgp

router bgp <asn>
 router-id <leaf-bgp-loopback-ip>
 neighbor <spine-loopback-ip> remote-as <asn>
  update-source loopback<bgp-id>
  address-family l2vpn evpn
   send-community both

 vrf <vrf-name>
  address-family ipv4 unicast
   redistribute direct route-map <route-map-name>
```

# VXLAN_L2VPN_BGP_EVPN_Leaf_Skeleton_Multicast_Replication

```text
nv overlay evpn
feature bgp
feature pim
feature interface-vlan
feature vn-segment-vlan-based
feature nv overlay

ip pim rp-address <rp-ip>

fabric forwarding anycast-gateway-mac <anycast-gateway-mac>

vlan <tenant-vlan-id>
 vn-segment <l2-vni>

vlan <l3vni-vlan-id>
 vn-segment <l3-vni>

vrf context <vrf-name>
 vni <l3-vni>
 rd auto
 address-family ipv4 unicast
  route-target both auto
  route-target both auto evpn

interface <underlay-uplink-interface>
 ip pim sparse-mode

interface loopback<nve-id>
 ip pim sparse-mode

interface vlan <l3vni-vlan-id>
 no shutdown
 vrf member <vrf-name>
 ip forward
 no ip redirects

interface vlan <tenant-vlan-id>
 no shutdown
 vrf member <vrf-name>
 ip address <gateway-ip>/<prefix-length>
 fabric forwarding mode anycast-gateway

interface nve1
 no shutdown
 source-interface loopback<nve-id>
 host-reachability protocol bgp
 member vni <l3-vni> associate-vrf
 member vni <l2-vni>
  mcast-group <multicast-group>

router bgp <asn>
 router-id <leaf-bgp-loopback-ip>
 neighbor <spine-loopback-ip> remote-as <asn>
  update-source loopback<bgp-id>
  address-family l2vpn evpn
   send-community both

 vrf <vrf-name>
  address-family ipv4 unicast
   redistribute direct route-map <route-map-name>
```


# VXLAN_L2VPN_BGP_EVPN_Spine_iBGP_Route_Reflector_Skeleton

```text
nv overlay evpn
feature bgp

router bgp <asn>
 router-id <spine-loopback-ip>
 neighbor <leaf-loopback-ip> remote-as <asn>
  update-source loopback<bgp-id>
  address-family l2vpn evpn
   send-community both
   route-reflector-client
```

# VXLAN_L2VPN_BGP_EVPN_Spine_eBGP_Skeleton

```text
nv overlay evpn
feature bgp

route-map <next-hop-unchanged-route-map> permit 10
 set ip next-hop unchanged

router bgp <spine-asn>
 router-id <spine-loopback-ip>
 address-family l2vpn evpn
  retain route-target all

 neighbor <leaf-loopback-ip> remote-as <leaf-asn>
  update-source loopback<bgp-id>
  address-family l2vpn evpn
   send-community both
   disable-peer-as-check
   route-map <next-hop-unchanged-route-map> out
```


# VXLAN_L2VPN_BGP_EVPN_Verification_Commands

| Task | Command | Expected Result |
|---|---|---|
| Verify NVE interface | `show nve interface` | `nve1` is up and uses the expected source loopback |
| Verify NVE peers | `show nve peers` | Remote VTEPs appear as up |
| Verify VNI state | `show nve vni` | L2 and L3 VNIs appear as up |
| Verify VRF-to-VNI mapping | `show nve vrf` | Tenant VRF maps to the expected L3 VNI |
| Verify EVPN BGP sessions | `show bgp l2vpn evpn summary` | EVPN neighbors are established |
| Verify EVPN routes | `show bgp l2vpn evpn` | EVPN route entries are present |
| Verify MAC routes | `show l2route evpn mac all` | Local and remote MAC routes are installed |
| Verify MAC/IP routes | `show l2route evpn mac-ip all` | MAC/IP bindings are learned through EVPN |
| Verify IMET routes | `show l2route evpn imet all` | Inclusive multicast Ethernet tag routes exist for BUM replication |
| Verify flood-list routes | `show l2route evpn fl all` | Flood-list routes exist for BUM forwarding |
| Verify VLAN-to-VNI mapping | `show vxlan` | VLAN maps to the expected VNI |
| Verify tenant ARP table | `show ip arp vrf <vrf-name>` | Tenant endpoint ARP entries are present |
| Verify tenant routing table | `show ip route vrf <vrf-name>` | Tenant routes exist in the correct VRF |
| Verify multicast replication if used | `show ip mroute <multicast-group>` | Multicast state exists for the VXLAN group |
| Verify PIM neighbors if multicast is used | `show ip pim neighbor` | PIM neighbors are formed on L3 uplinks |
| Verify endpoint reachability | `ping <remote-host-ip>` | Host-to-host traffic succeeds across VXLAN EVPN |
# VXLAN_L2VPN_BGP_EVPN_Rollback

| Step | Task                                          | Device       | Command                                                                                   | Expected Result                              |
| ---: | --------------------------------------------- | ------------ | ----------------------------------------------------------------------------------------- | -------------------------------------------- |
|    1 | Remove L2 VNI from NVE                        | Leaf         | `interface nve1` then `no member vni <l2-vni>`                                            | L2 VNI is removed from NVE                   |
|    2 | Remove L3 VNI from NVE                        | Leaf         | `interface nve1` then `no member vni <l3-vni> associate-vrf`                              | L3 VNI is removed from NVE                   |
|    3 | Shut NVE if removing VXLAN entirely           | Leaf         | `interface nve1` then `shutdown`                                                          | NVE stops VXLAN forwarding                   |
|    4 | Remove tenant SVI anycast gateway mode        | Leaf         | `interface vlan <tenant-vlan-id>` then `no fabric forwarding mode anycast-gateway`        | Tenant SVI no longer acts as anycast gateway |
|    5 | Remove tenant SVI                             | Leaf         | `no interface vlan <tenant-vlan-id>`                                                      | Tenant SVI is removed                        |
|    6 | Remove L3VNI SVI                              | Leaf         | `no interface vlan <l3vni-vlan-id>`                                                       | L3VNI SVI is removed                         |
|    7 | Remove VLAN-to-VNI mapping                    | Leaf         | `vlan <tenant-vlan-id>` then `no vn-segment <l2-vni>`                                     | Tenant VLAN no longer maps to L2 VNI         |
|    8 | Remove L3VNI VLAN mapping                     | Leaf         | `vlan <l3vni-vlan-id>` then `no vn-segment <l3-vni>`                                      | L3VNI VLAN no longer maps to L3 VNI          |
|    9 | Remove VRF VNI binding                        | Leaf         | `vrf context <vrf-name>` then `no vni <l3-vni>`                                           | VRF is no longer tied to L3 VNI              |
|   10 | Remove EVPN BGP neighbor AF                   | Leaf / Spine | `router bgp <asn>` then `neighbor <peer-loopback-ip>` then `no address-family l2vpn evpn` | EVPN address-family is removed for the peer  |
|   11 | Remove BGP neighbor if no longer needed       | Leaf / Spine | `router bgp <asn>` then `no neighbor <peer-loopback-ip>`                                  | BGP neighbor is removed                      |
|   12 | Remove multicast config if not used elsewhere | Leaf / Spine | `no ip pim rp-address <rp-ip>`                                                            | RP config is removed                         |
|   13 | Remove PIM from uplinks if not used elsewhere | Leaf / Spine | `interface <underlay-interface>` then `no ip pim sparse-mode`                             | Interface no longer participates in PIM      |
|   14 | Save rollback state                           | Leaf / Spine | `copy running-config startup-config`                                                      | Rollback configuration is saved              |
# VXLAN_L2VPN_BGP_EVPN_Failure_Checks

| Symptom | Command | What Usually Broke |
|---|---|---|
| EVPN BGP neighbor is down | `show bgp l2vpn evpn summary` | Wrong neighbor IP, wrong AS, missing update-source, loopback unreachable, or EVPN AF missing |
| EVPN routes missing | `show bgp l2vpn evpn` | Neighbor down, missing `send-community both`, route-target mismatch, or route reflector issue |
| NVE peer missing | `show nve peers` | EVPN control plane down, VTEP loopback unreachable, or NVE source loopback wrong |
| L2 VNI down | `show nve vni` | Missing VLAN-to-VNI mapping, missing NVE member VNI, or NVE interface shut |
| L3 VNI down | `show nve vrf` | Missing `member vni <l3-vni> associate-vrf`, missing VRF `vni`, or L3VNI SVI problem |
| VLAN not mapped to VNI | `show vxlan` | Missing `vn-segment <vni>` under VLAN |
| MAC routes missing | `show l2route evpn mac all` | Endpoint not learned, EVPN route type 2 not received, or BGP EVPN issue |
| MAC/IP routes missing | `show l2route evpn mac-ip all` | Host ARP not learned, anycast gateway issue, or no endpoint traffic |
| IMET routes missing | `show l2route evpn imet all` | Missing ingress replication config, EVPN route exchange issue, or L2 VNI not active |
| Flood-list routes missing | `show l2route evpn fl all` | BUM replication mode is missing or EVPN has not learned remote VTEPs |
| Ingress replication fails | `show nve peers` | Remote VTEP discovery through EVPN is broken |
| Multicast replication fails | `show ip mroute <multicast-group>` | PIM sparse-mode, RP, RPF, or multicast group configuration is broken |
| Tenant gateway fails | `show running-config interface vlan <tenant-vlan-id>` | Missing VRF binding, IP address, or anycast gateway mode |
| Tenant routes missing | `show ip route vrf <vrf-name>` | Missing direct redistribution, wrong route-map, or tenant SVI down |
| Spine has no useful EVPN routes in eBGP design | `show bgp l2vpn evpn` | Missing `retain route-target all`, next-hop handling issue, or route-target mismatch |
| vPC VXLAN inconsistency | `show vpc consistency-parameters global` | NVE source, VLAN-to-VNI mapping, shared secondary VTEP IP, or VNI config mismatch |