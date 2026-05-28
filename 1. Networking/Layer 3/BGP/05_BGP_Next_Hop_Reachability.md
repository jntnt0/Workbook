

```
# Index
# Source_Basis
# BGP_Next_Hop_Reachability_Mental_Model
# BGP_Next_Hop_Reachability_Configuration_Checklist
# BGP_Next_Hop_Reachability_Next_Hop_Self_Skeleton
# BGP_Next_Hop_Reachability_Next_Hop_Self_Peer_Group_Skeleton
# BGP_Next_Hop_Reachability_Advertise_Next_Hop_Into_OSPF_Skeleton
# BGP_Next_Hop_Reachability_Static_Route_To_Next_Hop_Skeleton
# BGP_Next_Hop_Reachability_eBGP_To_iBGP_Baseline_Skeleton
# BGP_Next_Hop_Reachability_Verification_Skeleton
# BGP_Next_Hop_Reachability_Verification_Commands
# BGP_Next_Hop_Reachability_Rollback
# BGP_Next_Hop_Reachability_Failure_Checks
# Attached_Labs
```

# BGP_Next_Hop_Reachability_Mental_Model
| Concept                | Operational Meaning                                                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| BGP next hop           | The IP address the router must reach to forward traffic for a BGP prefix                                                             |
| BGP table visibility   | A route can appear in `show bgp ipv4 unicast` without being usable for forwarding                                                    |
| Valid path             | A BGP path must have a reachable next hop before it can become best                                                                  |
| Best path              | Marked with `>` in the BGP table                                                                                                     |
| Installed route        | Route appears in the routing table after BGP selects it and the RIB accepts it                                                       |
| `*` marker             | BGP path is valid                                                                                                                    |
| `>` marker             | BGP selected the path as best                                                                                                        |
| Missing `>`            | Path is not selected as best, often because next hop is unreachable or another path wins                                             |
| RIB failure            | BGP selected a route but could not install it in the routing table                                                                   |
| eBGP next-hop behavior | eBGP usually changes next hop to the advertising eBGP router                                                                         |
| iBGP next-hop behavior | iBGP usually preserves the next hop learned from eBGP                                                                                |
| eBGP-to-iBGP problem   | Internal routers may learn an external prefix with a next hop they cannot reach                                                      |
| `next-hop-self`        | Advertising router rewrites next hop to itself before sending route to peer                                                          |
| IGP reachability fix   | Advertise the external next-hop address or transit subnet into the IGP                                                               |
| Static route fix       | Add a route to the BGP next-hop address when appropriate in labs or simple designs                                                   |
| Recursive lookup       | Router looks up the BGP next hop in the routing table before forwarding                                                              |
| IGP metric to next hop | If multiple BGP paths tie through earlier attributes, lower IGP cost to next hop can decide best path                                |
| Blunt rule             | If the BGP route is visible but not installed, check next-hop reachability before touching weight, local preference, MED, or AS path |
|                        |                                                                                                                                      |
# BGP_Next_Hop_Reachability_Configuration_Checklist
| Step | Task                                                       | Device             | Command                                                                                     | Expected Result                                                                  |             |                                          |                  |                               |
| ---: | ---------------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | ----------- | ---------------------------------------- | ---------------- | ----------------------------- |
|    1 | Confirm BGP neighbor is established                        | Router             | `show bgp ipv4 unicast summary`                                                             | Neighbor shows a number under `State/PfxRcd`                                     |             |                                          |                  |                               |
|    2 | Confirm route appears in BGP table                         | Router             | `show bgp ipv4 unicast`                                                                     | Prefix is visible in BGP table                                                   |             |                                          |                  |                               |
|    3 | Confirm specific BGP prefix details                        | Router             | `show bgp ipv4 unicast <prefix>`                                                            | Next hop, AS path, local preference, weight, and best-path state are visible     |             |                                          |                  |                               |
|    4 | Identify the BGP next hop                                  | Router             | `show bgp ipv4 unicast <prefix>`                                                            | Next-hop IP is known                                                             |             |                                          |                  |                               |
|    5 | Check whether path is valid                                | Router             | `show bgp ipv4 unicast <prefix>`                                                            | Path should show `*` if valid                                                    |             |                                          |                  |                               |
|    6 | Check whether path is best                                 | Router             | `show bgp ipv4 unicast <prefix>`                                                            | Best path should show `>`                                                        |             |                                          |                  |                               |
|    7 | Check routing table install                                | Router             | `show ip route <prefix>`                                                                    | Prefix appears in RIB if BGP path is selected and installable                    |             |                                          |                  |                               |
|    8 | Check route to BGP next hop                                | Router             | `show ip route <next-hop-ip>`                                                               | Router has a route to the next-hop IP                                            |             |                                          |                  |                               |
|    9 | Check CEF forwarding to next hop                           | Router             | `show ip cef <next-hop-ip>`                                                                 | CEF has forwarding entry to next hop                                             |             |                                          |                  |                               |
|   10 | Check sourced reachability to next hop                     | Router             | `ping <next-hop-ip> source <local-source-ip>`                                               | Ping succeeds from expected local source                                         |             |                                          |                  |                               |
|   11 | Check traceroute to next hop                               | Router             | `traceroute <next-hop-ip> source <local-source-ip>`                                         | Path toward next hop is visible                                                  |             |                                          |                  |                               |
|   12 | Confirm whether next hop is external or internal           | Router / Notes     | `<next-hop-ip location>`                                                                    | You know whether next hop belongs to eBGP edge, iBGP loopback, or transit subnet |             |                                          |                  |                               |
|   13 | Confirm eBGP-to-iBGP next-hop preservation issue           | iBGP Router        | `show bgp ipv4 unicast <prefix>`                                                            | iBGP peer sees external next hop it cannot reach                                 |             |                                          |                  |                               |
|   14 | Confirm advertising router toward iBGP peer                | Edge Router        | `show bgp ipv4 unicast neighbors <ibgp-peer-ip> advertised-routes`                          | Edge router advertises prefix to iBGP peer                                       |             |                                          |                  |                               |
|   15 | Enter configuration mode on advertising edge router        | Edge Router        | `configure terminal`                                                                        | Router enters global config mode                                                 |             |                                          |                  |                               |
|   16 | Enter BGP process                                          | Edge Router        | `router bgp <local-asn>`                                                                    | Router enters BGP config mode                                                    |             |                                          |                  |                               |
|   17 | Enter IPv4 unicast address family                          | Edge Router        | `address-family ipv4 unicast`                                                               | Router enters IPv4 unicast AFI                                                   |             |                                          |                  |                               |
|   18 | Configure next-hop-self toward iBGP peer                   | Edge Router        | `neighbor <ibgp-peer-ip> next-hop-self`                                                     | Edge router rewrites next hop to itself for routes sent to iBGP peer             |             |                                          |                  |                               |
|   19 | Configure next-hop-self toward peer group if used          | Edge Router        | `neighbor <peer-group-name> next-hop-self`                                                  | All iBGP peers in group receive reachable next hop                               |             |                                          |                  |                               |
|   20 | Exit address-family mode                                   | Edge Router        | `exit-address-family`                                                                       | Router returns to BGP config mode                                                |             |                                          |                  |                               |
|   21 | Save configuration                                         | Edge Router        | `copy running-config startup-config`                                                        | Configuration is saved                                                           |             |                                          |                  |                               |
|   22 | Soft clear outbound if policy/session needs refresh        | Edge Router        | `clear bgp ipv4 unicast <ibgp-peer-ip> soft out`                                            | Routes are re-advertised without hard reset                                      |             |                                          |                  |                               |
|   23 | Verify next hop changed on iBGP peer                       | iBGP Router        | `show bgp ipv4 unicast <prefix>`                                                            | Next hop is now the advertising edge router or configured source                 |             |                                          |                  |                               |
|   24 | Verify next-hop route exists                               | iBGP Router        | `show ip route <new-next-hop-ip>`                                                           | iBGP router can route to the new next hop                                        |             |                                          |                  |                               |
|   25 | Verify route installs in RIB                               | iBGP Router        | `show ip route <prefix>`                                                                    | BGP route installs if no better route blocks it                                  |             |                                          |                  |                               |
|   26 | Verify forwarding entry                                    | iBGP Router        | `show ip cef <prefix>`                                                                      | CEF points toward the reachable next hop                                         |             |                                          |                  |                               |
|   27 | Alternative fix: advertise next-hop network into IGP       | Edge / Core Router | `router ospf <process-id>` then `network <next-hop-network> <wildcard-mask> area <area-id>` | IGP carries route to the original next-hop network                               |             |                                          |                  |                               |
|   28 | Alternative fix: add static route to next hop              | iBGP Router        | `ip route <next-hop-ip> 255.255.255.255 <reachable-next-hop>`                               | Router can resolve original BGP next hop                                         |             |                                          |                  |                               |
|   29 | Verify original next-hop reachability after IGP/static fix | iBGP Router        | `show ip route <original-next-hop-ip>`                                                      | Original next hop is reachable                                                   |             |                                          |                  |                               |
|   30 | Verify BGP best path after reachability fix                | iBGP Router        | `show bgp ipv4 unicast <prefix>`                                                            | Path shows valid and best if it wins                                             |             |                                          |                  |                               |
|   31 | Verify route table after reachability fix                  | iBGP Router        | `show ip route <prefix>`                                                                    | Prefix appears as BGP route if selected                                          |             |                                          |                  |                               |
|   32 | Verify data-plane reachability                             | iBGP Router        | `ping <destination-ip> source <local-source-ip>`                                            | Destination is reachable if return path also works                               |             |                                          |                  |                               |
|   33 | Verify return path                                         | Remote Router      | `show ip route <local-source-ip>`                                                           | Remote side has route back to source                                             |             |                                          |                  |                               |
|   34 | Verify no route policy is hiding the route                 | Router             | `show running-config                                                                        | include route-map                                                                | prefix-list | filter-list                              | distribute-list` | Policy filters are identified |
|   35 | Verify logs                                                | Router             | `show logging                                                                               | include BGP                                                                      | ADJCHANGE`  | BGP session or update events are visible |                  |                               |

