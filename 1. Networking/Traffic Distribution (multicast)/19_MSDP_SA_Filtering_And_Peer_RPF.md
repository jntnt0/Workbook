Cisco’s MSDP configuration guides confirm the ip msdp sa-filter in and ip msdp sa-filter out syntax, including peer-specific filtering with ACLs, route maps, RP lists, and RP route maps. They also document ip msdp sa-limit for limiting accepted SA messages per peer.  

MSDP_SA_Filtering_And_Peer_RPF.md

MSDP_SA_Filtering_And_Peer_RPF

# MSDP_SA_Filtering_And_Peer_RPF_Mental_Model
| Concept | Operational Meaning |
|---|---|
| MSDP SA message | Source Active message advertises that a source is sending to a multicast group |
| SA tuple | The useful policy object is usually `(S,G)`: source address and multicast group address |
| SA cache | Table of active source/group entries learned locally or from MSDP peers |
| SA filtering | Policy that controls which SA messages are accepted from a peer or advertised to a peer |
| Inbound SA filter | Controls which SA messages received from a specific MSDP peer are accepted locally |
| Outbound SA filter | Controls which SA messages are sent to a specific MSDP peer |
| ACL matching | Extended ACL source field normally represents the multicast source, and destination field represents the multicast group |
| Route-map filtering | Route maps give more flexible filtering than raw ACLs and are safer for complex policy |
| RP filtering | `rp-list` or `rp-route-map` filters based on the originating RP for the SA |
| Peer-RPF | Loop-prevention check that decides whether an SA arrived from the correct MSDP peer |
| Peer-RPF failure | SA is rejected because it came from a peer that is not the logical best peer back to the originating RP |
| Mesh group | MSDP mesh group reduces SA reflooding and avoids peer-RPF problems inside a full-mesh RP core |
| Default peer | Used in simple single-peer MSDP designs to avoid peer-RPF complexity |
| SA limit | Protects the router by limiting how many SA cache entries can be accepted from a peer |
| Policy goal | Admit only useful, trusted source/group information and prevent SA loops, leaks, and cache pollution |
| Verification truth | `show ip msdp peer`, `show ip msdp sa-cache`, `show ip mroute`, `show ip route <ORIGINATING_RP>`, and MSDP debug prove whether SA filtering and peer-RPF are working |
# MSDP_SA_Filtering_And_Peer_RPF_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify MSDP peers, local RP, remote RP, source groups, and policy direction | RP routers | `show ip msdp peer`<br>`show ip msdp sa-cache` | MSDP peers, SA entries, and filtering targets are known |
| 2 | Confirm MSDP peer sessions are established before applying filters | RP routers | `show ip msdp peer` | Peer state is established |
| 3 | Confirm TCP 639 session exists if peer state is unclear | RP routers | `show tcp brief | include 639` | MSDP TCP session is visible |
| 4 | Confirm route to the remote MSDP peer loopback | RP routers | `show ip route <REMOTE_MSDP_PEER_ADDRESS>` | Route exists to the peer source address |
| 5 | Confirm route to the originating RP used in peer-RPF checks | RP routers | `show ip route <ORIGINATING_RP_ADDRESS>` | Route exists toward the RP that originated the SA |
| 6 | Review current SA cache before filtering | RP routers | `show ip msdp sa-cache` | Existing learned `(S,G)` entries are visible |
| 7 | Identify SA entries to permit | RP routers | `show ip msdp sa-cache | include <ALLOWED_SOURCE_OR_GROUP>` | Allowed source/group entries are known |
| 8 | Identify SA entries to deny | RP routers | `show ip msdp sa-cache | include <DENIED_SOURCE_OR_GROUP>` | Denied source/group entries are known |
| 9 | Create extended ACL for SA filtering by source and group | RP router | `conf t`<br>`ip access-list extended <SA_FILTER_ACL>` | Extended ACL configuration mode opens |
| 10 | Permit trusted source/group SA entries | RP router | `permit ip <ALLOWED_SOURCE_NETWORK> <SOURCE_WILDCARD> <ALLOWED_GROUP_RANGE> <GROUP_WILDCARD>` | Trusted `(S,G)` entries are permitted by the ACL |
| 11 | Deny untrusted source/group SA entries | RP router | `deny ip <DENIED_SOURCE_NETWORK> <SOURCE_WILDCARD> <DENIED_GROUP_RANGE> <GROUP_WILDCARD>` | Unwanted `(S,G)` entries are denied by the ACL |
| 12 | Add final permit if policy is deny-specific | RP router | `permit ip any any` | Other valid SA messages are not accidentally blocked |
| 13 | Verify ACL order before applying it | RP router | `show access-lists <SA_FILTER_ACL>` | Specific permits and denies appear in the intended order |
| 14 | Apply inbound SA filter to block unwanted SAs received from a peer | RP router | `conf t`<br>`ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> list <SA_FILTER_ACL>` | Router accepts only matching inbound SAs from that peer |
| 15 | Apply outbound SA filter to block unwanted SAs sent to a peer | RP router | `conf t`<br>`ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> list <SA_FILTER_ACL>` | Router sends only matching outbound SAs to that peer |
| 16 | Create route map for SA filtering if ACL-only policy is not enough | RP router | `conf t`<br>`route-map <SA_FILTER_ROUTE_MAP> permit 10`<br>`match ip address <SA_FILTER_ACL>` | Route map permits only SA entries matched by ACL |
| 17 | Add explicit deny sequence to route map if needed | RP router | `route-map <SA_FILTER_ROUTE_MAP> deny 20` | Route map blocks all unmatched SA entries |
| 18 | Apply inbound route-map SA filter | RP router | `conf t`<br>`ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> route-map <SA_FILTER_ROUTE_MAP>` | Incoming SA messages from peer must pass route map |
| 19 | Apply outbound route-map SA filter | RP router | `conf t`<br>`ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> route-map <SA_FILTER_ROUTE_MAP>` | Outgoing SA messages to peer must pass route map |
| 20 | Create RP-list ACL if filtering by originating RP | RP router | `conf t`<br>`ip access-list standard <RP_FILTER_ACL>`<br>`permit <ALLOWED_ORIGINATING_RP>` | ACL matches permitted originating RP addresses |
| 21 | Apply inbound filter based on originating RP | RP router | `conf t`<br>`ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> rp-list <RP_FILTER_ACL>` | Router accepts SAs only from permitted originating RPs |
| 22 | Apply outbound filter based on originating RP | RP router | `conf t`<br>`ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> rp-list <RP_FILTER_ACL>` | Router advertises SAs only for permitted originating RPs |
| 23 | Configure SA limit for noisy or untrusted peer | RP router | `conf t`<br>`ip msdp sa-limit <REMOTE_MSDP_PEER_ADDRESS> <SA_LIMIT>` | Router limits number of accepted SA cache entries from peer |
| 24 | Verify SA filter commands are installed | RP router | `show running-config | include ^ip msdp sa-filter` | Inbound and outbound filters appear under running configuration |
| 25 | Verify SA limit is installed | RP router | `show running-config | include ^ip msdp sa-limit` | SA limit appears for the selected peer |
| 26 | Clear SA cache for a clean policy test | RP router | `clear ip msdp sa-cache *` | Old SA cache entries are removed |
| 27 | Generate allowed multicast source traffic | Allowed source router | `ping <ALLOWED_MULTICAST_GROUP> source <ALLOWED_SOURCE_INTERFACE_OR_IP> repeat 5` | Allowed source creates valid local SA state |
| 28 | Verify allowed SA is accepted or advertised | RP routers | `show ip msdp sa-cache | include <ALLOWED_SOURCE_IP>` | Allowed `(S,G)` appears in SA cache where expected |
| 29 | Generate denied multicast source traffic | Denied source router | `ping <DENIED_MULTICAST_GROUP> source <DENIED_SOURCE_INTERFACE_OR_IP> repeat 5` | Denied source attempts to create SA state |
| 30 | Verify denied SA is blocked | RP routers | `show ip msdp sa-cache | include <DENIED_SOURCE_IP>` | Denied `(S,G)` is absent from filtered side |
| 31 | Verify multicast forwarding for allowed group | RP and last-hop routers | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Allowed source/group has multicast forwarding state |
| 32 | Verify multicast forwarding does not occur for denied group | RP and last-hop routers | `show ip mroute <DENIED_MULTICAST_GROUP>` | Denied source/group does not build useful remote forwarding state |
| 33 | Check peer-RPF path for accepted SA originator | RP router | `show ip route <ORIGINATING_RP_ADDRESS>` | Route points toward the expected MSDP peer or peer topology |
| 34 | Confirm MSDP mesh group if using full-mesh Anycast RP | RP routers | `show running-config | include ^ip msdp mesh-group` | Mesh-group entries exist for all intended mesh peers |
| 35 | Use default peer only in simple single-peer designs | RP router | `show running-config | include ^ip msdp default-peer` | Default peer is present only when deliberately required |
| 36 | Use controlled MSDP debugging if SA behavior is unclear | RP router | `debug ip msdp` | SA filtering, peer events, and peer-RPF messages are visible |
| 37 | Disable debugging after test | RP router | `undebug all` | Debugging is disabled |
| 38 | Save working SA filter and peer-RPF configuration | RP routers | `copy running-config startup-config` | MSDP SA filtering configuration is preserved |
# MSDP_SA_Filtering_And_Peer_RPF_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <REMOTE_MSDP_PEER_ADDRESS>        = remote MSDP peer unique loopback
! <ORIGINATING_RP_ADDRESS>          = RP address that originated the SA
! <SA_FILTER_ACL>                   = extended ACL matching source/group SA tuples
! <SA_FILTER_ROUTE_MAP>             = route map used for SA filtering
! <RP_FILTER_ACL>                   = standard ACL matching originating RP addresses
! <ALLOWED_SOURCE_NETWORK>          = permitted multicast source network
! <DENIED_SOURCE_NETWORK>           = denied multicast source network
! <SOURCE_WILDCARD>                 = wildcard mask for source network
! <ALLOWED_GROUP_RANGE>             = permitted multicast group range
! <DENIED_GROUP_RANGE>              = denied multicast group range
! <GROUP_WILDCARD>                  = wildcard mask for group range
! <ALLOWED_ORIGINATING_RP>           = permitted originating RP address
! <SA_LIMIT>                        = maximum SA cache entries allowed from peer
! <ALLOWED_MULTICAST_GROUP>          = test group that should pass SA filter
! <DENIED_MULTICAST_GROUP>           = test group that should be filtered
! <ALLOWED_SOURCE_IP>                = allowed multicast source IP
! <DENIED_SOURCE_IP>                 = denied multicast source IP
! <ALLOWED_SOURCE_INTERFACE_OR_IP>   = source interface or IP for allowed source test
! <DENIED_SOURCE_INTERFACE_OR_IP>    = source interface or IP for denied source test
! =========================================================
! BASELINE MSDP CHECKS
! =========================================================
show ip msdp peer
show ip msdp sa-cache
show ip route <REMOTE_MSDP_PEER_ADDRESS>
show ip route <ORIGINATING_RP_ADDRESS>
show tcp brief | include 639
! =========================================================
! EXTENDED ACL FOR SOURCE/GROUP SA FILTERING
! Extended ACL source field = multicast source
! Extended ACL destination field = multicast group
! =========================================================
conf t
 ip access-list extended <SA_FILTER_ACL>
  permit ip <ALLOWED_SOURCE_NETWORK> <SOURCE_WILDCARD> <ALLOWED_GROUP_RANGE> <GROUP_WILDCARD>
  deny ip <DENIED_SOURCE_NETWORK> <SOURCE_WILDCARD> <DENIED_GROUP_RANGE> <GROUP_WILDCARD>
  permit ip any any
 exit
