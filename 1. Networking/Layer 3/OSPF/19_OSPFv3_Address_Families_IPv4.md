OSPFv3_Address_Families_IPv4.md

OSPFv3_Address_Families_IPv4

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF and OSPFv3 topics to `All_combined_part3.md`, `Ios-xs_combined_epubs.md`, and ENARSI material |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 9, OSPFv3 | Supports OSPFv3 fundamentals, link-local neighbor communication, OSPFv3 interface enablement, and `show ospfv3` verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 10, Troubleshooting OSPFv3 Address Families | Supports OSPFv3 AF behavior, IPv4 AF verification, `show ip protocols`, `show ospfv3 interface brief`, `show ospfv3 database`, and `show ip route ospfv3` |
| `Ios-xs_combined_epubs.md` | OSPFv3 address-family configuration sections | Provides `router ospfv3 <process-id>`, `address-family ipv4 unicast`, and interface syntax `ospfv3 <process-id> ipv4 area <area-id>` |
| Related lab | `ospf-v3-ipv4-final` | Main lab for using OSPFv3 address families to carry IPv4 unicast routes |
# OSPFv3_Address_Families_IPv4_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPFv3 AF IPv4 | OSPFv3 can carry IPv4 unicast routes using the IPv4 address family |
| Not OSPFv2 | OSPFv3 IPv4 AF is not compatible with classic OSPFv2 IPv4 neighbors |
| IPv6 transport | OSPFv3 still uses IPv6 link-local addressing and OSPF protocol 89 for neighbor communication, even when carrying IPv4 routes |
| Required global IPv6 routing | `ipv6 unicast-routing` must be enabled because OSPFv3 uses IPv6 for control-plane exchange |
| Interface IPv6 enablement | Interfaces need IPv6 enabled or an IPv6 link-local address so OSPFv3 can form neighbors |
| IPv4 data prefix | Interfaces still need IPv4 addresses if they are advertising IPv4 unicast prefixes |
| Single OSPFv3 process | One `router ospfv3 <process-id>` can support IPv4 and IPv6 address families |
| Address-family mode | `address-family ipv4 unicast` contains IPv4 AF-specific settings like router ID, passive interface policy, default origination, and redistribution |
| Interface activation | OSPFv3 AF is enabled directly under the interface with `ospfv3 <process-id> ipv4 area <area-id>` |
| Per-AF adjacency | OSPFv3 can maintain separate IPv4 and IPv6 AF neighbor/LSDB behavior under the same process |
| Router ID | IPv4 AF still needs a 32-bit OSPF router ID. Set it manually |
| AF precedence | If a setting exists both globally under `router ospfv3` and under an address family, the address-family setting wins for that AF |
| Interface parameter scope | OSPFv3 interface parameters without `ipv4` or `ipv6` may apply to all AFs. AF-specific interface parameters apply only to that AF |
| Routing table view | IPv4 routes learned through OSPFv3 are verified with `show ip route ospfv3` |
| Protocol view | IPv4 AF protocol state can also appear under `show ip protocols` as `ospfv3 <process-id>` |
| LSDB view | `show ospfv3 database` shows OSPFv3 database information by address family |
# OSPFv3_Address_Families_IPv4_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the design is OSPFv3 AF IPv4, not classic OSPFv2 | Design step | `OSPFv3 IPv4 AF required: yes` | Routers will use OSPFv3 commands to carry IPv4 routes |
| 2 | Confirm all neighbors support OSPFv3 IPv4 AF | All routers | `show version` | Platform and image support OSPFv3 address families |
| 3 | Confirm IPv4 addressing on routed interfaces | All routers | `show ip interface brief` | Interfaces have the expected IPv4 addresses and are `up/up` |
| 4 | Confirm IPv6 is enabled globally | All routers | `show running-config | include ipv6 unicast-routing` | `ipv6 unicast-routing` is present or known to be missing |
| 5 | Enable IPv6 unicast routing | All routers | `configure terminal` then `ipv6 unicast-routing` | Router can run IPv6-based OSPFv3 control plane |
| 6 | Confirm IPv6 link-local capability on OSPFv3 interfaces | All routers | `show ipv6 interface <interface-id>` | Interface has an IPv6 link-local address |
| 7 | Enable IPv6 on an interface if no link-local address exists | All routers | `interface <interface-id>` then `ipv6 enable` | Interface generates an IPv6 link-local address |
| 8 | Optionally set a deterministic link-local address for easier lab troubleshooting | All routers | `ipv6 address FE80::<router-id> link-local` | Interface uses a predictable link-local address |
| 9 | Enter OSPFv3 process configuration | All routers | `router ospfv3 <process-id>` | OSPFv3 process exists locally |
| 10 | Enter IPv4 unicast address family | All routers | `address-family ipv4 unicast` | Router enters OSPFv3 IPv4 AF configuration mode |
| 11 | Configure deterministic IPv4 AF router ID | All routers | `router-id <router-id>` | IPv4 AF has a unique stable router ID |
| 12 | Configure passive-by-default policy if the router has edge interfaces | All routers | `passive-interface default` | No interface forms neighbors unless explicitly allowed |
| 13 | Allow OSPFv3 IPv4 AF on neighbor-facing interfaces | All routers | `no passive-interface <neighbor-interface>` | Intended transit interfaces can form OSPFv3 IPv4 AF neighbors |
| 14 | Optionally originate a default route into the IPv4 AF | Edge or ABR router | `default-information originate` | IPv4 default route is originated into OSPFv3 AF when conditions are met |
| 15 | Exit address-family mode | All routers | `exit-address-family` | Router returns to OSPFv3 router configuration mode |
| 16 | Exit router configuration | All routers | `exit` | Router returns to global configuration mode |
| 17 | Enter a neighbor-facing interface | All routers | `interface <neighbor-interface>` | Interface configuration mode opens |
| 18 | Confirm or configure IPv4 address on the interface | All routers | `ip address <ipv4-address> <mask>` | Interface has IPv4 prefix to advertise or use for forwarding |
| 19 | Confirm or enable IPv6 link-local support | All routers | `ipv6 enable` | Interface has IPv6 link-local transport for OSPFv3 |
| 20 | Enable OSPFv3 IPv4 AF on the interface | All routers | `ospfv3 <process-id> ipv4 area <area-id>` | Interface participates in OSPFv3 IPv4 AF in the intended area |
| 21 | Optionally make dedicated two-router links point-to-point | Both ends of transit link | `ospfv3 network point-to-point` | Link avoids DR/BDR behavior if used as a true two-router transit |
| 22 | Optionally set OSPFv3 interface cost | All routers | `ospfv3 cost <cost>` | OSPFv3 path metric is controlled |
| 23 | Enable OSPFv3 IPv4 AF on loopback interfaces that should be advertised | All routers | `interface Loopback0` then `ospfv3 <process-id> ipv4 area <area-id>` | Loopback IPv4 prefix is injected into OSPFv3 IPv4 AF |
| 24 | Save the configuration | All routers | `end` then `write memory` | Configuration is saved |
| 25 | Verify IPv4 AF process state | All routers | `show ospfv3` | Output shows `OSPFv3 <process-id> address-family ipv4` |
| 26 | Verify IPv4 AF protocol state | All routers | `show ip protocols` | Routing protocol shows `ospfv3 <process-id>` for IPv4 |
| 27 | Verify interface participation by AF | All routers | `show ospfv3 interface brief` | Interfaces appear with `AF ipv4`, correct process, area, cost, state, and neighbor count |
| 28 | Verify detailed interface state | All routers | `show ospfv3 interface <interface-id>` | Output shows IPv4 AF interface details, router ID, area, timers, and neighbor count |
| 29 | Verify neighbor adjacency | All routers | `show ospfv3 neighbor` | OSPFv3 IPv4 AF neighbors reach `FULL` |
| 30 | Verify LSDB by address family | All routers | `show ospfv3 database` | IPv4 AF LSAs are present |
| 31 | Verify IPv4 OSPFv3 route installation | All routers | `show ip route ospfv3` | IPv4 routes learned through OSPFv3 appear in the IPv4 RIB |
| 32 | Verify specific learned IPv4 prefix | All routers | `show ip route <remote-ipv4-prefix>` | Route points to the expected OSPFv3 next hop |
| 33 | Verify IPv4 forwarding | All routers | `ping <remote-ipv4-loopback>` | Remote IPv4 loopback or prefix responds |
| 34 | Verify forwarding path | All routers | `traceroute <remote-ipv4-loopback>` | Path follows expected OSPFv3 IPv4 AF route |
# OSPFv3_Address_Families_IPv4_Skeleton
! =========================================================
! OSPFv3 IPv4 address-family baseline
! OSPFv3 uses IPv6 link-local control-plane transport
! even when carrying IPv4 unicast routes.
! =========================================================
configure terminal
ipv6 unicast-routing
router ospfv3 <PROCESS_ID>
 address-family ipv4 unicast
  router-id <ROUTER_ID>
  passive-interface default
  no passive-interface <TRANSIT_INTERFACE_1>
  no passive-interface <TRANSIT_INTERFACE_2>
 exit-address-family
