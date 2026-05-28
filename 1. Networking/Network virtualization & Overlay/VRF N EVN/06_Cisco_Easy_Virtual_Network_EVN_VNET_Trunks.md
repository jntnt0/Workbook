

Cisco’s EVN guide confirms the critical pieces: EVN uses VRFs plus global VNET tags, the same VNET tag must match across routers, vnet trunk carries tagged VRF traffic, and the hidden dot1Q subinterfaces inherit the trunk interface configuration. Cisco’s verification list includes show vnet tag, show running-config vnet, show vrf list, show vnet detail, and show vnet counters.  

Cisco_Easy_Virtual_Network_EVN_VNET_Trunks.md

Cisco_Easy_Virtual_Network_EVN_VNET_Trunks

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Local KB index check | No dedicated EVN source was found in the local KB, so Cisco EVN documentation is used for EVN-specific syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 18, `Implementing and Verifying VRF-Lite` | Supports the VRF-Lite baseline that EVN builds on |
| `Cisco Easy Virtual Network Configuration Guide` | `Configuring Easy Virtual Network` | Supports `vrf definition`, `vnet tag`, `address-family ipv4`, `vnet trunk`, `vnet name`, and OSPF per-VRF syntax |
| `Cisco Easy Virtual Network Configuration Example` | `Configure EVN` | Supports VNET trunk behavior, hidden dot1Q subinterfaces, inherited interface configuration, and same-tag requirements |
| `Cisco Easy Virtual Network Configuration Guide` | `Verifying EVN Configurations` | Supports `show vnet tag`, `show running-config vnet`, `show vrf list`, `show vnet detail`, and `show vnet counters` |
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EVN | Cisco Easy Virtual Network simplifies VRF-Lite transport across routed infrastructure |
| Virtual network | A VRF with a `vnet tag` assigned |
| VNET tag | The numeric tag that identifies the same virtual network across routers |
| Same tag rule | The same VRF must use the same VNET tag on every EVN router |
| VNET trunk | A routed interface that carries multiple virtual networks using dot1Q tags |
| Hidden subinterfaces | EVN automatically creates hidden dot1Q subinterfaces for VRFs that have VNET tags |
| Inherited trunk config | IP address and supported interface commands on the trunk are inherited by the hidden subinterfaces |
| Edge interface | A normal VRF-bound interface that connects a host, VLAN, or access segment into one virtual network |
| VRF list | Optional filter that limits which VRFs are allowed on a specific VNET trunk |
| `vnet name` | Optional per-VRF trunk submode used to override inherited settings for one virtual network |
| Per-VRF routing | EVN transport does not replace routing. Each VRF still needs its own routing protocol instance or static routes |
| Global VNET | Untagged traffic remains in the global routing table |
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the image supports EVN before building the lab | All EVN routers | `show parser dump | include vnet trunk` | `vnet trunk` syntax exists on the platform |
| 2 | Confirm the routed trunk interfaces are present and not switchports | All EVN routers | `show ip interface brief` | Core-facing interfaces are available for routed EVN trunks |
| 3 | Confirm no manual subinterfaces already exist on the future VNET trunk | All EVN routers | `show running-config interface GigabitEthernet0/0` | Interface is clean enough for EVN auto-created subinterfaces |
| 4 | Enter global configuration mode | All EVN routers | `configure terminal` | Router enters configuration mode |
| 5 | Create the RED virtual network | All EVN routers | `vrf definition RED` | RED VRF definition mode opens |
| 6 | Assign RED VNET tag | All EVN routers | `vnet tag 100` | RED is associated with VNET tag 100 |
| 7 | Enable RED IPv4 address family | All EVN routers | `address-family ipv4` | RED IPv4 address family opens |
| 8 | Exit RED address-family mode | All EVN routers | `exit-address-family` | Router returns to VRF definition mode |
| 9 | Create the GREEN virtual network | All EVN routers | `vrf definition GREEN` | GREEN VRF definition mode opens |
| 10 | Assign GREEN VNET tag | All EVN routers | `vnet tag 200` | GREEN is associated with VNET tag 200 |
| 11 | Enable GREEN IPv4 address family | All EVN routers | `address-family ipv4` | GREEN IPv4 address family opens |
| 12 | Exit GREEN address-family mode | All EVN routers | `exit-address-family` | Router returns to VRF definition mode |
| 13 | Verify VNET tag consistency locally | All EVN routers | `show vnet tag` | RED shows tag 100 and GREEN shows tag 200 |
| 14 | Configure the physical core-facing interface | EVN router | `interface GigabitEthernet0/0` | Trunk interface configuration mode opens |
| 15 | Make the core-facing interface a VNET trunk | EVN router | `vnet trunk` | Interface becomes an EVN trunk carrying all tagged VRFs by default |
| 16 | Assign the EVN trunk transport IP address | EVN router | `ip address 192.168.12.1 255.255.255.252` | Trunk IP is applied and inherited by hidden VNET subinterfaces |
| 17 | Bring up the VNET trunk | EVN router | `no shutdown` | Physical trunk interface can transition up/up |
| 18 | Repeat VNET trunk configuration on the far-end router | Neighbor EVN router | `interface GigabitEthernet0/0` then `vnet trunk` then `ip address 192.168.12.2 255.255.255.252` | Far-end EVN trunk uses the same transport subnet |
| 19 | Verify hidden RED and GREEN subinterfaces were derived | EVN router | `show derived-config | section GigabitEthernet0/0` | Hidden dot1Q subinterfaces appear for tags 100 and 200 |
| 20 | Verify VNET counters | EVN router | `show vnet counters` | Configured VNETs, trunk interfaces, and VNET subinterfaces are counted |
| 21 | Verify VNET detail | EVN router | `show vnet detail` | VNETs appear with correct tag and interface information |
| 22 | Optional: create a trunk allow-list for selected VRFs | EVN router | `vrf list EVN_ALLOWED` | VRF list configuration mode opens |
| 23 | Optional: add RED to the trunk allow-list | EVN router | `member RED` | RED becomes allowed in the list |
| 24 | Optional: add GREEN to the trunk allow-list | EVN router | `member GREEN` | GREEN becomes allowed in the list |
| 25 | Optional: apply the VRF list to the trunk | EVN router | `interface GigabitEthernet0/0` then `vnet trunk list EVN_ALLOWED` | Only listed VRFs are carried over the VNET trunk |
| 26 | Optional: verify the VRF list | EVN router | `show vrf list EVN_ALLOWED` | RED and GREEN appear as members |
| 27 | Optional: enter per-VRF trunk submode for RED | EVN router | `interface GigabitEthernet0/0` then `vnet name RED` | Router enters RED-specific VNET interface submode |
| 28 | Optional: tune RED OSPF cost on this trunk only | EVN router | `ip ospf cost 100` | RED hidden subinterface uses the overridden OSPF cost |
| 29 | Optional: exit VNET interface submode | EVN router | `exit-if-vnet` | Router returns to normal interface configuration mode |
| 30 | Configure RED edge interface | EVN edge router | `interface GigabitEthernet1/0` then `vrf forwarding RED` then `ip address 10.10.10.1 255.255.255.0` | RED access network enters the RED VRF |
| 31 | Configure GREEN edge interface | EVN edge router | `interface GigabitEthernet2/0` then `vrf forwarding GREEN` then `ip address 172.16.10.1 255.255.255.0` | GREEN access network enters the GREEN VRF |
| 32 | Configure RED routing over the VNET trunk | EVN router | `router ospf 10 vrf RED` then `network 192.168.12.0 0.0.0.3 area 0` | RED OSPF runs over the RED VNET trunk subinterface |
| 33 | Advertise RED access network | EVN router | `network 10.10.10.0 0.0.0.255 area 0` | RED access subnet is advertised inside RED |
| 34 | Configure GREEN routing over the VNET trunk | EVN router | `router ospf 20 vrf GREEN` then `network 192.168.12.0 0.0.0.3 area 0` | GREEN OSPF runs over the GREEN VNET trunk subinterface |
| 35 | Advertise GREEN access network | EVN router | `network 172.16.10.0 0.0.0.255 area 0` | GREEN access subnet is advertised inside GREEN |
| 36 | Verify RED OSPF adjacency | EVN router | `show ip ospf vrf RED neighbor` | RED neighbor forms over the hidden RED VNET subinterface |
| 37 | Verify GREEN OSPF adjacency | EVN router | `show ip ospf vrf GREEN neighbor` | GREEN neighbor forms over the hidden GREEN VNET subinterface |
| 38 | Verify RED learned routes | EVN router | `show ip route vrf RED ospf` | RED remote prefixes are learned over EVN transport |
| 39 | Verify GREEN learned routes | EVN router | `show ip route vrf GREEN ospf` | GREEN remote prefixes are learned over EVN transport |
| 40 | Test RED reachability across EVN | EVN router | `ping vrf RED 10.10.20.1` | RED remote destination replies |
| 41 | Test GREEN reachability across EVN | EVN router | `ping vrf GREEN 172.16.20.1` | GREEN remote destination replies |
| 42 | Prove RED and GREEN remain isolated | EVN router | `ping vrf RED 172.16.20.1` | Ping fails unless route replication or route leaking is configured |
| 43 | Save the working EVN VNET trunk baseline | All EVN routers | `write memory` | EVN VNET trunk configuration survives reload |
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Skeleton
configure terminal
vrf definition RED
 vnet tag 100
 address-family ipv4
 exit-address-family
