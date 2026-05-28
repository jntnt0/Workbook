

```
# Index
# Source_Basis
# BGP_Table_Reading_And_Startup_Baseline_Mental_Model
# BGP_Table_Reading_And_Startup_Baseline_Configuration_Checklist
# BGP_Table_Reading_And_Startup_Baseline_Minimal_eBGP_Skeleton
# BGP_Table_Reading_And_Startup_Baseline_Standardized_AFI_Skeleton
# BGP_Table_Reading_And_Startup_Baseline_Loopback_Peering_Skeleton
# BGP_Table_Reading_And_Startup_Baseline_Table_Reading_Skeleton
# BGP_Table_Reading_And_Startup_Baseline_Verification_Commands
# BGP_Table_Reading_And_Startup_Baseline_Rollback
# BGP_Table_Reading_And_Startup_Baseline_Failure_Checks
# Attached_Labs
```

# BGP_Table_Reading_And_Startup_Baseline_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP startup baseline | The minimum state needed before deeper BGP policy troubleshooting matters |
| BGP process | `router bgp <asn>` starts the BGP process for the local AS |
| BGP router ID | Stable identifier for the BGP speaker; changing it resets sessions |
| Neighbor statement | Defines the peer IP and remote AS; wrong IP or ASN prevents session formation |
| AFI/SAFI | Address-family context that controls which routes are exchanged |
| Neighbor activation | Required when default IPv4 unicast activation is disabled |
| BGP table | Loc-RIB view of BGP-learned and locally originated prefixes |
| Routing table | RIB/FIB install view; a BGP route can exist in BGP but not install into routing table |
| `State/PfxRcd` number | Neighbor is established; number shows received prefixes |
| `Idle` or `Active` | Neighbor relationship is not established |
| Next-hop reachability | BGP path must have a reachable next hop before it can be valid for best-path/install |
| `*` valid | BGP considers the path valid |
| `>` best | BGP selected this path as best |
| `i` internal | Route was learned through iBGP |
| `r` RIB failure | BGP selected a route but did not install it in the routing table |
| Origin code `i` | Prefix originated through a BGP `network` statement |
| Origin code `?` | Prefix was redistributed into BGP |
| Weight | Local Cisco-only path preference; higher wins and does not leave the router |
| Local preference | AS-wide preference; higher wins and is carried within iBGP |
| AS path | Sequence of ASNs the route crossed; shorter is generally preferred after higher-priority attributes |
| Startup troubleshooting order | Interface, route to peer, ping with source, TCP/179, neighbor IP/ASN, update-source, TTL, authentication, AF activation |
| Blunt rule | Do not troubleshoot BGP attributes until the neighbor is established and you can correctly read the BGP table |



