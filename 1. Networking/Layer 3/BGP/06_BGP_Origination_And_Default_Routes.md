
```
# Index
# Source_Basis
# BGP_Origination_And_Default_Routes_Mental_Model
# BGP_Origination_And_Default_Routes_Configuration_Checklist
# BGP_Origination_And_Default_Routes_Network_Statement_Skeleton
# BGP_Origination_And_Default_Routes_Static_Route_Plus_Network_Skeleton
# BGP_Origination_And_Default_Routes_Default_With_Network_Skeleton
# BGP_Origination_And_Default_Routes_Neighbor_Default_Originate_Skeleton
# BGP_Origination_And_Default_Routes_Conditional_Default_Originate_Skeleton
# BGP_Origination_And_Default_Routes_Redistribute_Static_Controlled_Skeleton
# BGP_Origination_And_Default_Routes_Redistribute_Connected_Controlled_Skeleton
# BGP_Origination_And_Default_Routes_Verification_Skeleton
# BGP_Origination_And_Default_Routes_Verification_Commands
# BGP_Origination_And_Default_Routes_Rollback
# BGP_Origination_And_Default_Routes_Failure_Checks
# Attached_Labs
```

# BGP_Origination_And_Default_Routes_Mental_Model
| Concept                                  | Operational Meaning                                                                                                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BGP origination                          | The act of injecting a route into the local BGP Loc-RIB                                                                                                             |
| `network` statement                      | Tells BGP to originate an exact route that already exists in the routing table                                                                                      |
| Exact RIB match                          | The BGP `network <network> mask <mask>` command does nothing unless that exact prefix and mask exist in the RIB                                                     |
| `network` is not interface activation    | Unlike OSPF or EIGRP, BGP `network` does not enable BGP on an interface                                                                                             |
| Locally originated route                 | A route originated by the local BGP process                                                                                                                         |
| Local next hop `0.0.0.0`                 | BGP table marker showing the route was originated locally on that router                                                                                            |
| Local weight `32768`                     | Cisco default weight for locally originated BGP routes                                                                                                              |
| Origin code `i`                          | Usually means the route was originated with a BGP `network` statement                                                                                               |
| Origin code `?`                          | Usually means the route was redistributed into BGP                                                                                                                  |
| Redistribution into BGP                  | Imports routes from another source, such as static, connected, OSPF, or EIGRP                                                                                       |
| Redistribution risk                      | Can inject too many routes or the wrong routes unless controlled with route maps                                                                                    |
| Default route                            | `0.0.0.0/0`, used when no more specific route exists                                                                                                                |
| Default via `network`                    | Advertises `0.0.0.0/0` only if an exact default route exists in the local RIB                                                                                       |
| Default via `neighbor default-originate` | Sends default route to a specific neighbor or peer group                                                                                                            |
| Conditional default                      | Uses route map logic so the default route is advertised only when required conditions are true                                                                      |
| Static default route                     | Often used as the RIB object that lets BGP originate `0.0.0.0/0`                                                                                                    |
| `Null0` route                            | Can create an exact local RIB route for controlled origination or summaries                                                                                         |
| Locally originated path preference       | Local origination is a BGP best-path step after weight and local preference                                                                                         |
| Blunt rule                               | BGP does not advertise what you wish existed. It advertises what the BGP process is explicitly told to originate and what survives RIB, policy, and next-hop checks |
|                                          |                                                                                                                                                                     |

