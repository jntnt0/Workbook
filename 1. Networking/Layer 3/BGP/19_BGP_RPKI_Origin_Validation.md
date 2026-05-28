

The project source has RPKI validation-code visibility, but the actual configuration syntax is thin there. For the missing syntax, I used official Cisco IOS XE 17.x documentation: Cisco documents bgp rpki server tcp ... port ... refresh ..., RPKI states Valid/Invalid/Not Found, iBGP RPKI-state announcement with extended communities, bgp bestpath prefix-validate, and route-map match rpki policy handling.  

BGP_RPKI_Origin_Validation.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP path attributes, policy, AS path behavior, route maps, and BGP design behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22802-22806 | BGP table status and RPKI validation codes | Supports reading RPKI validation codes: `V` valid, `I` invalid, and `N` not found |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 27546-27550 | BGP table codes including RPKI validation codes | Supports identifying RPKI validation state in BGP table output |
| Cisco IOS XE 17.x IP Routing Configuration Guide, BGP Origin AS Validation | RPKI origin validation overview | Supports using an RPKI server to validate whether prefixes originate from expected ASNs |
| Cisco IOS XE 17.x IP Routing Configuration Guide, BGP Origin AS Validation | RPKI server and RTR behavior | Supports `bgp rpki server tcp <ip> port <port> refresh <seconds>` and router-to-cache behavior |
| Cisco IOS XE 17.x IP Routing Configuration Guide, BGP Origin AS Validation | Validation states | Supports Valid, Invalid, and Not Found state meanings |
| Cisco IOS XE 17.x IP Routing Configuration Guide, BGP Origin AS Validation | iBGP RPKI state announcement | Supports `neighbor <ip> send-community extended` and `neighbor <ip> announce rpki state` |
| Cisco IOS XE 17.x IP Routing Configuration Guide, BGP Origin AS Validation | Prefix validation behavior | Supports `bgp bestpath prefix-validate disable` and `bgp bestpath prefix-validate allow-invalid` |
| Cisco IOS XE 17.x IP Routing Configuration Guide, BGP Origin AS Validation | RPKI route-map policy | Supports `match rpki {valid | invalid | not-found}` and setting attributes such as local preference |
# BGP_RPKI_Origin_Validation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| RPKI | Resource Public Key Infrastructure used to validate whether an AS is authorized to originate a prefix |
| ROA | Route Origin Authorization that says which AS may originate a prefix and up to what maximum prefix length |
| ROV | Route Origin Validation, the BGP-side process that checks received BGP routes against RPKI data |
| RPKI cache / validator | Server that validates ROAs and serves validated prefix-origin data to routers |
| RTR protocol | Router-to-RPKI-cache protocol used by the router to download validated prefix-origin records |
| Origin AS | Last AS in the AS_PATH, treated as the AS that originated the route |
| Prefix length validation | A route can be invalid because the prefix length is longer than the ROA maximum length |
| Valid | Prefix and origin AS match RPKI data |
| Invalid | Prefix matches RPKI data, but origin AS or max-length does not match |
| Not Found | No matching RPKI record exists for the prefix/origin pair |
| Default validation preference | Valid is preferred over Not Found, and Invalid is least preferred or rejected by default behavior |
| Invalid route policy | You can reject invalids or keep them with poor preference for visibility/testing |
| `allow-invalid` | Allows Invalid routes to remain usable so route-map policy can lower preference instead of hard rejection |
| `prefix-validate disable` | Downloads RPKI data but disables validation use for the address family |
| `match rpki` | Route-map match condition based on RPKI state |
| RPKI state propagation | iBGP routers can carry validation state using an extended community |
| `send-community extended` | Required before announcing RPKI state to an iBGP neighbor |
| `announce rpki state` | Sends and receives RPKI validation state with an iBGP neighbor |
| Not eBGP state propagation | RPKI state extended community is for iBGP, not normal eBGP propagation |
| RPKI is not route filtering by itself | Validation state must be acted on by default behavior or policy |
| Blunt rule | RPKI does not prove the path is good. It only validates whether the origin AS is authorized to originate the prefix |
# BGP_RPKI_Origin_Validation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | External and internal BGP peers are established |
| 2 | Identify RPKI cache server | Notes | `<rpki-cache-ip>` | RPKI validator/cache IP address is known |
| 3 | Identify RTR TCP port | Notes | `<rpki-cache-port>` | RTR server port is known, commonly TCP 323 or lab-specific port |
| 4 | Identify refresh interval | Notes | `<refresh-seconds>` | Router refresh interval is known |
| 5 | Confirm router can reach RPKI cache | Router | `ping <rpki-cache-ip> source <source-ip>` | RPKI cache is reachable from router source |
| 6 | Confirm TCP path to RPKI cache | Router / Path | `show access-lists` or firewall check | TCP to RPKI cache port is not blocked |
| 7 | Confirm local BGP AS | Router | `show running-config | section router bgp` | Local BGP process is known |
| 8 | Confirm address family | Notes | `ipv4 unicast` or `ipv6 unicast` | Validation AFI is known |
| 9 | Confirm target route policy style | Notes | `reject-invalid`, `lower-invalid-localpref`, or `monitor-only` | RPKI enforcement behavior is chosen before configuration |
| 10 | Confirm whether iBGP routers will query cache directly | Notes | `direct-cache` or `iBGP-state-propagation` | Validation state distribution model is chosen |
| 11 | Review current BGP config | Router | `show running-config | section router bgp` | Current RPKI, neighbor, and route-map config is visible |
| 12 | Review current route maps | Router | `show running-config | section route-map` | Existing route-map logic is visible |
| 13 | Review current RPKI validation state in BGP table | Router | `show bgp ipv4 unicast` | RPKI code legend shows `V`, `I`, `N` if validation is active |
| 14 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 15 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 16 | Configure IPv4 RPKI cache server | Router | `bgp rpki server tcp <rpki-cache-ipv4> port <port> refresh <seconds>` | Router connects to RPKI cache and downloads prefix-origin data |
| 17 | Configure IPv6 RPKI cache server if used | Router | `bgp rpki server tcp <rpki-cache-ipv6> port <port> refresh <seconds>` | Router connects to IPv6 RPKI cache address |
| 18 | Configure second RPKI cache server for redundancy | Router | `bgp rpki server tcp <second-rpki-cache-ip> port <port> refresh <seconds>` | Router can download RPKI data from multiple caches |
| 19 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 20 | Keep default validation behavior if rejecting Invalid routes is desired | Router | `no bgp bestpath prefix-validate allow-invalid` | Invalid routes are not preferred or installed by default behavior |
| 21 | Allow Invalid routes only if route-map policy will downgrade instead of reject | Router | `bgp bestpath prefix-validate allow-invalid` | Invalid routes can remain candidates for policy treatment |
| 22 | Disable validation use only for testing or monitor-only download | Router | `bgp bestpath prefix-validate disable` | Router still connects to cache but does not use validation state |
| 23 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 24 | Enable extended communities to iBGP peer if propagating RPKI state | Router | `neighbor <ibgp-neighbor-ip> send-community extended` | Extended communities are sent to iBGP neighbor |
| 25 | Announce RPKI state to iBGP peer | Router | `neighbor <ibgp-neighbor-ip> announce rpki state` | RPKI state is sent and received through iBGP extended community |
| 26 | Use peer policy/template only if both commands live together | Router | `<peer-policy-template-with-send-community-and-announce-rpki>` | RPKI state announcement is inherited consistently |
| 27 | Create route map for RPKI invalid policy | Router | `route-map <rpki-rm-name> permit 10` | Route-map sequence exists |
| 28 | Match Invalid RPKI state | Router | `match rpki invalid` | Route map matches Invalid routes |
| 29 | Set low local preference for Invalid routes | Router | `set local-preference <low-value>` | Invalid routes are deprioritized |
| 30 | Create route map for Not Found policy | Router | `route-map <rpki-rm-name> permit 20` | Not Found sequence exists |
| 31 | Match Not Found RPKI state | Router | `match rpki not-found` | Route map matches Not Found routes |
| 32 | Set normal local preference for Not Found routes | Router | `set local-preference <normal-value>` | Not Found routes are usable but less preferred than Valid |
| 33 | Create route map for Valid policy | Router | `route-map <rpki-rm-name> permit 30` | Valid sequence exists |
| 34 | Match Valid RPKI state | Router | `match rpki valid` | Route map matches Valid routes |
| 35 | Set high local preference for Valid routes | Router | `set local-preference <high-value>` | Valid routes are preferred |
| 36 | Add permissive fallback sequence | Router | `route-map <rpki-rm-name> permit 40` | Unmatched routes are not accidentally denied |
| 37 | Enter BGP process again | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 38 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 39 | Apply RPKI route map inbound to eBGP peer | Router | `neighbor <ebgp-neighbor-ip> route-map <rpki-rm-name> in` | Inbound routes are treated based on RPKI state |
| 40 | Apply RPKI route map inbound to peer group if used | Router | `neighbor <peer-group-name> route-map <rpki-rm-name> in` | Peer group inherits RPKI policy |
| 41 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 42 | Save configuration | Router | `copy running-config startup-config` | RPKI configuration is saved |
| 43 | Soft clear inbound from eBGP peer | Router | `clear bgp ipv4 unicast <ebgp-neighbor-ip> soft in` | Routes are reprocessed with RPKI policy |
| 44 | Verify BGP RPKI server config | Router | `show running-config | include rpki|prefix-validate|announce rpki` | RPKI server and validation commands are visible |
| 45 | Verify BGP table validation codes | Router | `show bgp ipv4 unicast` | Routes show `V`, `I`, or `N` validation state where supported |
| 46 | Verify specific prefix validation state | Router | `show bgp ipv4 unicast <prefix>` | Prefix shows RPKI validation state |
| 47 | Verify Valid route preference | Router | `show bgp ipv4 unicast <valid-prefix>` | Valid route is accepted and preferred if competing states exist |
| 48 | Verify Not Found route behavior | Router | `show bgp ipv4 unicast <notfound-prefix>` | Not Found route is accepted unless policy denies it |
| 49 | Verify Invalid route behavior | Router | `show bgp ipv4 unicast <invalid-prefix>` | Invalid route is rejected, not best, or low-preference based on policy |
| 50 | Verify route-map counters | Router | `show route-map <rpki-rm-name>` | Valid, Invalid, and Not Found counters increment |
| 51 | Verify local route install | Router | `show ip route <prefix>` | Only accepted/selected route installs |
| 52 | Verify peer RPKI state announcement config | Router | `show running-config | section router bgp` | iBGP peer has `send-community extended` and `announce rpki state` |
| 53 | Verify iBGP peer sees validation state | iBGP Peer | `show bgp ipv4 unicast <prefix>` | iBGP peer receives RPKI state if announcement is configured |
| 54 | Verify no accidental eBGP state propagation | eBGP Peer | `show bgp ipv4 unicast <prefix>` | RPKI state extended community is not relied on for eBGP propagation |
| 55 | Verify logs | Router | `show logging | include RPKI|BGP|ADJCHANGE` | Cache/server/session or BGP policy events are visible if platform logs them |
# BGP_RPKI_Origin_Validation_RPKI_Server_Skeleton
configure terminal
router bgp <local-asn>
 bgp rpki server tcp <rpki-cache-ip> port <port> refresh <seconds>