# BGP_Table_Reading_And_Startup_Baseline_Configuration_Checklist
| Step | Task                                                                     | Device         | Command                                                           | Expected Result                                                                       |                                                                |                                           |                  |                                   |
| ---: | ------------------------------------------------------------------------ | -------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------- | ----------------------------------------- | ---------------- | --------------------------------- |
|    1 | Confirm interface baseline                                               | Router         | `show ip interface brief`                                         | Local peer-facing interface or loopback is up/up                                      |                                                                |                                           |                  |                                   |
|    2 | Confirm local interface addressing                                       | Router         | `show running-config interface <interface>`                       | Interface has expected IP address and is not shut down                                |                                                                |                                           |                  |                                   |
|    3 | Confirm local AS number                                                  | Router / Notes | `<local-asn>`                                                     | Local BGP AS is known                                                                 |                                                                |                                           |                  |                                   |
|    4 | Confirm neighbor IP                                                      | Router / Notes | `<neighbor-ip>`                                                   | Peer address used in neighbor statement is known                                      |                                                                |                                           |                  |                                   |
|    5 | Confirm remote AS number                                                 | Router / Notes | `<remote-asn>`                                                    | Peer AS is known                                                                      |                                                                |                                           |                  |                                   |
|    6 | Confirm peering type                                                     | Router / Notes | `eBGP` or `iBGP`                                                  | Local and remote AS relationship is clear                                             |                                                                |                                           |                  |                                   |
|    7 | Confirm source interface requirement                                     | Router / Notes | `<physical-interface>` or `<loopback-interface>`                  | You know whether BGP will source from physical IP or loopback                         |                                                                |                                           |                  |                                   |
|    8 | Confirm route to neighbor                                                | Router         | `show ip route <neighbor-ip>`                                     | Route to neighbor exists and is not only assumed                                      |                                                                |                                           |                  |                                   |
|    9 | Confirm sourced reachability to neighbor                                 | Router         | `ping <neighbor-ip> source <local-source-ip>`                     | Ping succeeds from the same source BGP will use                                       |                                                                |                                           |                  |                                   |
|   10 | Confirm reverse reachability from peer                                   | Peer Router    | `ping <local-source-ip> source <neighbor-source-ip>`              | Peer can reach the local BGP source address                                           |                                                                |                                           |                  |                                   |
|   11 | Confirm TCP/179 is not blocked                                           | Router / Path  | `show access-lists` or firewall check                             | ACL/firewall does not block BGP TCP/179                                               |                                                                |                                           |                  |                                   |
|   12 | Enter configuration mode                                                 | Router         | `configure terminal`                                              | Router enters global configuration mode                                               |                                                                |                                           |                  |                                   |
|   13 | Start BGP process                                                        | Router         | `router bgp <local-asn>`                                          | Router enters BGP configuration mode                                                  |                                                                |                                           |                  |                                   |
|   14 | Set stable BGP router ID                                                 | Router         | `bgp router-id <router-id>`                                       | Router ID is deterministic and stable                                                 |                                                                |                                           |                  |                                   |
|   15 | Enable BGP neighbor change logging                                       | Router         | `bgp log-neighbor-changes`                                        | Neighbor up/down events are logged                                                    |                                                                |                                           |                  |                                   |
|   16 | Disable automatic IPv4 neighbor activation if standardizing AFI behavior | Router         | `no bgp default ipv4-unicast`                                     | IPv4 neighbors must be activated under AFI explicitly                                 |                                                                |                                           |                  |                                   |
|   17 | Configure neighbor remote AS                                             | Router         | `neighbor <neighbor-ip> remote-as <remote-asn>`                   | BGP peer is defined                                                                   |                                                                |                                           |                  |                                   |
|   18 | Add neighbor description                                                 | Router         | `neighbor <neighbor-ip> description <description>`                | Peer purpose is documented                                                            |                                                                |                                           |                  |                                   |
|   19 | Configure update source if peering through loopbacks                     | Router         | `neighbor <neighbor-ip> update-source <loopback-interface>`       | BGP packets source from expected loopback                                             |                                                                |                                           |                  |                                   |
|   20 | Configure eBGP multihop only if peer is not directly connected           | Router         | `neighbor <neighbor-ip> ebgp-multihop <ttl>`                      | eBGP packets can reach non-direct peer                                                |                                                                |                                           |                  |                                   |
|   21 | Configure password only if both peers use authentication                 | Router         | `neighbor <neighbor-ip> password <password>`                      | BGP TCP MD5/authentication matches peer                                               |                                                                |                                           |                  |                                   |
|   22 | Enter IPv4 unicast address family                                        | Router         | `address-family ipv4 unicast`                                     | Router enters IPv4 unicast BGP AFI                                                    |                                                                |                                           |                  |                                   |
|   23 | Activate neighbor in IPv4 unicast AFI                                    | Router         | `neighbor <neighbor-ip> activate`                                 | Neighbor exchanges IPv4 unicast NLRI                                                  |                                                                |                                           |                  |                                   |
|   24 | Advertise local connected prefix if exact route exists in RIB            | Router         | `network <network> mask <subnet-mask>`                            | Exact matching route can be injected into BGP                                         |                                                                |                                           |                  |                                   |
|   25 | Verify exact route exists for `network` statement                        | Router         | `show ip route <network> <subnet-mask>`                           | Route exists in RIB; otherwise BGP will not originate it                              |                                                                |                                           |                  |                                   |
|   26 | Exit address-family mode                                                 | Router         | `exit-address-family`                                             | Router returns to BGP config mode                                                     |                                                                |                                           |                  |                                   |
|   27 | Save configuration                                                       | Router         | `copy running-config startup-config`                              | BGP startup baseline is saved                                                         |                                                                |                                           |                  |                                   |
|   28 | Verify BGP process exists                                                | Router         | `show running-config                                              | section router bgp`                                                                   | BGP process, neighbor, AFI, and network statements are visible |                                           |                  |                                   |
|   29 | Verify BGP summary                                                       | Router         | `show bgp ipv4 unicast summary`                                   | Neighbor appears in summary output                                                    |                                                                |                                           |                  |                                   |
|   30 | Interpret `State/PfxRcd`                                                 | Router         | `show bgp ipv4 unicast summary`                                   | A number means established; `Idle` or `Active` means session problem                  |                                                                |                                           |                  |                                   |
|   31 | Verify neighbor detail if summary is unclear                             | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Neighbor state, remote AS, timers, capabilities, and AFI info are visible             |                                                                |                                           |                  |                                   |
|   32 | Verify legacy equivalent summary if needed                               | Router         | `show ip bgp summary`                                             | Same practical IPv4 unicast summary view appears                                      |                                                                |                                           |                  |                                   |
|   33 | Verify BGP table                                                         | Router         | `show bgp ipv4 unicast`                                           | BGP Loc-RIB entries appear                                                            |                                                                |                                           |                  |                                   |
|   34 | Verify legacy equivalent BGP table if needed                             | Router         | `show ip bgp`                                                     | Same IPv4 unicast BGP table appears                                                   |                                                                |                                           |                  |                                   |
|   35 | Read status codes                                                        | Router         | `show bgp ipv4 unicast`                                           | `*` valid and `>` best are identified                                                 |                                                                |                                           |                  |                                   |
|   36 | Read origin codes                                                        | Router         | `show bgp ipv4 unicast`                                           | `i`, `e`, and `?` origin meanings are identified                                      |                                                                |                                           |                  |                                   |
|   37 | Verify next hop                                                          | Router         | `show bgp ipv4 unicast`                                           | Next-hop value is visible for each prefix                                             |                                                                |                                           |                  |                                   |
|   38 | Verify next-hop reachability                                             | Router         | `show ip route <next-hop-ip>`                                     | Next hop is reachable in RIB                                                          |                                                                |                                           |                  |                                   |
|   39 | Verify best BGP path details                                             | Router         | `show bgp ipv4 unicast <prefix>`                                  | Best path, attributes, next hop, and reason are visible                               |                                                                |                                           |                  |                                   |
|   40 | Verify route installed in routing table                                  | Router         | `show ip route <prefix>`                                          | Best BGP path installs if no better administrative-distance route wins                |                                                                |                                           |                  |                                   |
|   41 | Verify forwarding entry                                                  | Router         | `show ip cef <prefix>`                                            | Forwarding path points to expected next hop/interface                                 |                                                                |                                           |                  |                                   |
|   42 | Verify neighbor message counters                                         | Router         | `show bgp ipv4 unicast summary`                                   | MsgRcvd and MsgSent increment over time                                               |                                                                |                                           |                  |                                   |
|   43 | Verify input/output queues                                               | Router         | `show bgp ipv4 unicast summary`                                   | InQ and OutQ should usually be 0 in stable state                                      |                                                                |                                           |                  |                                   |
|   44 | Verify advertised routes if needed                                       | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Router is advertising expected prefixes                                               |                                                                |                                           |                  |                                   |
|   45 | Verify received routes if supported                                      | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> received-routes`   | Router sees expected inbound routes if soft reconfiguration/feature support allows it |                                                                |                                           |                  |                                   |
|   46 | Verify route policy is not hiding route                                  | Router         | `show running-config                                              | include route-map                                                                     | prefix-list                                                    | filter-list                               | distribute-list` | Filters and policy are identified |
|   47 | Verify log messages                                                      | Router         | `show logging                                                     | include BGP                                                                           | ADJCHANGE`                                                     | BGP neighbor up/down messages are visible |                  |                                   |
|   48 | Clear BGP softly after policy-only change                                | Router         | `clear bgp ipv4 unicast <neighbor-ip> soft in`                    | Inbound policy is refreshed without hard-resetting session                            |                                                                |                                           |                  |                                   |
|   49 | Clear BGP softly outbound after policy-only change                       | Router         | `clear bgp ipv4 unicast <neighbor-ip> soft out`                   | Outbound policy is refreshed without full teardown                                    |                                                                |                                           |                  |                                   |
|   50 | Avoid hard clear unless necessary                                        | Router         | `clear bgp ipv4 unicast <neighbor-ip>`                            | Session resets only when intentional                                                  |                                                                |                                           |                  |                                   |


```
# BGP_Table_Reading_And_Startup_Baseline_Minimal_eBGP_Skeleton
configure terminal

