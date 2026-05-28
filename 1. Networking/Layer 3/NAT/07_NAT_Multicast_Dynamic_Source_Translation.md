Cisco’s IP Multicast Dynamic NAT chapter says this feature translates multicast packet source addresses, creates a one-to-one mapping from inside local source to outside global pool address, and does not support multicast destination NAT, PAT overload for multicast, source-and-destination translation, or unicast-to-multicast translation. Its documented summary flow is NAT pool, ACL, ip nat inside source list ... pool ..., multicast routing, PIM sparse mode, and classic inside/outside NAT interface roles.  

NAT_Multicast_Dynamic_Source_Translation.md

NAT_Multicast_Dynamic_Source_Translation

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `iosxe_combined_pdfs_.md` NAT guide reference | Points IOS XE NAT configuration content to the IOS XE combined source |
| `iosxe_combined_pdfs_.md` | `NAT Configuration Guide` > NAT address conservation | Supports NAT pool, ACL, inside source dynamic NAT, and inside/outside NAT role syntax |
| Cisco IOS XE 17.x IP Addressing Configuration Guide | `IP Multicast Dynamic NAT` | Supports multicast source NAT, one-to-one dynamic source mapping, multicast NAT restrictions, and configuration steps |
| Cisco IOS XE 17.x IP Addressing Configuration Guide | `Configuring IP Multicast Dynamic NAT` | Provides `ip nat pool`, `access-list`, `ip nat inside source list ... pool`, `ip multicast-routing distributed`, `ip pim sparse-mode`, `ip nat inside`, and `ip nat outside` sequence |
| Related lab | `nat-multicast-final` | Lab target for multicast dynamic source NAT |
# NAT_Multicast_Dynamic_Source_Translation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Multicast source NAT | The router translates the multicast packet source address before forwarding multicast traffic across the outside domain. |
| Not multicast group NAT | This feature does not translate the multicast destination group address. The group remains the group. |
| One-to-one dynamic mapping | An inside local multicast source is mapped to one outside global address from the NAT pool. |
| No multicast PAT overload | Multicast dynamic NAT does not use port overload. Do not design this like normal inside-to-outside internet PAT. |
| NAT pool role | The pool provides outside global source addresses used to represent inside multicast sources. |
| ACL role | The NAT ACL selects inside source addresses eligible for translation. |
| PIM dependency | Multicast routing must work. PIM must be enabled on the relevant interfaces so multicast state can form. |
| NAT domain dependency | This is classic NAT. The source side uses `ip nat inside`; the outside multicast domain uses `ip nat outside`. |
| RPF still matters | Multicast forwarding still depends on RPF checks. NAT does not remove multicast routing requirements. |
| MFIB caveat | Multicast ACL logic may need to permit both pre-NAT and post-NAT source addresses so multicast forwarding state can form correctly. |
| Verification principle | Verify NAT and multicast separately: NAT table proves source translation, while multicast state proves forwarding. |
# NAT_Multicast_Dynamic_Source_Translation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are up | NAT-RTR | `show ip interface brief` | Inside and outside routed interfaces show up/up |
| 2 | Confirm current NAT state before changes | NAT-RTR | `show running-config | include ip nat` | Existing NAT rules and interface roles are visible |
| 3 | Confirm multicast routing state before changes | NAT-RTR | `show running-config | include ip multicast-routing|ip pim` | Existing multicast routing and PIM configuration is visible |
| 4 | Confirm route toward the inside multicast source | NAT-RTR | `show ip route <INSIDE_MULTICAST_SOURCE_IP>` | Router has a valid route toward the inside source |
| 5 | Confirm route toward the RP if sparse mode is used | NAT-RTR | `show ip route <RP_IP>` | Router has a valid route toward the RP |
| 6 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters global configuration mode |
| 7 | Create the outside global NAT pool | NAT-RTR | `ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>` | Pool exists for one-to-one multicast source translations |
| 8 | Create the NAT ACL for inside multicast sources | NAT-RTR | `access-list <NAT_ACL_ID> permit <INSIDE_SOURCE_SUBNET> <WILDCARD_MASK> any` | ACL matches inside source addresses that should be translated |
| 9 | Bind the NAT ACL to the NAT pool | NAT-RTR | `ip nat inside source list <NAT_ACL_ID> pool <POOL_NAME>` | Dynamic source NAT rule exists without overload |
| 10 | Enable multicast routing | NAT-RTR | `ip multicast-routing distributed` | Multicast routing is enabled with distributed switching support |
| 11 | Use non-distributed multicast routing if the image requires it | NAT-RTR | `ip multicast-routing` | Multicast routing is enabled on images that do not support the distributed keyword |
| 12 | Enter the inside source-facing interface | NAT-RTR | `interface <INSIDE_INTERFACE>` | Device enters interface configuration mode |
| 13 | Configure inside interface IP addressing if not already configured | NAT-RTR | `ip address <INSIDE_INTERFACE_IP> <INSIDE_MASK>` | Interface has the correct inside-side IP address |
| 14 | Enable PIM on the inside interface | NAT-RTR | `ip pim sparse-mode` | PIM is active on the source-facing interface |
| 15 | Mark the source-facing interface as NAT inside | NAT-RTR | `ip nat inside` | Interface is classified as the NAT inside domain |
| 16 | Return to global configuration mode | NAT-RTR | `exit` | Device returns to global configuration mode |
| 17 | Enter the outside multicast-facing interface | NAT-RTR | `interface <OUTSIDE_INTERFACE>` | Device enters outside interface configuration mode |
| 18 | Configure outside interface IP addressing if not already configured | NAT-RTR | `ip address <OUTSIDE_INTERFACE_IP> <OUTSIDE_MASK>` | Interface has the correct outside-side IP address |
| 19 | Enable PIM on the outside interface | NAT-RTR | `ip pim sparse-mode` | PIM is active on the outside multicast-facing interface |
| 20 | Mark the multicast-facing interface as NAT outside | NAT-RTR | `ip nat outside` | Interface is classified as the NAT outside domain |
| 21 | Return to global configuration mode | NAT-RTR | `exit` | Device returns to global configuration mode |
| 22 | Configure RP information if sparse mode requires manual RP | NAT-RTR | `ip pim rp-address <RP_IP>` | Router knows the RP for sparse-mode multicast groups |
| 23 | Optional: permit both original and translated sources in multicast filtering ACLs | NAT-RTR | `permit ip host <INSIDE_SOURCE_IP> any` and `permit ip host <TRANSLATED_SOURCE_IP> any` | Multicast filters do not block pre-NAT or post-NAT source state |
| 24 | Exit configuration mode | NAT-RTR | `end` | Device returns to privileged EXEC mode |
| 25 | Verify NAT pool and NAT rule | NAT-RTR | `show running-config | include ip nat pool|ip nat inside source` | NAT pool and dynamic source rule are present |
| 26 | Verify interface NAT and PIM roles | NAT-RTR | `show running-config | section interface` | Inside and outside interfaces show both PIM and NAT role commands |
| 27 | Verify PIM neighbors | NAT-RTR | `show ip pim neighbor` | Expected PIM neighbors are present |
| 28 | Verify receiver joins | Receiver-RTR | `show ip igmp groups` | Receiver interface shows membership for the multicast group |
| 29 | Generate multicast source traffic | Source Host | `ping <MULTICAST_GROUP_IP>` | Source sends multicast packets toward the group |
| 30 | Verify multicast route state | NAT-RTR | `show ip mroute <MULTICAST_GROUP_IP>` | Multicast route entry forms for the source and group |
| 31 | Verify MFIB state if supported | NAT-RTR | `show ip mfib <MULTICAST_GROUP_IP>` | Forwarding entry exists for multicast traffic |
| 32 | Verify NAT translation table | NAT-RTR | `show ip nat translations` | Inside local source maps to outside global pool address |
| 33 | Verify detailed NAT state | NAT-RTR | `show ip nat translations verbose` | Translation details show the expected inside local and inside global source mapping |
| 34 | Verify multicast counters | NAT-RTR | `show ip mroute count` | Packet counters increment for the multicast stream |
| 35 | Save the working configuration | NAT-RTR | `write memory` | Multicast dynamic NAT configuration survives reload |
# NAT_Multicast_Dynamic_Source_Translation_Skeleton
conf t
!
! Pool used for translated multicast source addresses.
ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
!
! ACL selects inside multicast source addresses.
access-list <NAT_ACL_ID> permit <INSIDE_SOURCE_SUBNET> <WILDCARD_MASK> any
!
! Dynamic source NAT. Do not use overload for multicast dynamic NAT.
ip nat inside source list <NAT_ACL_ID> pool <POOL_NAME>
!
! Cisco IOS XE documented multicast NAT flow uses this form.
ip multicast-routing distributed
!
! If the image does not support the distributed keyword, use:
! ip multicast-routing
!
interface <INSIDE_INTERFACE>
 description INSIDE_MULTICAST_SOURCE_SIDE
 ip address <INSIDE_INTERFACE_IP> <INSIDE_MASK>
 ip pim sparse-mode
 ip nat inside
 no shutdown
