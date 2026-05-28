

BGP_iBGP_Scalability_Route_Reflectors.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for iBGP scaling, route reflectors, route-reflector clients, originator ID, cluster list, and BGP best-path behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP route-reflector configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23299-23320 | Route reflector overview and rules | Supports the iBGP full-mesh problem, route-reflector role, client role, and route reflection behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23320-23362 | Route reflector configuration | Supports `neighbor <ip> route-reflector-client` under BGP address-family configuration |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23375-23413 | Route reflector rule behavior | Supports client-to-client, client-to-nonclient, and nonclient-to-client advertisement behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23413-23424 | Route reflector loop prevention | Supports Originator ID and Cluster List attributes |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23424-23436 | Originator and Cluster List verification | Supports viewing Originator and Cluster List in BGP path details |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 24377-24379 | Route reflector command reference | Supports `neighbor <ip-address> route-reflector-client` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26827 and 27809-27815 | Best-path tie-breakers | Supports lowest originator ID and minimum cluster list length in BGP path selection |
| `iosxe_combined_pdfs_.md` lines 34115-34119 | IOS XE route-reflector-client syntax | Supports `neighbor {ip-address | peer-group-name} route-reflector-client` |
| `iosxe_combined_pdfs_.md` lines 34125-34129 | IOS XE cluster ID syntax | Supports `bgp cluster-id <cluster-id>` |
| `iosxe_combined_pdfs_.md` lines 34135-34145 | Client-to-client reflection control | Supports `no bgp client-to-client reflection` when disabling client-to-client reflection is required |
# BGP_iBGP_Scalability_Route_Reflectors_Mental_Model
| Concept | Operational Meaning |
|---|---|
| iBGP full mesh | Normal iBGP requires every iBGP router to peer with every other iBGP router so routes are not hidden |
| iBGP split-horizon rule | A route learned from one iBGP peer is not advertised to another iBGP peer by default |
| Route reflector | iBGP router allowed to reflect selected iBGP-learned routes to other iBGP peers |
| Route-reflector client | iBGP peer configured on the route reflector as a client |
| Nonclient | iBGP peer of the route reflector that is not marked as a route-reflector client |
| Client configuration location | Only the route reflector is configured with `route-reflector-client`; clients do not configure themselves as clients |
| Address-family specific behavior | Route reflection is configured under the relevant BGP address family |
| Client-learned route | Route learned by the RR from a client can be reflected to other clients and nonclients |
| Nonclient-learned route | Route learned by the RR from a nonclient is reflected only to clients |
| eBGP-learned route | Route learned from eBGP can be advertised to clients and nonclients like normal BGP advertisement |
| Originator ID | Loop-prevention attribute identifying the router that originally injected the route into iBGP |
| Cluster ID | Identifier for a route-reflector cluster; defaults to the RR BGP router ID unless manually set |
| Cluster List | Loop-prevention attribute where RRs append cluster IDs as a route is reflected |
| Originator loop prevention | Router rejects a reflected route if its own router ID appears as the Originator ID |
| Cluster loop prevention | RR rejects a route if its own cluster ID already appears in the Cluster List |
| Client-to-client reflection | By default, an RR can reflect routes from one client to another client |
| `no bgp client-to-client reflection` | Disables client-to-client reflection when clients are fully meshed or design requires it |
| Route reflector redundancy | Usually requires multiple RRs or clients peered to more than one RR |
| Route reflector placement | RR should be topologically stable and have good IGP reachability to clients |
| Next-hop behavior | RR reflects routes but does not automatically solve unreachable next hops |
| Best path still applies | RR usually reflects its selected best path unless additional path behavior exists separately |
| Path hiding | RR can hide alternate paths because it reflects only selected paths by default |
| Blunt rule | Route reflectors scale iBGP control-plane peering. They do not replace next-hop reachability, good IGP design, or route policy sanity |
# BGP_iBGP_Scalability_Route_Reflectors_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm local AS | All BGP Routers | `<local-asn>` | All iBGP routers use the same AS number |
| 2 | Identify route reflector | Notes | `<rr-router>` | Router selected to reflect iBGP routes |
| 3 | Identify route-reflector clients | Notes | `<client-router-list>` | Client routers are known |
| 4 | Identify nonclient iBGP peers | Notes | `<nonclient-router-list>` | Nonclient peers are known |
| 5 | Confirm route-reflector cluster design | Notes | `<cluster-id>` or default router ID | Cluster ID plan is known |
| 6 | Confirm RR loopback address | RR | `show ip interface brief | include Loopback` | RR loopback is up/up |
| 7 | Confirm client loopback addresses | Clients | `show ip interface brief | include Loopback` | Client loopbacks are up/up |
| 8 | Confirm IGP reachability to RR loopback | Clients | `show ip route <rr-loopback-ip>` | Clients can reach RR loopback |
| 9 | Confirm IGP reachability to client loopbacks | RR | `show ip route <client-loopback-ip>` | RR can reach client loopbacks |
| 10 | Confirm sourced reachability from RR to client | RR | `ping <client-loopback-ip> source <rr-loopback-ip>` | RR can reach client from BGP source |
| 11 | Confirm sourced reachability from client to RR | Client | `ping <rr-loopback-ip> source <client-loopback-ip>` | Client can reach RR from BGP source |
| 12 | Confirm BGP neighbor baseline on RR | RR | `show bgp ipv4 unicast summary` | Existing iBGP peers are visible |
| 13 | Confirm BGP neighbor baseline on client | Client | `show bgp ipv4 unicast summary` | Client sees RR as established or ready to configure |
| 14 | Confirm current iBGP full-mesh gap | Notes / Router | `show bgp ipv4 unicast summary` | Missing full mesh is known and RR is the chosen scaling tool |
| 15 | Confirm target address family | Notes | `ipv4 unicast` | Route reflection AFI is known |
| 16 | Confirm prefixes used to test reflection | Notes | `<test-prefix-list>` | Prefixes originated by clients, nonclients, or eBGP edges are known |
| 17 | Review existing BGP config on RR | RR | `show running-config | section router bgp` | Neighbor and AFI config is visible |
| 18 | Review existing BGP config on clients | Clients | `show running-config | section router bgp` | Client neighbor and AFI config is visible |
| 19 | Enter configuration mode on RR | RR | `configure terminal` | RR enters global configuration mode |
| 20 | Enter BGP process on RR | RR | `router bgp <local-asn>` | RR enters BGP configuration mode |
| 21 | Set stable RR router ID | RR | `bgp router-id <rr-router-id>` | RR has deterministic BGP router ID |
| 22 | Configure cluster ID if using explicit cluster design | RR | `bgp cluster-id <cluster-id>` | RR uses configured cluster ID instead of default router ID |
| 23 | Standardize explicit AF activation if used | RR | `no bgp default ipv4-unicast` | Neighbors must be activated under AFI |
| 24 | Configure RR neighbor to client | RR | `neighbor <client-loopback-ip> remote-as <local-asn>` | Client iBGP neighbor is defined |
| 25 | Configure RR update source | RR | `neighbor <client-loopback-ip> update-source Loopback0` | RR sources BGP from Loopback0 |
| 26 | Add RR neighbor description | RR | `neighbor <client-loopback-ip> description RR-client-<name>` | Neighbor purpose is documented |
| 27 | Enter IPv4 unicast AFI on RR | RR | `address-family ipv4 unicast` | RR enters IPv4 unicast AFI |
| 28 | Activate client neighbor on RR | RR | `neighbor <client-loopback-ip> activate` | RR and client can exchange IPv4 unicast NLRI |
| 29 | Mark client as route-reflector client | RR | `neighbor <client-loopback-ip> route-reflector-client` | RR reflects eligible iBGP routes for that client |
| 30 | Repeat client config for each RR client | RR | `neighbor <client-loopback-ip> route-reflector-client` | All intended clients are marked correctly |
| 31 | Configure nonclient iBGP neighbor on RR if required | RR | `neighbor <nonclient-loopback-ip> remote-as <local-asn>` | Nonclient iBGP peer is defined |
| 32 | Activate nonclient neighbor without client marking | RR | `neighbor <nonclient-loopback-ip> activate` | Nonclient peer exchanges IPv4 unicast but is not a client |
| 33 | Leave client-to-client reflection enabled by default | RR | `no command needed` | RR can reflect from one client to another |
| 34 | Disable client-to-client reflection only if clients are otherwise fully meshed | RR | `no bgp client-to-client reflection` | RR no longer reflects client routes to other clients |
| 35 | Exit RR address-family mode | RR | `exit-address-family` | RR returns to BGP config mode |
| 36 | Save RR configuration | RR | `copy running-config startup-config` | RR config is saved |
| 37 | Enter configuration mode on client | Client | `configure terminal` | Client enters global configuration mode |
| 38 | Enter BGP process on client | Client | `router bgp <local-asn>` | Client enters BGP config mode |
| 39 | Set stable client router ID | Client | `bgp router-id <client-router-id>` | Client has deterministic BGP router ID |
| 40 | Configure client neighbor to RR | Client | `neighbor <rr-loopback-ip> remote-as <local-asn>` | Client iBGP neighbor to RR is defined |
| 41 | Configure client update source | Client | `neighbor <rr-loopback-ip> update-source Loopback0` | Client sources BGP from Loopback0 |
| 42 | Add client neighbor description | Client | `neighbor <rr-loopback-ip> description Route-reflector` | Neighbor purpose is documented |
| 43 | Enter IPv4 unicast AFI on client | Client | `address-family ipv4 unicast` | Client enters IPv4 unicast AFI |
| 44 | Activate RR neighbor on client | Client | `neighbor <rr-loopback-ip> activate` | Client exchanges IPv4 unicast with RR |
| 45 | Do not configure `route-reflector-client` on client | Client | `No route-reflector-client command` | Client remains normal iBGP peer from its own perspective |
| 46 | Originate test client prefix if needed | Client | `network <client-prefix> mask <client-mask>` | Client injects test prefix into BGP if exact route exists |
| 47 | Verify exact route for test prefix | Client | `show ip route <client-prefix> <client-mask>` | Exact route exists for BGP `network` command |
| 48 | Exit client AFI mode | Client | `exit-address-family` | Client returns to BGP config mode |
| 49 | Save client config | Client | `copy running-config startup-config` | Client config is saved |
| 50 | Verify RR-client BGP session | RR | `show bgp ipv4 unicast summary` | Client neighbors show established state |
| 51 | Verify client-to-RR BGP session | Client | `show bgp ipv4 unicast summary` | RR neighbor shows established state |
| 52 | Verify RR client configuration | RR | `show running-config | section router bgp` | Client neighbors show `route-reflector-client` under AFI |
| 53 | Verify route learned by RR from client | RR | `show bgp ipv4 unicast <client-prefix>` | RR sees client-originated route |
| 54 | Verify route reflected to another client | Other Client | `show bgp ipv4 unicast <client-prefix>` | Other client sees reflected route |
| 55 | Verify originator ID on reflected route | Other Client | `show bgp ipv4 unicast <client-prefix>` | Output shows Originator attribute if displayed |
| 56 | Verify cluster list on reflected route | Other Client | `show bgp ipv4 unicast <client-prefix>` | Output shows Cluster List if route crossed RR |
| 57 | Verify route from nonclient reflects only to clients | Client / Nonclient | `show bgp ipv4 unicast <nonclient-prefix>` | Client sees route; other nonclients do not receive it from RR |
| 58 | Verify route from client reflects to nonclient if design includes nonclient peer | Nonclient | `show bgp ipv4 unicast <client-prefix>` | Nonclient sees client-originated reflected route |
| 59 | Verify route install on client | Client | `show ip route <reflected-prefix>` | Reflected BGP route installs if valid and selected |
| 60 | Verify next-hop reachability for reflected route | Client | `show ip route <bgp-next-hop-ip>` | Client can reach BGP next hop |
| 61 | Fix next-hop if required on edge/RR policy | RR / Edge | `neighbor <client-ip> next-hop-self` under AFI if appropriate | Client receives reachable next hop where design requires it |
| 62 | Verify forwarding to reflected prefix | Client | `show ip cef <reflected-prefix>` | CEF points to reachable next hop |
| 63 | Verify RR advertised routes to client | RR | `show bgp ipv4 unicast neighbors <client-ip> advertised-routes` | RR advertises reflected routes to client |
| 64 | Verify BGP best-path reason if path hiding is suspected | RR / Client | `show bgp ipv4 unicast <prefix> best-path-reason` | Best-path decision is understood |
# BGP_iBGP_Scalability_Route_Reflectors_Basic_RR_Skeleton
configure terminal
router bgp <local-asn>
 bgp router-id <rr-router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <client1-loopback-ip> remote-as <local-asn>
 neighbor <client1-loopback-ip> update-source Loopback0
 neighbor <client1-loopback-ip> description RR-client-1
 neighbor <client2-loopback-ip> remote-as <local-asn>
 neighbor <client2-loopback-ip> update-source Loopback0
 neighbor <client2-loopback-ip> description RR-client-2
 address-family ipv4 unicast
  neighbor <client1-loopback-ip> activate
  neighbor <client1-loopback-ip> route-reflector-client
  neighbor <client2-loopback-ip> activate
  neighbor <client2-loopback-ip> route-reflector-client
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_Client_Skeleton
configure terminal
router bgp <local-asn>
 bgp router-id <client-router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <rr-loopback-ip> remote-as <local-asn>
 neighbor <rr-loopback-ip> update-source Loopback0
 neighbor <rr-loopback-ip> description Route-reflector
 address-family ipv4 unicast
  neighbor <rr-loopback-ip> activate
  network <client-prefix> mask <client-mask>
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_With_Explicit_Cluster_ID_Skeleton
configure terminal
router bgp <local-asn>
 bgp router-id <rr-router-id>
 bgp cluster-id <cluster-id>
 address-family ipv4 unicast
  neighbor <client-loopback-ip> activate
  neighbor <client-loopback-ip> route-reflector-client
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_Peer_Group_Skeleton
configure terminal
router bgp <local-asn>
 bgp router-id <rr-router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor RR-CLIENTS peer-group
 neighbor RR-CLIENTS remote-as <local-asn>
 neighbor RR-CLIENTS update-source Loopback0
 neighbor <client1-loopback-ip> peer-group RR-CLIENTS
 neighbor <client2-loopback-ip> peer-group RR-CLIENTS
 address-family ipv4 unicast
  neighbor RR-CLIENTS activate
  neighbor RR-CLIENTS route-reflector-client
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_Disable_Client_To_Client_Reflection_Skeleton
configure terminal
router bgp <local-asn>
 no bgp client-to-client reflection
