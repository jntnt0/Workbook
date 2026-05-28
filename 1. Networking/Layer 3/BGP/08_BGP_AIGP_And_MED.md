


```
# Index
# Source_Basis
# BGP_AIGP_And_MED_Mental_Model
# BGP_AIGP_And_MED_Configuration_Checklist
# BGP_AIGP_And_MED_AIGP_Neighbor_Enable_Skeleton
# BGP_AIGP_And_MED_AIGP_Set_IGP_Metric_Skeleton
# BGP_AIGP_And_MED_AIGP_Set_Manual_Metric_Skeleton
# BGP_AIGP_And_MED_MED_Inbound_Skeleton
# BGP_AIGP_And_MED_MED_Outbound_Skeleton
# BGP_AIGP_And_MED_Missing_MED_Behavior_Skeleton
# BGP_AIGP_And_MED_MED_Comparison_Tuning_Skeleton
# BGP_AIGP_And_MED_Verification_Skeleton
# BGP_AIGP_And_MED_Verification_Commands
# BGP_AIGP_And_MED_Rollback
# BGP_AIGP_And_MED_Failure_Checks
# Attached_Labs
```
# BGP_AIGP_And_MED_Mental_Model
| Concept                             | Operational Meaning                                                                                                                                     |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AIGP                                | Optional BGP path attribute that carries accumulated internal path cost across a controlled BGP domain                                                  |
| AIGP scope                          | Best suited when multiple ASNs are under one administrative control with consistent routing policy                                                      |
| AIGP is optional nontransitive      | It is not automatically carried everywhere and must be supported/enabled between peers                                                                  |
| AIGP neighbor agreement             | Both BGP peers must support and use AIGP for the metric to be exchanged                                                                                 |
| `neighbor <ip> aigp`                | Enables AIGP exchange with a BGP neighbor under the address family                                                                                      |
| AIGP metric                         | 32-bit value used by BGP as a path-selection input                                                                                                      |
| `set aigp-metric igp-metric`        | Copies the IGP metric into the AIGP value during redistribution or policy handling                                                                      |
| `set aigp-metric <metric>`          | Manually assigns an AIGP value with a route map                                                                                                         |
| Lower AIGP wins                     | When compared, the lower AIGP metric is preferred                                                                                                       |
| AIGP beats AS path length           | AIGP is evaluated before shortest AS path in the best-path process                                                                                      |
| Recursive AIGP                      | If the next hop is recursive, BGP can add the cost to reach the next hop into the derived AIGP metric                                                   |
| MED                                 | Nontransitive BGP attribute used to suggest preferred inbound entry point into an AS                                                                    |
| MED metric                          | 32-bit value shown in the BGP table Metric column                                                                                                       |
| Lower MED wins                      | Lower MED is preferred over higher MED                                                                                                                  |
| MED same-AS expectation             | MED is normally compared when candidate paths come from the same neighboring ASN                                                                        |
| Missing MED default                 | Missing MED is usually treated like 0, which can make a path without MED unexpectedly preferred                                                         |
| `bgp bestpath med missing-as-worst` | Treats missing MED as worst possible value                                                                                                              |
| `default-metric <metric>`           | Sets a default metric for paths received without MED                                                                                                    |
| `bgp always-compare-med`            | Allows MED comparison across different AS paths                                                                                                         |
| `bgp deterministic-med`             | Groups paths with identical AS paths so MED comparison is consistent regardless of update arrival order                                                 |
| Blunt rule                          | AIGP is for controlled internal cost-aware BGP domains. MED is for telling a neighboring AS which inbound link you prefer. They are not interchangeable |



