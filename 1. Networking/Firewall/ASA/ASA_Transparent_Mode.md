



# ASA_Transparent_Mode_Mental_Model
| Concept                    | Operational Meaning                                                                                               |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Transparent firewall       | ASA acts like a Layer 2 bridge while still enforcing firewall policy                                              |
| Routed firewall            | ASA interfaces are Layer 3 gateways with separate IP subnets                                                      |
| Bridge group               | Logical Layer 2 domain containing transparent firewall member interfaces                                          |
| BVI                        | Bridge Virtual Interface that owns the ASA management/control IP for that bridge group                            |
| Same subnet placement      | Inside and outside bridge-group member interfaces sit in the same Layer 3 subnet                                  |
| No routed interface IPs    | Physical bridge-group member interfaces do not receive IP addresses                                               |
| Management IP              | ASA uses the BVI IP or dedicated management interface for SSH, ASDM, syslog, AAA, SNMP, ARP, and control traffic  |
| L2F table                  | ASA bridge forwarding table that maps MAC addresses to interfaces                                                 |
| Static L2F entry           | Manually pins a MAC address to an ASA interface                                                                   |
| MAC learning               | ASA dynamically learns MAC-to-interface mappings unless disabled                                                  |
| ARP inspection             | ASA validates ARP replies/gratuitous ARP against static ARP entries                                               |
| Extended ACL               | Filters IP traffic in transparent mode                                                                            |
| EtherType ACL              | Filters non-IP or Layer 2 EtherType traffic in transparent mode                                                   |
| BPDU handling              | ASA forwards BPDUs by default, but EtherType ACLs can block them                                                  |
| Dynamic routing limitation | RIP, OSPF, and EIGRP are not supported through ASA transparent firewall routing functions                         |
| NAT in transparent mode    | NAT can be configured, but must be deliberate and verified carefully                                              |
| Mode conversion risk       | Switching between routed and transparent mode clears the running configuration                                    |
| Blunt rule                 | Transparent mode is a firewall bridge, not a router. If you configure it like routed mode, you will break the lab |

