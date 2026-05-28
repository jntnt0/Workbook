
Local source checked first. _KB_INDEX.md points IOS XE NAT to iosxe_combined_pdfs_.md; the local source supports the NAT syntax pieces, but the VASI-specific inter-VRF service-path workflow is clearer in Cisco’s VASI NAT document. Cisco states that IOS XE does not support classic inter-VRF NAT like older IOS, and that IOS XE uses VASI pairs for inter-VRF NAT service insertion. Cisco’s VASI guide also shows vasileft1 / vasiright1 pairing, VRF-specific static routes through the VASI pair, and NAT configured on either the vasiright side or vasileft side depending on where the translation should occur.  

NAT_VASI_Inter_VRF_Service_Path_NAT.md

NAT_VASI_Inter_VRF_Service_Path_NAT

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `iosxe_combined_pdfs_.md` NAT guide reference | Points IOS XE NAT configuration content to the IOS XE combined source |
| `iosxe_combined_pdfs_.md` | `NAT Configuration Guide` > NAT address conservation | Supports NAT pool, NAT ACL, static NAT, dynamic NAT, overload, VRF keyword, and NAT verification syntax |
| Cisco IOS XE VASI NAT document | `Configure VRF-Aware Software Infrastructure NAT on Cisco IOS XE` | Supports VASI as the IOS XE method for inter-VRF NAT |
| Cisco IOS XE VASI NAT document | `VASI Interface Configuration` | Supports `vasileft` and `vasiright` paired virtual interfaces assigned to different VRFs |
| Cisco IOS XE VASI NAT document | `Scenario 1 - NAT on Vasiright` | Supports NAT after traffic crosses from `vasileft` into `vasiright` |
| Cisco IOS XE VASI NAT document | `Scenario 2 - NAT on Vasileft` | Supports NAT before traffic crosses from `vasileft` into `vasiright` |
| Cisco IOS XE VASI configuration guide | `Configuring the VRF-Aware Software Infrastructure` | Supports VASI as a service framework for NAT, ACLs, firewall, IPsec, IPv4, IPv6, and routing between VRFs |
| Related lab | `nat-vasi-final` | Lab target for VASI inter-VRF service-path NAT |
# NAT_VASI_Inter_VRF_Service_Path_NAT_Mental_Model
| Concept | Operational Meaning |
|---|---|
| VASI | VRF-Aware Software Infrastructure. It provides virtual paired interfaces used to steer traffic between VRFs through services such as NAT, firewall, ACLs, and IPsec. |
| VASI pair | `vasileft1` automatically pairs with `vasiright1`. Matching interface numbers form the pair. Traffic entering one side is internally handed to the other side. |
| Inter-VRF service path | VASI is not just route leaking. It forces traffic through a service insertion point where NAT can be applied between VRFs. |
| VRF_LEFT | The source-side VRF in the common lab model. It usually contains the original inside host or source network. |
| VRF_RIGHT | The destination-side or outside-service VRF in the common lab model. It usually contains the remote destination or WAN-facing network. |
| NAT on `vasiright` | Traffic crosses from `vasileft` into `vasiright` first, then NAT is applied in `VRF_RIGHT` between `vasiright` as inside and the physical WAN as outside. |
| NAT on `vasileft` | NAT is applied in `VRF_LEFT` before traffic crosses the VASI pair. The physical source interface is inside and `vasileft` is outside. |
| Static routes are mandatory | Each VRF needs routes that point traffic into the correct VASI side. Without those routes, traffic stays in the wrong VRF or never enters the service path. |
| NAT VRF keyword | The `vrf <VRF_NAME>` keyword binds the NAT rule to the VRF where NAT is being performed. |
| Do not NAT to the WAN interface IP | In the Cisco VASI NAT example, translating to the WAN interface IP can cause the router to treat return traffic as destined to itself instead of forwarding it back through the VASI path. |
| Do not mark both VASI sides outside | Cisco notes that configuring both interfaces in a VASI pair as NAT outside is not supported. |
| Verification split | Verify VRF routes, VASI pair state, NAT interface roles, and NAT translations separately. One good output does not prove the whole path works. |
# NAT_VASI_Inter_VRF_Service_Path_NAT_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm IOS XE device supports VASI interfaces | NAT-RTR | `configure terminal` then `interface vasileft ?` | CLI exposes VASI interface support |
| 2 | Confirm current VRF baseline | NAT-RTR | `show vrf` | Required VRFs are visible or confirmed missing before build |
| 3 | Confirm routed interfaces are up | NAT-RTR | `show ip interface brief` | Physical interfaces and existing routed paths show up/up |
| 4 | Confirm no conflicting NAT state exists | NAT-RTR | `show running-config | include ip nat` | Existing NAT rules and NAT roles are known before changes |
| 5 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters global configuration mode |
| 6 | Create the left/source VRF | NAT-RTR | `vrf definition <VRF_LEFT>` | Left/source VRF is created |
| 7 | Configure the left/source VRF RD | NAT-RTR | `rd <LEFT_RD>` | VRF has a route distinguisher |
| 8 | Enable IPv4 address family for the left/source VRF | NAT-RTR | `address-family ipv4` then `exit-address-family` | IPv4 routing context exists for the VRF |
| 9 | Create the right/destination VRF | NAT-RTR | `vrf definition <VRF_RIGHT>` | Right/destination VRF is created |
| 10 | Configure the right/destination VRF RD | NAT-RTR | `rd <RIGHT_RD>` | VRF has a route distinguisher |
| 11 | Enable IPv4 address family for the right/destination VRF | NAT-RTR | `address-family ipv4` then `exit-address-family` | IPv4 routing context exists for the VRF |
| 12 | Configure the source-side physical interface in the left VRF | NAT-RTR | `interface <LEFT_PHYSICAL_INTERFACE>` then `vrf forwarding <VRF_LEFT>` | Source-facing interface is assigned to the left VRF |
| 13 | Assign the source-side physical IP address | NAT-RTR | `ip address <LEFT_PHYSICAL_IP> <LEFT_PHYSICAL_MASK>` | Interface has the correct source-side IP address |
| 14 | Bring up the source-side physical interface | NAT-RTR | `no shutdown` | Interface is enabled |
| 15 | Configure the destination-side physical interface in the right VRF | NAT-RTR | `interface <RIGHT_PHYSICAL_INTERFACE>` then `vrf forwarding <VRF_RIGHT>` | Destination-facing interface is assigned to the right VRF |
| 16 | Assign the destination-side physical IP address | NAT-RTR | `ip address <RIGHT_PHYSICAL_IP> <RIGHT_PHYSICAL_MASK>` | Interface has the correct destination-side IP address |
| 17 | Bring up the destination-side physical interface | NAT-RTR | `no shutdown` | Interface is enabled |
| 18 | Configure the left VASI interface | NAT-RTR | `interface vasileft<VASI_ID>` | Device enters left VASI interface configuration mode |
| 19 | Place the left VASI interface in the left VRF | NAT-RTR | `vrf forwarding <VRF_LEFT>` | `vasileft<VASI_ID>` belongs to the left VRF |
| 20 | Address the left VASI interface | NAT-RTR | `ip address <VASI_LEFT_IP> <VASI_MASK>` | Left VASI side has the correct point-to-point IP address |
| 21 | Optional: disable keepalives on the left VASI side | NAT-RTR | `no keepalive` | VASI interface avoids keepalive dependency in lab builds |
| 22 | Configure the right VASI interface | NAT-RTR | `interface vasiright<VASI_ID>` | Device enters right VASI interface configuration mode |
| 23 | Place the right VASI interface in the right VRF | NAT-RTR | `vrf forwarding <VRF_RIGHT>` | `vasiright<VASI_ID>` belongs to the right VRF |
| 24 | Address the right VASI interface | NAT-RTR | `ip address <VASI_RIGHT_IP> <VASI_MASK>` | Right VASI side has the correct point-to-point IP address |
| 25 | Optional: disable keepalives on the right VASI side | NAT-RTR | `no keepalive` | VASI interface avoids keepalive dependency in lab builds |
| 26 | Configure left VRF route toward right-side destinations through VASI | NAT-RTR | `ip route vrf <VRF_LEFT> <RIGHT_DEST_SUBNET> <RIGHT_DEST_MASK> vasileft<VASI_ID> <VASI_RIGHT_IP>` | Left VRF sends right-side traffic into the VASI pair |
| 27 | Configure right VRF route back toward original source subnet through VASI | NAT-RTR | `ip route vrf <VRF_RIGHT> <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_MASK> vasiright<VASI_ID> <VASI_LEFT_IP>` | Right VRF can route return traffic toward the original source path |
| 28 | Choose NAT placement model | NAT-RTR | `show run interface vasileft<VASI_ID>` and `show run interface vasiright<VASI_ID>` | You know whether NAT will be applied on the `vasiright` side or the `vasileft` side |
| 29 | For NAT on `vasiright`, mark `vasiright` as NAT inside | NAT-RTR | `interface vasiright<VASI_ID>` then `ip nat inside` | Traffic entering right VRF from VASI is classified as NAT inside |
| 30 | For NAT on `vasiright`, mark the right physical interface as NAT outside | NAT-RTR | `interface <RIGHT_PHYSICAL_INTERFACE>` then `ip nat outside` | Right/WAN-facing interface is classified as NAT outside |
| 31 | For static NAT on `vasiright`, configure static source NAT in the right VRF | NAT-RTR | `ip nat inside source static <LEFT_SOURCE_HOST_IP> <NAT_GLOBAL_IP> vrf <VRF_RIGHT>` | Source host is statically translated in the right VRF |
| 32 | For dynamic NAT on `vasiright`, create traffic-matching ACL | NAT-RTR | `ip access-list extended <NAT_ACL>` then `permit ip <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_WILDCARD> <RIGHT_DEST_SUBNET> <RIGHT_DEST_WILDCARD>` | ACL selects the inter-VRF flow to translate |
| 33 | For dynamic NAT on `vasiright`, create NAT pool | NAT-RTR | `ip nat pool <NAT_POOL> <NAT_GLOBAL_START> <NAT_GLOBAL_END> prefix-length <PREFIX_LENGTH>` | NAT pool exists in the right VRF address space |
| 34 | For dynamic NAT on `vasiright`, bind ACL to pool with VRF keyword | NAT-RTR | `ip nat inside source list <NAT_ACL> pool <NAT_POOL> vrf <VRF_RIGHT> overload` | Matching traffic is translated using the right VRF NAT pool |
| 35 | For NAT on `vasileft`, mark left physical interface as NAT inside | NAT-RTR | `interface <LEFT_PHYSICAL_INTERFACE>` then `ip nat inside` | Source-facing physical interface is classified as NAT inside |
| 36 | For NAT on `vasileft`, mark `vasileft` as NAT outside | NAT-RTR | `interface vasileft<VASI_ID>` then `ip nat outside` | Traffic leaving left VRF toward the VASI pair is classified as NAT outside |
| 37 | For static NAT on `vasileft`, configure static source NAT in the left VRF | NAT-RTR | `ip nat inside source static <LEFT_SOURCE_HOST_IP> <NAT_GLOBAL_IP> vrf <VRF_LEFT>` | Source host is statically translated before crossing VASI |
| 38 | For dynamic NAT on `vasileft`, create traffic-matching ACL | NAT-RTR | `ip access-list extended <NAT_ACL>` then `permit ip <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_WILDCARD> <RIGHT_DEST_SUBNET> <RIGHT_DEST_WILDCARD>` | ACL selects the inter-VRF flow to translate |
| 39 | For dynamic NAT on `vasileft`, create NAT pool | NAT-RTR | `ip nat pool <NAT_POOL> <NAT_GLOBAL_START> <NAT_GLOBAL_END> prefix-length <PREFIX_LENGTH>` | NAT pool exists for translated source addresses |
| 40 | For dynamic NAT on `vasileft`, bind ACL to pool with VRF keyword | NAT-RTR | `ip nat inside source list <NAT_ACL> pool <NAT_POOL> vrf <VRF_LEFT> overload` | Matching traffic is translated in the left VRF before VASI handoff |
| 41 | For NAT on `vasileft`, add right VRF route to translated address through VASI | NAT-RTR | `ip route vrf <VRF_RIGHT> <NAT_GLOBAL_IP_OR_SUBNET> <MASK> vasiright<VASI_ID> <VASI_LEFT_IP>` | Right VRF returns traffic toward translated source addresses through VASI |
| 42 | Confirm VASI interface addressing and VRF placement | NAT-RTR | `show running-config interface vasileft<VASI_ID>` and `show running-config interface vasiright<VASI_ID>` | Each VASI side is in the correct VRF with correct IP addressing |
| 43 | Confirm VRF routes point to the VASI pair | NAT-RTR | `show ip route vrf <VRF_LEFT>` and `show ip route vrf <VRF_RIGHT>` | Each VRF has routes through the correct VASI side |
| 44 | Confirm NAT is bound to the correct VRF | NAT-RTR | `show running-config | include ip nat inside source` | NAT rule includes the correct `vrf <VRF_NAME>` keyword |
| 45 | Confirm NAT interface roles are not both outside on the VASI pair | NAT-RTR | `show running-config interface vasileft<VASI_ID>` and `show running-config interface vasiright<VASI_ID>` | Design does not configure both VASI sides as `ip nat outside` |
| 46 | Generate traffic from left/source host to right/destination host | Source Host | `ping <RIGHT_DEST_IP>` | Inter-VRF traffic enters the VASI path and triggers translation |
| 47 | Verify VRF-specific NAT translations | NAT-RTR | `show ip nat translations vrf <NAT_VRF_NAME>` | Translation appears in the VRF where NAT is configured |
| 48 | Verify all NAT translations if VRF-specific output is unavailable | NAT-RTR | `show ip nat translations` | Expected inside local and inside global mapping appears |
| 49 | Verify route and translation together | NAT-RTR | `show ip route vrf <VRF_LEFT> <RIGHT_DEST_IP>` and `show ip route vrf <VRF_RIGHT> <NAT_GLOBAL_IP>` | Forward and return paths resolve through VASI correctly |
| 50 | Save the working configuration | NAT-RTR | `write memory` | VASI inter-VRF NAT configuration survives reload |
# NAT_VASI_Inter_VRF_Service_Path_NAT_Skeleton
conf t
!
vrf definition <VRF_LEFT>
 rd <LEFT_RD>
 !
 address-family ipv4
 exit-address-family