# BGP_AIGP_And_MED_Configuration_Checklist
| Step | Task                                                                              | Device          | Command                                                             | Expected Result                                              |                                                                            |                  |                 |                                                  |
| ---: | --------------------------------------------------------------------------------- | --------------- | ------------------------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------- | ---------------- | --------------- | ------------------------------------------------ |
|    1 | Confirm BGP neighbors are established                                             | Router          | `show bgp ipv4 unicast summary`                                     | Relevant neighbors show a number under `State/PfxRcd`        |                                                                            |                  |                 |                                                  |
|    2 | Confirm target prefix exists in BGP                                               | Router          | `show bgp ipv4 unicast <prefix>`                                    | Prefix appears with candidate paths                          |                                                                            |                  |                 |                                                  |
|    3 | Confirm next-hop reachability                                                     | Router          | `show ip route <next-hop-ip>`                                       | BGP next hop is reachable                                    |                                                                            |                  |                 |                                                  |
|    4 | Confirm current best path                                                         | Router          | `show bgp ipv4 unicast <prefix>`                                    | Current best path is marked with `>`                         |                                                                            |                  |                 |                                                  |
|    5 | Confirm why current path won                                                      | Router          | `show bgp ipv4 unicast <prefix> best-path-reason`                   | Best-path reason is visible if supported                     |                                                                            |                  |                 |                                                  |
|    6 | Decide whether this is an AIGP problem or MED problem                             | Notes           | `AIGP` or `MED`                                                     | Attribute choice matches the traffic-engineering goal        |                                                                            |                  |                 |                                                  |
|    7 | Use AIGP only for controlled multi-AS/internal-domain cost logic                  | Notes           | `<controlled-domain-confirmed>`                                     | AIGP is not being used across unrelated external ASNs        |                                                                            |                  |                 |                                                  |
|    8 | Use MED for neighboring-AS inbound entry preference                               | Notes           | `<same-neighboring-as-or-policy-confirmed>`                         | MED is being used where comparison will actually matter      |                                                                            |                  |                 |                                                  |
|    9 | Confirm whether candidate MED paths come from the same ASN                        | Router          | `show bgp ipv4 unicast <prefix>`                                    | AS path source makes MED comparison expectations clear       |                                                                            |                  |                 |                                                  |
|   10 | Review existing route maps                                                        | Router          | `show running-config                                                | section route-map`                                           | Existing match and set logic is known                                      |                  |                 |                                                  |
|   11 | Review existing prefix lists                                                      | Router          | `show running-config                                                | include ip prefix-list`                                      | Existing prefix match logic is known                                       |                  |                 |                                                  |
|   12 | Review existing BGP policy                                                        | Router          | `show running-config                                                | section router bgp`                                          | Neighbor policy, AIGP, MED tuning, and address-family settings are visible |                  |                 |                                                  |
|   13 | Create prefix list for target prefix                                              | Router          | `ip prefix-list <pl-name> permit <prefix>/<length>`                 | Prefix list matches only the intended route                  |                                                                            |                  |                 |                                                  |
|   14 | Create AIGP route map                                                             | Router          | `route-map <aigp-rm-name> permit 10`                                | AIGP policy sequence exists                                  |                                                                            |                  |                 |                                                  |
|   15 | Match target prefix for AIGP                                                      | Router          | `match ip address prefix-list <pl-name>`                            | Route map matches only intended prefixes                     |                                                                            |                  |                 |                                                  |
|   16 | Set AIGP from IGP metric                                                          | Router          | `set aigp-metric igp-metric`                                        | Matching route receives IGP-derived AIGP metric              |                                                                            |                  |                 |                                                  |
|   17 | Set manual AIGP value if lab requires fixed metric                                | Router          | `set aigp-metric <metric>`                                          | Matching route receives explicit AIGP metric                 |                                                                            |                  |                 |                                                  |
|   18 | Add permissive AIGP fallback sequence                                             | Router          | `route-map <aigp-rm-name> permit 20`                                | Unmatched routes are not accidentally denied                 |                                                                            |                  |                 |                                                  |
|   19 | Enter BGP process                                                                 | Router          | `router bgp <local-asn>`                                            | Router enters BGP configuration mode                         |                                                                            |                  |                 |                                                  |
|   20 | Enter IPv4 unicast address family                                                 | Router          | `address-family ipv4 unicast`                                       | Router enters IPv4 unicast AFI                               |                                                                            |                  |                 |                                                  |
|   21 | Enable AIGP on neighbor                                                           | Router          | `neighbor <neighbor-ip> aigp`                                       | Neighbor can exchange AIGP attribute                         |                                                                            |                  |                 |                                                  |
|   22 | Apply inbound AIGP route map if setting AIGP on received routes                   | Router          | `neighbor <neighbor-ip> route-map <aigp-rm-name> in`                | Received matching prefixes get AIGP policy                   |                                                                            |                  |                 |                                                  |
|   23 | Apply outbound AIGP route map if setting AIGP on advertised routes                | Router          | `neighbor <neighbor-ip> route-map <aigp-rm-name> out`               | Advertised matching prefixes get AIGP policy                 |                                                                            |                  |                 |                                                  |
|   24 | Redistribute IGP into BGP with AIGP route map if required                         | Router          | `redistribute <igp-protocol> <process-id> route-map <aigp-rm-name>` | Redistributed routes receive AIGP metric policy              |                                                                            |                  |                 |                                                  |
|   25 | Soft clear inbound after inbound AIGP policy change                               | Router          | `clear bgp ipv4 unicast <neighbor-ip> soft in`                      | Inbound routes are reprocessed                               |                                                                            |                  |                 |                                                  |
|   26 | Soft clear outbound after outbound AIGP policy change                             | Router          | `clear bgp ipv4 unicast <neighbor-ip> soft out`                     | Outbound routes are re-advertised                            |                                                                            |                  |                 |                                                  |
|   27 | Create MED route map                                                              | Router          | `route-map <med-rm-name> permit 10`                                 | MED policy sequence exists                                   |                                                                            |                  |                 |                                                  |
|   28 | Match target prefix for MED                                                       | Router          | `match ip address prefix-list <pl-name>`                            | Route map matches intended prefix                            |                                                                            |                  |                 |                                                  |
|   29 | Set MED value                                                                     | Router          | `set metric <med-value>`                                            | Matching route receives intended MED                         |                                                                            |                  |                 |                                                  |
|   30 | Add permissive MED fallback sequence                                              | Router          | `route-map <med-rm-name> permit 20`                                 | Unmatched routes are not accidentally denied                 |                                                                            |                  |                 |                                                  |
|   31 | Apply inbound MED route map if manipulating received paths locally                | Router          | `neighbor <neighbor-ip> route-map <med-rm-name> in`                 | Local router changes MED on received matching paths          |                                                                            |                  |                 |                                                  |
|   32 | Apply outbound MED route map if advertising MED to neighbor                       | Router          | `neighbor <neighbor-ip> route-map <med-rm-name> out`                | Neighbor receives matching routes with intended MED          |                                                                            |                  |                 |                                                  |
|   33 | Configure missing MED as worst if required across the AS                          | Router          | `bgp bestpath med missing-as-worst`                                 | Missing MED is treated as least preferred                    |                                                                            |                  |                 |                                                  |
|   34 | Configure default MED for missing MED if required                                 | Router          | `default-metric <metric>`                                           | Routes missing MED use the configured default metric         |                                                                            |                  |                 |                                                  |
|   35 | Enable always-compare MED only if comparing across different AS paths is intended | Router          | `bgp always-compare-med`                                            | MED can be compared across different neighboring ASNs        |                                                                            |                  |                 |                                                  |
|   36 | Enable deterministic MED for consistent MED behavior                              | Router          | `bgp deterministic-med`                                             | MED comparisons are grouped consistently by AS path          |                                                                            |                  |                 |                                                  |
|   37 | Deploy MED comparison tuning consistently across the AS                           | All BGP Routers | `show running-config                                                | include always-compare-med                                   | deterministic-med                                                          | missing-as-worst | default-metric` | All routers use the same MED comparison behavior |
|   38 | Exit address-family mode                                                          | Router          | `exit-address-family`                                               | Router returns to BGP config mode                            |                                                                            |                  |                 |                                                  |
|   39 | Save configuration                                                                | Router          | `copy running-config startup-config`                                | Configuration is saved                                       |                                                                            |                  |                 |                                                  |
|   40 | Verify route-map hit count                                                        | Router          | `show route-map <rm-name>`                                          | Route-map counters increment                                 |                                                                            |                  |                 |                                                  |
|   41 | Verify prefix-list hit count                                                      | Router          | `show ip prefix-list <pl-name>`                                     | Prefix-list counters increment if supported                  |                                                                            |                  |                 |                                                  |
|   42 | Verify BGP path attributes                                                        | Router          | `show bgp ipv4 unicast <prefix>`                                    | AIGP and/or Metric column reflects intended values           |                                                                            |                  |                 |                                                  |
|   43 | Verify best-path reason                                                           | Router          | `show bgp ipv4 unicast <prefix> best-path-reason`                   | AIGP or MED explains the selected path if it was decisive    |                                                                            |                  |                 |                                                  |
|   44 | Verify local RIB install                                                          | Router          | `show ip route <prefix>`                                            | Desired BGP path installs if selected and valid              |                                                                            |                  |                 |                                                  |
|   45 | Verify forwarding path                                                            | Router          | `show ip cef <prefix>`                                              | CEF points toward intended next hop/interface                |                                                                            |                  |                 |                                                  |
|   46 | Verify peer received MED if outbound MED is used                                  | Neighbor Router | `show bgp ipv4 unicast <prefix>`                                    | Neighbor sees intended MED value                             |                                                                            |                  |                 |                                                  |
|   47 | Verify peer received AIGP if AIGP is enabled                                      | Neighbor Router | `show bgp ipv4 unicast <prefix>`                                    | Peer sees AIGP-enabled path behavior if platform displays it |                                                                            |                  |                 |                                                  |
|   48 | Verify actual traffic path                                                        | Router          | `traceroute <destination-ip> source <source-ip>`                    | Traffic follows intended selected BGP path                   |                                                                            |                  |                 |                                                  |


