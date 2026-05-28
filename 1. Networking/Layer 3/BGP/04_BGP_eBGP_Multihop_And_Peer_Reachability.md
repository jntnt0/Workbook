

```
# Index
# Source_Basis
# BGP_eBGP_Multihop_And_Peer_Reachability_Mental_Model
# BGP_eBGP_Multihop_And_Peer_Reachability_Configuration_Checklist
# BGP_eBGP_Multihop_And_Peer_Reachability_Loopback_eBGP_Skeleton
# BGP_eBGP_Multihop_And_Peer_Reachability_Static_Route_To_Peer_Skeleton
# BGP_eBGP_Multihop_And_Peer_Reachability_IGP_To_Peer_Loopbacks_Skeleton
# BGP_eBGP_Multihop_And_Peer_Reachability_TTL_Security_Skeleton
# BGP_eBGP_Multihop_And_Peer_Reachability_Authentication_Skeleton
# BGP_eBGP_Multihop_And_Peer_Reachability_Verification_Skeleton
# BGP_eBGP_Multihop_And_Peer_Reachability_Verification_Commands
# BGP_eBGP_Multihop_And_Peer_Reachability_Rollback
# BGP_eBGP_Multihop_And_Peer_Reachability_Failure_Checks
# Attached_Labs
```

BGP_eBGP_Multihop_And_Peer_Reachability_Mental_Model

| Concept                     | Operational Meaning                                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Direct eBGP                 | eBGP peer is directly connected and normally works with TTL 1                                                         |
| eBGP multihop               | Allows eBGP peers to form a session across more than one router hop                                                   |
| TTL 1 default               | Normal eBGP packets expire if the peer is not directly connected                                                      |
| `ebgp-multihop`             | Raises the TTL for a specific eBGP neighbor                                                                           |
| Loopback peering            | Uses loopback addresses as BGP endpoints for stability across physical link failure                                   |
| `update-source`             | Forces BGP packets to source from the loopback or chosen interface                                                    |
| Peer source matching        | The received BGP packet source IP must match the configured neighbor IP                                               |
| Route to peer               | Each router must have a route to the other router’s BGP source address                                                |
| Reverse reachability        | Both routers must route back to each other’s BGP source addresses                                                     |
| TCP/179                     | BGP rides over TCP port 179; IP reachability alone is not enough if TCP/179 is blocked                                |
| eBGP multihop security risk | Raising TTL allows more distant devices to try reaching the BGP listener unless filtered or protected                 |
| TTL security                | Alternative hardening method that expects the peer to be within a configured hop distance                             |
| AFI activation              | Neighbor must be activated under IPv4 unicast if default IPv4 activation is disabled                                  |
| Normal failure state        | `Idle` or `Active` means the session is not established                                                               |
| Established state           | A number in `State/PfxRcd` means the session is established                                                           |
| Blunt rule                  | For eBGP multihop, do not touch attributes until route-to-peer, source IP, TTL, TCP/179, and AF activation are proven |

BGP_eBGP_Multihop_And_Peer_Reachability_Configuration_Checklist