exit
vrf definition GREEN
 vnet tag 200
 address-family ipv4
 exit-address-family
exit
interface GigabitEthernet0/0
 description EVN_VNET_TRUNK_TO_PEER
 vnet trunk
 ip address 192.168.12.1 255.255.255.252
 no shutdown
interface GigabitEthernet1/0
 description RED_EDGE_INTERFACE
 vrf forwarding RED
 ip address 10.10.10.1 255.255.255.0
 no shutdown
interface GigabitEthernet2/0
 description GREEN_EDGE_INTERFACE
 vrf forwarding GREEN
 ip address 172.16.10.1 255.255.255.0
 no shutdown
router ospf 10 vrf RED
 network 192.168.12.0 0.0.0.3 area 0
 network 10.10.10.0 0.0.0.255 area 0
router ospf 20 vrf GREEN
 network 192.168.12.0 0.0.0.3 area 0
 network 172.16.10.0 0.0.0.255 area 0
end
write memory
! Optional: restrict which VRFs ride a specific EVN trunk.
configure terminal
vrf list EVN_ALLOWED
 member RED
 member GREEN
 exit-vrf-list
interface GigabitEthernet0/0
 vnet trunk list EVN_ALLOWED
end
write memory
! Optional: override inherited trunk attributes for one VNET.
configure terminal
interface GigabitEthernet0/0
 vnet name RED
  ip ospf cost 100
 exit-if-vnet