# ASA_Transparent_Mode_Configuration_Checklist
| Step | Task                                                                          | Device            | Command                                                                                                                                           | Expected Result                                                                     |
| ---: | ----------------------------------------------------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
|    1 | Confirm current firewall mode                                                 | ASA               | `show firewall`                                                                                                                                   | ASA current mode is known before changes                                            |
|    2 | Confirm whether this is production or lab                                     | ASA / Notes       | `<lab-or-production>`                                                                                                                             | Risk is understood before changing firewall mode                                    |
|    3 | Backup current running config                                                 | ASA               | `copy running-config disk0:/routed-before-transparent.cfg`                                                                                        | Existing routed-mode config is saved                                                |
|    4 | Export config externally if possible                                          | ASA / Workstation | `copy running-config tftp:` or `copy running-config scp:`                                                                                         | Recovery copy exists outside the ASA                                                |
|    5 | Confirm console access                                                        | ASA / Notes       | `<console-access-confirmed>`                                                                                                                      | You can recover if SSH/ASDM access drops                                            |
|    6 | Confirm transparent design subnet                                             | ASA / Notes       | `<transparent-subnet> <mask>`                                                                                                                     | Inside/outside bridged segment subnet is known                                      |
|    7 | Confirm BVI management IP                                                     | ASA / Notes       | `<bvi-ip> <mask>`                                                                                                                                 | ASA BVI IP is in the bridged subnet                                                 |
|    8 | Confirm upstream router/default gateway                                       | ASA / Notes       | `<gateway-ip>`                                                                                                                                    | ASA management/control traffic has a default gateway plan                           |
|    9 | Confirm bridge group ID                                                       | ASA / Notes       | `<bridge-group-id>`                                                                                                                               | Bridge group number is selected, usually `1`                                        |
|   10 | Confirm bridge member interfaces                                              | ASA / Notes       | `<inside-interface> <outside-interface>`                                                                                                          | Physical interfaces to bridge are known                                             |
|   11 | Confirm optional management interface                                         | ASA / Notes       | `<management-interface>`                                                                                                                          | Dedicated management interface is planned if used                                   |
|   12 | Confirm management subnet and gateway                                         | ASA / Notes       | `<mgmt-ip> <mgmt-mask> <mgmt-gateway>`                                                                                                            | Out-of-band management path is known                                                |
|   13 | Confirm ACL policy requirement                                                | ASA / Notes       | `<permit-policy>`                                                                                                                                 | IP and non-IP traffic policy is known before cutting over                           |
|   14 | Confirm whether EtherType ACL is needed                                       | ASA / Notes       | `IPX, BPDU, MPLS, routing protocol, multicast, or other non-IP traffic`                                                                           | Non-IP / Layer 2 control traffic requirement is known                               |
|   15 | Confirm whether same-security traffic is needed                               | ASA / Notes       | `same-security-traffic permit inter-interface`                                                                                                    | Required if traffic must pass between same-security interfaces                      |
|   16 | Confirm whether NAT is needed                                                 | ASA / Notes       | `<nat-required-or-not>`                                                                                                                           | Transparent NAT requirement is known                                                |
|   17 | Confirm whether ARP inspection is needed                                      | ASA / Notes       | `<arp-inspection-required-or-not>`                                                                                                                | ARP security requirement is known                                                   |
|   18 | Confirm whether static MAC/L2F entries are needed                             | ASA / Notes       | `<static-mac-required-or-not>`                                                                                                                    | Static forwarding requirement is known                                              |
|   19 | Enter configuration mode                                                      | ASA               | `configure terminal`                                                                                                                              | ASA enters global configuration mode                                                |
|   20 | Enable transparent firewall mode                                              | ASA               | `firewall transparent`                                                                                                                            | ASA changes to transparent mode and clears incompatible running configuration       |
|   21 | Verify transparent mode after conversion                                      | ASA               | `show firewall`                                                                                                                                   | Output shows `Firewall mode: Transparent`                                           |
|   22 | Restore basic hostname if needed                                              | ASA               | `hostname <asa-hostname>`                                                                                                                         | ASA hostname is set                                                                 |
|   23 | Configure outside bridge member                                               | ASA               | `interface <outside-interface>`                                                                                                                   | ASA enters outside interface mode                                                   |
|   24 | Bring outside interface up                                                    | ASA               | `no shutdown`                                                                                                                                     | Outside interface is administratively enabled                                       |
|   25 | Name outside interface                                                        | ASA               | `nameif outside`                                                                                                                                  | Interface logical name is `outside`                                                 |
|   26 | Set outside security level                                                    | ASA               | `security-level 0`                                                                                                                                | Outside security level is set                                                       |
|   27 | Assign outside to bridge group                                                | ASA               | `bridge-group <bridge-group-id>`                                                                                                                  | Outside joins the transparent bridge group                                          |
|   28 | Configure inside bridge member                                                | ASA               | `interface <inside-interface>`                                                                                                                    | ASA enters inside interface mode                                                    |
|   29 | Bring inside interface up                                                     | ASA               | `no shutdown`                                                                                                                                     | Inside interface is administratively enabled                                        |
|   30 | Name inside interface                                                         | ASA               | `nameif inside`                                                                                                                                   | Interface logical name is `inside`                                                  |
|   31 | Set inside security level                                                     | ASA               | `security-level 100`                                                                                                                              | Inside security level is set                                                        |
|   32 | Assign inside to bridge group                                                 | ASA               | `bridge-group <bridge-group-id>`                                                                                                                  | Inside joins the same bridge group                                                  |
|   33 | Configure BVI interface                                                       | ASA               | `interface BVI<bridge-group-id>`                                                                                                                  | ASA enters BVI interface mode                                                       |
|   34 | Assign BVI IP address                                                         | ASA               | `ip address <bvi-ip> <bvi-mask>`                                                                                                                  | ASA has management/control IP in the bridged subnet                                 |
|   35 | Configure dedicated management interface if used                              | ASA               | `interface <management-interface>`                                                                                                                | ASA enters management interface mode                                                |
|   36 | Mark management interface as management-only if required                      | ASA               | `management-only`                                                                                                                                 | Interface is used only for management                                               |
|   37 | Name management interface                                                     | ASA               | `nameif management`                                                                                                                               | Interface logical name is `management`                                              |
|   38 | Set management security level                                                 | ASA               | `security-level 100`                                                                                                                              | Management interface security level is set                                          |
|   39 | Assign management IP                                                          | ASA               | `ip address <mgmt-ip> <mgmt-mask>`                                                                                                                | Dedicated management interface has IP address                                       |
|   40 | Bring management interface up                                                 | ASA               | `no shutdown`                                                                                                                                     | Management interface is enabled                                                     |
|   41 | Configure default route through management if using OOB management            | ASA               | `route management 0.0.0.0 0.0.0.0 <mgmt-gateway>`                                                                                                 | ASA management traffic has OOB default route                                        |
|   42 | Configure default route through bridged interface if no OOB management exists | ASA               | `route inside 0.0.0.0 0.0.0.0 <inside-gateway-ip>`                                                                                                | ASA management/control traffic has gateway through BVI path                         |
|   43 | Enable SSH if remote CLI management is needed                                 | ASA               | `ssh <mgmt-source-network> <mgmt-source-mask> <management-or-bvi-interface-name>`                                                                 | SSH is allowed from trusted management subnet                                       |
|   44 | Enable ASDM/HTTP if needed                                                    | ASA               | `http server enable`                                                                                                                              | ASA HTTP/ASDM server is enabled                                                     |
|   45 | Permit ASDM source subnet                                                     | ASA               | `http <mgmt-source-network> <mgmt-source-mask> <management-or-bvi-interface-name>`                                                                | ASDM is allowed from trusted management subnet                                      |
|   46 | Permit same-security inter-interface traffic only if required                 | ASA               | `same-security-traffic permit inter-interface`                                                                                                    | ASA allows traffic between same-security interfaces                                 |
|   47 | Create inside IP ACL if filtering IP traffic                                  | ASA               | `access-list <inside-ip-acl> extended permit ip <source> <mask> <destination> <mask>`                                                             | IP traffic permit rule exists                                                       |
|   48 | Apply inside IP ACL inbound                                                   | ASA               | `access-group <inside-ip-acl> in interface inside`                                                                                                | Inside inbound IP policy is active                                                  |
|   49 | Create outside IP ACL if filtering IP traffic                                 | ASA               | `access-list <outside-ip-acl> extended permit tcp <source> <mask> host <destination-ip> eq <port>`                                                | Outside inbound IP permit rule exists                                               |
|   50 | Apply outside IP ACL inbound                                                  | ASA               | `access-group <outside-ip-acl> in interface outside`                                                                                              | Outside inbound IP policy is active                                                 |
|   51 | Create EtherType ACL if non-IP traffic must be controlled                     | ASA               | `access-list <ethertype-acl> ethertype permit <ether-value-or-keyword>`                                                                           | EtherType policy exists                                                             |
|   52 | Apply EtherType ACL inbound on required interface                             | ASA               | `access-group <ethertype-acl> in interface <interface-name>`                                                                                      | EtherType policy is active                                                          |
|   53 | Avoid accidental EtherType deny-all unless intended                           | ASA / Notes       | `No explicit ethertype deny any unless design requires it`                                                                                        | IP traffic is not accidentally blocked by EtherType policy                          |
|   54 | Configure optional static NAT real object                                     | ASA               | `object network <real-object>` then `host <real-ip>`                                                                                              | Real host object exists                                                             |
|   55 | Configure optional static NAT mapped object                                   | ASA               | `object network <mapped-object>` then `host <mapped-ip>`                                                                                          | Mapped host object exists                                                           |
|   56 | Configure optional transparent static NAT                                     | ASA               | `nat (inside,outside) source static <real-object> <mapped-object>`                                                                                | Transparent-mode static NAT exists                                                  |
|   57 | Confirm upstream route for mapped NAT IP if needed                            | Upstream Router   | `ip route <mapped-ip> 255.255.255.255 <next-hop-ip>`                                                                                              | Upstream device knows how to reach translated address                               |
|   58 | Configure optional static ARP entry before no-flood ARP inspection            | ASA               | `arp <interface-name> <ip-address> <mac-address>`                                                                                                 | ASA has trusted ARP entry                                                           |
|   59 | Enable ARP inspection with flood behavior if cautious                         | ASA               | `arp-inspection <interface-name> enable flood`                                                                                                    | Unknown ARP entries are flooded rather than dropped                                 |
|   60 | Enable ARP inspection with no-flood only if all static ARP entries are known  | ASA               | `arp-inspection <interface-name> enable no-flood`                                                                                                 | Unknown ARP entries are dropped                                                     |
|   61 | Configure optional static L2F entry                                           | ASA               | `mac-address-table static <interface-name> <mac-address>`                                                                                         | MAC is pinned to the expected interface                                             |
|   62 | Disable MAC learning only if static MAC plan is complete                      | ASA               | `mac-learn <interface-name> disable`                                                                                                              | ASA stops dynamic MAC learning on that interface                                    |
|   63 | Tune L2F aging if needed                                                      | ASA               | `mac-address-table aging-time <minutes>`                                                                                                          | MAC aging timer is adjusted                                                         |
|   64 | Save configuration                                                            | ASA               | `write memory`                                                                                                                                    | Transparent firewall configuration is saved                                         |
|   65 | Verify firewall mode                                                          | ASA               | `show firewall`                                                                                                                                   | Output shows transparent mode                                                       |
|   66 | Verify interface state                                                        | ASA               | `show interface ip brief`                                                                                                                         | Bridge members and management/BVI are up/up where expected                          |
|   67 | Verify bridge group mapping                                                   | ASA               | `show running-config interface <interface-name>`                                                                                                  | Interfaces show correct `bridge-group` membership                                   |
|   68 | Verify BVI IP                                                                 | ASA               | `show running-config interface BVI<bridge-group-id>`                                                                                              | BVI has expected IP and mask                                                        |
|   69 | Verify routing for management/control traffic                                 | ASA               | `show route`                                                                                                                                      | Default route points to expected management or bridged gateway                      |
|   70 | Verify L2F table                                                              | ASA               | `show mac-address-table`                                                                                                                          | ASA learns expected MAC addresses on expected interfaces                            |
|   71 | Verify ARP inspection state                                                   | ASA               | `show arp-inspection`                                                                                                                             | ARP inspection is enabled or disabled as intended                                   |
|   72 | Verify ACL hit counts                                                         | ASA               | `show access-list`                                                                                                                                | Expected ACL hit counts increment                                                   |
|   73 | Verify connection table                                                       | ASA               | `show conn`                                                                                                                                       | IP sessions through transparent firewall appear                                     |
|   74 | Test host-to-gateway reachability                                             | Host              | `ping <default-gateway-ip>`                                                                                                                       | Host can reach gateway through ASA if ACL allows ICMP                               |
|   75 | Test host-to-host or host-to-service reachability                             | Host              | `ping <remote-host-ip>` or service-specific test                                                                                                  | Permitted traffic crosses transparent ASA                                           |
|   76 | Capture ingress and egress if traffic fails                                   | ASA               | `capture <cap1> interface <int1> match ip host <src-ip> host <dst-ip>` and `capture <cap2> interface <int2> match ip host <src-ip> host <dst-ip>` | Confirms whether packet enters and exits ASA                                        |
|   77 | Check drops if traffic fails                                                  | ASA               | `show asp drop`                                                                                                                                   | Drop reason points to ACL, ARP inspection, L2F, NAT, inspection, or interface issue |
|   78 | Debug L2F only when needed                                                    | ASA               | `debug mac-address-table`                                                                                                                         | ASA shows MAC learning behavior                                                     |
|   79 | Debug ARP inspection only when needed                                         | ASA               | `debug arp-inspection`                                                                                                                            | ASA shows ARP inspection permit/drop behavior                                       |
|   80 | Debug host move only when needed                                              | ASA               | `debug l2-indication`                                                                                                                             | ASA shows host move or static MAC conflict behavior                                 |
|   81 | Stop debugs                                                                   | ASA               | `undebug all`                                                                                                                                     | Debug output stops                                                                  |

