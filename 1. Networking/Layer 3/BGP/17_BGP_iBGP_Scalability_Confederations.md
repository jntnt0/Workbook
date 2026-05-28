


BGP_iBGP_Scalability_Confederations.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP attributes, route reflectors, confederations, communities, route maps, prefix lists, AS path filters, iBGP scalability, MP-BGP, and IPv6 BGP |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23441-23445 | Confederation overview | Supports confederations as an alternative to iBGP full mesh, using member ASNs inside a larger confederation AS |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23451-23459 | Confederation configuration steps | Supports configuring BGP under member AS, setting `bgp confederation identifier`, configuring `bgp confederation peers`, and then normal BGP neighbors |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23463-23515 | Confederation CLI examples | Supports `router bgp <member-asn>`, `bgp confederation identifier <confed-asn>`, and `bgp confederation peers <member-asn>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23527-23539 | Confederation behavior differences | Supports AS_CONFED_SEQUENCE, MED behavior inside confederation, LOCAL_PREF behavior inside confederation, next-hop behavior, and stripping confederation ASNs externally |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23542-23608 | Confederation verification output | Supports checking AS path parentheses, `confed-internal`, and `confed-external` path details |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24349-24355 | Confederation command reference | Supports `bgp confederation identifier <as-number>` and `bgp confederation peers <member-asn>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25928-25930 | `no-export` in confederations | Supports the rule that `no-export` can still pass between sub-ASs inside a confederation but not outside the confederation |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 25988-25990 | `local-AS` community in confederations | Supports the rule that `local-AS` limits advertisement to the local member AS/sub-AS |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 27382-27382 | AS_CONFED_SEQUENCE path length behavior | Supports the rule that confederation AS path is not counted like normal AS path length |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 27778-27782 | Best-path preference order | Supports eBGP, confederation member AS peers, and iBGP comparison behavior |
| `iosxe_combined_pdfs_.md` lines 33305-33321 | Routing domain confederations | Supports IOS XE confederation feature coverage |
| `iosxe_combined_pdfs_.md` lines 34009-34025 | IOS XE confederation syntax | Supports `bgp confederation identifier <autonomous-system>` and `bgp confederation peers <member-asn...>` |
# BGP_iBGP_Scalability_Confederations_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP confederation | Splits one large BGP AS into smaller internal member ASs while appearing externally as one AS |
| Confederation identifier | The public or external-facing AS number that outside peers see |
| Member AS | Internal sub-AS used inside the confederation |
| Private member ASNs | Member ASs are usually private ASNs, commonly 64512 through 65534 in classic examples |
| External peer view | Routers outside the confederation peer with the confederation identifier, not the internal member AS |
| Internal member-AS peer | Router in one member AS peers with router in another member AS using normal neighbor syntax plus `bgp confederation peers` |
| Confederation peer | A peer in another member AS inside the same confederation |
| Same member AS iBGP | Routers inside the same member AS use normal iBGP behavior |
| Inter-member peering | Peering between member ASs behaves like a special eBGP relationship inside the confederation |
| iBGP scale problem | Confederations reduce the need for one massive full mesh by dividing iBGP into smaller member AS domains |
| Confederation AS path | Internal member AS path is shown as AS_CONFED_SEQUENCE, usually inside parentheses in the AS path |
| AS_CONFED_SEQUENCE | Internal confederation AS path segment used for loop prevention inside the confederation |
| AS_CONFED_SEQUENCE stripping | Internal member ASNs are removed when the route leaves the confederation |
| AS path length behavior | AS_CONFED_SEQUENCE is not counted the same as normal AS path length for shortest-AS-path selection |
| LOCAL_PREF inside confederation | Local preference can pass between member ASs inside the confederation but does not leave the confederation |
| MED inside confederation | MED can pass between member ASs inside the confederation but does not leave the confederation |
| Next-hop behavior | Next hop for routes exchanged between member ASs is not automatically rewritten like normal external eBGP in every case |
| `confed-internal` | Route learned from a peer in the same member AS or internal confederation relationship, depending output context |
| `confed-external` | Route learned from another member AS inside the confederation |
| Route reflector compatibility | Route reflectors can still be used inside a member AS |
| `no-export` community | Can still advertise between member ASs inside the confederation, but not outside the confederation |
| `local-AS` community | Keeps the route inside the local member AS/sub-AS and blocks advertisement to other member ASs |
| Blunt rule | Confederations are not basic iBGP. They are an AS-scaling design. Get the identifier, member ASs, confederation peers, next hop, and external AS appearance correct before touching policy |
# BGP_iBGP_Scalability_Confederations_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify external confederation AS | Notes | `<confed-asn>` | The AS number seen by external peers is known |
| 2 | Identify member ASs | Notes | `<member-asn-1>, <member-asn-2>, <member-asn-3>` | Internal sub-AS layout is known |
| 3 | Identify routers in each member AS | Notes | `<router-to-member-as-map>` | Every router belongs to the correct member AS |
| 4 | Identify inter-member peerings | Notes | `<member-as-border-links>` | Routers that peer between member ASs are known |
| 5 | Identify external peerings | Notes | `<external-peer-list>` | Routers peering outside the confederation are known |
| 6 | Identify same-member iBGP peerings | Notes | `<same-member-ibgp-peers>` | Normal iBGP sessions inside each member AS are known |
| 7 | Confirm loopback plan | All BGP Routers | `<loopback-ip-list>` | Stable BGP source addresses are known |
| 8 | Confirm IGP reachability inside each member AS | Member Routers | `show ip route <peer-loopback-ip>` | Same-member iBGP loopbacks are reachable |
| 9 | Confirm inter-member link reachability | Border Routers | `ping <inter-member-peer-ip> source <local-source-ip>` | Member AS border peers can reach each other |
| 10 | Confirm external peer reachability | Edge Router | `ping <external-peer-ip> source <local-source-ip>` | External BGP peer is reachable |
| 11 | Confirm TCP/179 path | Routers / Path | `show access-lists` or path ACL check | TCP/179 is not blocked |
| 12 | Confirm existing BGP config | Routers | `show running-config | section router bgp` | Existing BGP config is visible |
| 13 | Confirm current BGP neighbor state | Routers | `show bgp ipv4 unicast summary` | Baseline neighbor state is known |
| 14 | Confirm current route visibility | Routers | `show bgp ipv4 unicast` | Baseline BGP table is known |
| 15 | Configure router in member AS | Router | `configure terminal` then `router bgp <member-asn>` | Router runs BGP using its internal member AS |
| 16 | Configure confederation identifier | Router | `bgp confederation identifier <confed-asn>` | Router knows the external-facing confederation AS |
| 17 | Configure confederation peer ASs on border routers | Border Router | `bgp confederation peers <other-member-asn>` | Router treats listed member ASs as confederation peers |
| 18 | Configure multiple confederation peer ASs if needed | Border Router | `bgp confederation peers <member-asn-1> <member-asn-2>` | Router recognizes all directly peered member ASs |
| 19 | Set stable router ID | Router | `bgp router-id <router-id>` | BGP router ID is deterministic |
| 20 | Enable BGP neighbor logging | Router | `bgp log-neighbor-changes` | Neighbor state changes are logged |
| 21 | Standardize explicit AF activation | Router | `no bgp default ipv4-unicast` | IPv4 unicast neighbors must be activated under AFI |
| 22 | Configure same-member iBGP neighbor | Router | `neighbor <same-member-peer-ip> remote-as <local-member-asn>` | Neighbor is configured as iBGP inside same member AS |
| 23 | Configure same-member update source if loopbacks are used | Router | `neighbor <same-member-peer-ip> update-source Loopback0` | iBGP packets source from loopback |
| 24 | Configure inter-member confederation neighbor | Border Router | `neighbor <confed-peer-ip> remote-as <other-member-asn>` | Neighbor is configured to another member AS inside the confederation |
| 25 | Configure external eBGP neighbor from confederation edge | Edge Router | `neighbor <external-peer-ip> remote-as <external-asn>` | Edge router peers with outside AS |
| 26 | Configure external peer to use confederation identifier | External Router | `neighbor <confed-edge-ip> remote-as <confed-asn>` | Outside peer sees the whole confederation as one AS |
| 27 | Configure neighbor descriptions | Routers | `neighbor <neighbor-ip> description <description>` | Peer purpose is documented |
| 28 | Configure eBGP multihop only if needed | Routers | `neighbor <neighbor-ip> ebgp-multihop <ttl>` | Non-direct external or inter-member session can form if design requires it |
| 29 | Enter IPv4 unicast AFI | Routers | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 30 | Activate same-member iBGP neighbor | Router | `neighbor <same-member-peer-ip> activate` | Same-member iBGP peer exchanges IPv4 unicast routes |
| 31 | Activate inter-member confederation neighbor | Border Router | `neighbor <confed-peer-ip> activate` | Confederation member-AS peer exchanges IPv4 unicast routes |
| 32 | Activate external neighbor | Edge Router | `neighbor <external-peer-ip> activate` | External peer exchanges IPv4 unicast routes |
| 33 | Configure route reflector client inside a member AS only if needed | RR Router | `neighbor <client-ip> route-reflector-client` | Route reflection is used inside a member AS where required |
| 34 | Originate test prefix if needed | Router | `network <prefix> mask <mask>` | Test prefix is injected into BGP if exact RIB route exists |
| 35 | Verify exact route for network statement | Router | `show ip route <prefix> <mask>` | Exact route exists before BGP origination |
| 36 | Configure next-hop-self only where design requires it | Router | `neighbor <neighbor-ip> next-hop-self` | Peer receives reachable next hop |
| 37 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 38 | Save configuration | Router | `copy running-config startup-config` | Confederation config is saved |
| 39 | Verify BGP process AS | Router | `show running-config | section router bgp` | Router runs under member AS, not confederation identifier |
| 40 | Verify confederation identifier | Router | `show running-config | include bgp confederation identifier` | Correct external-facing confederation AS is configured |
| 41 | Verify confederation peers | Border Router | `show running-config | include bgp confederation peers` | Correct member ASs are listed |
| 42 | Verify same-member iBGP session | Router | `show bgp ipv4 unicast summary` | Same-member iBGP neighbor is established |
| 43 | Verify inter-member confederation session | Border Router | `show bgp ipv4 unicast summary` | Neighbor in other member AS is established |
| 44 | Verify external session | Edge Router | `show bgp ipv4 unicast summary` | External peer is established |
| 45 | Verify external peer sees confederation AS | External Router | `show bgp ipv4 unicast summary` | Neighbor AS appears as `<confed-asn>` |
| 46 | Verify route learned across member ASs | Router | `show bgp ipv4 unicast <prefix>` | Route appears through confederation path |
| 47 | Verify AS_CONFED_SEQUENCE | Router | `show bgp ipv4 unicast <prefix>` | Member ASNs appear in parentheses or confed path notation |
| 48 | Verify confed-internal/confed-external marker | Router | `show bgp ipv4 unicast <prefix>` | Output identifies confederation route type if platform displays it |
| 49 | Verify confed ASNs are stripped externally | External Router | `show bgp ipv4 unicast <prefix>` | External peer sees only normal AS path with confederation AS, not internal member AS sequence |
| 50 | Verify local preference behavior inside confederation | Member Router | `show bgp ipv4 unicast <prefix>` | Local preference is visible across member ASs where expected |
| 51 | Verify MED behavior inside confederation | Member Router | `show bgp ipv4 unicast <prefix>` | MED remains visible inside confederation where expected |
| 52 | Verify next-hop reachability | Router | `show ip route <bgp-next-hop-ip>` | BGP next hop is reachable |
| 53 | Verify route install | Router | `show ip route <prefix>` | BGP route installs if selected and valid |
| 54 | Verify forwarding path | Router | `show ip cef <prefix>` | FIB points toward expected next hop/interface |
| 55 | Verify advertised routes to external peer | Edge Router | `show bgp ipv4 unicast neighbors <external-peer-ip> advertised-routes` | External peer receives routes with internal member ASs hidden |
| 56 | Verify advertised routes to confederation peer | Border Router | `show bgp ipv4 unicast neighbors <confed-peer-ip> advertised-routes` | Member AS peer receives expected routes |
| 57 | Verify no route leak outside confederation | External Router | `show bgp ipv4 unicast` | Only intended prefixes are visible externally |
| 58 | Verify logs | Router | `show logging | include BGP|ADJCHANGE` | No unexpected session reset or AS mismatch appears |
# BGP_iBGP_Scalability_Confederations_Member_AS_Basic_Skeleton
configure terminal
router bgp <local-member-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 bgp confederation identifier <confed-asn>
 no bgp default ipv4-unicast
 neighbor <same-member-peer-ip> remote-as <local-member-asn>
 neighbor <same-member-peer-ip> update-source Loopback0
 neighbor <same-member-peer-ip> description Same-member-iBGP
 address-family ipv4 unicast
  neighbor <same-member-peer-ip> activate
  network <local-prefix> mask <local-mask>
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Confederations_Inter_Member_Peer_Skeleton
configure terminal
router bgp <local-member-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 bgp confederation identifier <confed-asn>
 bgp confederation peers <other-member-asn>
 no bgp default ipv4-unicast
 neighbor <confed-peer-ip> remote-as <other-member-asn>
 neighbor <confed-peer-ip> description Confederation-peer-to-member-AS-<other-member-asn>
 address-family ipv4 unicast
  neighbor <confed-peer-ip> activate
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Confederations_External_Edge_Skeleton
configure terminal
router bgp <local-member-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 bgp confederation identifier <confed-asn>
 bgp confederation peers <other-member-asn>
 no bgp default ipv4-unicast
 neighbor <external-peer-ip> remote-as <external-asn>
 neighbor <external-peer-ip> description External-AS-<external-asn>
 address-family ipv4 unicast
  neighbor <external-peer-ip> activate
  network <local-prefix> mask <local-mask>
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Confederations_External_Peer_View_Skeleton
configure terminal
router bgp <external-asn>
 bgp router-id <external-router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <confed-edge-ip> remote-as <confed-asn>
 neighbor <confed-edge-ip> description Confederation-AS-<confed-asn>
 address-family ipv4 unicast
  neighbor <confed-edge-ip> activate
  network <external-prefix> mask <external-mask>
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Confederations_Route_Reflector_Inside_Member_AS_Skeleton
configure terminal
router bgp <local-member-asn>
 bgp router-id <rr-router-id>
 bgp log-neighbor-changes
 bgp confederation identifier <confed-asn>
 no bgp default ipv4-unicast
 neighbor <client-ip> remote-as <local-member-asn>
 neighbor <client-ip> update-source Loopback0
 neighbor <client-ip> description RR-client-inside-member-AS
 address-family ipv4 unicast
  neighbor <client-ip> activate
  neighbor <client-ip> route-reflector-client
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Confederations_Next_Hop_Self_Skeleton
configure terminal
router bgp <local-member-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> next-hop-self
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
# BGP_iBGP_Scalability_Confederations_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show ip route <prefix>
show ip route <bgp-next-hop-ip>
show ip cef <prefix>
show running-config | section router bgp
show running-config | include bgp confederation
show logging | include BGP|ADJCHANGE
# BGP_iBGP_Scalability_Confederations_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP sessions | `show bgp ipv4 unicast summary` | Same-member, inter-member, and external peers are established |
| Verify local BGP process ASN | `show running-config | section router bgp` | Router uses member AS under `router bgp` |
| Verify confederation identifier | `show running-config | include bgp confederation identifier` | Confederation external AS is configured |
| Verify confederation peers | `show running-config | include bgp confederation peers` | Other directly peered member ASs are listed |
| Verify same-member iBGP route | `show bgp ipv4 unicast <prefix>` | Route learned within same member AS appears as expected |
| Verify inter-member confederation route | `show bgp ipv4 unicast <prefix>` | Route learned from another member AS appears |
| Verify AS_CONFED_SEQUENCE | `show bgp ipv4 unicast <prefix>` | Internal member ASNs appear in parentheses or confed path notation |
| Verify confed-internal marker | `show bgp ipv4 unicast <prefix>` | Output shows `confed-internal` where platform displays it |
| Verify confed-external marker | `show bgp ipv4 unicast <prefix>` | Output shows `confed-external` where platform displays it |
| Verify external peer AS view | External peer `show bgp ipv4 unicast summary` | External peer sees neighbor AS as the confederation identifier |
| Verify confed ASNs are hidden externally | External peer `show bgp ipv4 unicast <prefix>` | Internal member ASNs are not visible outside the confederation |
| Verify local preference inside confederation | `show bgp ipv4 unicast <prefix>` | LOCAL_PREF is visible across member ASs where expected |
| Verify MED inside confederation | `show bgp ipv4 unicast <prefix>` | MED is visible across member ASs where expected |
| Verify next-hop reachability | `show ip route <bgp-next-hop-ip>` | BGP next hop is reachable |
| Verify route install | `show ip route <prefix>` | BGP route installs if selected and valid |
| Verify forwarding entry | `show ip cef <prefix>` | CEF points to expected next hop/interface |
| Verify advertised routes to member AS peer | `show bgp ipv4 unicast neighbors <confed-peer-ip> advertised-routes` | Expected routes are advertised to confederation peer |
| Verify advertised routes to external peer | `show bgp ipv4 unicast neighbors <external-peer-ip> advertised-routes` | Routes advertised externally hide internal member AS details |
| Verify route reflector inside member AS | `show running-config | section router bgp` | `route-reflector-client` appears only where used inside a member AS |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No AS mismatch or unexpected reset appears |
# BGP_iBGP_Scalability_Confederations_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current confederation config | Router | `show running-config | section router bgp` | Member AS, confed identifier, confed peers, and neighbors are visible |
| 2 | Identify active sessions | Router | `show bgp ipv4 unicast summary` | Same-member, inter-member, and external peers are visible |
| 3 | Identify confederation routes | Router | `show bgp ipv4 unicast <prefix>` | AS_CONFED_SEQUENCE and confed markers are visible |
| 4 | Confirm replacement design before removing confederation | Notes | `full mesh / route reflector / standalone AS` | iBGP reachability will not be destroyed blindly |
| 5 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 6 | Enter BGP process under member AS | Router | `router bgp <local-member-asn>` | Router enters BGP configuration mode |
| 7 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 8 | Deactivate inter-member neighbor | Router | `no neighbor <confed-peer-ip> activate` | Inter-member IPv4 unicast exchange stops |
| 9 | Remove next-hop-self if lab-only | Router | `no neighbor <neighbor-ip> next-hop-self` | Next-hop rewrite is removed |
| 10 | Remove route-reflector-client if lab-only | Router | `no neighbor <client-ip> route-reflector-client` | Client marking is removed |
| 11 | Exit address-family mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 12 | Remove inter-member neighbor | Router | `no neighbor <confed-peer-ip> remote-as <other-member-asn>` | Confederation peer neighbor is removed |
| 13 | Remove external neighbor if full lab rollback is intended | Router | `no neighbor <external-peer-ip> remote-as <external-asn>` | External neighbor is removed |
| 14 | Remove same-member iBGP neighbor if full lab rollback is intended | Router | `no neighbor <same-member-peer-ip> remote-as <local-member-asn>` | Same-member iBGP neighbor is removed |
| 15 | Remove confederation peer AS list | Router | `no bgp confederation peers <other-member-asn>` | Other member AS is no longer treated as confederation peer |
| 16 | Remove confederation identifier | Router | `no bgp confederation identifier <confed-asn>` | Router no longer participates in that confederation |
| 17 | Remove BGP process only if full rollback is intended | Router | `no router bgp <local-member-asn>` | BGP process is removed |
| 18 | Soft clear remaining neighbors if policy only changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Advertisement state is refreshed without hard reset |
| 19 | Verify confederation config removed | Router | `show running-config | include bgp confederation` | Confederation commands are gone |
| 20 | Verify route behavior after rollback | Router | `show bgp ipv4 unicast <prefix>` | Routes now reflect replacement design |
| 21 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_iBGP_Scalability_Confederations_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Inter-member BGP session stays Idle | `show bgp ipv4 unicast summary` | Missing reachability, wrong `remote-as`, missing `bgp confederation peers`, or TCP/179 blocked |
| External peer rejects confed edge | External peer `show running-config | section router bgp` | External peer used member AS instead of confederation identifier |
| Confed router appears as wrong AS externally | External peer `show bgp ipv4 unicast summary` | `bgp confederation identifier` missing or wrong |
| Internal member ASNs leak externally | External peer `show bgp ipv4 unicast <prefix>` | Confederation edge or identifier is misconfigured |
| Confederation peer behaves like normal eBGP | `show running-config | include bgp confederation peers` | Missing `bgp confederation peers <member-asn>` |
| Same-member iBGP route not advertised to another iBGP peer | `show bgp ipv4 unicast neighbors <peer> advertised-routes` | Normal iBGP split-horizon issue; need full mesh or route reflector inside member AS |
| Route learned across member AS but not installed | `show ip route <prefix>` and `show ip route <bgp-next-hop-ip>` | Next hop is unreachable |
| Confed route shows unexpected next hop | `show bgp ipv4 unicast <prefix>` | Confederation next-hop behavior differs from normal eBGP assumption |
| Route needs next-hop-self | `show ip route <bgp-next-hop-ip>` | Peer cannot resolve advertised next hop |
| AS_CONFED_SEQUENCE missing internally | `show bgp ipv4 unicast <prefix>` | Route did not cross member AS boundary or confederation peering is wrong |
| AS_CONFED_SEQUENCE visible externally | External peer `show bgp ipv4 unicast <prefix>` | Confederation boundary behavior is broken or peer is not truly external |
| Local preference not visible inside confederation | `show bgp ipv4 unicast <prefix>` | Policy removed/overrode it or route did not pass through intended confed path |
| MED behavior differs from expectation | `show bgp ipv4 unicast <prefix>` | MED remains inside confederation but does not leave it |
| `no-export` route still crosses member ASs | `show bgp ipv4 unicast community no-export` | Expected behavior inside confederation; it should not leave confederation |
| `local-AS` route does not cross member AS boundary | `show bgp ipv4 unicast community local-AS` | Expected behavior; local-AS blocks advertisement outside local member AS/sub-AS |
| External peer sees no route | Edge `show bgp ipv4 unicast neighbors <external-peer> advertised-routes` | Outbound policy, no valid route, next-hop issue, or confed boundary misconfig |
| Inter-member peer sees no route | Border `show bgp ipv4 unicast neighbors <confed-peer> advertised-routes` | Prefix not best/valid, policy blocks it, or neighbor AFI not activated |
| BGP table shows route but RIB does not | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | RIB failure, next-hop issue, or better administrative distance route |
| Peering with loopbacks fails | `ping <peer-loopback> source <local-loopback>` | IGP/static reachability to loopbacks is missing |
| Confederation design becomes harder than full mesh | Topology review | Too few routers or too small a lab to justify confederation complexity |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used when soft refresh would have worked |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-confederation-final` | `BGP_iBGP_Scalability_Confederations.md` | Primary lab for member ASs, confederation identifier, confederation peers, internal AS path behavior, and external AS hiding |
# Index
# Source_Basis
# BGP_iBGP_Scalability_Confederations_Mental_Model
# BGP_iBGP_Scalability_Confederations_Configuration_Checklist
# BGP_iBGP_Scalability_Confederations_Member_AS_Basic_Skeleton
# BGP_iBGP_Scalability_Confederations_Inter_Member_Peer_Skeleton
# BGP_iBGP_Scalability_Confederations_External_Edge_Skeleton
# BGP_iBGP_Scalability_Confederations_External_Peer_View_Skeleton
# BGP_iBGP_Scalability_Confederations_Route_Reflector_Inside_Member_AS_Skeleton
# BGP_iBGP_Scalability_Confederations_Next_Hop_Self_Skeleton
# BGP_iBGP_Scalability_Confederations_Verification_Skeleton
# BGP_iBGP_Scalability_Confederations_Verification_Commands
# BGP_iBGP_Scalability_Confederations_Rollback
# BGP_iBGP_Scalability_Confederations_Failure_Checks
# Attached_Labs

Blunt rule: confederations make one AS look like many inside, and many inside look like one outside. If the outside peer sees your member ASNs, the boundary is wrong. If internal peers cannot install routes, the confederation probably worked and your next-hop or policy design is broken.