end
! =========================================================
! INBOUND SA FILTER
! Controls SAs accepted from the selected MSDP peer.
! =========================================================
conf t
 ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> list <SA_FILTER_ACL>
end
! =========================================================
! OUTBOUND SA FILTER
! Controls SAs advertised to the selected MSDP peer.
! =========================================================
conf t
 ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> list <SA_FILTER_ACL>
end
! =========================================================
! ROUTE-MAP BASED SA FILTER
! Use this when policy needs route-map structure.
! =========================================================
conf t
 route-map <SA_FILTER_ROUTE_MAP> permit 10
  match ip address <SA_FILTER_ACL>
 exit
 route-map <SA_FILTER_ROUTE_MAP> deny 20
 exit
end
conf t
 ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> route-map <SA_FILTER_ROUTE_MAP>
 ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> route-map <SA_FILTER_ROUTE_MAP>
end
! =========================================================
! RP-LIST BASED SA FILTER
! Filters based on originating RP address.
! =========================================================
conf t
 ip access-list standard <RP_FILTER_ACL>
  permit <ALLOWED_ORIGINATING_RP>
 exit
 ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> rp-list <RP_FILTER_ACL>
 ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> rp-list <RP_FILTER_ACL>
end
! =========================================================
! SA CACHE LIMIT
! Protects the router from excessive SA entries from a peer.
! =========================================================
conf t
 ip msdp sa-limit <REMOTE_MSDP_PEER_ADDRESS> <SA_LIMIT>