exit
interface <TRANSIT_INTERFACE_1>
 ip address <LOCAL_IPV4_ADDRESS> <MASK>
 ipv6 enable
 ospfv3 <PROCESS_ID> ipv4 area <AREA_ID>
 no shutdown
exit
interface <TRANSIT_INTERFACE_2>
 ip address <LOCAL_IPV4_ADDRESS> <MASK>
 ipv6 enable
 ospfv3 <PROCESS_ID> ipv4 area <AREA_ID>
 no shutdown
exit
interface Loopback0
 ip address <LOOPBACK_IPV4_ADDRESS> <MASK>
 ipv6 enable
 ospfv3 <PROCESS_ID> ipv4 area <AREA_ID>
exit
end
write memory
! =========================================================
! Point-to-point transit link option
! Use only on true two-router transit links.
! =========================================================
configure terminal
interface <TRANSIT_INTERFACE>
 ospfv3 network point-to-point
end
write memory
! =========================================================
! IPv4 AF default route origination
! Configure under the IPv4 address family, not the parent process,
! when the default should apply only to IPv4 AF.
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family ipv4 unicast
  default-information originate
 exit-address-family
end
write memory
! =========================================================
! Example
! =========================================================
configure terminal
ipv6 unicast-routing
router ospfv3 10
 address-family ipv4 unicast
  router-id 1.1.1.1
  passive-interface default
  no passive-interface GigabitEthernet0/0
 exit-address-family