end
write memory
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Verification_Commands
show parser dump | include vnet trunk
show vrf
show vrf detail
show vnet tag
show vnet detail
show vnet counters
show running-config vnet
show running-config vrf RED
show running-config vrf GREEN
show vrf list
show vrf list EVN_ALLOWED
show running-config interface GigabitEthernet0/0
show derived-config | section GigabitEthernet0/0
show derived-config interface GigabitEthernet0/0.100
show derived-config interface GigabitEthernet0/0.200
show ip interface brief
show vrf ipv4 unicast interfaces
show ip vrf interfaces
show ip ospf vrf RED neighbor
show ip ospf vrf GREEN neighbor
show ip protocols vrf RED
show ip protocols vrf GREEN
show ip route
show ip route vrf RED
show ip route vrf GREEN
show ip route vrf RED ospf
show ip route vrf GREEN ospf
show ip cef vrf RED 10.10.20.1
show ip cef vrf GREEN 172.16.20.1
ping vrf RED 10.10.20.1
ping vrf GREEN 172.16.20.1
ping vrf RED 172.16.20.1
traceroute vrf RED 10.10.20.1
traceroute vrf GREEN 172.16.20.1
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Rollback
configure terminal
router ospf 10 vrf RED
 no network 192.168.12.0 0.0.0.3 area 0
 no network 10.10.10.0 0.0.0.255 area 0