# BGP_Origination_And_Default_Routes_Configuration_Checklist
| Step | Task                                                                                   | Device         | Command                                                           | Expected Result                                                     |                                                                             |              |                                      |
| ---: | -------------------------------------------------------------------------------------- | -------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------- | ------------ | ------------------------------------ |
|    1 | Confirm BGP neighbor baseline                                                          | Router         | `show bgp ipv4 unicast summary`                                   | Neighbor shows a number under `State/PfxRcd`                        |                                                                             |              |                                      |
|    2 | Confirm local AS                                                                       | Router / Notes | `<local-asn>`                                                     | Local BGP AS is known                                               |                                                                             |              |                                      |
|    3 | Confirm target prefix to originate                                                     | Router / Notes | `<prefix> <mask>`                                                 | Prefix and exact mask are known                                     |                                                                             |              |                                      |
|    4 | Confirm whether prefix should be normal route or default route                         | Router / Notes | `<normal-prefix>` or `0.0.0.0/0`                                  | Origination type is clear                                           |                                                                             |              |                                      |
|    5 | Confirm whether route source should be `network`, redistribution, or default-originate | Router / Notes | `<origination-method>`                                            | Method is chosen deliberately                                       |                                                                             |              |                                      |
|    6 | Confirm exact RIB route exists for `network` method                                    | Router         | `show ip route <prefix> <mask>`                                   | Exact prefix and mask exist in the routing table                    |                                                                             |              |                                      |
|    7 | Create static route if exact RIB route is needed for origination                       | Router         | `ip route <prefix> <mask> <next-hop-or-null0>`                    | Exact route exists in RIB                                           |                                                                             |              |                                      |
|    8 | Create static default route if default must exist locally                              | Router         | `ip route 0.0.0.0 0.0.0.0 <next-hop-ip>`                          | Default route exists in RIB                                         |                                                                             |              |                                      |
|    9 | Confirm default route exists before advertising it with `network`                      | Router         | `show ip route 0.0.0.0 0.0.0.0`                                   | Exact `0.0.0.0/0` route exists                                      |                                                                             |              |                                      |
|   10 | Review current BGP config                                                              | Router         | `show running-config                                              | section router bgp`                                                 | Existing neighbors, AFI, network statements, and redistribution are visible |              |                                      |
|   11 | Review existing route maps                                                             | Router         | `show running-config                                              | section route-map`                                                  | Route maps are visible                                                      |              |                                      |
|   12 | Review existing prefix lists                                                           | Router         | `show running-config                                              | include ip prefix-list`                                             | Prefix lists are visible                                                    |              |                                      |
|   13 | Enter configuration mode                                                               | Router         | `configure terminal`                                              | Router enters global configuration mode                             |                                                                             |              |                                      |
|   14 | Enter BGP process                                                                      | Router         | `router bgp <local-asn>`                                          | Router enters BGP configuration mode                                |                                                                             |              |                                      |
|   15 | Standardize explicit AF activation if used                                             | Router         | `no bgp default ipv4-unicast`                                     | Neighbors must be activated under address-family                    |                                                                             |              |                                      |
|   16 | Enter IPv4 unicast address family                                                      | Router         | `address-family ipv4 unicast`                                     | Router enters IPv4 unicast AFI                                      |                                                                             |              |                                      |
|   17 | Activate neighbor if not already active                                                | Router         | `neighbor <neighbor-ip> activate`                                 | Neighbor can exchange IPv4 unicast NLRI                             |                                                                             |              |                                      |
|   18 | Originate normal prefix with `network`                                                 | Router         | `network <prefix> mask <mask>`                                    | Exact RIB prefix is injected into BGP                               |                                                                             |              |                                      |
|   19 | Originate default route with `network` if default exists in RIB                        | Router         | `network 0.0.0.0 mask 0.0.0.0`                                    | BGP originates `0.0.0.0/0` if exact default exists                  |                                                                             |              |                                      |
|   20 | Create prefix list for controlled redistribution                                       | Router         | `ip prefix-list <pl-name> permit <prefix>/<length>`               | Only intended prefix is matched                                     |                                                                             |              |                                      |
|   21 | Create route map for controlled redistribution                                         | Router         | `route-map <rm-name> permit <seq>`                                | Route map sequence exists                                           |                                                                             |              |                                      |
|   22 | Match prefix list in route map                                                         | Router         | `match ip address prefix-list <pl-name>`                          | Route map matches only intended routes                              |                                                                             |              |                                      |
|   23 | Set optional BGP attributes during redistribution                                      | Router         | `set metric <med>` or `set community <community>`                 | Redistributed routes receive intended attributes                    |                                                                             |              |                                      |
|   24 | Redistribute static routes into BGP with route map                                     | Router         | `redistribute static route-map <rm-name>`                         | Only matched static routes are injected into BGP                    |                                                                             |              |                                      |
|   25 | Redistribute connected routes into BGP with route map                                  | Router         | `redistribute connected route-map <rm-name>`                      | Only matched connected routes are injected into BGP                 |                                                                             |              |                                      |
|   26 | Avoid uncontrolled redistribution                                                      | Router / Notes | `No bare redistribute unless lab explicitly requires it`          | BGP does not accidentally ingest too many routes                    |                                                                             |              |                                      |
|   27 | Configure neighbor-specific default advertisement                                      | Router         | `neighbor <neighbor-ip> default-originate`                        | Default route is advertised to that neighbor                        |                                                                             |              |                                      |
|   28 | Configure conditional neighbor default advertisement                                   | Router         | `neighbor <neighbor-ip> default-originate route-map <rm-name>`    | Default route is advertised only when route-map condition allows it |                                                                             |              |                                      |
|   29 | Configure peer-group default advertisement if used                                     | Router         | `neighbor <peer-group-name> default-originate`                    | Default route is advertised to peer-group members                   |                                                                             |              |                                      |
|   30 | Exit address-family mode                                                               | Router         | `exit-address-family`                                             | Router returns to BGP config mode                                   |                                                                             |              |                                      |
|   31 | Save configuration                                                                     | Router         | `copy running-config startup-config`                              | Origination configuration is saved                                  |                                                                             |              |                                      |
|   32 | Verify BGP table                                                                       | Router         | `show bgp ipv4 unicast`                                           | Locally originated prefix appears                                   |                                                                             |              |                                      |
|   33 | Verify local origination marker                                                        | Router         | `show bgp ipv4 unicast <prefix>`                                  | Locally originated route shows next hop `0.0.0.0` where applicable  |                                                                             |              |                                      |
|   34 | Verify origin code for `network` route                                                 | Router         | `show bgp ipv4 unicast <prefix>`                                  | Origin code is `i`                                                  |                                                                             |              |                                      |
|   35 | Verify origin code for redistributed route                                             | Router         | `show bgp ipv4 unicast <prefix>`                                  | Origin code is usually `?`                                          |                                                                             |              |                                      |
|   36 | Verify local weight                                                                    | Router         | `show bgp ipv4 unicast <prefix>`                                  | Local originated route usually shows weight `32768`                 |                                                                             |              |                                      |
|   37 | Verify advertised routes                                                               | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Intended prefix or default route is advertised                      |                                                                             |              |                                      |
|   38 | Verify peer receives route                                                             | Peer Router    | `show bgp ipv4 unicast <prefix>`                                  | Peer sees the advertised prefix                                     |                                                                             |              |                                      |
|   39 | Verify peer receives default route                                                     | Peer Router    | `show bgp ipv4 unicast 0.0.0.0/0`                                 | Peer sees default route if advertised                               |                                                                             |              |                                      |
|   40 | Verify peer installs route                                                             | Peer Router    | `show ip route <prefix>`                                          | Route installs if valid and preferred                               |                                                                             |              |                                      |
|   41 | Verify peer installs default route                                                     | Peer Router    | `show ip route 0.0.0.0`                                           | Default route installs if valid and preferred                       |                                                                             |              |                                      |
|   42 | Verify next-hop reachability on peer                                                   | Peer Router    | `show ip route <next-hop-ip>`                                     | Peer can reach BGP next hop                                         |                                                                             |              |                                      |
|   43 | Verify traffic follows expected default path                                           | Peer Router    | `traceroute <external-ip> source <source-ip>`                     | Traffic follows intended default route path                         |                                                                             |              |                                      |
|   44 | Verify route policy did not block advertisement                                        | Router         | `show running-config                                              | include route-map                                                   | prefix-list                                                                 | filter-list` | Policy does not deny intended prefix |
|   45 | Soft clear outbound after policy-only change                                           | Router         | `clear bgp ipv4 unicast <neighbor-ip> soft out`                   | Advertisement updates without hard reset                            |                                                                             |              |                                      |
|      |                                                                                        |                |                                                                   |                                                                     |                                                                             |              |                                      |
|      |                                                                                        |                |                                                                   |                                                                     |                                                                             |              |                                      |