```
# BGP_AIGP_And_MED_AIGP_Neighbor_Enable_Skeleton
configure terminal

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> aigp
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_AIGP_Set_IGP_Metric_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <aigp-rm-name> permit 10
 match ip address prefix-list <pl-name>
 set aigp-metric igp-metric

route-map <aigp-rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> aigp
  neighbor <neighbor-ip> route-map <aigp-rm-name> in
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_AIGP_Set_Manual_Metric_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <aigp-rm-name> permit 10
 match ip address prefix-list <pl-name>
 set aigp-metric <metric>

route-map <aigp-rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> aigp
  neighbor <neighbor-ip> route-map <aigp-rm-name> in
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_MED_Inbound_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <med-rm-name> permit 10
 match ip address prefix-list <pl-name>
 set metric <med-value>

route-map <med-rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <med-rm-name> in
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft in
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_MED_Outbound_Skeleton
configure terminal

ip prefix-list <pl-name> permit <prefix>/<length>

route-map <med-rm-name> permit 10
 match ip address prefix-list <pl-name>
 set metric <med-value>

route-map <med-rm-name> permit 20

router bgp <local-asn>
 address-family ipv4 unicast
  neighbor <neighbor-ip> route-map <med-rm-name> out
 exit-address-family

end
clear bgp ipv4 unicast <neighbor-ip> soft out
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_Missing_MED_Behavior_Skeleton
configure terminal

router bgp <local-asn>
 bgp bestpath med missing-as-worst
 default-metric <metric>

end
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_MED_Comparison_Tuning_Skeleton
configure terminal

router bgp <local-asn>
 bgp deterministic-med
 bgp always-compare-med

end
copy running-config startup-config
```