| Step | Task                                                                 | Device         | Command                                                           | Expected Result                                                                      |                                                                             |                                                  |
| ---: | -------------------------------------------------------------------- | -------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- | ------------------------------------------------ |
|    1 | Confirm local AS number                                              | Router / Notes | `<local-asn>`                                                     | Local BGP AS is known                                                                |                                                                             |                                                  |
|    2 | Confirm remote AS number                                             | Router / Notes | `<remote-asn>`                                                    | Remote BGP AS is known and differs from local AS                                     |                                                                             |                                                  |
|    3 | Confirm neighbor endpoint address                                    | Router / Notes | `<neighbor-loopback-or-remote-ip>`                                | Peer address used in BGP neighbor statement is known                                 |                                                                             |                                                  |
|    4 | Confirm local source address                                         | Router / Notes | `<local-loopback-or-source-ip>`                                   | Local BGP source address is known                                                    |                                                                             |                                                  |
|    5 | Confirm session type                                                 | Router / Notes | `eBGP multihop`                                                   | Local AS and remote AS differ, and peers are not directly connected or use loopbacks |                                                                             |                                                  |
|    6 | Confirm physical path interface state                                | Router         | `show ip interface brief`                                         | Interfaces along local path are up/up                                                |                                                                             |                                                  |
|    7 | Confirm local loopback if used                                       | Router         | `show ip interface brief                                          | include Loopback`                                                                    | Loopback interface is up/up                                                 |                                                  |
|    8 | Confirm local source interface config                                | Router         | `show running-config interface <source-interface>`                | Source interface has expected IP address                                             |                                                                             |                                                  |
|    9 | Confirm route to neighbor endpoint                                   | Router         | `show ip route <neighbor-ip>`                                     | Router has a specific path to the neighbor endpoint                                  |                                                                             |                                                  |
|   10 | Confirm neighbor has route back to local source                      | Peer Router    | `show ip route <local-source-ip>`                                 | Peer has a path back to local BGP source                                             |                                                                             |                                                  |
|   11 | Confirm sourced ping to peer                                         | Router         | `ping <neighbor-ip> source <local-source-ip>`                     | Ping succeeds from the same source BGP will use                                      |                                                                             |                                                  |
|   12 | Confirm reverse sourced ping                                         | Peer Router    | `ping <local-source-ip> source <neighbor-source-ip>`              | Peer can reach the local source address                                              |                                                                             |                                                  |
|   13 | Confirm TCP/179 is not blocked                                       | Router / Path  | `show access-lists` or path firewall check                        | ACL/firewall path does not block TCP/179                                             |                                                                             |                                                  |
|   14 | Confirm desired TTL value                                            | Router / Notes | `<ttl-value>`                                                     | TTL is large enough to reach peer, usually slightly above actual hop count           |                                                                             |                                                  |
|   15 | Confirm whether TTL security will be used instead of normal multihop | Router / Notes | `ebgp-multihop` or `ttl-security`                                 | Only one approach is selected deliberately                                           |                                                                             |                                                  |
|   16 | Enter global configuration mode                                      | Router         | `configure terminal`                                              | Router enters configuration mode                                                     |                                                                             |                                                  |
|   17 | Start BGP process                                                    | Router         | `router bgp <local-asn>`                                          | Router enters BGP configuration mode                                                 |                                                                             |                                                  |
|   18 | Set stable router ID                                                 | Router         | `bgp router-id <router-id>`                                       | Router ID is deterministic                                                           |                                                                             |                                                  |
|   19 | Enable neighbor logging                                              | Router         | `bgp log-neighbor-changes`                                        | Neighbor state changes are logged                                                    |                                                                             |                                                  |
|   20 | Standardize explicit AF activation                                   | Router         | `no bgp default ipv4-unicast`                                     | Neighbors must be activated under AFI                                                |                                                                             |                                                  |
|   21 | Configure eBGP neighbor remote AS                                    | Router         | `neighbor <neighbor-ip> remote-as <remote-asn>`                   | Neighbor is defined as eBGP peer                                                     |                                                                             |                                                  |
|   22 | Add neighbor description                                             | Router         | `neighbor <neighbor-ip> description <description>`                | Peer purpose is documented                                                           |                                                                             |                                                  |
|   23 | Configure update source if using loopback or non-default source      | Router         | `neighbor <neighbor-ip> update-source <source-interface>`         | BGP packets use the expected source IP                                               |                                                                             |                                                  |
|   24 | Configure eBGP multihop                                              | Router         | `neighbor <neighbor-ip> ebgp-multihop <ttl>`                      | eBGP packets can reach non-direct peer                                               |                                                                             |                                                  |
|   25 | Configure neighbor password only if both peers use it                | Router         | `neighbor <neighbor-ip> password <password>`                      | BGP authentication matches peer                                                      |                                                                             |                                                  |
|   26 | Configure timers only if required                                    | Router         | `neighbor <neighbor-ip> timers <keepalive> <holdtime>`            | Timer behavior is set intentionally                                                  |                                                                             |                                                  |
|   27 | Enter IPv4 unicast address family                                    | Router         | `address-family ipv4 unicast`                                     | Router enters IPv4 unicast AFI                                                       |                                                                             |                                                  |
|   28 | Activate neighbor under IPv4 unicast                                 | Router         | `neighbor <neighbor-ip> activate`                                 | Neighbor can exchange IPv4 unicast NLRI                                              |                                                                             |                                                  |
|   29 | Advertise local prefix only if exact route exists                    | Router         | `network <network> mask <subnet-mask>`                            | Prefix is eligible for BGP advertisement                                             |                                                                             |                                                  |
|   30 | Verify exact route for network statement                             | Router         | `show ip route <network> <subnet-mask>`                           | Exact route exists in RIB                                                            |                                                                             |                                                  |
|   31 | Exit address-family mode                                             | Router         | `exit-address-family`                                             | Router returns to BGP config mode                                                    |                                                                             |                                                  |
|   32 | Save configuration                                                   | Router         | `copy running-config startup-config`                              | Configuration is saved                                                               |                                                                             |                                                  |
|   33 | Verify BGP config                                                    | Router         | `show running-config                                              | section router bgp`                                                                  | Neighbor, remote AS, update-source, multihop, and AF activation are visible |                                                  |
|   34 | Verify route to peer still exists                                    | Router         | `show ip route <neighbor-ip>`                                     | Route to peer endpoint remains present                                               |                                                                             |                                                  |
|   35 | Verify sourced ping still works                                      | Router         | `ping <neighbor-ip> source <local-source-ip>`                     | Source-specific reachability succeeds                                                |                                                                             |                                                  |
|   36 | Verify BGP summary                                                   | Router         | `show bgp ipv4 unicast summary`                                   | Neighbor appears in summary                                                          |                                                                             |                                                  |
|   37 | Confirm established state                                            | Router         | `show bgp ipv4 unicast summary`                                   | `State/PfxRcd` shows a number                                                        |                                                                             |                                                  |
|   38 | Verify neighbor detail                                               | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Neighbor state, external link type, source, TTL, and capabilities are visible        |                                                                             |                                                  |
|   39 | Verify BGP table                                                     | Router         | `show bgp ipv4 unicast`                                           | Expected prefixes appear                                                             |                                                                             |                                                  |
|   40 | Verify specific prefix                                               | Router         | `show bgp ipv4 unicast <prefix>`                                  | Next hop, AS path, and best path are visible                                         |                                                                             |                                                  |
|   41 | Verify route install                                                 | Router         | `show ip route <prefix>`                                          | BGP route installs if valid and preferred                                            |                                                                             |                                                  |
|   42 | Verify advertised routes                                             | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected prefixes are sent to peer                                                   |                                                                             |                                                  |
|   43 | Verify logs                                                          | Router         | `show logging                                                     | include BGP                                                                          | ADJCHANGE`                                                                  | Neighbor establishment or failure reason appears |