!
vrf definition <VRF_RIGHT>
 rd <RIGHT_RD>
 !
 address-family ipv4
 exit-address-family
!
interface <LEFT_PHYSICAL_INTERFACE>
 description SOURCE_SIDE_PHYSICAL_INTERFACE
 vrf forwarding <VRF_LEFT>
 ip address <LEFT_PHYSICAL_IP> <LEFT_PHYSICAL_MASK>
 no shutdown
!
interface <RIGHT_PHYSICAL_INTERFACE>
 description DESTINATION_OR_WAN_SIDE_PHYSICAL_INTERFACE
 vrf forwarding <VRF_RIGHT>
 ip address <RIGHT_PHYSICAL_IP> <RIGHT_PHYSICAL_MASK>
 no shutdown
!
interface vasileft<VASI_ID>
 description VASI_LEFT_IN_<VRF_LEFT>
 vrf forwarding <VRF_LEFT>
 ip address <VASI_LEFT_IP> <VASI_MASK>
 no keepalive
 no shutdown
!
interface vasiright<VASI_ID>
 description VASI_RIGHT_IN_<VRF_RIGHT>
 vrf forwarding <VRF_RIGHT>
 ip address <VASI_RIGHT_IP> <VASI_MASK>
 no keepalive
 no shutdown