end
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Dual_RPKI_Server_Skeleton
configure terminal
router bgp <local-asn>
 bgp rpki server tcp <rpki-cache-ip-1> port <port-1> refresh <seconds>
 bgp rpki server tcp <rpki-cache-ip-2> port <port-2> refresh <seconds>
end
copy running-config startup-config
# BGP_RPKI_Origin_Validation_iBGP_State_Announcement_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <ibgp-neighbor-ip> send-community extended
  neighbor <ibgp-neighbor-ip> announce rpki state
 exit-address-family
end
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Reject_Invalid_Default_Behavior_Skeleton
configure terminal
router bgp <local-asn>
 bgp rpki server tcp <rpki-cache-ip> port <port> refresh <seconds>
 address-family ipv4 unicast
  no bgp bestpath prefix-validate allow-invalid
 exit-address-family
end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Allow_Invalid_For_Route_Map_Policy_Skeleton
configure terminal
router bgp <local-asn>
 bgp rpki server tcp <rpki-cache-ip> port <port> refresh <seconds>
 address-family ipv4 unicast
  bgp bestpath prefix-validate allow-invalid
 exit-address-family
end
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Monitor_Only_Disable_Validation_Skeleton
configure terminal
router bgp <local-asn>
 bgp rpki server tcp <rpki-cache-ip> port <port> refresh <seconds>
 address-family ipv4 unicast
  bgp bestpath prefix-validate disable
 exit-address-family
