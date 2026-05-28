

```
# Index
# Source_Basis
# BGP_Session_Foundation_eBGP_iBGP_Mental_Model
# BGP_Session_Foundation_eBGP_iBGP_Configuration_Checklist
# BGP_Session_Foundation_eBGP_iBGP_Direct_eBGP_Skeleton
# BGP_Session_Foundation_eBGP_iBGP_Direct_iBGP_Skeleton
# BGP_Session_Foundation_eBGP_iBGP_iBGP_Loopback_Peering_Skeleton
# BGP_Session_Foundation_eBGP_iBGP_eBGP_Loopback_Peering_Skeleton
# BGP_Session_Foundation_eBGP_iBGP_eBGP_To_iBGP_Next_Hop_Self_Skeleton
# BGP_Session_Foundation_eBGP_iBGP_Startup_Verification_Skeleton
# BGP_Session_Foundation_eBGP_iBGP_Verification_Commands
# BGP_Session_Foundation_eBGP_iBGP_Rollback
# BGP_Session_Foundation_eBGP_iBGP_Failure_Checks
# Attached_Labs
```


# BGP_Session_Foundation_eBGP_iBGP_Mental_Model
| Concept                      | Operational Meaning                                                                                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| BGP session                  | A TCP/179 neighbor relationship between two BGP speakers                                                                                                |
| BGP speaker                  | Router running a BGP process                                                                                                                            |
| BGP peer / neighbor          | Router configured with `neighbor <ip> remote-as <asn>`                                                                                                  |
| Local AS                     | ASN configured under `router bgp <local-asn>`                                                                                                           |
| Remote AS                    | ASN configured in the neighbor statement                                                                                                                |
| eBGP                         | Neighbor is in a different AS                                                                                                                           |
| iBGP                         | Neighbor is in the same AS                                                                                                                              |
| eBGP administrative distance | eBGP routes install with AD 20 by default                                                                                                               |
| iBGP administrative distance | iBGP routes install with AD 200 by default                                                                                                              |
| eBGP TTL                     | eBGP uses TTL 1 by default, so direct reachability is expected                                                                                          |
| iBGP TTL                     | iBGP supports multihop behavior by default                                                                                                              |
| eBGP next-hop behavior       | eBGP usually rewrites next hop to the advertising router                                                                                                |
| iBGP next-hop behavior       | iBGP usually preserves next hop, which can break route validity if the next hop is unreachable                                                          |
| AS path prepend              | eBGP prepends the local AS when advertising to another AS                                                                                               |
| iBGP AS path behavior        | iBGP does not prepend the local AS inside the same AS                                                                                                   |
| iBGP full mesh rule          | A route learned from one iBGP peer is not advertised to another iBGP peer unless using route reflection or confederation                                |
| Loopback peering             | More stable BGP peering method because loopbacks remain reachable through IGP path changes                                                              |
| `update-source`              | Forces BGP packets to source from a specific interface, commonly Loopback0                                                                              |
| `ebgp-multihop`              | Required when eBGP peers are not directly connected                                                                                                     |
| AFI/SAFI activation          | Neighbor must be active under the address family before exchanging routes when default IPv4 activation is disabled                                      |
| `State/PfxRcd` number        | BGP session is established; number shows received prefixes                                                                                              |
| `Idle` or `Active`           | BGP session is not established                                                                                                                          |
| Blunt rule                   | eBGP/iBGP foundation is not about policy yet. First prove TCP reachability, source IP correctness, AS correctness, AF activation, and next-hop validity |
# BGP_Session_Foundation_eBGP_iBGP_Configuration_Checklist
| Step | Task                                                                            | Device         | Command                                                           | Expected Result                                                              |                                                                       |                                 |
| ---: | ------------------------------------------------------------------------------- | -------------- | ----------------------------------------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------- |
|    1 | Confirm local AS number                                                         | Router / Notes | `<local-asn>`                                                     | Local BGP AS is known                                                        |                                                                       |                                 |
|    2 | Confirm neighbor IP                                                             | Router / Notes | `<neighbor-ip>`                                                   | Peer IP address is known                                                     |                                                                       |                                 |
|    3 | Confirm remote AS number                                                        | Router / Notes | `<remote-asn>`                                                    | Peer AS is known                                                             |                                                                       |                                 |
|    4 | Classify session type                                                           | Router / Notes | `local-asn == remote-asn` or `local-asn != remote-asn`            | Same AS means iBGP; different AS means eBGP                                  |                                                                       |                                 |
|    5 | Confirm peer-facing interface state                                             | Router         | `show ip interface brief`                                         | Peer-facing interface is up/up                                               |                                                                       |                                 |
|    6 | Confirm local source IP                                                         | Router         | `show running-config interface <source-interface>`                | Source interface has expected IP address                                     |                                                                       |                                 |
|    7 | Confirm route to neighbor                                                       | Router         | `show ip route <neighbor-ip>`                                     | Router has a route to the peer address                                       |                                                                       |                                 |
|    8 | Confirm sourced ping to neighbor                                                | Router         | `ping <neighbor-ip> source <local-source-ip>`                     | Peer is reachable from the BGP source address                                |                                                                       |                                 |
|    9 | Confirm reverse reachability                                                    | Peer Router    | `ping <local-source-ip> source <neighbor-source-ip>`              | Peer can reach local BGP source address                                      |                                                                       |                                 |
|   10 | Confirm TCP/179 is not blocked                                                  | Router / Path  | `show access-lists` or firewall/path check                        | ACL/firewall path does not block TCP/179                                     |                                                                       |                                 |
|   11 | Confirm router ID plan                                                          | Router / Notes | `<router-id>`                                                     | Stable BGP router ID is selected                                             |                                                                       |                                 |
|   12 | Enter global configuration mode                                                 | Router         | `configure terminal`                                              | Router enters configuration mode                                             |                                                                       |                                 |
|   13 | Start BGP process                                                               | Router         | `router bgp <local-asn>`                                          | Router enters BGP configuration mode                                         |                                                                       |                                 |
|   14 | Configure stable router ID                                                      | Router         | `bgp router-id <router-id>`                                       | BGP router ID is deterministic                                               |                                                                       |                                 |
|   15 | Enable neighbor change logging                                                  | Router         | `bgp log-neighbor-changes`                                        | BGP neighbor events are logged                                               |                                                                       |                                 |
|   16 | Standardize explicit AF activation                                              | Router         | `no bgp default ipv4-unicast`                                     | IPv4 unicast neighbors must be activated under AFI                           |                                                                       |                                 |
|   17 | Configure neighbor remote AS                                                    | Router         | `neighbor <neighbor-ip> remote-as <remote-asn>`                   | Neighbor is defined with correct peer AS                                     |                                                                       |                                 |
|   18 | Add neighbor description                                                        | Router         | `neighbor <neighbor-ip> description <description>`                | Peer purpose is documented                                                   |                                                                       |                                 |
|   19 | Configure BGP password only if both peers use it                                | Router         | `neighbor <neighbor-ip> password <password>`                      | TCP MD5/authentication matches peer                                          |                                                                       |                                 |
|   20 | Configure timers only if required by design                                     | Router         | `neighbor <neighbor-ip> timers <keepalive> <holdtime>`            | Neighbor timers are set deliberately                                         |                                                                       |                                 |
|   21 | Configure loopback source if peering to loopback                                | Router         | `neighbor <neighbor-loopback-ip> update-source Loopback0`         | BGP packets source from Loopback0                                            |                                                                       |                                 |
|   22 | Configure eBGP multihop only for non-direct eBGP                                | Router         | `neighbor <neighbor-ip> ebgp-multihop <ttl>`                      | eBGP can reach non-direct peer                                               |                                                                       |                                 |
|   23 | Enter IPv4 unicast address family                                               | Router         | `address-family ipv4 unicast`                                     | Router enters IPv4 unicast AFI                                               |                                                                       |                                 |
|   24 | Activate neighbor under IPv4 unicast                                            | Router         | `neighbor <neighbor-ip> activate`                                 | Neighbor can exchange IPv4 unicast NLRI                                      |                                                                       |                                 |
|   25 | Configure `next-hop-self` toward iBGP peer when advertising eBGP-learned routes | Router         | `neighbor <ibgp-neighbor-ip> next-hop-self`                       | iBGP peer receives reachable next hop                                        |                                                                       |                                 |
|   26 | Advertise local prefix only if exact route exists                               | Router         | `network <network> mask <subnet-mask>`                            | Prefix is eligible for local BGP origination                                 |                                                                       |                                 |
|   27 | Verify exact route for network statement                                        | Router         | `show ip route <network> <subnet-mask>`                           | Exact prefix exists in RIB                                                   |                                                                       |                                 |
|   28 | Exit address-family mode                                                        | Router         | `exit-address-family`                                             | Router returns to BGP config mode                                            |                                                                       |                                 |
|   29 | Save configuration                                                              | Router         | `copy running-config startup-config`                              | Configuration is saved                                                       |                                                                       |                                 |
|   30 | Verify BGP configuration                                                        | Router         | `show running-config                                              | section router bgp`                                                          | BGP process, neighbor, AFI, update-source, and activation are visible |                                 |
|   31 | Verify BGP summary                                                              | Router         | `show bgp ipv4 unicast summary`                                   | Neighbor appears in summary                                                  |                                                                       |                                 |
|   32 | Confirm established state                                                       | Router         | `show bgp ipv4 unicast summary`                                   | `State/PfxRcd` shows a number, not `Idle` or `Active`                        |                                                                       |                                 |
|   33 | Verify neighbor detail                                                          | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | State is Established and remote AS/type are correct                          |                                                                       |                                 |
|   34 | Verify session type                                                             | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Output shows internal or external link as expected                           |                                                                       |                                 |
|   35 | Verify message counters                                                         | Router         | `show bgp ipv4 unicast summary`                                   | `MsgRcvd` and `MsgSent` increment                                            |                                                                       |                                 |
|   36 | Verify queues                                                                   | Router         | `show bgp ipv4 unicast summary`                                   | `InQ` and `OutQ` are normally 0                                              |                                                                       |                                 |
|   37 | Verify BGP table                                                                | Router         | `show bgp ipv4 unicast`                                           | BGP table contains expected prefixes                                         |                                                                       |                                 |
|   38 | Verify specific prefix attributes                                               | Router         | `show bgp ipv4 unicast <prefix>`                                  | Path source, next hop, AS path, localpref, weight, and best path are visible |                                                                       |                                 |
|   39 | Verify route installed in RIB                                                   | Router         | `show ip route <prefix>`                                          | BGP route installs if valid and preferred                                    |                                                                       |                                 |
|   40 | Verify eBGP route AD                                                            | Router         | `show ip route <ebgp-prefix>`                                     | eBGP route shows AD 20 unless changed                                        |                                                                       |                                 |
|   41 | Verify iBGP route AD                                                            | Router         | `show ip route <ibgp-prefix>`                                     | iBGP route shows AD 200 unless changed                                       |                                                                       |                                 |
|   42 | Verify next-hop reachability                                                    | Router         | `show ip route <next-hop-ip>`                                     | BGP next hop is reachable                                                    |                                                                       |                                 |
|   43 | Verify advertised routes                                                        | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected prefixes are sent to peer                                           |                                                                       |                                 |
|   44 | Verify received prefix count                                                    | Router         | `show bgp ipv4 unicast summary`                                   | Prefix count matches expectation                                             |                                                                       |                                 |
|   45 | Verify logs                                                                     | Router         | `show logging                                                     | include BGP                                                                  | ADJCHANGE`                                                            | BGP session changes are visible |
```
# BGP_Session_Foundation_eBGP_iBGP_Direct_eBGP_Skeleton
configure terminal

router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <ebgp-neighbor-ip> remote-as <remote-asn>
 neighbor <ebgp-neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <ebgp-neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Session_Foundation_eBGP_iBGP_Direct_iBGP_Skeleton
configure terminal

router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <ibgp-neighbor-ip> remote-as <local-asn>
 neighbor <ibgp-neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <ibgp-neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Session_Foundation_eBGP_iBGP_iBGP_Loopback_Peering_Skeleton
configure terminal

interface Loopback0
 ip address <local-loopback-ip> 255.255.255.255

router bgp <local-asn>
 bgp router-id <local-loopback-ip>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <remote-loopback-ip> remote-as <local-asn>
 neighbor <remote-loopback-ip> description <description>
 neighbor <remote-loopback-ip> update-source Loopback0

 address-family ipv4 unicast
  neighbor <remote-loopback-ip> activate
  network <local-loopback-ip> mask 255.255.255.255
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Session_Foundation_eBGP_iBGP_eBGP_Loopback_Peering_Skeleton
configure terminal

interface Loopback0
 ip address <local-loopback-ip> 255.255.255.255

router bgp <local-asn>
 bgp router-id <local-loopback-ip>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <remote-loopback-ip> remote-as <remote-asn>
 neighbor <remote-loopback-ip> description <description>
 neighbor <remote-loopback-ip> update-source Loopback0
 neighbor <remote-loopback-ip> ebgp-multihop <ttl>

 address-family ipv4 unicast
  neighbor <remote-loopback-ip> activate
  network <local-loopback-ip> mask 255.255.255.255
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Session_Foundation_eBGP_iBGP_eBGP_To_iBGP_Next_Hop_Self_Skeleton
configure terminal

router bgp <local-asn>
 no bgp default ipv4-unicast
 neighbor <ebgp-neighbor-ip> remote-as <external-asn>
 neighbor <ibgp-neighbor-ip> remote-as <local-asn>

 address-family ipv4 unicast
  neighbor <ebgp-neighbor-ip> activate
  neighbor <ibgp-neighbor-ip> activate
  neighbor <ibgp-neighbor-ip> next-hop-self
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_Session_Foundation_eBGP_iBGP_Startup_Verification_Skeleton
show ip interface brief
show ip route <neighbor-ip>
ping <neighbor-ip> source <local-source-ip>
show running-config | section router bgp
show bgp ipv4 unicast summary
show bgp ipv4 unicast neighbors <neighbor-ip>
show bgp ipv4 unicast
show bgp ipv4 unicast <prefix>
show ip route <prefix>
show logging | include BGP|ADJCHANGE
```