end
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_Next_Hop_Self_To_Clients_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <client-loopback-ip> next-hop-self
 exit-address-family
end
clear bgp ipv4 unicast <client-loopback-ip> soft out
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_Redundant_RR_Client_Skeleton
configure terminal
router bgp <local-asn>
 bgp router-id <client-router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <rr1-loopback-ip> remote-as <local-asn>
 neighbor <rr1-loopback-ip> update-source Loopback0
 neighbor <rr1-loopback-ip> description RR1
 neighbor <rr2-loopback-ip> remote-as <local-asn>
 neighbor <rr2-loopback-ip> update-source Loopback0
 neighbor <rr2-loopback-ip> description RR2
 address-family ipv4 unicast
  neighbor <rr1-loopback-ip> activate
  neighbor <rr2-loopback-ip> activate
 exit-address-family
end
copy running-config startup-config
# BGP_iBGP_Scalability_Route_Reflectors_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show ip route <prefix>
show ip route <bgp-next-hop-ip>
show ip cef <prefix>
show running-config | section router bgp
show logging | include BGP|ADJCHANGE
# BGP_iBGP_Scalability_Route_Reflectors_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP sessions on RR | `show bgp ipv4 unicast summary` | Client and nonclient sessions are established |
| Verify BGP sessions on client | `show bgp ipv4 unicast summary` | RR session is established |
| Verify RR config | `show running-config | section router bgp` | Client neighbors show `route-reflector-client` under address-family on RR only |
| Verify cluster ID | `show running-config | include bgp cluster-id` | Explicit cluster ID appears if configured |
| Verify client-to-client reflection setting | `show running-config | include client-to-client` | Command appears only if client-to-client reflection was disabled |
| Verify client prefix on originator | `show bgp ipv4 unicast <client-prefix>` | Originating client shows locally originated prefix |
| Verify route learned by RR | `show bgp ipv4 unicast <client-prefix>` on RR | RR sees client-originated route |
| Verify route reflected to another client | `show bgp ipv4 unicast <client-prefix>` on other client | Other client sees reflected prefix |
| Verify route reflected to nonclient | `show bgp ipv4 unicast <client-prefix>` on nonclient | Nonclient sees client-originated route if peered with RR |
| Verify nonclient route reflected to client | `show bgp ipv4 unicast <nonclient-prefix>` on client | Client sees nonclient route via RR |
| Verify nonclient route not reflected to another nonclient | `show bgp ipv4 unicast <nonclient-prefix>` on other nonclient | Other nonclient does not receive it from RR unless separately peered |
| Verify Originator ID | `show bgp ipv4 unicast <reflected-prefix>` | Reflected route shows Originator ID when platform displays it |
| Verify Cluster List | `show bgp ipv4 unicast <reflected-prefix>` | Reflected route shows Cluster List when platform displays it |
| Verify reflected route install | `show ip route <reflected-prefix>` | Reflected route installs if valid and selected |
| Verify next-hop reachability | `show ip route <bgp-next-hop-ip>` | Client can reach BGP next hop |
| Verify forwarding entry | `show ip cef <reflected-prefix>` | CEF points toward expected next hop/interface |
| Verify RR advertised routes | `show bgp ipv4 unicast neighbors <client-ip> advertised-routes` | RR advertises expected reflected routes to client |
| Verify best-path reason | `show bgp ipv4 unicast <prefix> best-path-reason` | Best path decision is understood if supported |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected session resets occurred |
# BGP_iBGP_Scalability_Route_Reflectors_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify RR configuration | RR | `show running-config | section router bgp` | Route-reflector-client commands, cluster ID, and neighbors are visible |
| 2 | Identify reflected routes | Client / RR | `show bgp ipv4 unicast <prefix>` | Reflected routes and attributes are visible |
| 3 | Identify advertised reflected routes | RR | `show bgp ipv4 unicast neighbors <client-ip> advertised-routes` | Current reflected advertisements are visible |
| 4 | Enter configuration mode on RR | RR | `configure terminal` | RR enters global configuration mode |
| 5 | Enter BGP process on RR | RR | `router bgp <local-asn>` | RR enters BGP config mode |
| 6 | Enter IPv4 unicast AFI | RR | `address-family ipv4 unicast` | RR enters IPv4 unicast AFI |
| 7 | Remove route-reflector-client from one client | RR | `no neighbor <client-loopback-ip> route-reflector-client` | Neighbor is no longer an RR client |
| 8 | Remove route-reflector-client from peer group if used | RR | `no neighbor <peer-group-name> route-reflector-client` | Peer group no longer acts as RR clients |
| 9 | Remove next-hop-self to clients if lab-only | RR | `no neighbor <client-loopback-ip> next-hop-self` | Next-hop rewrite is removed |
| 10 | Exit AFI mode | RR | `exit-address-family` | RR returns to BGP config mode |
| 11 | Remove cluster ID if lab-only | RR | `no bgp cluster-id <cluster-id>` | RR returns to default cluster ID behavior |
| 12 | Re-enable client-to-client reflection if disabled | RR | `bgp client-to-client reflection` | Client-to-client reflection returns to default behavior if supported |
| 13 | Remove client neighbor entirely if full rollback is intended | RR | `no neighbor <client-loopback-ip> remote-as <local-asn>` | Client neighbor is removed from RR |
| 14 | Remove peer group if lab-only | RR | `no neighbor <peer-group-name> peer-group` | Peer group is removed |
| 15 | Soft clear outbound from RR | RR | `clear bgp ipv4 unicast <client-loopback-ip> soft out` | Client receives updated route advertisement behavior |
| 16 | Verify route-reflector-client removed | RR | `show running-config | section router bgp` | Removed client no longer has RR-client marking |
| 17 | Verify reflected route withdrawal | Client | `show bgp ipv4 unicast <reflected-prefix>` | Route is absent unless learned through another valid iBGP path |
| 18 | Restore full mesh if RR function is removed | All iBGP Routers | `neighbor <peer-loopback-ip> remote-as <local-asn>` | iBGP reachability remains intact after RR removal |
| 19 | Save rollback | RR | `copy running-config startup-config` | Rollback is saved |
# BGP_iBGP_Scalability_Route_Reflectors_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Client does not receive reflected route | RR `show bgp ipv4 unicast neighbors <client-ip> advertised-routes` | Client not marked as route-reflector-client, wrong AFI, route not best on RR, or outbound policy blocks it |
| Route-reflector-client command appears on client | Client `show running-config | section router bgp` | Misunderstanding of role; only RR marks clients |
| RR does not learn client route | RR `show bgp ipv4 unicast <client-prefix>` | Client did not originate route, neighbor down, AFI not activated, or policy blocks it |
| Client route not originated | Client `show ip route <client-prefix> <mask>` | Exact route for BGP `network` statement is missing |
| Reflected route appears but does not install | Client `show ip route <prefix>` | Next hop unreachable, better route exists, or RIB failure |
| Reflected route next hop unreachable | Client `show ip route <bgp-next-hop-ip>` | IGP does not carry next-hop route or `next-hop-self` is needed |
| Nonclient does not receive route from another nonclient | RR advertised-routes check | Expected behavior; RR reflects nonclient-learned routes only to clients |
| Client-to-client reflection does not occur | RR `show running-config | include client-to-client` | `no bgp client-to-client reflection` is configured or policy blocks route |
| Route reflection creates path hiding | RR and client `show bgp ipv4 unicast <prefix>` | RR reflects only its best path by default; consider topology, policy, or Additional Paths |
| Originator ID causes route rejection | `show bgp ipv4 unicast <prefix>` and router IDs | Router sees its own router ID in Originator ID |
| Cluster List causes route rejection | `show bgp ipv4 unicast <prefix>` | RR sees its own cluster ID in Cluster List |
| Duplicate cluster ID causes unexpected rejection | `show running-config | include bgp cluster-id` | Multiple RRs share cluster ID incorrectly for the design |
| Route reflector cluster design unclear | Config review | Cluster IDs and client placement were not planned |
| RR sessions flap | `show logging | include BGP|ADJCHANGE` | IGP instability to loopbacks, source mismatch, or TCP/179 issue |
| Client can ping RR physical IP but BGP fails | `ping <rr-loopback-ip> source <client-loopback-ip>` | Loopback-to-loopback reachability is broken |
| RR has clients in wrong address family | `show running-config | section address-family` | `route-reflector-client` configured under wrong AFI or neighbor not activated |
| Policy blocks reflected route | `show running-config | include route-map|prefix-list|filter-list|distribute-list` | Route-map/filter attached inbound or outbound blocks prefix |
| Full mesh removed before RR verified | `show bgp ipv4 unicast summary` | iBGP topology lost reachability because RR was not working first |
| Redundant RR clients choose odd path | `show bgp ipv4 unicast <prefix> best-path-reason` | Originator ID, cluster list length, IGP metric, or route policy changed best path |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Used hard clear instead of soft route refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-route-reflector-final` | `BGP_iBGP_Scalability_Route_Reflectors.md` | Primary route-reflector lab for iBGP scaling, client configuration, reflected route behavior, and full-mesh reduction |
| `bgp-route-reflector-ibgp-final` | `BGP_iBGP_Scalability_Route_Reflectors.md` | Route-reflector iBGP variant lab for client/nonclient behavior and reflected iBGP route verification |
# Index
# Source_Basis
# BGP_iBGP_Scalability_Route_Reflectors_Mental_Model
# BGP_iBGP_Scalability_Route_Reflectors_Configuration_Checklist
# BGP_iBGP_Scalability_Route_Reflectors_Basic_RR_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Client_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_With_Explicit_Cluster_ID_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Peer_Group_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Disable_Client_To_Client_Reflection_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Next_Hop_Self_To_Clients_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Redundant_RR_Client_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Verification_Skeleton
# BGP_iBGP_Scalability_Route_Reflectors_Verification_Commands
# BGP_iBGP_Scalability_Route_Reflectors_Rollback
# BGP_iBGP_Scalability_Route_Reflectors_Failure_Checks
# Attached_Labs

Blunt rule: route reflectors solve the iBGP full-mesh problem, not next-hop reachability. If the client sees the reflected route but cannot install it, the RR worked and your next-hop or policy design is broken.