end
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Route_Map_Local_Preference_Skeleton
configure terminal
route-map <rpki-rm-name> permit 10
 match rpki invalid
 set local-preference <invalid-low-local-pref>
route-map <rpki-rm-name> permit 20
 match rpki not-found
 set local-preference <notfound-normal-local-pref>
route-map <rpki-rm-name> permit 30
 match rpki valid
 set local-preference <valid-high-local-pref>
route-map <rpki-rm-name> permit 40
router bgp <local-asn>
 address-family ipv4 unicast
  bgp bestpath prefix-validate allow-invalid
  neighbor <ebgp-neighbor-ip> route-map <rpki-rm-name> in
 exit-address-family
end
clear bgp ipv4 unicast <ebgp-neighbor-ip> soft in
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Route_Map_Deny_Invalid_Skeleton
configure terminal
route-map <rpki-rm-name> deny 10
 match rpki invalid
route-map <rpki-rm-name> permit 20
 match rpki not-found
route-map <rpki-rm-name> permit 30
 match rpki valid
route-map <rpki-rm-name> permit 40
router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <ebgp-neighbor-ip> route-map <rpki-rm-name> in
 exit-address-family
end
clear bgp ipv4 unicast <ebgp-neighbor-ip> soft in
copy running-config startup-config
# BGP_RPKI_Origin_Validation_IPv6_Skeleton
configure terminal
router bgp <local-asn>
 bgp rpki server tcp <rpki-cache-ip> port <port> refresh <seconds>
 address-family ipv6 unicast
  bgp bestpath prefix-validate allow-invalid
  neighbor <ipv6-ebgp-neighbor> route-map <rpki-rm-name> in
 exit-address-family