# BGP_Session_Foundation_eBGP_iBGP_Verification_Commands
| Task                          | Command                                                           | Expected Result                                                                     |                                                              |                              |
| ----------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------ | ---------------------------- |
| Verify interface state        | `show ip interface brief`                                         | Peer-facing interface or loopback path is up/up                                     |                                                              |                              |
| Verify local source interface | `show running-config interface <source-interface>`                | Source interface has expected IP address                                            |                                                              |                              |
| Verify route to peer          | `show ip route <neighbor-ip>`                                     | Route to neighbor exists                                                            |                                                              |                              |
| Verify sourced reachability   | `ping <neighbor-ip> source <local-source-ip>`                     | Peer is reachable from BGP source IP                                                |                                                              |                              |
| Verify reverse reachability   | `ping <local-source-ip> source <neighbor-source-ip>` on peer      | Peer can reach local BGP source IP                                                  |                                                              |                              |
| Verify BGP process            | `show running-config                                              | section router bgp`                                                                 | Local AS, router ID, neighbor, and AFI config are present    |                              |
| Verify neighbor remote AS     | `show running-config                                              | section router bgp`                                                                 | Neighbor has correct `remote-as` value                       |                              |
| Verify update-source          | `show running-config                                              | section router bgp`                                                                 | Loopback peering has `neighbor <ip> update-source Loopback0` |                              |
| Verify eBGP multihop          | `show running-config                                              | section router bgp`                                                                 | Non-direct eBGP has `neighbor <ip> ebgp-multihop <ttl>`      |                              |
| Verify AF activation          | `show running-config                                              | section address-family`                                                             | Neighbor is activated under IPv4 unicast                     |                              |
| Verify BGP summary            | `show bgp ipv4 unicast summary`                                   | Neighbor appears                                                                    |                                                              |                              |
| Verify established session    | `show bgp ipv4 unicast summary`                                   | `State/PfxRcd` shows a number                                                       |                                                              |                              |
| Verify neighbor detail        | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | State is Established                                                                |                                                              |                              |
| Verify session type           | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Neighbor shows internal or external link as expected                                |                                                              |                              |
| Verify negotiated timers      | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Keepalive and hold timers are visible                                               |                                                              |                              |
| Verify capabilities           | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Route refresh, 4-byte ASN, and address-family capabilities are visible if supported |                                                              |                              |
| Verify BGP table              | `show bgp ipv4 unicast`                                           | Expected prefixes appear                                                            |                                                              |                              |
| Verify specific prefix        | `show bgp ipv4 unicast <prefix>`                                  | Path source, next hop, AS path, and best path are visible                           |                                                              |                              |
| Verify next-hop reachability  | `show ip route <next-hop-ip>`                                     | BGP next hop is reachable                                                           |                                                              |                              |
| Verify RIB install            | `show ip route <prefix>`                                          | BGP route installs if valid and preferred                                           |                                                              |                              |
| Verify eBGP AD                | `show ip route <ebgp-prefix>`                                     | Route shows BGP AD 20 unless changed                                                |                                                              |                              |
| Verify iBGP AD                | `show ip route <ibgp-prefix>`                                     | Route shows BGP AD 200 unless changed                                               |                                                              |                              |
| Verify advertised routes      | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected prefixes are advertised to peer                                            |                                                              |                              |
| Verify queues                 | `show bgp ipv4 unicast summary`                                   | `InQ` and `OutQ` are usually 0                                                      |                                                              |                              |
| Verify logs                   | `show logging                                                     | include BGP                                                                         | ADJCHANGE`                                                   | Neighbor changes are visible |