```
# BGP_eBGP_Multihop_And_Peer_Reachability_Loopback_eBGP_Skeleton
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
  network <local-prefix> mask <local-mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_eBGP_Multihop_And_Peer_Reachability_Static_Route_To_Peer_Skeleton
configure terminal

ip route <remote-loopback-ip> 255.255.255.255 <next-hop-ip>
ip route <remote-advertised-prefix> <remote-mask> <next-hop-ip>

router bgp <local-asn>
 neighbor <remote-loopback-ip> remote-as <remote-asn>
 neighbor <remote-loopback-ip> update-source Loopback0
 neighbor <remote-loopback-ip> ebgp-multihop <ttl>

 address-family ipv4 unicast
  neighbor <remote-loopback-ip> activate
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_eBGP_Multihop_And_Peer_Reachability_IGP_To_Peer_Loopbacks_Skeleton
configure terminal

interface Loopback0
 ip address <local-loopback-ip> 255.255.255.255

router ospf <process-id>
 network <local-loopback-ip> 0.0.0.0 area <area-id>
 network <transit-network> <wildcard-mask> area <area-id>

router bgp <local-asn>
 bgp router-id <local-loopback-ip>
 no bgp default ipv4-unicast
 neighbor <remote-loopback-ip> remote-as <remote-asn>
 neighbor <remote-loopback-ip> update-source Loopback0
 neighbor <remote-loopback-ip> ebgp-multihop <ttl>

 address-family ipv4 unicast
  neighbor <remote-loopback-ip> activate
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_eBGP_Multihop_And_Peer_Reachability_TTL_Security_Skeleton
configure terminal

router bgp <local-asn>
 neighbor <neighbor-ip> remote-as <remote-asn>
 neighbor <neighbor-ip> update-source <source-interface>
 neighbor <neighbor-ip> ttl-security hops <hop-count>

 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
 exit-address-family

end
copy running-config startup-config
```

```
# BGP_eBGP_Multihop_And_Peer_Reachability_Authentication_Skeleton
configure terminal

router bgp <local-asn>
 neighbor <neighbor-ip> password <password>

end
copy running-config startup-config
```