```
# BGP_Origination_And_Default_Routes_Network_Statement_Skeleton
configure terminal

router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast

 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
  network <prefix> mask <mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Origination_And_Default_Routes_Static_Route_Plus_Network_Skeleton
configure terminal

ip route <prefix> <mask> <next-hop-ip-or-null0>

router bgp <local-asn>
 address-family ipv4 unicast
  network <prefix> mask <mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Origination_And_Default_Routes_Default_With_Network_Skeleton
configure terminal

ip route 0.0.0.0 0.0.0.0 <next-hop-ip>

router bgp <local-asn>
 address-family ipv4 unicast
  network 0.0.0.0 mask 0.0.0.0
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Origination_And_Default_Routes_Neighbor_Default_Originate_Skeleton
configure terminal

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> default-originate
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Origination_And_Default_Routes_Conditional_Default_Originate_Skeleton
configure terminal

ip prefix-list <condition-pl> permit <tracked-prefix>/<length>

route-map <default-condition-rm> permit 10
 match ip address prefix-list <condition-pl>

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> default-originate route-map <default-condition-rm>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Origination_And_Default_Routes_Redistribute_Static_Controlled_Skeleton
configure terminal

ip prefix-list <static-to-bgp-pl> permit <prefix>/<length>

route-map <static-to-bgp-rm> permit 10
 match ip address prefix-list <static-to-bgp-pl>

router bgp <local-asn>
 address-family ipv4 unicast
  redistribute static route-map <static-to-bgp-rm>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Origination_And_Default_Routes_Redistribute_Connected_Controlled_Skeleton
configure terminal

ip prefix-list <connected-to-bgp-pl> permit <prefix>/<length>

route-map <connected-to-bgp-rm> permit 10
 match ip address prefix-list <connected-to-bgp-pl>

router bgp <local-asn>
 address-family ipv4 unicast
  redistribute connected route-map <connected-to-bgp-rm>
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Origination_And_Default_Routes_Verification_Skeleton
show bgp ipv4 unicast summary
show ip route <prefix>
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast 0.0.0.0/0
show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes
show ip route 0.0.0.0
show running-config | section router bgp
show running-config | section route-map
show running-config | include ip prefix-list
show logging | include BGP|ADJCHANGE
```