```
# BGP_Next_Hop_Reachability_Next_Hop_Self_Skeleton
configure terminal

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <ibgp-peer-ip> next-hop-self
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Next_Hop_Reachability_Next_Hop_Self_Peer_Group_Skeleton
configure terminal

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <peer-group-name> next-hop-self
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Next_Hop_Reachability_Advertise_Next_Hop_Into_OSPF_Skeleton
configure terminal

router ospf <process-id>
 network <next-hop-transit-network> <wildcard-mask> area <area-id>

end
copy running-config startup-config
```

```
# BGP_Next_Hop_Reachability_Static_Route_To_Next_Hop_Skeleton
configure terminal

ip route <bgp-next-hop-ip> 255.255.255.255 <reachable-next-hop-ip>

end
copy running-config startup-config
```

```
# BGP_Next_Hop_Reachability_eBGP_To_iBGP_Baseline_Skeleton
configure terminal

router bgp <local-asn>
 no bgp default ipv4-unicast
 neighbor <ebgp-peer-ip> remote-as <external-asn>
 neighbor <ibgp-peer-ip> remote-as <local-asn>

 address-family ipv4 unicast
  neighbor <ebgp-peer-ip> activate
  neighbor <ibgp-peer-ip> activate
  neighbor <ibgp-peer-ip> next-hop-self
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Next_Hop_Reachability_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast
show bgp ipv4 unicast <prefix>
show ip route <next-hop-ip>
ping <next-hop-ip> source <local-source-ip>
show ip route <prefix>
show ip cef <prefix>
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show logging | include BGP|ADJCHANGE
```


