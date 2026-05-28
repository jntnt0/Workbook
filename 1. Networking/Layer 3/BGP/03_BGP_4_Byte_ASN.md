

```
# Index
# Source_Basis
# BGP_4_Byte_ASN_Mental_Model
# BGP_4_Byte_ASN_Configuration_Checklist
# BGP_4_Byte_ASN_ASPLAIN_eBGP_Skeleton
# BGP_4_Byte_ASN_ASDOT_eBGP_Skeleton
# BGP_4_Byte_ASN_iBGP_Skeleton
# BGP_4_Byte_ASN_Loopback_eBGP_Skeleton
# BGP_4_Byte_ASN_Notation_Conversion_Reference
# BGP_4_Byte_ASN_Verification_Commands
# BGP_4_Byte_ASN_Rollback
# BGP_4_Byte_ASN_Failure_Checks
# Attached_Labs
```
# BGP_4_Byte_ASN_Mental_Model
| Concept                  | Operational Meaning                                                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| ASN                      | Autonomous System Number identifies a BGP routing domain                                                                                        |
| 2-byte ASN               | Traditional ASN range up to 65535                                                                                                               |
| 4-byte ASN               | Extended 32-bit ASN range up to 4294967295                                                                                                      |
| Private 2-byte ASN range | 64512 through 65534                                                                                                                             |
| Private 4-byte ASN range | 4200000000 through 4294967294                                                                                                                   |
| AS identity              | The local ASN under `router bgp <asn>` defines the router's AS identity                                                                         |
| Peer AS identity         | The neighbor `remote-as <asn>` defines what AS the peer is expected to belong to                                                                |
| eBGP relationship        | Local ASN and remote ASN are different                                                                                                          |
| iBGP relationship        | Local ASN and remote ASN are the same                                                                                                           |
| AS_PATH                  | BGP path attribute that records ASNs crossed by a route advertisement                                                                           |
| Loop prevention          | BGP rejects routes that contain its own ASN in AS_PATH                                                                                          |
| ASPLAIN                  | Decimal 4-byte ASN format, such as `4200000001`                                                                                                 |
| ASDOT                    | Dotted 4-byte ASN format, such as `64086.59905`                                                                                                 |
| AS notation display      | `bgp asnotation dot` changes how 4-byte ASNs display and how regex matching should be written                                                   |
| BGP OPEN message         | Session establishment includes the originating router ASN                                                                                       |
| Compatibility risk       | Old devices, scripts, regex filters, monitoring tools, and documentation may misread 4-byte ASN notation                                        |
| Blunt rule               | 4-byte ASN is not a new BGP feature. It is still normal BGP. The risk is notation, AS_PATH matching, remote-as accuracy, and tool compatibility |