# BGP_Origination_And_Default_Routes_Verification_Commands
| Task                                                       | Command                                                           | Expected Result                                               |                                                                              |                                        |
| ---------------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------- |
| Verify neighbor state                                      | `show bgp ipv4 unicast summary`                                   | Neighbor shows a number under `State/PfxRcd`                  |                                                                              |                                        |
| Verify exact RIB route for `network` statement             | `show ip route <prefix> <mask>`                                   | Exact prefix and mask exist                                   |                                                                              |                                        |
| Verify exact default route for default `network` statement | `show ip route 0.0.0.0 0.0.0.0`                                   | Exact default route exists                                    |                                                                              |                                        |
| Verify BGP config                                          | `show running-config                                              | section router bgp`                                           | Network statements, redistribution, and default-originate config are visible |                                        |
| Verify route map config                                    | `show running-config                                              | section route-map`                                            | Route map match/set logic is visible                                         |                                        |
| Verify prefix list config                                  | `show running-config                                              | include ip prefix-list`                                       | Prefix list permits only intended routes                                     |                                        |
| Verify BGP table                                           | `show bgp ipv4 unicast`                                           | Originated prefixes appear                                    |                                                                              |                                        |
| Verify specific originated prefix                          | `show bgp ipv4 unicast <prefix>`                                  | Prefix shows expected next hop, origin code, weight, and path |                                                                              |                                        |
| Verify locally originated next hop                         | `show bgp ipv4 unicast <prefix>`                                  | Locally originated route often shows next hop `0.0.0.0`       |                                                                              |                                        |
| Verify `network` origin code                               | `show bgp ipv4 unicast <prefix>`                                  | Origin code is `i`                                            |                                                                              |                                        |
| Verify redistributed origin code                           | `show bgp ipv4 unicast <prefix>`                                  | Origin code is usually `?`                                    |                                                                              |                                        |
| Verify local weight                                        | `show bgp ipv4 unicast <prefix>`                                  | Locally originated route usually has weight `32768`           |                                                                              |                                        |
| Verify default route in BGP                                | `show bgp ipv4 unicast 0.0.0.0/0`                                 | Default route appears if originated                           |                                                                              |                                        |
| Verify advertised routes                                   | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Intended prefix or default is advertised                      |                                                                              |                                        |
| Verify peer received prefix                                | `show bgp ipv4 unicast <prefix>` on peer                          | Peer sees advertised route                                    |                                                                              |                                        |
| Verify peer received default                               | `show bgp ipv4 unicast 0.0.0.0/0` on peer                         | Peer sees default route                                       |                                                                              |                                        |
| Verify peer route install                                  | `show ip route <prefix>` on peer                                  | Peer installs route if valid and preferred                    |                                                                              |                                        |
| Verify peer default install                                | `show ip route 0.0.0.0` on peer                                   | Peer installs BGP default if valid and preferred              |                                                                              |                                        |
| Verify peer next-hop reachability                          | `show ip route <next-hop-ip>` on peer                             | Peer can reach next hop                                       |                                                                              |                                        |
| Verify forwarding path                                     | `show ip cef <prefix>`                                            | CEF points toward expected next hop/interface                 |                                                                              |                                        |
| Verify traffic path                                        | `traceroute <destination-ip> source <source-ip>`                  | Traffic follows expected route or default path                |                                                                              |                                        |
| Verify BGP logs                                            | `show logging                                                     | include BGP                                                   | ADJCHANGE`                                                                   | Neighbor and update events are visible |