```
# ASA_Transparent_Mode_Basic_Bridge_Group_Skeleton
configure terminal

firewall transparent

hostname <asa-hostname>

interface <outside-interface>
 no shutdown
 nameif outside
 security-level 0
 bridge-group <bridge-group-id>

interface <inside-interface>
 no shutdown
 nameif inside
 security-level 100
 bridge-group <bridge-group-id>

interface BVI<bridge-group-id>
 ip address <bvi-ip> <bvi-mask>

route inside 0.0.0.0 0.0.0.0 <inside-gateway-ip>

write memory
```


```
# ASA_Transparent_Mode_Dedicated_Management_Skeleton
configure terminal

interface <management-interface>
 no shutdown
 management-only
 nameif management
 security-level 100
 ip address <mgmt-ip> <mgmt-mask>

route management 0.0.0.0 0.0.0.0 <mgmt-gateway>

ssh <mgmt-source-network> <mgmt-source-mask> management
http server enable
http <mgmt-source-network> <mgmt-source-mask> management

write memory
```

```
# ASA_Transparent_Mode_IP_ACL_Skeleton
configure terminal

access-list <inside-ip-acl> remark Permit selected IP traffic through transparent ASA
access-list <inside-ip-acl> extended permit ip <inside-source-network> <inside-source-mask> <outside-destination-network> <outside-destination-mask>
access-group <inside-ip-acl> in interface inside

access-list <outside-ip-acl> remark Permit selected return or inbound IP traffic through transparent ASA
access-list <outside-ip-acl> extended permit ip <outside-source-network> <outside-source-mask> <inside-destination-network> <inside-destination-mask>
access-group <outside-ip-acl> in interface outside

write memory
```