# BGP_Session_Foundation_eBGP_iBGP_Rollback
| Step | Task                                                 | Device | Command                                                   | Expected Result                                  |                                            |
| ---: | ---------------------------------------------------- | ------ | --------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------ |
|    1 | Identify BGP process                                 | Router | `show running-config                                      | section router bgp`                              | Local AS and BGP configuration are visible |
|    2 | Identify active neighbors                            | Router | `show bgp ipv4 unicast summary`                           | Current BGP peers and state are visible          |                                            |
|    3 | Identify advertised networks                         | Router | `show running-config                                      | section router bgp`                              | `network` statements are visible           |
|    4 | Enter configuration mode                             | Router | `configure terminal`                                      | Router enters global configuration mode          |                                            |
|    5 | Enter BGP process                                    | Router | `router bgp <local-asn>`                                  | Router enters BGP configuration mode             |                                            |
|    6 | Enter IPv4 unicast AFI                               | Router | `address-family ipv4 unicast`                             | Router enters IPv4 unicast address-family mode   |                                            |
|    7 | Remove network statement                             | Router | `no network <network> mask <subnet-mask>`                 | Prefix is no longer originated                   |                                            |
|    8 | Remove next-hop-self if configured                   | Router | `no neighbor <neighbor-ip> next-hop-self`                 | Next-hop rewrite is removed                      |                                            |
|    9 | Deactivate neighbor from AFI                         | Router | `no neighbor <neighbor-ip> activate`                      | Neighbor no longer exchanges IPv4 unicast NLRI   |                                            |
|   10 | Exit address-family mode                             | Router | `exit-address-family`                                     | Router returns to BGP config mode                |                                            |
|   11 | Remove eBGP multihop if configured                   | Router | `no neighbor <neighbor-ip> ebgp-multihop <ttl>`           | eBGP multihop is removed                         |                                            |
|   12 | Remove update-source if configured                   | Router | `no neighbor <neighbor-ip> update-source <interface>`     | Update-source override is removed                |                                            |
|   13 | Remove neighbor password if configured               | Router | `no neighbor <neighbor-ip> password <password>`           | Neighbor authentication is removed               |                                            |
|   14 | Remove neighbor timers if configured                 | Router | `no neighbor <neighbor-ip> timers <keepalive> <holdtime>` | Custom timers are removed                        |                                            |
|   15 | Remove neighbor                                      | Router | `no neighbor <neighbor-ip> remote-as <remote-asn>`        | BGP neighbor is removed                          |                                            |
|   16 | Remove BGP process only if full rollback is intended | Router | `no router bgp <local-asn>`                               | Entire BGP process is removed                    |                                            |
|   17 | Verify neighbor removed                              | Router | `show bgp ipv4 unicast summary`                           | Neighbor no longer appears                       |                                            |
|   18 | Verify route removal                                 | Router | `show bgp ipv4 unicast <prefix>`                          | Prefix is absent or no longer locally originated |                                            |
|   19 | Save rollback                                        | Router | `copy running-config startup-config`                      | Rollback is saved                                |                                            |