# BGP_Origination_And_Default_Routes_Rollback
| Step | Task                                         | Device      | Command                                                           | Expected Result                                       |                                                                                |
| ---: | -------------------------------------------- | ----------- | ----------------------------------------------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------ |
|    1 | Identify current BGP origination config      | Router      | `show running-config                                              | section router bgp`                                   | Network statements, redistribution, and default-originate commands are visible |
|    2 | Identify route maps                          | Router      | `show running-config                                              | section route-map`                                    | Route maps used for redistribution/default origination are visible             |
|    3 | Identify prefix lists                        | Router      | `show running-config                                              | include ip prefix-list`                               | Prefix lists used by route maps are visible                                    |
|    4 | Identify static routes used for origination  | Router      | `show running-config                                              | include ^ip route`                                    | Static route or static default route is visible                                |
|    5 | Enter configuration mode                     | Router      | `configure terminal`                                              | Router enters global configuration mode               |                                                                                |
|    6 | Enter BGP process                            | Router      | `router bgp <local-asn>`                                          | Router enters BGP configuration mode                  |                                                                                |
|    7 | Enter IPv4 unicast AFI                       | Router      | `address-family ipv4 unicast`                                     | Router enters IPv4 unicast address-family             |                                                                                |
|    8 | Remove normal `network` statement            | Router      | `no network <prefix> mask <mask>`                                 | Prefix is no longer locally originated by `network`   |                                                                                |
|    9 | Remove default `network` statement           | Router      | `no network 0.0.0.0 mask 0.0.0.0`                                 | Default is no longer originated by `network`          |                                                                                |
|   10 | Remove neighbor default-originate            | Router      | `no neighbor <neighbor-ip> default-originate`                     | Neighbor no longer receives default from this command |                                                                                |
|   11 | Remove conditional default-originate         | Router      | `no neighbor <neighbor-ip> default-originate route-map <rm-name>` | Conditional default advertisement is removed          |                                                                                |
|   12 | Remove static redistribution                 | Router      | `no redistribute static route-map <rm-name>`                      | Static routes are no longer redistributed into BGP    |                                                                                |
|   13 | Remove connected redistribution              | Router      | `no redistribute connected route-map <rm-name>`                   | Connected routes are no longer redistributed into BGP |                                                                                |
|   14 | Exit address-family mode                     | Router      | `exit-address-family`                                             | Router returns to BGP config mode                     |                                                                                |
|   15 | Remove lab-only route map                    | Router      | `no route-map <rm-name> permit <seq>`                             | Route map sequence is removed                         |                                                                                |
|   16 | Remove lab-only prefix list                  | Router      | `no ip prefix-list <pl-name>`                                     | Prefix list is removed                                |                                                                                |
|   17 | Remove lab-only static route                 | Router      | `no ip route <prefix> <mask> <next-hop-ip-or-null0>`              | Static route used for origination is removed          |                                                                                |
|   18 | Remove lab-only static default route         | Router      | `no ip route 0.0.0.0 0.0.0.0 <next-hop-ip>`                       | Static default route is removed                       |                                                                                |
|   19 | Soft clear outbound updates                  | Router      | `clear bgp ipv4 unicast <neighbor-ip> soft out`                   | Peer receives updated advertisement set               |                                                                                |
|   20 | Verify route removed from local BGP          | Router      | `show bgp ipv4 unicast <prefix>`                                  | Prefix is absent or no longer locally originated      |                                                                                |
|   21 | Verify default removed from peer if intended | Peer Router | `show bgp ipv4 unicast 0.0.0.0/0`                                 | Default route is absent if rollback intended          |                                                                                |
|   22 | Save rollback                                | Router      | `copy running-config startup-config`                              | Rollback is saved                                     |                                                                                |