exit
interface GigabitEthernet0/0
 ip address 10.12.1.1 255.255.255.0
 ipv6 enable
 ospfv3 10 ipv4 area 0
 no shutdown
exit
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 ipv6 enable
 ospfv3 10 ipv4 area 0
exit
end
write memory
# OSPFv3_Address_Families_IPv4_Verification_Commands
show running-config | include ipv6 unicast-routing
show running-config | section router ospfv3
show running-config interface <interface-id>
show ip interface brief
show ipv6 interface brief
show ipv6 interface <interface-id>
show ip protocols
show ospfv3
show ospfv3 interface brief
show ospfv3 interface <interface-id>
show ospfv3 neighbor
show ospfv3 neighbor detail
show ospfv3 database
show ospfv3 database router
show ospfv3 database prefix
show ip route ospfv3
show ip route <remote-ipv4-prefix>
ping <neighbor-ipv4-address>
ping <remote-ipv4-loopback>
traceroute <remote-ipv4-loopback>
show logging | include OSPFv3|OSPF|ADJCHG|link-local|IPv6
# OSPFv3_Address_Families_IPv4_Rollback
! =========================================================
! Remove OSPFv3 IPv4 AF from one interface
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ospfv3 <PROCESS_ID> ipv4 area <AREA_ID>
end
write memory
! =========================================================
! Remove OSPFv3 point-to-point network type
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ospfv3 network point-to-point
end
write memory
! =========================================================
! Remove IPv4 AF default route origination
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family ipv4 unicast
  no default-information originate
 exit-address-family
end
write memory
! =========================================================
! Remove passive-interface exception
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family ipv4 unicast
  passive-interface <INTERFACE_ID>
 exit-address-family
end
write memory
! =========================================================
! Remove passive-by-default policy
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family ipv4 unicast
  no passive-interface default
 exit-address-family