end
! =========================================================
! CLEAN TEST STATE
! =========================================================
clear ip msdp sa-cache *
! =========================================================
! TEST ALLOWED SOURCE
! =========================================================
ping <ALLOWED_MULTICAST_GROUP> source <ALLOWED_SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! TEST DENIED SOURCE
! =========================================================
ping <DENIED_MULTICAST_GROUP> source <DENIED_SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show running-config | include ^ip msdp
show access-lists <SA_FILTER_ACL>
show access-lists <RP_FILTER_ACL>
show route-map <SA_FILTER_ROUTE_MAP>
show ip msdp peer
show ip msdp sa-cache
show ip mroute <ALLOWED_MULTICAST_GROUP>
show ip mroute <DENIED_MULTICAST_GROUP>
show ip route <ORIGINATING_RP_ADDRESS>
! =========================================================
! CONTROLLED DEBUG
! Use briefly only during SA filtering and peer-RPF testing.
! =========================================================
debug ip msdp
undebug all
# MSDP_SA_Filtering_And_Peer_RPF_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| MSDP peer state | RP routers | `show ip msdp peer` | Peer is established |
| MSDP TCP state | RP routers | `show tcp brief | include 639` | TCP 639 session exists |
| SA cache baseline | RP routers | `show ip msdp sa-cache` | Existing source/group entries are visible |
| SA filter configuration | RP router | `show running-config | include ^ip msdp sa-filter` | Inbound and/or outbound filters are present |
| SA limit configuration | RP router | `show running-config | include ^ip msdp sa-limit` | SA limit is configured for selected peer |
| Extended ACL contents | RP router | `show access-lists <SA_FILTER_ACL>` | ACL permits and denies intended `(S,G)` entries |
| RP-list ACL contents | RP router | `show access-lists <RP_FILTER_ACL>` | ACL permits intended originating RPs |
| Route-map contents | RP router | `show route-map <SA_FILTER_ROUTE_MAP>` | Route map matches intended ACL and has expected permit or deny actions |
| Allowed SA accepted | RP router | `show ip msdp sa-cache | include <ALLOWED_SOURCE_IP>` | Allowed source/group appears in SA cache |
| Denied SA blocked | RP router | `show ip msdp sa-cache | include <DENIED_SOURCE_IP>` | Denied source/group is absent from filtered side |
| Allowed multicast forwarding | RP and last-hop routers | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Allowed `(S,G)` creates valid forwarding state |
| Denied multicast forwarding | RP and last-hop routers | `show ip mroute <DENIED_MULTICAST_GROUP>` | Denied `(S,G)` does not create useful remote forwarding state |
| Route to MSDP peer | RP router | `show ip route <REMOTE_MSDP_PEER_ADDRESS>` | Route exists toward peer loopback |
| Route to originating RP | RP router | `show ip route <ORIGINATING_RP_ADDRESS>` | Route used by peer-RPF is sane |
| Mesh group config | RP routers | `show running-config | include ^ip msdp mesh-group` | Mesh-group exists if full-mesh Anycast RP core is used |
| Default peer config | RP router | `show running-config | include ^ip msdp default-peer` | Default peer exists only in deliberate simple single-peer design |
| Peer-RPF troubleshooting | RP router | `debug ip msdp` | Debug shows whether SAs pass or fail peer-RPF |
| Debug cleanup | RP router | `show debugging`<br>`undebug all` | No MSDP debug remains active |
# MSDP_SA_Filtering_And_Peer_RPF_Rollback
! =========================================================
! REMOVE SA LIMIT
! =========================================================
conf t
 no ip msdp sa-limit <REMOTE_MSDP_PEER_ADDRESS> <SA_LIMIT>