```
# BGP_eBGP_Multihop_And_Peer_Reachability_Verification_Skeleton
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


BGP_eBGP_Multihop_And_Peer_Reachability_Verification_Commands

| Task                                | Command                                                           | Expected Result                                              |                                                                                            |                                                       |
| ----------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------ | ----------------------------------------------------- |
| Verify interface state              | `show ip interface brief`                                         | Peer path interface and local loopback are up/up             |                                                                                            |                                                       |
| Verify local source interface       | `show running-config interface <source-interface>`                | Source interface has expected IP                             |                                                                                            |                                                       |
| Verify route to peer                | `show ip route <neighbor-ip>`                                     | Route to eBGP peer endpoint exists                           |                                                                                            |                                                       |
| Verify reverse route on peer        | `show ip route <local-source-ip>`                                 | Peer has route back to local BGP source                      |                                                                                            |                                                       |
| Verify sourced reachability         | `ping <neighbor-ip> source <local-source-ip>`                     | Ping succeeds from expected BGP source                       |                                                                                            |                                                       |
| Verify reverse sourced reachability | `ping <local-source-ip> source <neighbor-source-ip>`              | Peer can reach local source                                  |                                                                                            |                                                       |
| Verify BGP process                  | `show running-config                                              | section router bgp`                                          | BGP process, neighbor, update-source, multihop, and AFI are visible                        |                                                       |
| Verify remote AS                    | `show running-config                                              | section router bgp`                                          | Neighbor has correct `remote-as`                                                           |                                                       |
| Verify update-source                | `show running-config                                              | section router bgp`                                          | Loopback or selected source interface is configured                                        |                                                       |
| Verify eBGP multihop                | `show running-config                                              | section router bgp`                                          | `neighbor <ip> ebgp-multihop <ttl>` is present                                             |                                                       |
| Verify TTL security if used         | `show running-config                                              | section router bgp`                                          | `neighbor <ip> ttl-security hops <hop-count>` is present instead of normal multihop design |                                                       |
| Verify AF activation                | `show running-config                                              | section address-family`                                      | Neighbor is activated under IPv4 unicast                                                   |                                                       |
| Verify BGP summary                  | `show bgp ipv4 unicast summary`                                   | Neighbor appears                                             |                                                                                            |                                                       |
| Verify established state            | `show bgp ipv4 unicast summary`                                   | `State/PfxRcd` shows a number                                |                                                                                            |                                                       |
| Verify neighbor detail              | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | State is Established and external link is shown              |                                                                                            |                                                       |
| Verify source and connection detail | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Local host/source and foreign host/peer details match design |                                                                                            |                                                       |
| Verify message counters             | `show bgp ipv4 unicast summary`                                   | `MsgRcvd` and `MsgSent` increment                            |                                                                                            |                                                       |
| Verify queues                       | `show bgp ipv4 unicast summary`                                   | `InQ` and `OutQ` are normally 0                              |                                                                                            |                                                       |
| Verify BGP table                    | `show bgp ipv4 unicast`                                           | Expected routes appear                                       |                                                                                            |                                                       |
| Verify specific prefix              | `show bgp ipv4 unicast <prefix>`                                  | Next hop, AS path, and best path are visible                 |                                                                                            |                                                       |
| Verify RIB install                  | `show ip route <prefix>`                                          | BGP route installs if valid and preferred                    |                                                                                            |                                                       |
| Verify forwarding                   | `show ip cef <prefix>`                                            | CEF points to expected next hop/interface                    |                                                                                            |                                                       |
| Verify advertised routes            | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected prefixes are advertised                             |                                                                                            |                                                       |
| Verify logs                         | `show logging                                                     | include BGP                                                  | ADJCHANGE`                                                                                 | Neighbor state changes or failure reasons are visible |

# BGP_eBGP_Multihop_And_Peer_Reachability_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current BGP config | Router | `show running-config | section router bgp` | Neighbor, update-source, multihop, AFI, and network statements are visible |
| 2 | Identify current neighbor state | Router | `show bgp ipv4 unicast summary` | Current session state is known |
| 3 | Identify routes used for peer reachability | Router | `show ip route <neighbor-ip>` | Static or IGP route to peer is identified |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 6 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast address-family |
| 7 | Remove advertised prefix if lab-only | Router | `no network <network> mask <subnet-mask>` | Prefix is no longer advertised |
| 8 | Deactivate neighbor | Router | `no neighbor <neighbor-ip> activate` | Neighbor stops exchanging IPv4 unicast NLRI |
| 9 | Exit AFI mode | Router | `exit-address-family` | Router returns to BGP config mode |
| 10 | Remove eBGP multihop | Router | `no neighbor <neighbor-ip> ebgp-multihop <ttl>` | Multihop TTL override is removed |
| 11 | Remove TTL security if used | Router | `no neighbor <neighbor-ip> ttl-security hops <hop-count>` | TTL security setting is removed |
| 12 | Remove update-source | Router | `no neighbor <neighbor-ip> update-source <source-interface>` | Source override is removed |
| 13 | Remove authentication if lab-only | Router | `no neighbor <neighbor-ip> password <password>` | Neighbor password is removed |
| 14 | Remove neighbor | Router | `no neighbor <neighbor-ip> remote-as <remote-asn>` | Neighbor is removed |
| 15 | Remove static route to peer if lab-only | Router | `no ip route <remote-loopback-ip> 255.255.255.255 <next-hop-ip>` | Static peer reachability route is removed |
| 16 | Remove loopback if lab-only | Router | `no interface Loopback0` | Lab loopback is removed |
| 17 | Remove BGP process only if full rollback is intended | Router | `no router bgp <local-asn>` | BGP process is removed |
| 18 | Verify neighbor removal | Router | `show bgp ipv4 unicast summary` | Removed neighbor no longer appears |
| 19 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |

# BGP_eBGP_Multihop_And_Peer_Reachability_Failure_Checks
| Symptom                                              | Command                                                       | What Usually Broke                                                           |                                                                           |                                           |
| ---------------------------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ----------------------------------------- |
| Neighbor stuck Idle                                  | `show bgp ipv4 unicast summary`                               | No route to peer, wrong neighbor IP, peer not configured, or TCP/179 blocked |                                                                           |                                           |
| Neighbor stuck Active                                | `show bgp ipv4 unicast summary`                               | TCP connection cannot complete                                               |                                                                           |                                           |
| Direct eBGP works but loopback eBGP fails            | `show running-config                                          | section router bgp`                                                          | Missing `update-source`, missing `ebgp-multihop`, or no route to loopback |                                           |
| Ping works without source but BGP fails              | `ping <neighbor-ip> source <local-source-ip>`                 | BGP source IP cannot reach peer or peer cannot return to that source         |                                                                           |                                           |
| Peer receives packet from wrong source               | `show bgp ipv4 unicast neighbors <neighbor-ip>` and logs      | Missing or wrong `update-source`                                             |                                                                           |                                           |
| TTL expires before peer                              | `show running-config                                          | section router bgp`                                                          | Missing `ebgp-multihop` or TTL value too low                              |                                           |
| TTL security blocks session                          | `show running-config                                          | section router bgp`                                                          | `ttl-security hops` value does not match real hop distance                |                                           |
| TCP/179 blocked                                      | ACL/path check                                                | ACL/firewall blocks BGP                                                      |                                                                           |                                           |
| Authentication failure                               | `show logging                                                 | include BGP`                                                                 | Neighbor password mismatch                                                |                                           |
| Wrong AS configured                                  | `show logging                                                 | include BGP                                                                  | AS`                                                                       | `remote-as` does not match peer local AS  |
| Neighbor established but receives 0 prefixes         | `show bgp ipv4 unicast summary`                               | AFI not activated, no network statement, or policy blocks routes             |                                                                           |                                           |
| AFI activation missing                               | `show running-config                                          | section address-family`                                                      | Neighbor not activated under IPv4 unicast                                 |                                           |
| Network statement does not advertise route           | `show ip route <network> <subnet-mask>`                       | Exact route does not exist in RIB                                            |                                                                           |                                           |
| Prefix appears in BGP but not routing table          | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Next hop unreachable, better AD route exists, or RIB failure                 |                                                                           |                                           |
| Route to peer only exists through unstable path      | `show ip route <neighbor-ip>`                                 | Underlay/IGP/static route is unstable                                        |                                                                           |                                           |
| Session flaps during underlay changes                | `show logging                                                 | include BGP                                                                  | ADJCHANGE`                                                                | Peer reachability to loopback is unstable |
| eBGP multihop creates security exposure              | Config review                                                 | TTL raised without ACL, CoPP, TTL security, or peer filtering                |                                                                           |                                           |
| Neighbor appears established but no forwarding works | `show ip route <prefix>` and `show ip cef <prefix>`           | BGP session is up but data-plane route/next-hop is wrong                     |                                                                           |                                           |
| Queues stay nonzero                                  | `show bgp ipv4 unicast summary`                               | Peer instability, update churn, or control-plane issue                       |                                                                           |                                           |
| Hard clear caused unnecessary outage                 | `show logging                                                 | include BGP`                                                                 | Used hard clear instead of soft clear or waiting for TCP recovery         |                                           |

```
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-ebgp-multihop-final` | `BGP_eBGP_Multihop_And_Peer_Reachability.md` | Primary lab for non-direct eBGP, loopback peering, source address control, TTL behavior, and peer reachability verification |
```