# BGP_Session_Foundation_eBGP_iBGP_Failure_Checks
| Symptom                                          | Command                                                            | What Usually Broke                                                                                       |                                                                            |                                                                                        |
| ------------------------------------------------ | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Neighbor stuck Idle                              | `show bgp ipv4 unicast summary`                                    | Missing route to peer, wrong neighbor IP, peer not configured, or TCP/179 blocked                        |                                                                            |                                                                                        |
| Neighbor stuck Active                            | `show bgp ipv4 unicast summary`                                    | Router is trying TCP/179 but cannot complete the session                                                 |                                                                            |                                                                                        |
| Neighbor never appears                           | `show running-config                                               | section router bgp`                                                                                      | Neighbor statement missing or wrong BGP AS process                         |                                                                                        |
| eBGP direct peer fails                           | `show ip route <neighbor-ip>`                                      | Peer not directly reachable or wrong interface/IP                                                        |                                                                            |                                                                                        |
| eBGP loopback peer fails                         | `show running-config                                               | section router bgp`                                                                                      | Missing `update-source`, missing `ebgp-multihop`, or no route to loopback  |                                                                                        |
| iBGP loopback peer fails                         | `ping <remote-loopback-ip> source <local-loopback-ip>`             | IGP/static route to loopback is missing                                                                  |                                                                            |                                                                                        |
| Peer rejects connection                          | `show logging                                                      | include BGP`                                                                                             | Neighbor IP/source does not match configured neighbor statement            |                                                                                        |
| Wrong AS configured                              | `show bgp ipv4 unicast neighbors <neighbor-ip>`                    | `remote-as` does not match peer's local BGP AS                                                           |                                                                            |                                                                                        |
| Authentication failure                           | `show logging                                                      | include BGP`                                                                                             | Neighbor password mismatch                                                 |                                                                                        |
| Session flaps repeatedly                         | `show logging                                                      | include BGP                                                                                              | ADJCHANGE`                                                                 | Timers too aggressive, unstable link, duplicate router ID, or reachability instability |
| Session established but receives 0 prefixes      | `show bgp ipv4 unicast summary`                                    | Peer has no advertised routes, AFI not activated, policy blocks routes, or no exact network route exists |                                                                            |                                                                                        |
| Neighbor established but no IPv4 routes exchange | `show running-config                                               | section address-family`                                                                                  | Neighbor not activated under IPv4 unicast                                  |                                                                                        |
| Network statement does not advertise route       | `show ip route <network> <subnet-mask>`                            | Exact route does not exist in RIB                                                                        |                                                                            |                                                                                        |
| eBGP route installs but iBGP peer cannot use it  | `show bgp ipv4 unicast <prefix>`                                   | iBGP peer cannot reach the preserved eBGP next hop                                                       |                                                                            |                                                                                        |
| Route appears valid but not best                 | `show bgp ipv4 unicast <prefix>`                                   | Another BGP path wins best-path selection                                                                |                                                                            |                                                                                        |
| Route is best but not in RIB                     | `show bgp ipv4 unicast` and `show ip route <prefix>`               | Better administrative distance route exists or RIB failure                                               |                                                                            |                                                                                        |
| iBGP route not advertised to another iBGP peer   | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes`  | iBGP full-mesh rule blocks re-advertisement                                                              |                                                                            |                                                                                        |
| Full iBGP mesh missing                           | Topology review and `show bgp ipv4 unicast summary`                | iBGP peers are not fully meshed and no route reflector/confederation exists                              |                                                                            |                                                                                        |
| eBGP peer learned route with local AS in AS path | `show bgp ipv4 unicast <prefix>`                                   | AS path loop prevention rejects the route                                                                |                                                                            |                                                                                        |
| Prefix has unreachable next hop                  | `show bgp ipv4 unicast <prefix>` and `show ip route <next-hop-ip>` | Next hop is inaccessible                                                                                 |                                                                            |                                                                                        |
| BGP packets sourced from wrong IP                | `show bgp ipv4 unicast neighbors <neighbor-ip>`                    | Missing or wrong `update-source`                                                                         |                                                                            |                                                                                        |
| Queues are not zero                              | `show bgp ipv4 unicast summary`                                    | Peer/control-plane issue, stuck updates, or instability                                                  |                                                                            |                                                                                        |
| TCP reachable but BGP still down                 | `show running-config                                               | section router bgp`                                                                                      | AS mismatch, password mismatch, source mismatch, TTL issue, or AF mismatch |                                                                                        |
| Soft policy change did not affect routes         | `clear bgp ipv4 unicast <neighbor-ip> soft in`                     | Policy was changed but routes were not refreshed                                                         |                                                                            |                                                                                        |
| Hard clear caused avoidable outage               | `show logging                                                      | include BGP`                                                                                             | Used hard clear when soft clear would have worked                          |                                                                                        |
```