# BGP_4_Byte_ASN_Configuration_Checklist
| Step | Task                                                                      | Device         | Command                                                           | Expected Result                                                            |                                                      |                                                         |                                                    |
| ---: | ------------------------------------------------------------------------- | -------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------- |
|    1 | Confirm local 4-byte ASN plan                                             | Router / Notes | `<local-4-byte-asn>`                                              | Local ASN is selected, usually from private 4-byte range for lab use       |                                                      |                                                         |                                                    |
|    2 | Confirm peer 4-byte ASN plan                                              | Router / Notes | `<remote-4-byte-asn>`                                             | Remote ASN is known                                                        |                                                      |                                                         |                                                    |
|    3 | Confirm notation style                                                    | Router / Notes | `asplain` or `asdot`                                              | Team knows whether ASNs will be entered/read as decimal or dotted notation |                                                      |                                                         |                                                    |
|    4 | Confirm private ASN range if lab/internal                                 | Router / Notes | `4200000000-4294967294`                                           | Private 4-byte ASN range is used for lab/internal designs                  |                                                      |                                                         |                                                    |
|    5 | Confirm AS relationship                                                   | Router / Notes | `<local-asn> == <remote-asn>` or `<local-asn> != <remote-asn>`    | Same AS means iBGP; different AS means eBGP                                |                                                      |                                                         |                                                    |
|    6 | Confirm peer-facing interface state                                       | Router         | `show ip interface brief`                                         | Peer-facing interface is up/up                                             |                                                      |                                                         |                                                    |
|    7 | Confirm local source IP                                                   | Router         | `show running-config interface <source-interface>`                | Source interface has expected IP                                           |                                                      |                                                         |                                                    |
|    8 | Confirm route to neighbor                                                 | Router         | `show ip route <neighbor-ip>`                                     | Route to peer exists                                                       |                                                      |                                                         |                                                    |
|    9 | Confirm sourced reachability                                              | Router         | `ping <neighbor-ip> source <local-source-ip>`                     | Peer is reachable from BGP source address                                  |                                                      |                                                         |                                                    |
|   10 | Confirm reverse reachability                                              | Peer Router    | `ping <local-source-ip> source <neighbor-source-ip>`              | Peer can reach local BGP source address                                    |                                                      |                                                         |                                                    |
|   11 | Confirm TCP/179 path                                                      | Router / Path  | `show access-lists` or path ACL check                             | TCP/179 is not blocked                                                     |                                                      |                                                         |                                                    |
|   12 | Confirm existing BGP process                                              | Router         | `show running-config                                              | section router bgp`                                                        | Existing BGP config is visible                       |                                                         |                                                    |
|   13 | Confirm current ASN display style                                         | Router         | `show running-config                                              | include bgp asnotation`                                                    | Output shows whether dot notation display is enabled |                                                         |                                                    |
|   14 | Enter configuration mode                                                  | Router         | `configure terminal`                                              | Router enters global configuration mode                                    |                                                      |                                                         |                                                    |
|   15 | Start BGP using 4-byte ASPLAIN ASN                                        | Router         | `router bgp <local-4-byte-asn>`                                   | Router enters BGP config mode using decimal 4-byte ASN                     |                                                      |                                                         |                                                    |
|   16 | Start BGP using ASDOT notation if preferred/supported                     | Router         | `router bgp <high-order>.<low-order>`                             | Router enters BGP config mode using dotted ASN notation                    |                                                      |                                                         |                                                    |
|   17 | Set stable router ID                                                      | Router         | `bgp router-id <router-id>`                                       | Router ID is deterministic                                                 |                                                      |                                                         |                                                    |
|   18 | Enable neighbor change logging                                            | Router         | `bgp log-neighbor-changes`                                        | BGP neighbor up/down events are logged                                     |                                                      |                                                         |                                                    |
|   19 | Enable ASDOT display if desired                                           | Router         | `bgp asnotation dot`                                              | 4-byte ASNs display in dotted format                                       |                                                      |                                                         |                                                    |
|   20 | Keep ASPLAIN display if preferred                                         | Router         | `no bgp asnotation dot`                                           | 4-byte ASNs display in decimal format                                      |                                                      |                                                         |                                                    |
|   21 | Standardize explicit AF activation                                        | Router         | `no bgp default ipv4-unicast`                                     | Neighbors must be activated under address-family                           |                                                      |                                                         |                                                    |
|   22 | Configure neighbor with 4-byte remote ASN in ASPLAIN                      | Router         | `neighbor <neighbor-ip> remote-as <remote-4-byte-asn>`            | Neighbor is defined using decimal ASN                                      |                                                      |                                                         |                                                    |
|   23 | Configure neighbor with 4-byte remote ASN in ASDOT if preferred/supported | Router         | `neighbor <neighbor-ip> remote-as <high-order>.<low-order>`       | Neighbor is defined using dotted ASN                                       |                                                      |                                                         |                                                    |
|   24 | Add neighbor description                                                  | Router         | `neighbor <neighbor-ip> description <description>`                | Peer purpose is documented                                                 |                                                      |                                                         |                                                    |
|   25 | Configure update-source if using loopback peering                         | Router         | `neighbor <neighbor-ip> update-source Loopback0`                  | BGP packets source from Loopback0                                          |                                                      |                                                         |                                                    |
|   26 | Configure eBGP multihop only if eBGP peer is not directly connected       | Router         | `neighbor <neighbor-ip> ebgp-multihop <ttl>`                      | Non-direct eBGP peer can form                                              |                                                      |                                                         |                                                    |
|   27 | Configure password only if both peers use authentication                  | Router         | `neighbor <neighbor-ip> password <password>`                      | BGP authentication matches peer                                            |                                                      |                                                         |                                                    |
|   28 | Enter IPv4 unicast address family                                         | Router         | `address-family ipv4 unicast`                                     | Router enters IPv4 unicast AFI                                             |                                                      |                                                         |                                                    |
|   29 | Activate neighbor                                                         | Router         | `neighbor <neighbor-ip> activate`                                 | Neighbor can exchange IPv4 unicast NLRI                                    |                                                      |                                                         |                                                    |
|   30 | Advertise local prefix only if exact route exists                         | Router         | `network <network> mask <subnet-mask>`                            | Prefix is eligible for BGP origination                                     |                                                      |                                                         |                                                    |
|   31 | Verify exact local route exists                                           | Router         | `show ip route <network> <subnet-mask>`                           | Exact route exists in RIB                                                  |                                                      |                                                         |                                                    |
|   32 | Exit address-family mode                                                  | Router         | `exit-address-family`                                             | Router returns to BGP config mode                                          |                                                      |                                                         |                                                    |
|   33 | Save configuration                                                        | Router         | `copy running-config startup-config`                              | Configuration is saved                                                     |                                                      |                                                         |                                                    |
|   34 | Verify BGP process ASN                                                    | Router         | `show bgp ipv4 unicast summary`                                   | Local ASN appears as expected                                              |                                                      |                                                         |                                                    |
|   35 | Verify neighbor remote ASN                                                | Router         | `show bgp ipv4 unicast summary`                                   | Neighbor AS value appears as expected                                      |                                                      |                                                         |                                                    |
|   36 | Verify neighbor state                                                     | Router         | `show bgp ipv4 unicast summary`                                   | `State/PfxRcd` shows a number when established                             |                                                      |                                                         |                                                    |
|   37 | Verify neighbor detail                                                    | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip>`                   | Remote AS, local AS, capabilities, and state are correct                   |                                                      |                                                         |                                                    |
|   38 | Verify AS notation display                                                | Router         | `show bgp ipv4 unicast summary`                                   | 4-byte ASN displays in selected notation format                            |                                                      |                                                         |                                                    |
|   39 | Verify BGP table                                                          | Router         | `show bgp ipv4 unicast`                                           | Routes show expected AS_PATH with 4-byte ASNs                              |                                                      |                                                         |                                                    |
|   40 | Verify specific prefix AS_PATH                                            | Router         | `show bgp ipv4 unicast <prefix>`                                  | AS_PATH contains expected 4-byte ASNs                                      |                                                      |                                                         |                                                    |
|   41 | Verify installed route                                                    | Router         | `show ip route <prefix>`                                          | BGP route installs if valid and preferred                                  |                                                      |                                                         |                                                    |
|   42 | Verify advertised routes                                                  | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Expected prefixes are advertised with expected AS_PATH                     |                                                      |                                                         |                                                    |
|   43 | Verify received routes                                                    | Router         | `show bgp ipv4 unicast neighbors <neighbor-ip> routes`            | Expected inbound prefixes are visible                                      |                                                      |                                                         |                                                    |
|   44 | Verify AS path regex behavior if filters exist                            | Router         | `show running-config                                              | include as-path                                                            | ip as-path                                           | bgp asnotation`                                         | Regex format matches the active ASN notation style |
|   45 | Verify logs                                                               | Router         | `show logging                                                     | include BGP                                                                | ADJCHANGE`                                           | Neighbor establishment or ASN mismatch logs are visible |                                                    |

```
# BGP_4_Byte_ASN_ASPLAIN_eBGP_Skeleton
configure terminal