end
! =========================================================
! REMOVE INBOUND SA FILTERS
! =========================================================
conf t
 no ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> list <SA_FILTER_ACL>
 no ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> route-map <SA_FILTER_ROUTE_MAP>
 no ip msdp sa-filter in <REMOTE_MSDP_PEER_ADDRESS> rp-list <RP_FILTER_ACL>
end
! =========================================================
! REMOVE OUTBOUND SA FILTERS
! =========================================================
conf t
 no ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> list <SA_FILTER_ACL>
 no ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> route-map <SA_FILTER_ROUTE_MAP>
 no ip msdp sa-filter out <REMOTE_MSDP_PEER_ADDRESS> rp-list <RP_FILTER_ACL>
end
! =========================================================
! REMOVE ROUTE MAP
! =========================================================
conf t
 no route-map <SA_FILTER_ROUTE_MAP>
end
! =========================================================
! REMOVE EXTENDED SA FILTER ACL
! =========================================================
conf t
 no ip access-list extended <SA_FILTER_ACL>
end
! =========================================================
! REMOVE RP FILTER ACL
! =========================================================
conf t
 no ip access-list standard <RP_FILTER_ACL>
end
! =========================================================
! CLEAR MSDP AND MULTICAST STATE
! Command support varies by platform.
! =========================================================
clear ip msdp sa-cache *
clear ip mroute *
undebug all
# MSDP_SA_Filtering_And_Peer_RPF_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| All SAs disappear after filter is applied | ACL or route map has implicit deny with no final permit | RP router | `show access-lists <SA_FILTER_ACL>`<br>`show route-map <SA_FILTER_ROUTE_MAP>` | Add explicit permits for valid SAs or final `permit ip any any` if policy is deny-specific |
| Denied SA still appears | Filter applied in wrong direction or to wrong peer | RP router | `show running-config | include ^ip msdp sa-filter` | Apply `in` filter for received SAs or `out` filter for advertised SAs to the correct peer |
| Allowed SA is blocked | ACL source/group fields are reversed or ACL order is wrong | RP router | `show access-lists <SA_FILTER_ACL>` | Match source in ACL source field and group in ACL destination field |
| Filter does not affect local SA origination | `sa-filter in` controls received SAs, not local SA creation | Local RP | `show ip msdp sa-cache` | Use outbound SA filter or source admission controls depending on goal |
| Outbound filter blocks too much | Route map deny or ACL implicit deny catches all SAs | RP router | `show route-map <SA_FILTER_ROUTE_MAP>`<br>`show access-lists <SA_FILTER_ACL>` | Add correct permit sequence before deny sequence |
| SA limit reached | Peer is sending more SAs than configured limit | RP router | `show running-config | include sa-limit`<br>`show ip msdp sa-cache` | Raise SA limit or filter unwanted groups before accepting them |
| SA accepted from wrong RP | RP-list or RP route-map not configured | RP router | `show ip msdp sa-cache`<br>`show running-config | include rp-list|rp-route-map` | Add `rp-list` or `rp-route-map` to filter by originating RP |
| SA rejected despite ACL permit | Peer-RPF failure, not SA filter failure | RP router | `debug ip msdp`<br>`show ip route <ORIGINATING_RP_ADDRESS>` | Fix MSDP/BGP topology, use mesh group in RP core, or use default peer only in simple design |
| Peer-RPF fails in full mesh | Mesh group missing or inconsistent | RP routers | `show running-config | include mesh-group` | Configure matching MSDP mesh-group entries for full-mesh peers |
| Peer-RPF fails in interdomain design | MSDP peering does not align with BGP or mBGP path to originating RP | RP routers | `show ip route <ORIGINATING_RP_ADDRESS>`<br>`show ip msdp peer` | Align MSDP topology with BGP path or correct route preference |
| Default peer hides peer-RPF problem | `ip msdp default-peer` bypasses peer-RPF logic in a design that needs proper topology | RP router | `show running-config | include default-peer` | Remove default peer unless this is a single-peer edge design |
| MSDP peer is down | TCP 639, route, source loopback, or remote-as issue | RP routers | `show ip msdp peer`<br>`show tcp brief | include 639`<br>`show ip route <REMOTE_MSDP_PEER_ADDRESS>` | Fix loopback reachability, peer source, ACLs, or remote-as |
| SA cache empty before filter testing | No active source traffic exists | Source-side RP | `show ip mroute <GROUP>` | Generate source traffic and verify source registration to RP |
| SA exists but no multicast forwarding | Receiver interest, PIM forwarding, or RPF is broken | RP and last-hop routers | `show ip igmp groups`<br>`show ip mroute <GROUP>`<br>`show ip rpf <SOURCE_IP>` | Fix receiver join, OIL, PIM, or RPF |
| SSM traffic expected in MSDP SA cache | SSM does not use RP or MSDP | All multicast routers | `show running-config | include ip pim ssm` | Use ASM group for MSDP testing |
| ACL command rejected | Platform expects numbered extended ACL or supports different syntax | RP router | `ip msdp sa-filter in ?` | Use platform-supported ACL or route-map syntax |
| Clear command rejected | Platform uses different clear syntax | RP router | `clear ip msdp ?` | Use supported clear command or wait for SA cache aging |
| Debug floods console | MSDP debug left enabled | RP router | `show debugging` | Run `undebug all` |
##### Source_Basis
# MSDP_SA_Filtering_And_Peer_RPF_Mental_Model
# MSDP_SA_Filtering_And_Peer_RPF_Configuration_Checklist
# MSDP_SA_Filtering_And_Peer_RPF_Skeleton
# MSDP_SA_Filtering_And_Peer_RPF_Verification_Commands
# MSDP_SA_Filtering_And_Peer_RPF_Rollback
# MSDP_SA_Filtering_And_Peer_RPF_Failure_Checks