```

# Attached_Labs
| Lab Name                     | Attach Under This Note                | Why                                                                                                  |
| ---------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `bgp-ebgp-two-routers-final` | `BGP_Session_Foundation_eBGP_iBGP.md` | Basic direct eBGP session formation between two routers                                              |
| `bgp-ebp-two-routers-final`  | `BGP_Session_Foundation_eBGP_iBGP.md` | Likely typo of eBGP two-router lab; attach here instead of creating duplicate note                   |
| `bgp-ebgp-three-ases-final`  | `BGP_Session_Foundation_eBGP_iBGP.md` | Basic multi-AS eBGP session and AS path visibility                                                   |
| `bgp-ibgp-two-routers-final` | `BGP_Session_Foundation_eBGP_iBGP.md` | Basic iBGP session formation inside one AS                                                           |
| `bgp-ebgp-ibgp-final`        | `BGP_Session_Foundation_eBGP_iBGP.md` | Mixed eBGP and iBGP baseline behavior                                                                |
| `bgp-ccnp-route-lab-final`   | `BGP_Session_Foundation_eBGP_iBGP.md` | Broad CCNP BGP session baseline lab that reinforces neighbor state, route exchange, and verification |
# Attached_Labs
| Lab Name                     | Attach Under This Note                | Why                                                                                                  |
| ---------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `bgp-ebgp-two-routers-final` | `BGP_Session_Foundation_eBGP_iBGP.md` | Basic direct eBGP session formation between two routers                                              |
| `bgp-ebp-two-routers-final`  | `BGP_Session_Foundation_eBGP_iBGP.md` | Likely typo of eBGP two-router lab; attach here instead of creating duplicate note                   |
| `bgp-ebgp-three-ases-final`  | `BGP_Session_Foundation_eBGP_iBGP.md` | Basic multi-AS eBGP session and AS path visibility                                                   |
| `bgp-ibgp-two-routers-final` | `BGP_Session_Foundation_eBGP_iBGP.md` | Basic iBGP session formation inside one AS                                                           |
| `bgp-ebgp-ibgp-final`        | `BGP_Session_Foundation_eBGP_iBGP.md` | Mixed eBGP and iBGP baseline behavior                                                                |
| `bgp-ccnp-route-lab-final`   | `BGP_Session_Foundation_eBGP_iBGP.md` | Broad CCNP BGP session baseline lab that reinforces neighbor state, route exchange, and verification |