exit
router ospf 20 vrf GREEN
 no network 192.168.12.0 0.0.0.3 area 0
 no network 172.16.10.0 0.0.0.255 area 0
exit
no router ospf 10 vrf RED
no router ospf 20 vrf GREEN
interface GigabitEthernet0/0
 no vnet name RED
 no vnet trunk list EVN_ALLOWED
 no vnet trunk
 no ip address
 shutdown
 no description
interface GigabitEthernet1/0
 no ip address
 no vrf forwarding RED
 shutdown
 no description
interface GigabitEthernet2/0
 no ip address
 no vrf forwarding GREEN
 shutdown
 no description
no vrf list EVN_ALLOWED
vrf definition RED
 no vnet tag 100
 no address-family ipv4
exit
vrf definition GREEN
 no vnet tag 200
 no address-family ipv4
exit
no vrf definition RED
no vrf definition GREEN
end
write memory
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `vnet trunk` command is rejected | Platform or image does not support EVN | `show parser dump | include vnet trunk` | Use an EVN-capable IOS or IOS XE image, or fall back to manual VRF-Lite subinterfaces |
| VNET tag does not appear | VRF missing `vnet tag` or address family | `show vnet tag` | Add `vnet tag <number>` and `address-family ipv4` under the VRF |
| Hidden subinterfaces are not created | VRF has no VNET tag or trunk is not enabled | `show derived-config | section <interface>` | Configure `vnet tag` under VRF and `vnet trunk` on the physical routed interface |
| Hidden subinterfaces created with wrong tag | VNET tag mismatch | `show vnet tag` on both routers | Use the same tag for the same VRF on every EVN router |
| Trunk does not carry one VRF | VRF list blocks that VRF | `show vrf list` and `show running-config interface <interface>` | Add the VRF as a `member` or remove the trunk list |
| OSPF adjacency fails for one VRF only | Per-VRF routing process missing or wrong network statement | `show ip ospf vrf <VRF> neighbor` | Configure `router ospf <id> vrf <VRF>` and include the trunk subnet |
| OSPF adjacency for filtered VRF drops after applying trunk list | VRF was excluded from the applied list | `show vrf list <list-name>` | Add the missing VRF to the list or remove `vnet trunk list <list-name>` |
| RED reaches remote RED but not GREEN | Correct behavior when no route leaking or route replication exists | `show ip route vrf RED` | Configure EVN shared services or BGP/static leaking only if cross-VRF access is required |
| Routes appear in global instead of VRF | Routing process configured without `vrf <VRF>` | `show ip route` and `show ip route vrf <VRF>` | Move routing under `router ospf <id> vrf <VRF>` or the correct VRF routing context |
| Edge host subnet does not appear in VRF | Edge interface is not bound to the VRF or IP was removed after binding | `show vrf ipv4 unicast interfaces` | Apply `vrf forwarding <VRF>` first, then reapply the IP address |
| Per-VRF trunk tuning does not show in derived config | Override was applied in the wrong mode | `show derived-config interface <hidden-subinterface>` | Use `interface <trunk>` then `vnet name <VRF>` and apply the override there |
| Removing an override does not behave as expected | Some VNET submode commands require default form instead of no form | `show running-config interface <trunk>` | Use the documented `default <command>` or correct `no <command>` form inside `vnet name <VRF>` mode |
##### Source_Basis
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Mental_Model
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Configuration_Checklist
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Skeleton
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Verification_Commands
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Rollback
# Cisco_Easy_Virtual_Network_EVN_VNET_Trunks_Failure_Checks