!
! Route left VRF traffic toward right-side destinations through the left VASI side.
ip route vrf <VRF_LEFT> <RIGHT_DEST_SUBNET> <RIGHT_DEST_MASK> vasileft<VASI_ID> <VASI_RIGHT_IP>
!
! Route right VRF traffic back toward original left-side sources through the right VASI side.
ip route vrf <VRF_RIGHT> <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_MASK> vasiright<VASI_ID> <VASI_LEFT_IP>
!
! =========================================================
! Option A: NAT on vasiright side
! NAT occurs after traffic crosses from vasileft to vasiright.
! =========================================================
interface vasiright<VASI_ID>
 ip nat inside
!
interface <RIGHT_PHYSICAL_INTERFACE>
 ip nat outside
!
! Static NAT on right VRF
ip nat inside source static <LEFT_SOURCE_HOST_IP> <NAT_GLOBAL_IP> vrf <VRF_RIGHT>
!
! Dynamic PAT on right VRF
ip access-list extended <NAT_ACL>
 permit ip <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_WILDCARD> <RIGHT_DEST_SUBNET> <RIGHT_DEST_WILDCARD>
exit
ip nat pool <NAT_POOL> <NAT_GLOBAL_START> <NAT_GLOBAL_END> prefix-length <PREFIX_LENGTH>
ip nat inside source list <NAT_ACL> pool <NAT_POOL> vrf <VRF_RIGHT> overload
!
! =========================================================
! Option B: NAT on vasileft side
! NAT occurs before traffic crosses from vasileft to vasiright.
! Use this as an alternative, not stacked with Option A.
! =========================================================
! interface <LEFT_PHYSICAL_INTERFACE>
!  ip nat inside
! !
! interface vasileft<VASI_ID>
!  ip nat outside
! !
! ip nat inside source static <LEFT_SOURCE_HOST_IP> <NAT_GLOBAL_IP> vrf <VRF_LEFT>
! ip access-list extended <NAT_ACL>
!  permit ip <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_WILDCARD> <RIGHT_DEST_SUBNET> <RIGHT_DEST_WILDCARD>
! exit
! ip nat pool <NAT_POOL> <NAT_GLOBAL_START> <NAT_GLOBAL_END> prefix-length <PREFIX_LENGTH>
! ip nat inside source list <NAT_ACL> pool <NAT_POOL> vrf <VRF_LEFT> overload
! ip route vrf <VRF_RIGHT> <NAT_GLOBAL_IP_OR_SUBNET> <MASK> vasiright<VASI_ID> <VASI_LEFT_IP>
!
end
write memory
# NAT_VASI_Inter_VRF_Service_Path_NAT_Verification_Commands
show vrf
show ip interface brief
show ip interface brief vrf <VRF_LEFT>
show ip interface brief vrf <VRF_RIGHT>
show running-config interface <LEFT_PHYSICAL_INTERFACE>
show running-config interface <RIGHT_PHYSICAL_INTERFACE>
show running-config interface vasileft<VASI_ID>
show running-config interface vasiright<VASI_ID>
show running-config | include ip nat
show running-config | include ip route vrf
show ip route vrf <VRF_LEFT>
show ip route vrf <VRF_RIGHT>
show ip route vrf <VRF_LEFT> <RIGHT_DEST_IP>
show ip route vrf <VRF_RIGHT> <LEFT_SOURCE_IP>
show ip route vrf <VRF_RIGHT> <NAT_GLOBAL_IP>
show access-lists
show access-lists <NAT_ACL>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
show ip nat translations vrf <NAT_VRF_NAME>
ping vrf <VRF_LEFT> <RIGHT_DEST_IP> source <LEFT_SOURCE_IP>
ping vrf <VRF_RIGHT> <NAT_GLOBAL_IP>
traceroute vrf <VRF_LEFT> <RIGHT_DEST_IP> source <LEFT_SOURCE_IP>
clear ip nat translation *
show ip nat translations vrf <NAT_VRF_NAME>
# NAT_VASI_Inter_VRF_Service_Path_NAT_Rollback
conf t
!
no ip nat inside source list <NAT_ACL> pool <NAT_POOL> vrf <VRF_RIGHT> overload
no ip nat inside source list <NAT_ACL> pool <NAT_POOL> vrf <VRF_LEFT> overload
no ip nat inside source static <LEFT_SOURCE_HOST_IP> <NAT_GLOBAL_IP> vrf <VRF_RIGHT>
no ip nat inside source static <LEFT_SOURCE_HOST_IP> <NAT_GLOBAL_IP> vrf <VRF_LEFT>
no ip nat pool <NAT_POOL> <NAT_GLOBAL_START> <NAT_GLOBAL_END> prefix-length <PREFIX_LENGTH>
no ip access-list extended <NAT_ACL>
!
no ip route vrf <VRF_LEFT> <RIGHT_DEST_SUBNET> <RIGHT_DEST_MASK> vasileft<VASI_ID> <VASI_RIGHT_IP>
no ip route vrf <VRF_RIGHT> <LEFT_SOURCE_SUBNET> <LEFT_SOURCE_MASK> vasiright<VASI_ID> <VASI_LEFT_IP>
no ip route vrf <VRF_RIGHT> <NAT_GLOBAL_IP_OR_SUBNET> <MASK> vasiright<VASI_ID> <VASI_LEFT_IP>
!
interface <LEFT_PHYSICAL_INTERFACE>
 no ip nat inside
 no ip nat outside