```
# BGP_AIGP_And_MED_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show bgp ipv4 unicast <prefix> bestpath
show bgp ipv4 unicast <prefix> best-path-reason
show ip route <prefix>
show ip cef <prefix>
show route-map <rm-name>
show ip prefix-list <pl-name>
show running-config | section router bgp
show running-config | section route-map
show running-config | include deterministic-med|always-compare-med|missing-as-worst|default-metric
```


# BGP_AIGP_And_MED_Verification_Commands
| Task                         | Command                                           | Expected Result                                                       |                                                                               |                                                 |
| ---------------------------- | ------------------------------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------------- |
| Verify BGP session state     | `show bgp ipv4 unicast summary`                   | Neighbor is established                                               |                                                                               |                                                 |
| Verify target prefix         | `show bgp ipv4 unicast <prefix>`                  | Prefix has expected candidate paths                                   |                                                                               |                                                 |
| Verify current best path     | `show bgp ipv4 unicast <prefix>`                  | Best path is marked with `>`                                          |                                                                               |                                                 |
| Verify best-path reason      | `show bgp ipv4 unicast <prefix> best-path-reason` | Output shows whether AIGP, MED, or another attribute decided the path |                                                                               |                                                 |
| Verify best path only        | `show bgp ipv4 unicast <prefix> bestpath`         | Only selected best path is displayed if supported                     |                                                                               |                                                 |
| Verify AIGP neighbor config  | `show running-config                              | section router bgp`                                                   | `neighbor <ip> aigp` appears under address-family                             |                                                 |
| Verify AIGP route-map config | `show running-config                              | section route-map`                                                    | Route map contains `set aigp-metric igp-metric` or `set aigp-metric <metric>` |                                                 |
| Verify MED route-map config  | `show running-config                              | section route-map`                                                    | Route map contains `set metric <med-value>`                                   |                                                 |
| Verify route-map hits        | `show route-map <rm-name>`                        | Route-map counters increment                                          |                                                                               |                                                 |
| Verify prefix-list match     | `show ip prefix-list <pl-name>`                   | Prefix list matches intended prefix                                   |                                                                               |                                                 |
| Verify MED value             | `show bgp ipv4 unicast <prefix>`                  | Metric column shows expected MED                                      |                                                                               |                                                 |
| Verify missing MED behavior  | `show running-config                              | include missing-as-worst                                              | default-metric`                                                               | Missing MED behavior is configured consistently |
| Verify deterministic MED     | `show running-config                              | include deterministic-med`                                            | Deterministic MED is enabled where intended                                   |                                                 |
| Verify always-compare MED    | `show running-config                              | include always-compare-med`                                           | MED comparison across different AS paths is enabled only if intended          |                                                 |
| Verify route install         | `show ip route <prefix>`                          | Desired BGP path installs in RIB                                      |                                                                               |                                                 |
| Verify forwarding path       | `show ip cef <prefix>`                            | CEF points toward expected next hop/interface                         |                                                                               |                                                 |
| Verify peer-received MED     | `show bgp ipv4 unicast <prefix>` on peer          | Neighbor sees intended MED if MED was sent outbound                   |                                                                               |                                                 |
| Verify traffic path          | `traceroute <destination-ip> source <source-ip>`  | Traffic follows expected selected path                                |                                                                               |                                                 |
| Verify logs                  | `show logging                                     | include BGP                                                           | ADJCHANGE`                                                                    | No unexpected session reset or churn appears    |