# BGP_Next_Hop_Reachability_Verification_Commands
| Task                                 | Command                                                           | Expected Result                                                  |                                                            |                        |                  |                                       |
| ------------------------------------ | ----------------------------------------------------------------- | ---------------------------------------------------------------- | ---------------------------------------------------------- | ---------------------- | ---------------- | ------------------------------------- |
| Verify BGP neighbor state            | `show bgp ipv4 unicast summary`                                   | Neighbor shows a number under `State/PfxRcd`                     |                                                            |                        |                  |                                       |
| Verify BGP route visibility          | `show bgp ipv4 unicast`                                           | Prefix appears in BGP table                                      |                                                            |                        |                  |                                       |
| Verify specific prefix               | `show bgp ipv4 unicast <prefix>`                                  | Next hop, AS path, best path, and validity are visible           |                                                            |                        |                  |                                       |
| Verify valid marker                  | `show bgp ipv4 unicast <prefix>`                                  | Path has `*` if valid                                            |                                                            |                        |                  |                                       |
| Verify best marker                   | `show bgp ipv4 unicast <prefix>`                                  | Path has `>` if selected as best                                 |                                                            |                        |                  |                                       |
| Identify BGP next hop                | `show bgp ipv4 unicast <prefix>`                                  | Next-hop IP is visible                                           |                                                            |                        |                  |                                       |
| Verify next-hop route                | `show ip route <next-hop-ip>`                                     | Router has a route to the BGP next hop                           |                                                            |                        |                  |                                       |
| Verify next-hop CEF entry            | `show ip cef <next-hop-ip>`                                       | CEF can forward toward next hop                                  |                                                            |                        |                  |                                       |
| Verify sourced next-hop reachability | `ping <next-hop-ip> source <local-source-ip>`                     | Ping succeeds from expected source                               |                                                            |                        |                  |                                       |
| Verify path to next hop              | `traceroute <next-hop-ip> source <local-source-ip>`               | Path toward next hop is visible                                  |                                                            |                        |                  |                                       |
| Verify route installed in RIB        | `show ip route <prefix>`                                          | BGP route installs if selected and valid                         |                                                            |                        |                  |                                       |
| Verify CEF to destination prefix     | `show ip cef <prefix>`                                            | CEF points to expected next hop/interface                        |                                                            |                        |                  |                                       |
| Verify advertised next hop to peer   | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Prefix is advertised with expected next-hop behavior             |                                                            |                        |                  |                                       |
| Verify neighbor detail               | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Neighbor state, source, capabilities, and AFI status are visible |                                                            |                        |                  |                                       |
| Verify next-hop-self config          | `show running-config                                              | section router bgp`                                              | `neighbor <ip> next-hop-self` appears under address-family |                        |                  |                                       |
| Verify IGP route to next hop         | `show ip route <next-hop-ip>`                                     | IGP or static route resolves next hop                            |                                                            |                        |                  |                                       |
| Verify route policy                  | `show running-config                                              | include route-map                                                | prefix-list                                                | filter-list            | distribute-list` | Route filters/policies are identified |
| Verify logs                          | `show logging                                                     | include BGP                                                      | ADJCHANGE`                                                 | BGP events are visible |                  |                                       |