!
interface <RIGHT_PHYSICAL_INTERFACE>
 no ip nat inside
 no ip nat outside
!
interface vasileft<VASI_ID>
 no ip nat inside
 no ip nat outside
!
interface vasiright<VASI_ID>
 no ip nat inside
 no ip nat outside
!
no interface vasileft<VASI_ID>
no interface vasiright<VASI_ID>
!
no vrf definition <VRF_LEFT>
no vrf definition <VRF_RIGHT>
!
end
clear ip nat translation *
write memory
# NAT_VASI_Inter_VRF_Service_Path_NAT_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Traffic never reaches NAT | VRF route does not point into the VASI pair | `show ip route vrf <VRF_LEFT> <RIGHT_DEST_IP>` | Add or correct the route pointing to `vasileft<VASI_ID>` |
| Return traffic fails | Right VRF route points to the wrong return path | `show ip route vrf <VRF_RIGHT> <LEFT_SOURCE_IP>` or `show ip route vrf <VRF_RIGHT> <NAT_GLOBAL_IP>` | Add or correct the return route through `vasiright<VASI_ID>` |
| Static NAT exists but no sessions pass | NAT is configured under the wrong VRF | `show running-config | include ip nat inside source` | Use the correct `vrf <VRF_NAME>` keyword for the side where NAT is applied |
| `show ip nat translations vrf <VRF>` is empty | Testing the wrong NAT VRF or traffic does not match NAT | `show ip nat translations` and `show access-lists <NAT_ACL>` | Check the VRF where NAT is applied and fix the ACL match |
| Router treats NAT global as local to itself | NAT global address was set to the WAN interface IP | `show running-config | include ip nat inside source` | Use a routed pool or separate global address, not the WAN interface IP |
| Both VASI sides are marked outside | Unsupported VASI NAT role placement | `show running-config interface vasileft<VASI_ID>` and `show running-config interface vasiright<VASI_ID>` | Do not configure both VASI interfaces in a pair as `ip nat outside` |
| NAT on `vasiright` fails | `vasiright` is not `ip nat inside` or right physical interface is not `ip nat outside` | `show running-config interface vasiright<VASI_ID>` and `show running-config interface <RIGHT_PHYSICAL_INTERFACE>` | Put `ip nat inside` on `vasiright` and `ip nat outside` on the right physical interface |
| NAT on `vasileft` fails | Left physical interface and `vasileft` NAT roles are wrong | `show running-config interface <LEFT_PHYSICAL_INTERFACE>` and `show running-config interface vasileft<VASI_ID>` | Put `ip nat inside` on the left physical interface and `ip nat outside` on `vasileft` |
| VASI pair does not pass traffic | VASI interface numbers do not match | `show running-config interface vasileft<VASI_ID>` and `show running-config interface vasiright<VASI_ID>` | Use matching numbers, such as `vasileft1` with `vasiright1` |
| VRF ping fails before NAT testing | Baseline VRF routing or interface placement is broken | `show vrf`, `show ip interface brief vrf <VRF>`, `show ip route vrf <VRF>` | Fix VRF assignment, IP addressing, and static routes before NAT |
| Dynamic NAT ACL has zero hits | ACL does not match the original source and destination flow | `show access-lists <NAT_ACL>` | Correct source subnet, destination subnet, and wildcard masks |
| Pool NAT fails after first sessions | Pool is too small or overload is missing | `show ip nat statistics` | Add `overload` or increase the NAT pool |
| NAT output appears globally but not under VRF command | Platform output varies or command was run against the wrong VRF | `show ip nat translations` and `show ip nat translations vrf <VRF>` | Validate with both generic and VRF-specific NAT translation commands |
| Old behavior remains after changes | Stale NAT translation state persists | `show ip nat translations verbose` | Clear translations with `clear ip nat translation *` during lab resets |
##### Source_Basis
# NAT_VASI_Inter_VRF_Service_Path_NAT_Mental_Model
# NAT_VASI_Inter_VRF_Service_Path_NAT_Configuration_Checklist
# NAT_VASI_Inter_VRF_Service_Path_NAT_Skeleton
# NAT_VASI_Inter_VRF_Service_Path_NAT_Verification_Commands
# NAT_VASI_Inter_VRF_Service_Path_NAT_Rollback
# NAT_VASI_Inter_VRF_Service_Path_NAT_Failure_Checks