# BGP_Origination_And_Default_Routes_Failure_Checks
| Symptom                                              | Command                                                            | What Usually Broke                                                          |                                                                            |
| ---------------------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `network` statement does not advertise prefix        | `show ip route <prefix> <mask>`                                    | Exact route does not exist in the RIB                                       |                                                                            |
| Prefix exists in RIB but not in BGP                  | `show running-config                                               | section router bgp`                                                         | Missing `network` statement, wrong mask, wrong AFI, or route-map denies it |
| Prefix appears with wrong origin code                | `show bgp ipv4 unicast <prefix>`                                   | `network` gives `i`; redistribution usually gives `?`                       |                                                                            |
| Default route is not advertised by `network 0.0.0.0` | `show ip route 0.0.0.0 0.0.0.0`                                    | Exact default route does not exist locally                                  |                                                                            |
| Neighbor does not receive default route              | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes`  | `default-originate` missing, route-map blocks it, or neighbor not activated |                                                                            |
| Conditional default never advertises                 | `show route-map <rm-name>` and `show ip route <condition-prefix>`  | Route-map condition does not match                                          |                                                                            |
| Conditional default always advertises                | `show running-config                                               | section route-map`                                                          | Route-map is too broad or missing match condition                          |
| Redistribution injects too many routes               | `show bgp ipv4 unicast`                                            | Redistribution was configured without a restrictive route map               |                                                                            |
| Redistributed route missing                          | `show route-map <rm-name>` and `show ip route <prefix>`            | Route map does not match source route or source route is absent             |                                                                            |
| Peer receives route but does not install it          | Peer `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Next hop unreachable, better route exists, or policy rejects route          |                                                                            |
| Route appears in BGP but no `>` marker               | `show bgp ipv4 unicast <prefix>`                                   | Route is not selected as best                                               |                                                                            |
| Route appears with `r` RIB failure                   | `show bgp ipv4 unicast`                                            | BGP best route lost RIB install to another route source                     |                                                                            |
| Default route causes blackhole                       | `traceroute <external-ip>` and upstream route check                | Default was advertised without real upstream reachability                   |                                                                            |
| Null0 route causes unexpected drop                   | `show ip route <prefix>`                                           | Static Null0 route supports origination but catches unmatched traffic       |                                                                            |
| Peer gets full routes when only default was intended | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes`  | Outbound policy is missing or neighbor is not limited to default            |                                                                            |
| Peer gets default plus unwanted specifics            | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes`  | Route map/prefix list does not filter specifics                             |                                                                            |
| Soft policy change does not show up on peer          | `clear bgp ipv4 unicast <neighbor-ip> soft out`                    | Outbound updates were not refreshed                                         |                                                                            |
| Hard clear caused avoidable outage                   | `show logging                                                      | include BGP`                                                                | Hard clear used when soft outbound refresh was enough                      |

# Attached_Labs
| Lab Name                                | Attach Under This Note                  | Why                                                                                                    |
| --------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `bgp-default-final`                     | `BGP_Origination_And_Default_Routes.md` | Primary lab for default route origination and default advertisement behavior                           |
| `bgp-attribute-locally-originate-final` | `BGP_Origination_And_Default_Routes.md` | Primary lab for locally originated route behavior and how local origination affects BGP path selection |