router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 neighbor <neighbor-ip> remote-as <remote-asn>
 neighbor <neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```



```
# BGP_Table_Reading_And_Startup_Baseline_Standardized_AFI_Skeleton
configure terminal

router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <neighbor-ip> remote-as <remote-asn>
 neighbor <neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Table_Reading_And_Startup_Baseline_Loopback_Peering_Skeleton
configure terminal

interface Loopback0
 ip address <local-loopback-ip> 255.255.255.255

router bgp <local-asn>
 bgp router-id <local-loopback-ip>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <neighbor-loopback-ip> remote-as <remote-asn>
 neighbor <neighbor-loopback-ip> update-source Loopback0
 neighbor <neighbor-loopback-ip> ebgp-multihop <ttl>

 address-family ipv4 unicast
  neighbor <neighbor-loopback-ip> activate
  network <local-loopback-ip> mask 255.255.255.255
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_Table_Reading_And_Startup_Baseline_Table_Reading_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast
show bgp ipv4 unicast <prefix>
show ip route <prefix>
show ip cef <prefix>
show logging | include BGP|ADJCHANGE
```


# BGP_Table_Reading_And_Startup_Baseline_Verification_Commands
| Task                                           | Command                                                           | Expected Result                                                    |                                                                        |                                     |
| ---------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------------------- | ----------------------------------- |
| Verify interface state                         | `show ip interface brief`                                         | Peer-facing interface or loopback path is up/up                    |                                                                        |                                     |
| Verify route to neighbor                       | `show ip route <neighbor-ip>`                                     | Router has a non-broken path to peer                               |                                                                        |                                     |
| Verify sourced ping                            | `ping <neighbor-ip> source <local-source-ip>`                     | Peer is reachable from the BGP source address                      |                                                                        |                                     |
| Verify BGP config                              | `show running-config                                              | section router bgp`                                                | Local AS, router ID, neighbor, AFI, and network statements are visible |                                     |
| Verify BGP summary                             | `show bgp ipv4 unicast summary`                                   | Neighbor appears with a number in `State/PfxRcd` when established  |                                                                        |                                     |
| Verify legacy summary equivalent               | `show ip bgp summary`                                             | Same IPv4 unicast summary view appears                             |                                                                        |                                     |
| Verify neighbor detail                         | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | State is Established and remote AS/source/capabilities are correct |                                                                        |                                     |
| Verify BGP table                               | `show bgp ipv4 unicast`                                           | BGP Loc-RIB is visible                                             |                                                                        |                                     |
| Verify legacy table equivalent                 | `show ip bgp`                                                     | Same IPv4 unicast BGP table appears                                |                                                                        |                                     |
| Verify specific BGP prefix                     | `show bgp ipv4 unicast <prefix>`                                  | All paths and path attributes for prefix are visible               |                                                                        |                                     |
| Verify best path marker                        | `show bgp ipv4 unicast`                                           | Best route has `>`                                                 |                                                                        |                                     |
| Verify valid path marker                       | `show bgp ipv4 unicast`                                           | Valid route has `*`                                                |                                                                        |                                     |
| Verify internal route marker                   | `show bgp ipv4 unicast`                                           | iBGP-learned routes show internal marker                           |                                                                        |                                     |
| Verify RIB failure                             | `show bgp ipv4 unicast`                                           | `r` indicates BGP path did not install in RIB                      |                                                                        |                                     |
| Verify origin code                             | `show bgp ipv4 unicast`                                           | `i`, `e`, or `?` explains route origin type                        |                                                                        |                                     |
| Verify next-hop value                          | `show bgp ipv4 unicast`                                           | Next hop is visible for each path                                  |                                                                        |                                     |
| Verify next-hop route                          | `show ip route <next-hop-ip>`                                     | Next hop is reachable                                              |                                                                        |                                     |
| Verify route installed in RIB                  | `show ip route <prefix>`                                          | BGP route appears if selected and installable                      |                                                                        |                                     |
| Verify forwarding entry                        | `show ip cef <prefix>`                                            | FIB points to expected next hop/interface                          |                                                                        |                                     |
| Verify locally originated network route exists | `show ip route <network> <subnet-mask>`                           | Exact route exists for the BGP `network` command                   |                                                                        |                                     |
| Verify advertised routes                       | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected prefixes are advertised                                   |                                                                        |                                     |
| Verify received route count                    | `show bgp ipv4 unicast summary`                                   | Prefix count under `State/PfxRcd` matches expectation              |                                                                        |                                     |
| Verify queue health                            | `show bgp ipv4 unicast summary`                                   | `InQ` and `OutQ` are usually 0                                     |                                                                        |                                     |
| Verify log messages                            | `show logging                                                     | include BGP                                                        | ADJCHANGE`                                                             | Neighbor up/down events are visible |

# BGP_Table_Reading_And_Startup_Baseline_Rollback
| Step | Task                                                 | Device | Command                                                                      | Expected Result                                  |                                                            |
| ---: | ---------------------------------------------------- | ------ | ---------------------------------------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------------- |
|    1 | Identify BGP config                                  | Router | `show running-config                                                         | section router bgp`                              | Current BGP process, neighbors, and AFI config are visible |
|    2 | Identify advertised networks                         | Router | `show running-config                                                         | section router bgp`                              | `network` statements are known                             |
|    3 | Identify active sessions                             | Router | `show bgp ipv4 unicast summary`                                              | Current neighbors and state are visible          |                                                            |
|    4 | Enter configuration mode                             | Router | `configure terminal`                                                         | Router enters global configuration mode          |                                                            |
|    5 | Enter BGP process                                    | Router | `router bgp <local-asn>`                                                     | Router enters BGP config mode                    |                                                            |
|    6 | Remove advertised prefix                             | Router | `address-family ipv4 unicast` then `no network <network> mask <subnet-mask>` | Prefix is no longer originated into BGP          |                                                            |
|    7 | Remove neighbor activation                           | Router | `no neighbor <neighbor-ip> activate` under AFI                               | Neighbor is no longer active for IPv4 unicast    |                                                            |
|    8 | Exit address family                                  | Router | `exit-address-family`                                                        | Router returns to BGP config mode                |                                                            |
|    9 | Remove neighbor statement                            | Router | `no neighbor <neighbor-ip> remote-as <remote-asn>`                           | Neighbor is removed                              |                                                            |
|   10 | Remove update-source if configured separately        | Router | `no neighbor <neighbor-ip> update-source <interface>`                        | Update-source override is removed                |                                                            |
|   11 | Remove eBGP multihop if configured separately        | Router | `no neighbor <neighbor-ip> ebgp-multihop <ttl>`                              | eBGP multihop override is removed                |                                                            |
|   12 | Remove BGP process only if full rollback is intended | Router | `no router bgp <local-asn>`                                                  | Entire BGP process is removed                    |                                                            |
|   13 | Verify neighbor removed                              | Router | `show bgp ipv4 unicast summary`                                              | Removed neighbor no longer appears               |                                                            |
|   14 | Verify prefix no longer advertised                   | Router | `show bgp ipv4 unicast <prefix>`                                             | Prefix is absent or no longer locally originated |                                                            |
|   15 | Save rollback                                        | Router | `copy running-config startup-config`                                         | Rollback is saved                                |                                                            |

# BGP_Table_Reading_And_Startup_Baseline_Failure_Checks
| Symptom                                        | Command                                                         | What Usually Broke                                                                               |                                                                                |
| ---------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| Neighbor stuck Idle                            | `show bgp ipv4 unicast summary`                                 | No TCP session; route, reachability, ACL, wrong neighbor IP, or peer not configured              |                                                                                |
| Neighbor stuck Active                          | `show bgp ipv4 unicast summary`                                 | Router keeps trying TCP/179 but cannot complete session                                          |                                                                                |
| Neighbor never appears                         | `show running-config                                            | section router bgp`                                                                              | Neighbor statement missing or wrong BGP process                                |
| Ping to neighbor fails                         | `ping <neighbor-ip> source <local-source-ip>`                   | L3 reachability is broken                                                                        |                                                                                |
| Route to neighbor missing                      | `show ip route <neighbor-ip>`                                   | No route to peer source address                                                                  |                                                                                |
| Peering works one way only                     | `show bgp ipv4 unicast neighbors <neighbor-ip>` on both routers | Return route or peer source address mismatch                                                     |                                                                                |
| eBGP loopback peer fails                       | `show running-config                                            | section router bgp`                                                                              | Missing `update-source`, missing route to loopback, or missing `ebgp-multihop` |
| Neighbor source mismatch                       | `debug ip tcp transactions` or neighbor detail                  | Inbound BGP packet source does not match configured neighbor IP                                  |                                                                                |
| TCP/179 blocked                                | ACL/firewall check, `show access-lists`                         | ACL or firewall blocks BGP                                                                       |                                                                                |
| Authentication failure                         | `show logging                                                   | include BGP`                                                                                     | Password/MD5 mismatch                                                          |
| Neighbor established but receives 0 prefixes   | `show bgp ipv4 unicast summary`                                 | Peer is not advertising routes, AFI not activated, policy filter, or no valid network statements |                                                                                |
| `network` statement does not advertise prefix  | `show ip route <network> <mask>`                                | Exact route does not exist in RIB                                                                |                                                                                |
| Route appears in BGP but not routing table     | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>`   | Better AD route exists, next hop unreachable, or RIB failure                                     |                                                                                |
| BGP route has no `*` valid marker              | `show bgp ipv4 unicast <prefix>`                                | Next hop is unreachable or route failed validity check                                           |                                                                                |
| BGP route has `*` but no `>`                   | `show bgp ipv4 unicast <prefix>`                                | Valid path exists but another BGP path is better                                                 |                                                                                |
| BGP route shows `r` RIB failure                | `show bgp ipv4 unicast`                                         | BGP best path lost RIB install to another route source or install constraint                     |                                                                                |
| iBGP route not advertised to another iBGP peer | `show bgp ipv4 unicast` and peer topology                       | iBGP split-horizon rule; use full mesh, route reflector, or confederation                        |                                                                                |
| Prefix advertised with wrong origin code       | `show bgp ipv4 unicast <prefix>`                                | `network` statement vs redistribution source is different than expected                          |                                                                                |
| Next hop is wrong                              | `show bgp ipv4 unicast <prefix>`                                | eBGP-to-iBGP next-hop behavior or missing `next-hop-self`                                        |                                                                                |
| Queue counters stay nonzero                    | `show bgp ipv4 unicast summary`                                 | Control-plane congestion, stuck peer, or session issue                                           |                                                                                |
| Table output is misunderstood                  | `show bgp ipv4 unicast`                                         | Reading BGP table as routing table instead of Loc-RIB                                            |                                                                                |
| Hard clear caused outage                       | `show logging                                                   | include BGP`                                                                                     | Used hard `clear bgp` instead of soft policy refresh                           |


```
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `how-to-read-bgp-table-final` | `BGP_Table_Reading_And_Startup_Baseline.md` | Primary table-reading lab for status codes, origin codes, next hop, best path, and BGP vs RIB thinking |
| `bgp-troubleshooting-startup-final` | `BGP_Table_Reading_And_Startup_Baseline.md` | Primary startup baseline lab for neighbor state, reachability, route-to-peer, TCP/179, source IP, and initial BGP session checks |
```