```
# ASA_Transparent_Mode_EtherType_ACL_Skeleton
configure terminal

access-list <inside-ethertype-acl> remark Control non-IP Layer 2 traffic inbound on inside
access-list <inside-ethertype-acl> ethertype permit bpdu
access-list <inside-ethertype-acl> ethertype permit mpls-unicast
access-list <inside-ethertype-acl> ethertype permit mpls-multicast
access-group <inside-ethertype-acl> in interface inside

access-list <outside-ethertype-acl> remark Control non-IP Layer 2 traffic inbound on outside
access-list <outside-ethertype-acl> ethertype permit bpdu
access-list <outside-ethertype-acl> ethertype permit mpls-unicast
access-list <outside-ethertype-acl> ethertype permit mpls-multicast
access-group <outside-ethertype-acl> in interface outside

write memory
```


```
# ASA_Transparent_Mode_Static_NAT_Skeleton
configure terminal

object network <real-object>
 host <real-ip>

object network <mapped-object>
 host <mapped-ip>

nat (inside,outside) source static <real-object> <mapped-object>

write memory
```


```
# ASA_Transparent_Mode_ARP_Inspection_Skeleton
configure terminal

arp <interface-name> <trusted-ip> <trusted-mac>

arp-inspection <interface-name> enable no-flood

write memory
```