router bgp <local-4-byte-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 no bgp asnotation dot
 neighbor <neighbor-ip> remote-as <remote-4-byte-asn>
 neighbor <neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_4_Byte_ASN_ASDOT_eBGP_Skeleton
configure terminal

router bgp <local-asdot-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 bgp asnotation dot
 neighbor <neighbor-ip> remote-as <remote-asdot-asn>
 neighbor <neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_4_Byte_ASN_iBGP_Skeleton
configure terminal

router bgp <shared-4-byte-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <ibgp-neighbor-ip> remote-as <shared-4-byte-asn>
 neighbor <ibgp-neighbor-ip> description <description>

 address-family ipv4 unicast
  neighbor <ibgp-neighbor-ip> activate
  network <network> mask <subnet-mask>
 exit-address-family

end
copy running-config startup-config
```


```
# BGP_4_Byte_ASN_Loopback_eBGP_Skeleton
configure terminal

interface Loopback0
 ip address <local-loopback-ip> 255.255.255.255

router bgp <local-4-byte-asn>
 bgp router-id <local-loopback-ip>
 bgp log-neighbor-changes
 no bgp default ipv4-unicast
 neighbor <remote-loopback-ip> remote-as <remote-4-byte-asn>
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
# BGP_4_Byte_ASN_Notation_Conversion_Reference
ASPLAIN 65536 = ASDOT 1.0
ASPLAIN 65537 = ASDOT 1.1
ASPLAIN 65546 = ASDOT 1.10
ASPLAIN 4200000000 = ASDOT 64086.59904
ASPLAIN 4200000001 = ASDOT 64086.59905