# BGP_Next_Hop_Reachability_Rollback
| Step | Task                                             | Device | Command                                                                                                | Expected Result                                            |                                                        |
| ---: | ------------------------------------------------ | ------ | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------ |
|    1 | Identify BGP next-hop config                     | Router | `show running-config                                                                                   | section router bgp`                                        | `next-hop-self`, neighbors, and AFI config are visible |
|    2 | Identify static next-hop route if used           | Router | `show running-config                                                                                   | include ^ip route <bgp-next-hop-ip>`                       | Static route to next hop is visible                    |
|    3 | Identify IGP advertisement if used               | Router | `show running-config                                                                                   | section router ospf`                                       | OSPF network statement for next-hop network is visible |
|    4 | Enter configuration mode                         | Router | `configure terminal`                                                                                   | Router enters global configuration mode                    |                                                        |
|    5 | Enter BGP process                                | Router | `router bgp <local-asn>`                                                                               | Router enters BGP config mode                              |                                                        |
|    6 | Enter IPv4 unicast AFI                           | Router | `address-family ipv4 unicast`                                                                          | Router enters IPv4 unicast address-family                  |                                                        |
|    7 | Remove next-hop-self from specific peer          | Router | `no neighbor <ibgp-peer-ip> next-hop-self`                                                             | Next-hop rewrite is removed                                |                                                        |
|    8 | Remove next-hop-self from peer group if used     | Router | `no neighbor <peer-group-name> next-hop-self`                                                          | Peer-group next-hop rewrite is removed                     |                                                        |
|    9 | Exit AFI mode                                    | Router | `exit-address-family`                                                                                  | Router returns to BGP config mode                          |                                                        |
|   10 | Remove static route to next hop if lab-only      | Router | `no ip route <bgp-next-hop-ip> 255.255.255.255 <reachable-next-hop-ip>`                                | Static next-hop reachability route is removed              |                                                        |
|   11 | Remove OSPF advertisement if lab-only            | Router | `router ospf <process-id>` then `no network <next-hop-transit-network> <wildcard-mask> area <area-id>` | IGP no longer advertises that next-hop network             |                                                        |
|   12 | Soft clear outbound if next-hop-self was removed | Router | `clear bgp ipv4 unicast <ibgp-peer-ip> soft out`                                                       | Updated next-hop behavior is advertised without hard reset |                                                        |
|   13 | Verify next-hop behavior after rollback          | Router | `show bgp ipv4 unicast <prefix>`                                                                       | Prefix shows expected original next-hop behavior           |                                                        |
|   14 | Verify route install behavior                    | Router | `show ip route <prefix>`                                                                               | Route install state matches rollback design                |                                                        |
|   15 | Save rollback                                    | Router | `copy running-config startup-config`                                                                   | Rollback is saved                                          |                                                        |