!
interface <OUTSIDE_INTERFACE>
 description OUTSIDE_MULTICAST_DOMAIN_SIDE
 ip address <OUTSIDE_INTERFACE_IP> <OUTSIDE_MASK>
 ip pim sparse-mode
 ip nat outside
 no shutdown
!
! Required when using manual sparse-mode RP.
ip pim rp-address <RP_IP>
!
end
write memory
# NAT_Multicast_Dynamic_Source_Translation_Receiver_Skeleton
conf t
!
ip multicast-routing
!
interface <RECEIVER_FACING_INTERFACE>
 description RECEIVER_LAN
 ip address <RECEIVER_INTERFACE_IP> <RECEIVER_MASK>
 ip pim sparse-mode
 ip igmp join-group <MULTICAST_GROUP_IP>
 no shutdown
!
interface <UPSTREAM_INTERFACE>
 description TOWARD_NAT_RTR_OR_RP
 ip address <UPSTREAM_INTERFACE_IP> <UPSTREAM_MASK>
 ip pim sparse-mode
 no shutdown
!
ip pim rp-address <RP_IP>
!
end
write memory
# NAT_Multicast_Dynamic_Source_Translation_Verification_Commands
show ip interface brief
show running-config | include ip multicast-routing
show running-config | include ip nat
show running-config | include ip pim
show running-config | section interface
show access-lists
show access-lists <NAT_ACL_ID>
show ip route <INSIDE_MULTICAST_SOURCE_IP>
show ip route <RP_IP>
show ip pim neighbor
show ip pim rp mapping
show ip igmp groups
show ip mroute
show ip mroute <MULTICAST_GROUP_IP>
show ip mroute count
show ip mfib
show ip mfib <MULTICAST_GROUP_IP>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
ping <MULTICAST_GROUP_IP>
clear ip nat translation *
clear ip mroute *
show ip nat translations
show ip mroute <MULTICAST_GROUP_IP>
# NAT_Multicast_Dynamic_Source_Translation_Rollback
conf t
!
no ip nat inside source list <NAT_ACL_ID> pool <POOL_NAME>
no ip nat pool <POOL_NAME> <START_GLOBAL_IP> <END_GLOBAL_IP> netmask <GLOBAL_MASK>
no access-list <NAT_ACL_ID>
!
no ip pim rp-address <RP_IP>
!
interface <INSIDE_INTERFACE>
 no ip nat inside
 no ip pim sparse-mode