```
# ASA_Transparent_Mode_Static_L2F_MAC_Skeleton
configure terminal

mac-address-table static <interface-name> <mac-address>

mac-address-table aging-time <minutes>

write memory
```


```
# ASA_Transparent_Mode_Disable_MAC_Learning_Skeleton
configure terminal

mac-address-table static <interface-name> <mac-address>

mac-learn <interface-name> disable

write memory
```

```
# ASA_Transparent_Mode_Same_Security_Interfaces_Skeleton
configure terminal

same-security-traffic permit inter-interface

write memory
```


# ASA_Transparent_Mode_Verification_Commands
| Task                               | Command                                                                               | Expected Result                                                                 |
| ---------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Verify firewall mode               | `show firewall`                                                                       | Output shows `Firewall mode: Transparent`                                       |
| Verify interface state             | `show interface ip brief`                                                             | Bridge member interfaces and management/BVI interfaces are up/up where expected |
| Verify logical interface names     | `show nameif`                                                                         | Interfaces have expected names and security levels                              |
| Verify bridge group membership     | `show running-config interface <interface-name>`                                      | Member interfaces show correct `bridge-group <id>`                              |
| Verify BVI address                 | `show running-config interface BVI<bridge-group-id>`                                  | BVI has expected management/control IP                                          |
| Verify management route            | `show route`                                                                          | Default route points to expected management or bridged gateway                  |
| Verify SSH management access       | `show running-config ssh`                                                             | Trusted management subnet is allowed                                            |
| Verify ASDM/HTTP management access | `show running-config http`                                                            | HTTP server and trusted management subnet are configured                        |
| Verify same-security behavior      | `show running-config same-security-traffic`                                           | `same-security-traffic permit inter-interface` appears only if required         |
| Verify MAC forwarding table        | `show mac-address-table`                                                              | Expected MACs are learned on expected interfaces                                |
| Verify static MAC entries          | `show mac-address-table`                                                              | Static entries appear with correct interface                                    |
| Verify ARP table                   | `show arp`                                                                            | Expected ARP entries exist                                                      |
| Verify ARP inspection state        | `show arp-inspection`                                                                 | ARP inspection is enabled/disabled with expected flood/no-flood behavior        |
| Verify IP ACLs                     | `show access-list <ip-acl-name>`                                                      | Permit/deny entries and hit counts match traffic tests                          |
| Verify EtherType ACLs              | `show access-list <ethertype-acl-name>`                                               | EtherType entries and hit counts match non-IP traffic tests                     |
| Verify access-group bindings       | `show access-group`                                                                   | ACLs are applied to expected interfaces and directions                          |
| Verify transparent NAT             | `show nat`                                                                            | Optional NAT rules appear in expected order                                     |
| Verify transparent NAT details     | `show nat detail`                                                                     | NAT hit counts increment for translated traffic                                 |
| Verify translations                | `show xlate`                                                                          | Optional NAT translations appear only when expected                             |
| Verify connection table            | `show conn`                                                                           | TCP/UDP/ICMP sessions through transparent firewall appear                       |
| Test host-to-gateway path          | `ping <default-gateway-ip>` from host                                                 | Host can reach gateway if ACL permits                                           |
| Test host-to-service path          | `Test-NetConnection <server-ip> -Port <port>` or equivalent                           | Permitted service succeeds                                                      |
| Capture ingress traffic            | `capture <cap-in> interface <ingress-interface> match ip host <src-ip> host <dst-ip>` | Packet arrives on expected ingress interface                                    |
| Capture egress traffic             | `capture <cap-out> interface <egress-interface> match ip host <src-ip> host <dst-ip>` | Packet exits expected egress interface                                          |
| View capture                       | `show capture <capture-name>`                                                         | Captured packets confirm forwarding or drop point                               |
| Verify ASP drops                   | `show asp drop`                                                                       | No relevant ACL, ARP inspection, L2F, NAT, or inspection drops increment        |
| Debug MAC learning                 | `debug mac-address-table`                                                             | ASA shows L2F learning events                                                   |
| Debug ARP inspection               | `debug arp-inspection`                                                                | ASA shows ARP inspection forwarding/drop behavior                               |
| Debug L2 host move                 | `debug l2-indication`                                                                 | ASA shows host move or static MAC conflict behavior                             |
| Clear interface MAC table          | `clear mac-address-table <interface-name>`                                            | MAC entries for selected interface are cleared                                  |
| Clear full MAC table               | `clear mac-address-table`                                                             | Dynamic L2F entries are cleared                                                 |
| Stop debugs                        | `undebug all`                                                                         | Debug output stops                                                              |