# BGP_Next_Hop_Reachability_Failure_Checks
| Symptom                                                      | Command                                                            | What Usually Broke                                                     |                                                                                     |                                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------------ | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Prefix appears in BGP but not routing table                  | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>`      | BGP next hop is unreachable, better AD route exists, or RIB failure    |                                                                                     |                                                                              |
| BGP path has `*` but no `>`                                  | `show bgp ipv4 unicast <prefix>`                                   | Path is valid but not selected as best                                 |                                                                                     |                                                                              |
| BGP path has no valid marker                                 | `show bgp ipv4 unicast <prefix>`                                   | Next hop is unreachable or path fails validity check                   |                                                                                     |                                                                              |
| iBGP peer learns eBGP route but cannot install it            | `show bgp ipv4 unicast <prefix>`                                   | eBGP next hop was preserved and is unreachable from the iBGP peer      |                                                                                     |                                                                              |
| `next-hop-self` has no effect                                | `show running-config                                               | section router bgp`                                                    | Command is missing under correct address family, wrong peer, or route not refreshed |                                                                              |
| Next hop still shows external peer after fix                 | `show bgp ipv4 unicast <prefix>`                                   | `next-hop-self` configured on wrong router or wrong neighbor direction |                                                                                     |                                                                              |
| Route to next hop missing                                    | `show ip route <next-hop-ip>`                                      | IGP/static route to next hop is missing                                |                                                                                     |                                                                              |
| Ping to next hop fails                                       | `ping <next-hop-ip> source <local-source-ip>`                      | Underlay reachability, ACL, source route, or return path is broken     |                                                                                     |                                                                              |
| Static route to next hop works only temporarily              | `show ip route <next-hop-ip>`                                      | Lab shortcut hides underlay design issue                               |                                                                                     |                                                                              |
| IGP route exists but CEF forwarding fails                    | `show ip cef <next-hop-ip>`                                        | CEF adjacency, ARP, interface, or recursive lookup issue               |                                                                                     |                                                                              |
| Route installs after `next-hop-self` but traffic still fails | `show ip cef <prefix>` and return-path checks                      | Forward path works but return path is missing                          |                                                                                     |                                                                              |
| Remote side cannot reply                                     | Remote router `show ip route <source-ip>`                          | Return route to source is missing                                      |                                                                                     |                                                                              |
| Multiple BGP paths tie until IGP cost decides                | `show bgp ipv4 unicast <prefix>` and `show ip route <next-hop-ip>` | Lowest IGP metric to next hop decides after earlier attributes tie     |                                                                                     |                                                                              |
| Route policy hides next-hop issue                            | `show running-config                                               | include route-map                                                      | prefix-list`                                                                        | Policy changes route visibility, making next-hop problem look like filtering |
| Soft clear not done after config change                      | `show bgp ipv4 unicast <prefix>`                                   | Peer still has old advertised next-hop behavior                        |                                                                                     |                                                                              |
| Hard clear caused avoidable outage                           | `show logging                                                      | include BGP`                                                           | Hard reset used when soft outbound refresh was enough                               |                                                                              |

```
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-next-hop-final` | `BGP_Next_Hop_Reachability.md` | Primary lab for seeing BGP routes fail to install because the next hop is unreachable |
| `bgp-ebgp-over-ibgp-final` | `BGP_Next_Hop_Reachability.md` | Primary lab for eBGP-learned routes carried across iBGP where `next-hop-self` or IGP next-hop reachability is required |
```