# BGP_AIGP_And_MED_Rollback
| Step | Task                                           | Device | Command                                                  | Expected Result                                     |                                                       |
| ---: | ---------------------------------------------- | ------ | -------------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------- |
|    1 | Identify current BGP AIGP and MED config       | Router | `show running-config                                     | section router bgp`                                 | Neighbor AIGP, route maps, and MED tuning are visible |
|    2 | Identify route maps                            | Router | `show running-config                                     | section route-map`                                  | AIGP and MED route-map sequences are visible          |
|    3 | Identify prefix lists                          | Router | `show ip prefix-list`                                    | Prefix lists used by route maps are visible         |                                                       |
|    4 | Enter configuration mode                       | Router | `configure terminal`                                     | Router enters global configuration mode             |                                                       |
|    5 | Enter BGP process                              | Router | `router bgp <local-asn>`                                 | Router enters BGP configuration mode                |                                                       |
|    6 | Enter IPv4 unicast AFI                         | Router | `address-family ipv4 unicast`                            | Router enters IPv4 unicast address-family           |                                                       |
|    7 | Remove AIGP neighbor enablement                | Router | `no neighbor <neighbor-ip> aigp`                         | AIGP is no longer exchanged with that neighbor      |                                                       |
|    8 | Remove inbound AIGP route map                  | Router | `no neighbor <neighbor-ip> route-map <aigp-rm-name> in`  | Inbound AIGP policy is detached                     |                                                       |
|    9 | Remove outbound AIGP route map                 | Router | `no neighbor <neighbor-ip> route-map <aigp-rm-name> out` | Outbound AIGP policy is detached                    |                                                       |
|   10 | Remove inbound MED route map                   | Router | `no neighbor <neighbor-ip> route-map <med-rm-name> in`   | Inbound MED policy is detached                      |                                                       |
|   11 | Remove outbound MED route map                  | Router | `no neighbor <neighbor-ip> route-map <med-rm-name> out`  | Outbound MED policy is detached                     |                                                       |
|   12 | Exit AFI mode                                  | Router | `exit-address-family`                                    | Router returns to BGP config mode                   |                                                       |
|   13 | Remove missing MED as worst if lab-only        | Router | `no bgp bestpath med missing-as-worst`                   | Missing MED behavior returns to default             |                                                       |
|   14 | Remove default MED if lab-only                 | Router | `no default-metric <metric>`                             | Default MED override is removed                     |                                                       |
|   15 | Remove always-compare MED if lab-only          | Router | `no bgp always-compare-med`                              | MED is no longer compared across different AS paths |                                                       |
|   16 | Remove deterministic MED if lab-only           | Router | `no bgp deterministic-med`                               | Deterministic MED behavior is removed               |                                                       |
|   17 | Remove AIGP route map sequence                 | Router | `no route-map <aigp-rm-name> permit <seq>`               | AIGP route-map sequence is removed                  |                                                       |
|   18 | Remove MED route map sequence                  | Router | `no route-map <med-rm-name> permit <seq>`                | MED route-map sequence is removed                   |                                                       |
|   19 | Remove prefix list if lab-only                 | Router | `no ip prefix-list <pl-name>`                            | Prefix list is removed                              |                                                       |
|   20 | Soft clear inbound if inbound policy changed   | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in`           | Inbound routes are reprocessed                      |                                                       |
|   21 | Soft clear outbound if outbound policy changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out`          | Outbound advertisements are refreshed               |                                                       |
|   22 | Verify rollback                                | Router | `show bgp ipv4 unicast <prefix> best-path-reason`        | Best-path reason reflects rollback                  |                                                       |
|   23 | Save rollback                                  | Router | `copy running-config startup-config`                     | Rollback is saved                                   |                                                       |