Formula:
ASPLAIN = (<high-order> * 65536) + <low-order>
ASDOT = <high-order>.<low-order>
```


# BGP_4_Byte_ASN_Rollback
| Step | Task                                                 | Device | Command                                                   | Expected Result                                |                                          |
| ---: | ---------------------------------------------------- | ------ | --------------------------------------------------------- | ---------------------------------------------- | ---------------------------------------- |
|    1 | Identify BGP config                                  | Router | `show running-config                                      | section router bgp`                            | Current 4-byte ASN BGP config is visible |
|    2 | Identify active BGP sessions                         | Router | `show bgp ipv4 unicast summary`                           | Current neighbors and ASNs are visible         |                                          |
|    3 | Enter configuration mode                             | Router | `configure terminal`                                      | Router enters global configuration mode        |                                          |
|    4 | Enter BGP process                                    | Router | `router bgp <local-4-byte-asn>`                           | Router enters BGP config mode                  |                                          |
|    5 | Enter IPv4 unicast AFI                               | Router | `address-family ipv4 unicast`                             | Router enters IPv4 unicast address-family      |                                          |
|    6 | Remove advertised prefix                             | Router | `no network <network> mask <subnet-mask>`                 | Prefix is no longer originated                 |                                          |
|    7 | Deactivate neighbor                                  | Router | `no neighbor <neighbor-ip> activate`                      | Neighbor no longer exchanges IPv4 unicast NLRI |                                          |
|    8 | Exit AFI mode                                        | Router | `exit-address-family`                                     | Router returns to BGP config mode              |                                          |
|    9 | Remove eBGP multihop if used                         | Router | `no neighbor <neighbor-ip> ebgp-multihop <ttl>`           | eBGP multihop is removed                       |                                          |
|   10 | Remove update-source if used                         | Router | `no neighbor <neighbor-ip> update-source <interface>`     | Update-source is removed                       |                                          |
|   11 | Remove neighbor password if used                     | Router | `no neighbor <neighbor-ip> password <password>`           | Password is removed                            |                                          |
|   12 | Remove neighbor                                      | Router | `no neighbor <neighbor-ip> remote-as <remote-4-byte-asn>` | 4-byte ASN neighbor is removed                 |                                          |
|   13 | Restore ASPLAIN display if dot notation was lab-only | Router | `no bgp asnotation dot`                                   | ASN display returns to ASPLAIN                 |                                          |
|   14 | Remove BGP process only if full rollback is intended | Router | `no router bgp <local-4-byte-asn>`                        | BGP process is removed                         |                                          |
|   15 | Verify neighbor removed                              | Router | `show bgp ipv4 unicast summary`                           | Removed neighbor no longer appears             |                                          |
|   16 | Save rollback                                        | Router | `copy running-config startup-config`                      | Rollback is saved                              |                                          |

# BGP_4_Byte_ASN_Failure_Checks
| Symptom                                                | Command                                                       | What Usually Broke                                                                |                                                                           |                              |                                                                               |
| ------------------------------------------------------ | ------------------------------------------------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ---------------------------- | ----------------------------------------------------------------------------- |
| Neighbor stuck Idle                                    | `show bgp ipv4 unicast summary`                               | Route to peer missing, wrong neighbor IP, peer not configured, or TCP/179 blocked |                                                                           |                              |                                                                               |
| Neighbor stuck Active                                  | `show bgp ipv4 unicast summary`                               | TCP session cannot complete                                                       |                                                                           |                              |                                                                               |
| Neighbor rejects session                               | `show logging                                                 | include BGP                                                                       | AS`                                                                       | Local or remote ASN mismatch |                                                                               |
| Peer configured as eBGP when iBGP expected             | `show running-config                                          | section router bgp`                                                               | `neighbor remote-as` differs from local `router bgp` ASN                  |                              |                                                                               |
| Peer configured as iBGP when eBGP expected             | `show running-config                                          | section router bgp`                                                               | `neighbor remote-as` matches local ASN by mistake                         |                              |                                                                               |
| AS displays differently than expected                  | `show running-config                                          | include bgp asnotation`                                                           | `bgp asnotation dot` is enabled or disabled contrary to expectation       |                              |                                                                               |
| AS path regex stops matching                           | `show running-config                                          | include ip as-path                                                                | as-path access-list                                                       | bgp asnotation`              | Regex written for ASPLAIN while router displays/matches ASDOT, or the reverse |
| Monitoring tool reports wrong ASN                      | `show bgp ipv4 unicast summary`                               | Tool expects decimal ASN but router displays dotted notation                      |                                                                           |                              |                                                                               |
| Documentation and CLI disagree                         | `show bgp ipv4 unicast summary`                               | One source uses ASPLAIN and another uses ASDOT                                    |                                                                           |                              |                                                                               |
| Route rejected due to AS loop                          | `show bgp ipv4 unicast <prefix>` and logs                     | Local ASN appears in AS_PATH                                                      |                                                                           |                              |                                                                               |
| Prefix not advertised                                  | `show ip route <network> <subnet-mask>`                       | Exact route for `network` command does not exist                                  |                                                                           |                              |                                                                               |
| Neighbor established but receives 0 prefixes           | `show bgp ipv4 unicast summary`                               | AFI not activated, no advertised network, or outbound policy blocks routes        |                                                                           |                              |                                                                               |
| Route appears but does not install                     | `show bgp ipv4 unicast <prefix>` and `show ip route <prefix>` | Next hop unreachable, better AD route exists, or RIB failure                      |                                                                           |                              |                                                                               |
| 4-byte lab conflicts with real network                 | `show running-config                                          | section router bgp`                                                               | Lab used public/reserved ASN instead of private lab ASN                   |                              |                                                                               |
| eBGP loopback 4-byte session fails                     | `show running-config                                          | section router bgp`                                                               | Missing `update-source`, missing `ebgp-multihop`, or no route to loopback |                              |                                                                               |
| BGP table looks correct but teammate says ASN is wrong | `show bgp ipv4 unicast summary` and notation conversion check | Same ASN written in different notation                                            |                                                                           |                              |                                                                               |
| Soft policy change does not affect AS path filtering   | `clear bgp ipv4 unicast <neighbor-ip> soft in`                | Routes were not refreshed after AS path policy change                             |                                                                           |                              |                                                                               |

```
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-4-byte-asn-final` | `BGP_4_Byte_ASN.md` | Primary lab for 4-byte ASN session configuration, ASN notation, AS_PATH reading, and neighbor verification |
```