end
clear bgp ipv6 unicast <ipv6-ebgp-neighbor> soft in
copy running-config startup-config
# BGP_RPKI_Origin_Validation_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast
show bgp ipv4 unicast <prefix>
show ip route <prefix>
show route-map <rpki-rm-name>
show running-config | section router bgp
show running-config | section route-map
show logging | include RPKI|BGP|ADJCHANGE
# BGP_RPKI_Origin_Validation_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP neighbor state | `show bgp ipv4 unicast summary` | BGP peers are established |
| Verify RPKI config | `show running-config | include rpki|prefix-validate` | RPKI server and validation behavior are configured |
| Verify iBGP RPKI state announcement | `show running-config | include send-community extended|announce rpki state` | iBGP peer is configured to carry RPKI state |
| Verify route-map config | `show running-config | section route-map` | `match rpki` and set/deny behavior is correct |
| Verify route-map hits | `show route-map <rpki-rm-name>` | Route-map counters increment for Valid, Invalid, and Not Found paths |
| Verify BGP table validation codes | `show bgp ipv4 unicast` | Output shows RPKI validation code legend and route states where supported |
| Verify specific prefix state | `show bgp ipv4 unicast <prefix>` | Prefix shows Valid, Invalid, or Not Found state |
| Verify Valid route behavior | `show bgp ipv4 unicast <valid-prefix>` | Valid prefix is accepted and preferred where competing states exist |
| Verify Not Found route behavior | `show bgp ipv4 unicast <notfound-prefix>` | Not Found prefix is accepted unless policy denies it |
| Verify Invalid route behavior | `show bgp ipv4 unicast <invalid-prefix>` | Invalid prefix is rejected, non-best, or low-preference based on policy |
| Verify route install | `show ip route <prefix>` | Only selected valid/accepted BGP route installs |
| Verify forwarding entry | `show ip cef <prefix>` | FIB points toward selected next hop |
| Verify peer received filtered result | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Invalid or downgraded routes are advertised or withheld according to policy |
| Verify iBGP peer sees state | `show bgp ipv4 unicast <prefix>` on iBGP peer | iBGP peer sees RPKI state if state announcement is configured |
| Verify logs | `show logging | include RPKI|BGP|ADJCHANGE` | Cache/session/policy events are visible if logged |
# BGP_RPKI_Origin_Validation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current RPKI config | Router | `show running-config | section router bgp` | RPKI server, prefix-validate, state announcement, and neighbor policy are visible |
| 2 | Identify RPKI route maps | Router | `show running-config | section route-map` | `match rpki` route maps are visible |
| 3 | Identify active validation behavior | Router | `show bgp ipv4 unicast <prefix>` | Current validation state behavior is known |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 7 | Remove inbound RPKI route map | Router | `no neighbor <ebgp-neighbor-ip> route-map <rpki-rm-name> in` | RPKI state route-map policy is detached |
| 8 | Remove allow-invalid if lab-only | Router | `no bgp bestpath prefix-validate allow-invalid` | Invalid routes return to default handling |
| 9 | Remove validation disable if lab-only | Router | `no bgp bestpath prefix-validate disable` | Validation use returns to normal if RPKI server remains configured |
| 10 | Remove RPKI state announcement to iBGP peer | Router | `no neighbor <ibgp-neighbor-ip> announce rpki state` | RPKI state is no longer announced to that iBGP peer |
| 11 | Remove extended community send only if lab-only | Router | `no neighbor <ibgp-neighbor-ip> send-community extended` | Extended community sending is removed if not needed elsewhere |
| 12 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 13 | Remove RPKI cache server | Router | `no bgp rpki server tcp <rpki-cache-ip> port <port> refresh <seconds>` | Router stops using that RPKI cache |
| 14 | Remove second RPKI cache server if lab-only | Router | `no bgp rpki server tcp <second-rpki-cache-ip> port <port> refresh <seconds>` | Redundant cache entry is removed |
| 15 | Remove route-map sequences if lab-only | Router | `no route-map <rpki-rm-name> permit <seq>` or `no route-map <rpki-rm-name> deny <seq>` | RPKI route-map sequences are removed |
| 16 | Soft clear inbound | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Routes are reprocessed after rollback |
| 17 | Verify RPKI config removal | Router | `show running-config | include rpki|prefix-validate` | Removed RPKI commands no longer appear |
| 18 | Verify route behavior after rollback | Router | `show bgp ipv4 unicast <prefix>` | Route behavior matches rollback design |
| 19 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_RPKI_Origin_Validation_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| No RPKI validation codes appear | `show running-config | include rpki|prefix-validate` | RPKI not configured, validation disabled, or platform output differs |
| Router cannot use RPKI data | `ping <rpki-cache-ip> source <source-ip>` | Cache server unreachable or TCP port blocked |
| Cache connection never comes up | Path ACL/firewall check | RTR TCP port, routing, source IP, firewall, or cache process is wrong |
| All prefixes show Not Found | `show bgp ipv4 unicast` | ROAs are missing, cache data is stale, wrong cache, or lab has no ROA data |
| Expected Valid route shows Invalid | `show bgp ipv4 unicast <prefix>` | ROA origin AS mismatch or advertised prefix is longer than ROA max length |
| Expected Invalid route still installs | `show running-config | include allow-invalid` | `bgp bestpath prefix-validate allow-invalid` is enabled or route-map permits it |
| Invalid route is rejected before route-map can lower local preference | `show running-config | include allow-invalid` | `allow-invalid` is missing when using policy-based downgrade |
| Route-map has no hits | `show route-map <rpki-rm-name>` | Neighbor route-map not attached, wrong direction, no matching RPKI state, or routes not refreshed |
| RPKI route-map denies too much | `show running-config | section route-map` | Missing permissive fallback sequence |
| Not Found route preferred over Valid route | `show bgp ipv4 unicast <prefix>` | Validation disabled, route-map overwrote preference, or states are not actually competing for same prefix |
| Valid route not preferred | `show bgp ipv4 unicast <prefix> best-path-reason` | Validation disabled or route-map changed attributes unexpectedly |
| iBGP peer does not see RPKI state | `show running-config | include send-community extended|announce rpki state` | Missing extended community send or missing `announce rpki state` |
| `announce rpki state` removed unexpectedly | `show running-config | section router bgp` | Removing `send-community extended` also removed RPKI state announcement |
| eBGP peer does not receive RPKI state community | External peer check | Expected behavior; RPKI state extended community is not sent to eBGP peers |
| RPKI cache fails but BGP still accepts routes | Logs and `show running-config` | Router behaves without origin validation when cache connection is unavailable |
| Validation testing breaks production reachability | `show bgp ipv4 unicast <prefix>` | Invalid routes were rejected without staged monitor/downgrade policy |
| Soft clear does not update policy | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Wrong neighbor or wrong direction refreshed |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Hard reset used instead of soft inbound refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-rpki-final` | `BGP_RPKI_Origin_Validation.md` | Primary lab for RPKI origin validation, Valid/Invalid/Not Found states, `match rpki` policy, RPKI cache connectivity, and iBGP state propagation |
# Index
# Source_Basis
# BGP_RPKI_Origin_Validation_Mental_Model
# BGP_RPKI_Origin_Validation_Configuration_Checklist
# BGP_RPKI_Origin_Validation_RPKI_Server_Skeleton
# BGP_RPKI_Origin_Validation_Dual_RPKI_Server_Skeleton
# BGP_RPKI_Origin_Validation_iBGP_State_Announcement_Skeleton
# BGP_RPKI_Origin_Validation_Reject_Invalid_Default_Behavior_Skeleton
# BGP_RPKI_Origin_Validation_Allow_Invalid_For_Route_Map_Policy_Skeleton
# BGP_RPKI_Origin_Validation_Monitor_Only_Disable_Validation_Skeleton
# BGP_RPKI_Origin_Validation_Route_Map_Local_Preference_Skeleton
# BGP_RPKI_Origin_Validation_Route_Map_Deny_Invalid_Skeleton
# BGP_RPKI_Origin_Validation_IPv6_Skeleton
# BGP_RPKI_Origin_Validation_Verification_Skeleton
# BGP_RPKI_Origin_Validation_Verification_Commands
# BGP_RPKI_Origin_Validation_Rollback
# BGP_RPKI_Origin_Validation_Failure_Checks
# Attached_Labs

Blunt rule: RPKI validates origin authorization, not full path legitimacy. A Valid route can still take a bad path, and a Not Found route is not automatically malicious. Invalid is the one you treat harshly, but stage that policy before you break reachability.