# BGP_AIGP_And_MED_Failure_Checks
| Symptom                                     | Command                                                      | What Usually Broke                                                                        |                                                                       |                                         |                                                    |
| ------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | --------------------------------------- | -------------------------------------------------- |
| AIGP has no effect                          | `show running-config                                         | section router bgp`                                                                       | `neighbor <ip> aigp` missing on one or both peers                     |                                         |                                                    |
| AIGP route map has no hits                  | `show route-map <aigp-rm-name>`                              | Prefix list does not match or route map is attached wrong                                 |                                                                       |                                         |                                                    |
| AIGP path loses unexpectedly                | `show bgp ipv4 unicast <prefix> best-path-reason`            | Weight, local preference, locally originated route, or missing AIGP path took priority    |                                                                       |                                         |                                                    |
| AIGP beats shorter AS path                  | `show bgp ipv4 unicast <prefix>`                             | Expected behavior because AIGP is evaluated before AS path length                         |                                                                       |                                         |                                                    |
| AIGP metric looks higher than expected      | `show bgp ipv4 unicast <prefix>`                             | Recursive next-hop cost was added into derived AIGP metric                                |                                                                       |                                         |                                                    |
| AIGP missing on one path                    | `show bgp ipv4 unicast <prefix>`                             | Non-AIGP path may be ignored when AIGP paths exist                                        |                                                                       |                                         |                                                    |
| MED has no effect                           | `show bgp ipv4 unicast <prefix>`                             | Candidate paths are from different neighboring ASNs or earlier attributes already decided |                                                                       |                                         |                                                    |
| Lower MED does not win                      | `show bgp ipv4 unicast <prefix> best-path-reason`            | Weight, local preference, locally originated, AIGP, AS path, or origin decided first      |                                                                       |                                         |                                                    |
| MED route map has no hits                   | `show route-map <med-rm-name>`                               | Prefix list mismatch, wrong neighbor, wrong AFI, or wrong policy direction                |                                                                       |                                         |                                                    |
| MED missing path wins unexpectedly          | `show bgp ipv4 unicast <prefix>`                             | Missing MED is treated like 0 by default                                                  |                                                                       |                                         |                                                    |
| Missing MED should be worst but is not      | `show running-config                                         | include missing-as-worst`                                                                 | `bgp bestpath med missing-as-worst` is missing on one or more routers |                                         |                                                    |
| MED differs across routers in same AS       | `show running-config                                         | include deterministic-med                                                                 | always-compare-med                                                    | missing-as-worst`                       | MED behavior knobs are not configured consistently |
| Always-compare MED causes odd path changes  | `show bgp ipv4 unicast <prefix> best-path-reason`            | MED is now compared across different AS paths, changing normal behavior                   |                                                                       |                                         |                                                    |
| Deterministic MED not behaving consistently | `show running-config                                         | include deterministic-med`                                                                | Feature is not enabled consistently across the AS                     |                                         |                                                    |
| Route disappears after route map            | `show route-map <rm-name>`                                   | Missing permissive fallback sequence caused unmatched routes to be denied                 |                                                                       |                                         |                                                    |
| Peer does not see outbound MED              | Peer `show bgp ipv4 unicast <prefix>`                        | Route map applied inbound instead of outbound, or route was not refreshed                 |                                                                       |                                         |                                                    |
| Soft clear did not update attributes        | `clear bgp ipv4 unicast <neighbor-ip> soft in` or `soft out` | Wrong direction was refreshed                                                             |                                                                       |                                         |                                                    |
| Route is best in BGP but not installed      | `show ip route <prefix>`                                     | RIB failure, next-hop reachability problem, or better administrative distance route       |                                                                       |                                         |                                                    |
| Traffic does not follow selected BGP path   | `show ip cef <prefix>`                                       | CEF/recursive next-hop path differs from what you assumed                                 |                                                                       |                                         |                                                    |
| Hard clear caused avoidable outage          | `show logging                                                | include BGP                                                                               | ADJCHANGE`                                                            | Hard reset used instead of soft refresh |                                                    |


# Attached_Labs
| Lab Name                       | Attach Under This Note | Why                                                                                   |
| ------------------------------ | ---------------------- | ------------------------------------------------------------------------------------- |
| `bgp-aigp-final`               | `BGP_AIGP_And_MED.md`  | Base AIGP mechanism lab for enabling AIGP and understanding AIGP as a best-path input |
| `bgp-aigp-final-example-final` | `BGP_AIGP_And_MED.md`  | AIGP example lab for applied metric behavior                                          |
| `bgp-aigp-med-example-final`   | `BGP_AIGP_And_MED.md`  | Comparison lab showing why AIGP and MED are separate path-selection tools             |