# ASA_Transparent_Mode_Rollback
| Step | Task                                                               | Device      | Command                                                                   | Expected Result                                                          |
| ---: | ------------------------------------------------------------------ | ----------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
|    1 | Save transparent config before rollback                            | ASA         | `copy running-config disk0:/transparent.cfg`                              | Transparent-mode config backup exists                                    |
|    2 | Confirm console access                                             | ASA / Notes | `<console-access-confirmed>`                                              | Safe rollback path exists                                                |
|    3 | Record current mode                                                | ASA         | `show firewall`                                                           | ASA shows transparent mode before rollback                               |
|    4 | Enter configuration mode                                           | ASA         | `configure terminal`                                                      | ASA enters global configuration mode                                     |
|    5 | Remove ARP inspection if backing out feature only                  | ASA         | `clear configure arp-inspection`                                          | ARP inspection returns to default                                        |
|    6 | Remove static ARP entry if lab-only                                | ASA         | `no arp <interface-name> <trusted-ip> <trusted-mac>`                      | Static ARP entry is removed                                              |
|    7 | Remove static MAC entry if lab-only                                | ASA         | `no mac-address-table static <interface-name> <mac-address>`              | Static L2F entry is removed                                              |
|    8 | Re-enable MAC learning if disabled                                 | ASA         | `no mac-learn <interface-name> disable`                                   | ASA resumes dynamic MAC learning                                         |
|    9 | Reset MAC aging if changed                                         | ASA         | `no mac-address-table aging-time <minutes>`                               | MAC aging returns to default                                             |
|   10 | Remove EtherType ACL binding if lab-only                           | ASA         | `no access-group <ethertype-acl> in interface <interface-name>`           | EtherType ACL is detached                                                |
|   11 | Remove EtherType ACL if unused                                     | ASA         | `clear configure access-list <ethertype-acl>`                             | EtherType ACL is removed                                                 |
|   12 | Remove IP ACL binding if lab-only                                  | ASA         | `no access-group <ip-acl> in interface <interface-name>`                  | IP ACL is detached                                                       |
|   13 | Remove IP ACL if unused                                            | ASA         | `clear configure access-list <ip-acl>`                                    | IP ACL is removed                                                        |
|   14 | Remove optional NAT if lab-only                                    | ASA         | `no nat (inside,outside) source static <real-object> <mapped-object>`     | Transparent NAT rule is removed                                          |
|   15 | Remove NAT objects if unused                                       | ASA         | `no object network <real-object>` and `no object network <mapped-object>` | Lab-only NAT objects are removed                                         |
|   16 | Remove same-security inter-interface if lab-only                   | ASA         | `no same-security-traffic permit inter-interface`                         | Same-security inter-interface permission is removed                      |
|   17 | Clear connections before mode rollback                             | ASA         | `clear conn`                                                              | Existing state is cleared                                                |
|   18 | Clear translations if NAT changed                                  | ASA         | `clear xlate`                                                             | Existing NAT translations are cleared                                    |
|   19 | Clear MAC table                                                    | ASA         | `clear mac-address-table`                                                 | Dynamic L2F entries are cleared                                          |
|   20 | Convert back to routed mode only if full mode rollback is intended | ASA         | `no firewall transparent`                                                 | ASA changes to routed mode and clears incompatible running configuration |
|   21 | Verify routed mode                                                 | ASA         | `show firewall`                                                           | Output shows routed mode                                                 |
|   22 | Restore routed-mode config if needed                               | ASA         | `copy disk0:/routed-before-transparent.cfg running-config`                | Routed-mode baseline is restored                                         |
|   23 | Save rollback state                                                | ASA         | `write memory`                                                            | Rollback configuration is saved                                          |