end
write memory
! =========================================================
! Remove only the IPv4 unicast address family
! Use carefully if the OSPFv3 process also carries IPv6.
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 no address-family ipv4 unicast
end
write memory
! =========================================================
! Remove the full OSPFv3 process
! This also removes IPv6 AF if it exists under the same process.
! =========================================================
configure terminal
no router ospfv3 <PROCESS_ID>
end
write memory
! =========================================================
! Remove IPv6 enablement from an interface only if no IPv6 control plane
! or IPv6 routing protocol still depends on it.
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ipv6 enable
end
write memory
# OSPFv3_Address_Families_IPv4_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| OSPFv3 IPv4 AF neighbor does not form | `ipv6 unicast-routing` is missing | `show running-config | include ipv6 unicast-routing` | Configure `ipv6 unicast-routing` |
| OSPFv3 IPv4 AF neighbor does not form | Interface has no IPv6 link-local address | `show ipv6 interface <interface-id>` | Configure `ipv6 enable` or an IPv6 link-local address |
| OSPFv3 IPv4 AF neighbor does not form | Interface was enabled for OSPFv2 instead of OSPFv3 IPv4 AF | `show running-config interface <interface-id>` | Replace `ip ospf <process> area <area>` with `ospfv3 <process> ipv4 area <area>` if OSPFv3 AF is required |
| OSPFv3 IPv4 AF neighbor does not form with OSPFv2 router | OSPFv2 and OSPFv3 AF IPv4 are not compatible | `show ip protocols`, `show ospfv3 neighbor` | Use OSPFv3 IPv4 AF on both routers or convert both to OSPFv2 |
| Interface missing from IPv4 AF | OSPFv3 was enabled for IPv6 AF only | `show running-config interface <interface-id>`, `show ospfv3 interface brief` | Configure `ospfv3 <process-id> ipv4 area <area-id>` |
| Interface appears under IPv6 AF but not IPv4 AF | Wrong address family was applied | `show ospfv3 interface brief` | Add the IPv4 AF interface command |
| IPv4 route missing | IPv4 address missing on advertising interface | `show ip interface brief`, `show ospfv3 interface <interface-id>` | Configure the correct IPv4 address |
| IPv4 route missing | Interface is passive in IPv4 AF | `show ip protocols`, `show running-config | section router ospfv3` | Configure `no passive-interface <interface-id>` under `address-family ipv4 unicast` |
| IPv4 route missing | OSPFv3 IPv4 AF router ID missing or duplicated | `show ospfv3`, `show logging | include OSPFv3|router ID` | Configure unique `router-id <id>` under IPv4 AF |
| Neighbor stuck or flapping | Area mismatch | `show ospfv3 interface <interface-id>` | Put both ends of the link in the same area |
| Neighbor stuck or flapping | Timer mismatch | `show ospfv3 interface <interface-id> | include Hello|Dead` | Match hello and dead timers |
| Neighbor stuck in EXSTART or EXCHANGE | MTU mismatch | `show ospfv3 neighbor detail`, `show interface <interface-id>` | Match MTU or use the platform-supported OSPFv3 MTU ignore option only when required |
| Neighbor state shows DR/BDR unexpectedly | Ethernet default broadcast network type | `show ospfv3 interface <interface-id>` | Use `ospfv3 network point-to-point` only on true two-router transit links |
| `show ip route ospfv3` has no routes | LSDB has no usable IPv4 AF prefixes | `show ospfv3 database`, `show ospfv3 interface brief` | Verify IPv4 AF interface activation and prefix advertisement |
| `show ospfv3 database` shows IPv6 AF but not IPv4 AF | IPv4 address family was not created | `show running-config | section router ospfv3` | Configure `address-family ipv4 unicast` |
| Default route missing from IPv4 AF | `default-information originate` configured under wrong AF or missing | `show running-config | section router ospfv3`, `show ip route ospfv3` | Configure it under `address-family ipv4 unicast` |
| IPv6 routes appear but IPv4 routes do not | IPv6 AF is working but IPv4 AF is not enabled or not addressed | `show ospfv3 interface brief`, `show ip interface brief` | Enable IPv4 AF on interfaces and verify IPv4 addressing |
| IPv4 AF and IPv6 AF have different behavior | AF-specific configuration overrides parent OSPFv3 configuration | `show running-config | section router ospfv3` | Check settings under the IPv4 AF specifically |
| Ping to remote IPv4 prefix fails but route exists | Return path or data-plane filtering issue | `show ip route <source-prefix>` on return-side routers | Verify reverse route, ACLs, and interface state |
| Link-local address changes after reload or interface replacement | Link-local was auto-generated from interface identifier | `show ipv6 interface <interface-id>` | Configure deterministic `ipv6 address FE80::<id> link-local` if stable diagnostics are needed |
##### Source_Basis
# OSPFv3_Address_Families_IPv4_Mental_Model
# OSPFv3_Address_Families_IPv4_Configuration_Checklist
# OSPFv3_Address_Families_IPv4_Skeleton
# OSPFv3_Address_Families_IPv4_Verification_Commands
# OSPFv3_Address_Families_IPv4_Rollback
# OSPFv3_Address_Families_IPv4_Failure_Checks
# index of each title throughout note, not in table format