!
interface <OUTSIDE_INTERFACE>
 no ip nat outside
 no ip pim sparse-mode
!
no ip multicast-routing distributed
no ip multicast-routing
!
end
clear ip nat translation *
clear ip mroute *
write memory
# NAT_Multicast_Dynamic_Source_Translation_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No NAT translations appear | Multicast source traffic does not match the NAT ACL | `show access-lists <NAT_ACL_ID>` | Correct the ACL to match the inside source address |
| NAT command with overload was used | Multicast dynamic NAT does not support PAT overload | `show running-config | include ip nat inside source` | Remove `overload` and use one-to-one pool translation |
| Multicast group address is expected to change | Feature translates source address, not multicast destination group | `show ip mroute <MULTICAST_GROUP_IP>` | Use source NAT expectations only; do not expect group translation |
| PIM neighbors do not form | PIM is missing or routing between multicast routers is broken | `show ip pim neighbor` | Enable `ip pim sparse-mode` and fix unicast reachability |
| Receiver does not join the group | IGMP join is missing on receiver LAN | `show ip igmp groups` | Configure receiver application, test join, or `ip igmp join-group <GROUP>` in lab |
| `show ip mroute` has no `(S,G)` entry | RPF, PIM, RP, or multicast ACL state is wrong | `show ip mroute`, `show ip pim rp mapping`, `show ip route <SOURCE>` | Fix RPF route, RP config, or multicast ACL permits |
| MFIB does not contain expected entry | Multicast ACL permits only pre-NAT or only post-NAT address | `show ip mfib <GROUP>` | Permit both original and translated source addresses where multicast filtering is applied |
| NAT table forms but traffic is not delivered | Multicast forwarding path is broken after translation | `show ip mroute count` and `show ip pim neighbor` | Troubleshoot PIM, RPF, RP, and receiver joins separately from NAT |
| Source appears with wrong address to receivers | Wrong NAT pool was used | `show ip nat translations verbose` | Correct the pool address range and clear translations |
| Pool is exhausted | Not enough outside global addresses for inside multicast sources | `show ip nat statistics` | Increase pool size or reduce translated sources |
| Router rejects `ip multicast-routing distributed` | Image does not support the distributed keyword | `ip multicast-routing ?` | Use `ip multicast-routing` if supported by the platform |
| Existing state survives config changes | Old NAT or multicast state remains active | `show ip nat translations` and `show ip mroute` | Use `clear ip nat translation *` and `clear ip mroute *` during lab resets |
| Unicast NAT works but multicast NAT fails | Unicast NAT was validated, but multicast routing was never made functional | `show ip pim neighbor`, `show ip igmp groups`, `show ip mroute` | Build multicast routing first, then add multicast source NAT |
##### Source_Basis
# NAT_Multicast_Dynamic_Source_Translation_Mental_Model
# NAT_Multicast_Dynamic_Source_Translation_Configuration_Checklist
# NAT_Multicast_Dynamic_Source_Translation_Skeleton
# NAT_Multicast_Dynamic_Source_Translation_Receiver_Skeleton
# NAT_Multicast_Dynamic_Source_Translation_Verification_Commands
# NAT_Multicast_Dynamic_Source_Translation_Rollback
# NAT_Multicast_Dynamic_Source_Translation_Failure_Checks