# ASA_Transparent_Mode_Failure_Checks
| Symptom                                            | Command                                                                         | What Usually Broke                                                                       |
| -------------------------------------------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| ASA loses SSH/ASDM after mode change               | Console access and `show firewall`                                              | `firewall transparent` cleared running config and management was not rebuilt             |
| Interfaces cannot be assigned IPs                  | `show firewall`                                                                 | Expected behavior in transparent mode; use BVI IP instead                                |
| BVI cannot be reached                              | `show running-config interface BVI<id>`, `show route`, `show mac-address-table` | BVI IP wrong subnet, bridge members down, route missing, or ACL blocks management        |
| Hosts in same subnet cannot pass traffic           | `show mac-address-table`, `show access-list`, captures                          | MAC learning, ACL, ARP inspection, or bridge-group membership issue                      |
| ASA does not learn MAC address                     | `show mac-address-table` and `debug mac-address-table`                          | Interface down, wrong bridge group, MAC learning disabled, or no traffic seen            |
| Host moved and traffic fails                       | `debug l2-indication` and `show mac-address-table`                              | Stale MAC entry or static MAC pinned to old interface                                    |
| Clearing MAC table fixes traffic temporarily       | `show mac-address-table`                                                        | MAC aging, host move, asymmetric L2 path, or static MAC issue                            |
| ARP fails through firewall                         | `show arp-inspection`, `show arp`, `debug arp-inspection`                       | ARP inspection no-flood enabled without complete static ARP table                        |
| ARP inspection drops valid hosts                   | `show arp-inspection` and syslog                                                | Static ARP entry missing or wrong MAC/IP/interface mapping                               |
| Non-IP traffic fails                               | `show access-list <ethertype-acl>`                                              | EtherType ACL missing permit or explicit deny blocks required frame type                 |
| BPDU behavior is wrong                             | `show access-list <ethertype-acl>`                                              | EtherType ACL permits or denies BPDU contrary to design                                  |
| IP traffic fails after EtherType ACL change        | `show access-list <ethertype-acl>`                                              | Explicit `ethertype deny any` may block IP traffic unexpectedly                          |
| DHCP/routing/multicast traffic fails               | `show access-list`, captures                                                    | Connectionless/special traffic may need ACLs on both interfaces                          |
| Dynamic routing does not work as expected          | `show running-config router`                                                    | RIP, OSPF, and EIGRP are not supported as ASA routing functions in transparent mode      |
| NAT rule has zero hits                             | `show nat detail`                                                               | Traffic does not match real/mapped objects, interface pair, or route path                |
| Translated address is unreachable from upstream    | Upstream router route table                                                     | Upstream router lacks static route for mapped address                                    |
| Same-security interface traffic fails              | `show running-config same-security-traffic`                                     | Missing `same-security-traffic permit inter-interface`                                   |
| Interface ACL blocks traffic                       | `show access-list <ip-acl>`                                                     | ACL missing permit or applied to wrong interface/direction                               |
| Packet enters but does not exit ASA                | Captures on both interfaces                                                     | ACL, ARP inspection, L2F table, NAT, inspection, or MAC learning issue                   |
| `show conn` shows no connection for attempted flow | `show access-list`, captures                                                    | Packet is dropped before connection creation or traffic is non-IP/connectionless         |
| Management syslog/SNMP/AAA fails                   | `show route`, `show running-config logging`, `show running-config aaa-server`   | ASA control-plane route points wrong direction or management interface route missing     |
| Multimode context behaves differently              | `show firewall` from system context                                             | Context is routed while expected transparent, or transparent while expected routed       |
| Config disappeared after mode conversion           | `show startup-config`, disk backups                                             | Expected behavior when switching routed/transparent modes without saving/exporting first |
| Debug output floods terminal                       | `show debug`                                                                    | Debugs left enabled; run `undebug all`                                                   |
