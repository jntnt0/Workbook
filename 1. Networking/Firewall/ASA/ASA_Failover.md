
```
# Index
# Source_Basis
# ASA_Failover_Mental_Model
# ASA_Failover_Configuration_Checklist
# ASA_Failover_Active_Standby_Primary_Skeleton
# ASA_Failover_Active_Standby_Secondary_Skeleton
# ASA_Failover_Shared_Control_And_State_Link_Skeleton
# ASA_Failover_Redundant_Control_And_State_Link_Skeleton
# ASA_Failover_Data_Interface_Standby_IP_Skeleton
# ASA_Failover_Timers_And_Interface_Policy_Skeleton
# ASA_Failover_HTTP_State_Replication_Skeleton
# ASA_Failover_Manual_Switchover_Skeleton
# ASA_Failover_Recover_Standby_Config_Sync_Skeleton
# ASA_Failover_Active_Active_Context_Skeleton
# ASA_Failover_Verification_Commands
# ASA_Failover_Rollback
# ASA_Failover_Failure_Checks
```



# ASA_Failover_Mental_Model
| Concept                         | Operational Meaning                                                                                                                      |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Failover pair                   | Two ASA units operate as one redundant firewall system                                                                                   |
| Primary unit                    | Statically assigned role; does not always mean currently active                                                                          |
| Secondary unit                  | Statically assigned role; does not always mean currently standby                                                                         |
| Active role                     | Unit currently passing transit traffic and owning the active interface IP/MAC values                                                     |
| Standby role                    | Unit not passing normal transit traffic; receives replicated config and state                                                            |
| Active/Standby                  | Normal single-context failover design; one ASA active, one ASA standby                                                                   |
| Active/Active                   | Multiple-context-only design where failover groups can be active on different physical units                                             |
| Failover control link           | ASA-to-ASA link used for hello messages, configuration sync, and peer health                                                             |
| Stateful failover link          | ASA-to-ASA link used to replicate connection, ARP, route, and VPN state                                                                  |
| Shared failover/state link      | One link can carry both control and stateful replication                                                                                 |
| Separate failover/state links   | Control and stateful replication can use different isolated subnets                                                                      |
| Standby interface IP            | Secondary IP address configured on every monitored data interface                                                                        |
| Virtual MAC                     | Optional active/standby MAC pair that stabilizes upstream/downstream ARP behavior during switchover                                      |
| Interface monitoring            | ASA tracks data interface health and can trigger switchover if enough monitored interfaces fail                                          |
| Configuration replication       | Active unit pushes config to standby; changes should be made on active only                                                              |
| `write standby`                 | Active unit overwrites standby running config to repair config inconsistency                                                             |
| `no failover active`            | Manual command on active unit to force the peer to become active                                                                         |
| `failover active`               | Manual command on standby unit to make itself active                                                                                     |
| No preemption in Active/Standby | Former primary does not automatically become active again after recovery                                                                 |
| Blunt rule                      | Configure and manage the active ASA. Do not casually configure the standby, or you create sync problems that will bite during switchover |



# ASA_Failover_Configuration_Checklist
| Step | Task                                                                  | Device                | Command                                                                                                                | Expected Result                                                                 |                                                                             |
| ---: | --------------------------------------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
|    1 | Confirm both ASAs are same platform/model family                      | ASA1, ASA2            | `show version`                                                                                                         | Hardware/software details are visible for comparison                            |                                                                             |
|    2 | Confirm ASA software versions match                                   | ASA1, ASA2            | `show version`                                                                                                         | Both units run compatible or matching ASA software                              |                                                                             |
|    3 | Confirm licenses support failover                                     | ASA1, ASA2            | `show activation-key` or `show version`                                                                                | Failover and required encryption/license features are available                 |                                                                             |
|    4 | Confirm firewall mode                                                 | ASA1, ASA2            | `show firewall`                                                                                                        | Both units use the same firewall mode                                           |                                                                             |
|    5 | Confirm context mode                                                  | ASA1, ASA2            | `show mode`                                                                                                            | Both units use the same single or multiple context mode                         |                                                                             |
|    6 | Confirm Active/Standby design                                         | ASA / Notes           | `Active/Standby`                                                                                                       | Correct model is chosen for single-context failover                             |                                                                             |
|    7 | Confirm Active/Active only if multiple context mode is required       | ASA / Notes           | `Active/Active requires multiple-context mode`                                                                         | Active/Active is not confused with normal Active/Standby                        |                                                                             |
|    8 | Confirm data interface cabling symmetry                               | ASA1, ASA2 / Switches | `<same VLANs/same segments>`                                                                                           | Corresponding interfaces connect to the same L2 networks                        |                                                                             |
|    9 | Confirm failover link cabling                                         | ASA1, ASA2            | `<failover-link-cable>`                                                                                                | Failover link is isolated, preferably direct or dedicated                       |                                                                             |
|   10 | Confirm state link cabling                                            | ASA1, ASA2            | `<state-link-cable>`                                                                                                   | Stateful link is isolated if separate from control link                         |                                                                             |
|   11 | Confirm failover control subnet                                       | ASA / Notes           | `<failover-control-ip> <standby-ip> <mask>`                                                                            | Subnet does not overlap data interfaces or NAT policy                           |                                                                             |
|   12 | Confirm stateful failover subnet if separate                          | ASA / Notes           | `<state-link-ip> <standby-ip> <mask>`                                                                                  | Separate state subnet does not overlap any other interface                      |                                                                             |
|   13 | Confirm data active and standby IPs                                   | ASA / Notes           | `<active-ip> standby <standby-ip>`                                                                                     | Every monitored data interface has a standby IP plan                            |                                                                             |
|   14 | Confirm virtual MAC plan if used                                      | ASA / Notes           | `<active-mac> standby <standby-mac>`                                                                                   | Stable MAC values are planned for data interfaces                               |                                                                             |
|   15 | Backup existing primary config                                        | Primary ASA           | `copy running-config disk0:/before-failover-primary.cfg`                                                               | Recovery copy exists                                                            |                                                                             |
|   16 | Backup existing secondary config if any                               | Secondary ASA         | `copy running-config disk0:/before-failover-secondary.cfg`                                                             | Recovery copy exists                                                            |                                                                             |
|   17 | Clear secondary interface config if adding to existing primary        | Secondary ASA         | `clear configure interface`                                                                                            | Secondary has no conflicting interface config                                   |                                                                             |
|   18 | Enter configuration mode on primary                                   | Primary ASA           | `configure terminal`                                                                                                   | Primary enters global configuration mode                                        |                                                                             |
|   19 | Enable failover physical control interface                            | Primary ASA           | `interface <failover-control-physical-interface>` then `no shutdown`                                                   | Control physical interface is enabled                                           |                                                                             |
|   20 | Enable state physical interface if separate                           | Primary ASA           | `interface <state-physical-interface>` then `no shutdown`                                                              | State physical interface is enabled                                             |                                                                             |
|   21 | Create redundant failover interface if using redundant physical links | Primary ASA           | `interface Redundant<id>`                                                                                              | Redundant interface mode is entered                                             |                                                                             |
|   22 | Add first member to redundant failover interface                      | Primary ASA           | `member-interface <physical-interface-1>`                                                                              | First physical link joins redundant interface                                   |                                                                             |
|   23 | Add second member to redundant failover interface                     | Primary ASA           | `member-interface <physical-interface-2>`                                                                              | Second physical link joins redundant interface                                  |                                                                             |
|   24 | Set primary unit role                                                 | Primary ASA           | `failover lan unit primary`                                                                                            | Unit is statically designated primary                                           |                                                                             |
|   25 | Configure failover control interface                                  | Primary ASA           | `failover lan interface <failover-link-name> <interface>`                                                              | Interface becomes failover control link                                         |                                                                             |
|   26 | Configure failover control IPs                                        | Primary ASA           | `failover interface ip <failover-link-name> <primary-ip> <mask> standby <secondary-ip>`                                | Primary and secondary failover link IPs are defined                             |                                                                             |
|   27 | Configure stateful link using same link if desired                    | Primary ASA           | `failover link <failover-link-name>`                                                                                   | Stateful failover uses the failover control link                                |                                                                             |
|   28 | Configure separate stateful link if desired                           | Primary ASA           | `failover link <state-link-name> <state-interface>`                                                                    | Dedicated stateful replication link is defined                                  |                                                                             |
|   29 | Configure separate stateful link IPs if separate                      | Primary ASA           | `failover interface ip <state-link-name> <primary-state-ip> <mask> standby <secondary-state-ip>`                       | Stateful link IPs are defined                                                   |                                                                             |
|   30 | Encrypt failover/state traffic if link is not physically trusted      | Primary ASA           | `failover ipsec pre-shared-key <key>`                                                                                  | Failover control/state traffic is protected                                     |                                                                             |
|   31 | Enable HTTP connection replication only if required                   | Primary ASA           | `failover replication http`                                                                                            | HTTP connection state is replicated                                             |                                                                             |
|   32 | Configure unit failover timers if using custom timers                 | Primary ASA           | `failover polltime unit msec <poll-ms> holdtime msec <hold-ms>`                                                        | Unit hello and hold timers are set                                              |                                                                             |
|   33 | Configure interface monitoring timers if using custom timers          | Primary ASA           | `failover polltime interface msec <poll-ms> holdtime <seconds>`                                                        | Interface polling and hold timers are set                                       |                                                                             |
|   34 | Configure interface failure threshold if needed                       | Primary ASA           | `failover interface-policy <number-or-percent>`                                                                        | Switchover threshold is changed from default behavior                           |                                                                             |
|   35 | Configure outside data interface                                      | Primary ASA           | `interface <outside-interface>`                                                                                        | Outside interface mode is entered                                               |                                                                             |
|   36 | Name outside interface                                                | Primary ASA           | `nameif outside`                                                                                                       | Outside logical interface exists                                                |                                                                             |
|   37 | Set outside security level                                            | Primary ASA           | `security-level 0`                                                                                                     | Outside security level is set                                                   |                                                                             |
|   38 | Configure outside active and standby IPs                              | Primary ASA           | `ip address <outside-active-ip> <mask> standby <outside-standby-ip>`                                                   | Outside has active and standby addresses                                        |                                                                             |
|   39 | Configure outside virtual MACs if used                                | Primary ASA           | `mac-address <outside-active-mac> standby <outside-standby-mac>`                                                       | Outside MAC behavior is stable across failover                                  |                                                                             |
|   40 | Enable outside interface                                              | Primary ASA           | `no shutdown`                                                                                                          | Outside interface is enabled                                                    |                                                                             |
|   41 | Configure inside data interface                                       | Primary ASA           | `interface <inside-interface>`                                                                                         | Inside interface mode is entered                                                |                                                                             |
|   42 | Name inside interface                                                 | Primary ASA           | `nameif inside`                                                                                                        | Inside logical interface exists                                                 |                                                                             |
|   43 | Set inside security level                                             | Primary ASA           | `security-level 100`                                                                                                   | Inside security level is set                                                    |                                                                             |
|   44 | Configure inside active and standby IPs                               | Primary ASA           | `ip address <inside-active-ip> <mask> standby <inside-standby-ip>`                                                     | Inside has active and standby addresses                                         |                                                                             |
|   45 | Configure inside virtual MACs if used                                 | Primary ASA           | `mac-address <inside-active-mac> standby <inside-standby-mac>`                                                         | Inside MAC behavior is stable across failover                                   |                                                                             |
|   46 | Enable inside interface                                               | Primary ASA           | `no shutdown`                                                                                                          | Inside interface is enabled                                                     |                                                                             |
|   47 | Monitor outside interface                                             | Primary ASA           | `monitor-interface outside`                                                                                            | Failover monitors outside health                                                |                                                                             |
|   48 | Monitor inside interface                                              | Primary ASA           | `monitor-interface inside`                                                                                             | Failover monitors inside health                                                 |                                                                             |
|   49 | Exclude noncritical interface if needed                               | Primary ASA           | `no monitor-interface <interface-name>`                                                                                | Noncritical interface failure does not trigger failover                         |                                                                             |
|   50 | Enable failover on primary                                            | Primary ASA           | `failover`                                                                                                             | Primary enters failover negotiation and becomes active if no active mate exists |                                                                             |
|   51 | Verify primary active state                                           | Primary ASA           | `show failover                                                                                                         | include host`                                                                   | Output shows `This host: Primary - Active` and peer not detected or standby |
|   52 | Save primary config                                                   | Primary ASA           | `write memory`                                                                                                         | Primary startup config is saved                                                 |                                                                             |
|   53 | Prepare secondary physical failover interfaces                        | Secondary ASA         | `interface <failover-control-physical-interface>` then `no shutdown`                                                   | Secondary failover physical interface is enabled                                |                                                                             |
|   54 | Prepare secondary state physical interface if separate                | Secondary ASA         | `interface <state-physical-interface>` then `no shutdown`                                                              | Secondary state physical interface is enabled                                   |                                                                             |
|   55 | Create redundant failover interface on secondary if used              | Secondary ASA         | `interface Redundant<id>` then `member-interface <physical-interface-1>` and `member-interface <physical-interface-2>` | Secondary redundant failover link matches primary                               |                                                                             |
|   56 | Set secondary unit role                                               | Secondary ASA         | `failover lan unit secondary`                                                                                          | Unit is statically designated secondary                                         |                                                                             |
|   57 | Configure same failover control interface on secondary                | Secondary ASA         | `failover lan interface <failover-link-name> <interface>`                                                              | Secondary uses same logical failover interface name                             |                                                                             |
|   58 | Configure same failover control IP pair on secondary                  | Secondary ASA         | `failover interface ip <failover-link-name> <primary-ip> <mask> standby <secondary-ip>`                                | Same IP pair is configured; secondary uses standby address                      |                                                                             |
|   59 | Configure same stateful link on secondary                             | Secondary ASA         | `failover link <failover-link-name>` or `failover link <state-link-name> <state-interface>`                            | Secondary state link matches primary                                            |                                                                             |
|   60 | Configure same separate state IP pair on secondary if used            | Secondary ASA         | `failover interface ip <state-link-name> <primary-state-ip> <mask> standby <secondary-state-ip>`                       | Same state link IP pair is configured                                           |                                                                             |
|   61 | Configure same failover IPsec key on secondary if used                | Secondary ASA         | `failover ipsec pre-shared-key <key>`                                                                                  | Secondary can authenticate/protect failover link                                |                                                                             |
|   62 | Enable failover on secondary                                          | Secondary ASA         | `failover`                                                                                                             | Secondary detects active mate and begins configuration replication              |                                                                             |
|   63 | Verify secondary sync                                                 | Secondary ASA         | `show failover                                                                                                         | include host`                                                                   | Output shows `This host: Secondary - Standby Ready` and primary active      |
|   64 | Verify full failover status                                           | Active ASA            | `show failover`                                                                                                        | Control link, state link, monitored interfaces, and peer status are normal      |                                                                             |
|   65 | Verify failover config replication                                    | Active ASA            | `show running-config failover`                                                                                         | Primary and secondary failover config is consistent except unit role            |                                                                             |
|   66 | Verify standby received running config                                | Standby ASA           | `show running-config`                                                                                                  | Standby has replicated data interface, NAT, ACL, route, and VPN config          |                                                                             |
|   67 | Sync standby manually if needed                                       | Active ASA            | `write standby`                                                                                                        | Active overwrites standby running config                                        |                                                                             |
|   68 | Verify interface status on both units                                 | Active ASA            | `show failover`                                                                                                        | Data interfaces show `Normal` on both units                                     |                                                                             |
|   69 | Verify stateful replication counters                                  | Active ASA            | `show failover`                                                                                                        | Stateful update counters are visible and errors are not climbing                |                                                                             |
|   70 | Generate test traffic                                                 | Host / Router         | `ping <remote-host>` or service test                                                                                   | Transit traffic works through active ASA                                        |                                                                             |
|   71 | Verify connection replication                                         | Active ASA            | `show conn`                                                                                                            | Active has connection entries for test traffic                                  |                                                                             |
|   72 | Verify standby state if possible                                      | Standby ASA           | `show conn`                                                                                                            | Standby has replicated state where expected                                     |                                                                             |
|   73 | Test manual switchover from active                                    | Active ASA            | `no failover active`                                                                                                   | Peer becomes active                                                             |                                                                             |
|   74 | Verify new active unit                                                | New Active ASA        | `show failover                                                                                                         | include host`                                                                   | Former standby now shows active                                             |
|   75 | Verify traffic survives switchover                                    | Host / Router         | `ping <remote-host>` or active service test                                                                            | Traffic impact is minimal if stateful failover is working                       |                                                                             |
|   76 | Switch back manually if desired                                       | Standby or Active ASA | `failover active` or `no failover active`                                                                              | Desired unit becomes active                                                     |                                                                             |
|   77 | Review failover history                                               | ASA                   | `show failover history`                                                                                                | Recent state transitions and reasons are visible                                |                                                                             |
|   78 | Save final config                                                     | Active ASA            | `write memory`                                                                                                         | Final failover configuration is saved                                           |                                                                             |

```
# ASA_Failover_Active_Standby_Primary_Skeleton
configure terminal

interface <failover-control-interface>
 no shutdown

interface <state-interface>
 no shutdown

failover lan unit primary
failover lan interface <failover-link-name> <failover-control-interface>
failover interface ip <failover-link-name> <primary-failover-ip> <failover-mask> standby <secondary-failover-ip>

failover link <state-link-name> <state-interface>
failover interface ip <state-link-name> <primary-state-ip> <state-mask> standby <secondary-state-ip>

failover ipsec pre-shared-key <failover-key>

interface <outside-interface>
 nameif outside
 security-level 0
 ip address <outside-active-ip> <outside-mask> standby <outside-standby-ip>
 no shutdown

interface <inside-interface>
 nameif inside
 security-level 100
 ip address <inside-active-ip> <inside-mask> standby <inside-standby-ip>
 no shutdown

monitor-interface outside
monitor-interface inside

failover

write memory
```


```
# ASA_Failover_Active_Standby_Secondary_Skeleton
configure terminal

interface <failover-control-interface>
 no shutdown

interface <state-interface>
 no shutdown

failover lan unit secondary
failover lan interface <failover-link-name> <failover-control-interface>
failover interface ip <failover-link-name> <primary-failover-ip> <failover-mask> standby <secondary-failover-ip>

failover link <state-link-name> <state-interface>
failover interface ip <state-link-name> <primary-state-ip> <state-mask> standby <secondary-state-ip>

failover ipsec pre-shared-key <failover-key>

failover

write memory
```


```
# ASA_Failover_Shared_Control_And_State_Link_Skeleton
configure terminal

interface <failover-interface>
 no shutdown

failover lan unit <primary-or-secondary>
failover lan interface <failover-link-name> <failover-interface>
failover interface ip <failover-link-name> <primary-failover-ip> <failover-mask> standby <secondary-failover-ip>
failover link <failover-link-name>

failover

write memory
```


```
# ASA_Failover_Redundant_Control_And_State_Link_Skeleton
configure terminal

interface <physical-interface-1>
 no shutdown

interface <physical-interface-2>
 no shutdown

interface Redundant<id>
 description LAN Failover Interface
 member-interface <physical-interface-1>
 member-interface <physical-interface-2>

failover lan unit <primary-or-secondary>
failover lan interface <failover-link-name> Redundant<id>
failover interface ip <failover-link-name> <primary-failover-ip> <failover-mask> standby <secondary-failover-ip>
failover link <failover-link-name>

failover

write memory
```


```
# ASA_Failover_Data_Interface_Standby_IP_Skeleton
configure terminal

interface <outside-interface>
 nameif outside
 security-level 0
 mac-address <outside-active-mac> standby <outside-standby-mac>
 ip address <outside-active-ip> <outside-mask> standby <outside-standby-ip>
 no shutdown

interface <inside-interface>
 nameif inside
 security-level 100
 mac-address <inside-active-mac> standby <inside-standby-mac>
 ip address <inside-active-ip> <inside-mask> standby <inside-standby-ip>
 no shutdown

monitor-interface outside
monitor-interface inside

write memory
```


```
# ASA_Failover_Timers_And_Interface_Policy_Skeleton
configure terminal

failover polltime unit msec <unit-poll-ms> holdtime msec <unit-hold-ms>
failover polltime interface msec <interface-poll-ms> holdtime <interface-hold-seconds>
failover interface-policy <failed-interface-threshold>

write memory
```


```
# ASA_Failover_HTTP_State_Replication_Skeleton
configure terminal

failover replication http

write memory
```


```
# ASA_Failover_Manual_Switchover_Skeleton
# Run from the currently active ASA to force the peer active:
no failover active

# Or run from the standby ASA to make it active:
failover active
```


```
# ASA_Failover_Recover_Standby_Config_Sync_Skeleton
# Run from the active ASA after accidental standby-side config changes:
write standby
```

```
# ASA_Failover_Active_Active_Context_Skeleton
configure terminal

mode multiple

failover group 1
 primary
 preempt <seconds>
 interface-policy <threshold>
 failover polltime interface <seconds> holdtime <seconds>

failover group 2
 secondary
 preempt <seconds>

admin-context admin

context admin
 allocate-interface <admin-interface>
 config-url flash:/admin.cfg
 join-failover-group 1

context <context-a>
 allocate-interface <context-a-interface-1>
 allocate-interface <context-a-interface-2>
 config-url flash:/<context-a>.cfg
 join-failover-group 1

context <context-b>
 allocate-interface <context-b-interface-1>
 allocate-interface <context-b-interface-2>
 config-url flash:/<context-b>.cfg
 join-failover-group 2

write memory
```


# ASA_Failover_Verification_Commands
| Task                                   | Command                                                                | Expected Result                                                    |                                                  |
| -------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------ |
| Verify firewall mode                   | `show firewall`                                                        | Mode matches the failover design                                   |                                                  |
| Verify context mode                    | `show mode`                                                            | Single or multiple context mode matches design                     |                                                  |
| Verify software and platform           | `show version`                                                         | Peers are compatible                                               |                                                  |
| Verify license state                   | `show activation-key` or `show version`                                | Failover licensing is present where required                       |                                                  |
| Verify failover configuration          | `show running-config failover`                                         | Failover role, links, IPs, timers, and key are present             |                                                  |
| Verify failover state                  | `show failover`                                                        | Failover is on and one unit is active while peer is standby ready  |                                                  |
| Verify concise host roles              | `show failover                                                         | include host`                                                      | Output identifies this host and other host state |
| Verify control link                    | `show failover`                                                        | Failover LAN interface is up                                       |                                                  |
| Verify stateful link                   | `show failover`                                                        | Stateful failover link is up if configured                         |                                                  |
| Verify monitored interfaces            | `show failover`                                                        | Expected interfaces show `Normal`                                  |                                                  |
| Verify data interface standby IPs      | `show running-config interface <interface-name>`                       | Interface has `ip address <active-ip> <mask> standby <standby-ip>` |                                                  |
| Verify virtual MACs                    | `show running-config interface <interface-name>`                       | Interface has active and standby MACs if configured                |                                                  |
| Verify failover history                | `show failover history`                                                | Past failover transitions and reasons are visible                  |                                                  |
| Verify active connections              | `show conn`                                                            | Active unit has connection entries                                 |                                                  |
| Verify state replication counters      | `show failover`                                                        | Stateful object counters increment without rising xerr/rerr errors |                                                  |
| Verify NAT state after switchover      | `show xlate`                                                           | Expected translations exist after failover                         |                                                  |
| Verify ARP state                       | `show arp`                                                             | Active unit has expected ARP entries                               |                                                  |
| Verify routing after failover          | `show route`                                                           | Active unit has expected static or replicated routes               |                                                  |
| Verify VPN state replication           | `show crypto ikev1 sa`, `show crypto ikev2 sa`, `show crypto ipsec sa` | VPN SAs exist or rebuild as expected                               |                                                  |
| Verify standby logging if enabled      | `show running-config logging`                                          | `logging standby` appears only if intentionally configured         |                                                  |
| Verify failover debug only when needed | `debug fover fail`                                                     | Debug shows failover failure/sync details                          |                                                  |
| Stop debugs                            | `undebug all`                                                          | Debug output stops                                                 |                                                  |

# ASA_Failover_Rollback
| Step | Task                                                                    | Device      | Command                                                                                    | Expected Result                                  |
| ---: | ----------------------------------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------ |
|    1 | Confirm current failover state                                          | ASA         | `show failover`                                                                            | Active and standby roles are known               |
|    2 | Backup current config                                                   | Active ASA  | `copy running-config disk0:/before-failover-rollback.cfg`                                  | Recovery copy exists                             |
|    3 | Move production traffic to intended unit if needed                      | ASA         | `failover active` or `no failover active`                                                  | Desired unit is active before rollback           |
|    4 | Enter configuration mode                                                | Active ASA  | `configure terminal`                                                                       | ASA enters global configuration mode             |
|    5 | Disable failover on active if fully removing pair                       | Active ASA  | `no failover`                                                                              | Failover operation stops on active               |
|    6 | Disable failover on standby if accessible                               | Standby ASA | `no failover`                                                                              | Failover operation stops on standby              |
|    7 | Remove failover control link config                                     | ASA         | `no failover lan interface <failover-link-name> <interface>`                               | Failover control link is removed                 |
|    8 | Remove stateful link config                                             | ASA         | `no failover link <state-link-name> <interface>`                                           | Stateful link is removed                         |
|    9 | Remove failover interface IPs                                           | ASA         | `no failover interface ip <failover-link-name> <primary-ip> <mask> standby <secondary-ip>` | Failover IP pair is removed                      |
|   10 | Remove failover IPsec key                                               | ASA         | `no failover ipsec pre-shared-key <failover-key>`                                          | Failover link encryption key is removed          |
|   11 | Remove HTTP replication if enabled                                      | ASA         | `no failover replication http`                                                             | HTTP state replication is disabled               |
|   12 | Remove custom failover timers                                           | ASA         | `no failover polltime unit` and `no failover polltime interface`                           | Timers return to default                         |
|   13 | Remove custom interface policy                                          | ASA         | `no failover interface-policy <threshold>`                                                 | Interface failure threshold returns to default   |
|   14 | Remove data interface monitoring if required                            | ASA         | `no monitor-interface <interface-name>`                                                    | Interface is no longer monitored                 |
|   15 | Remove standby IPs only if reverting to standalone interface addressing | ASA         | `interface <interface-name>` then `ip address <active-ip> <mask>`                          | Interface no longer has standby IP               |
|   16 | Remove virtual MACs if no longer needed                                 | ASA         | `interface <interface-name>` then `no mac-address <active-mac> standby <standby-mac>`      | Interface uses default/burned-in MAC behavior    |
|   17 | Remove redundant failover interface if lab-only                         | ASA         | `interface Redundant<id>` then `no member-interface <physical-interface>`                  | Redundant failover interface members are removed |
|   18 | Restore standalone interface config if needed                           | ASA         | `copy disk0:/before-failover-primary.cfg running-config`                                   | Standalone config is restored                    |
|   19 | Clear stale connections                                                 | ASA         | `clear conn`                                                                               | Old connection state is cleared                  |
|   20 | Clear stale translations                                                | ASA         | `clear xlate`                                                                              | Old NAT translations are cleared                 |
|   21 | Verify failover is off                                                  | ASA         | `show failover`                                                                            | Failover is disabled                             |
|   22 | Save rollback state                                                     | ASA         | `write memory`                                                                             | Rollback is saved                                |


# ASA_Failover_Failure_Checks
| Symptom                                            | Command                                                                | What Usually Broke                                                                                                      |
| -------------------------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Secondary never joins pair                         | `show failover`                                                        | Failover link down, wrong unit role, wrong failover IPs, software mismatch, license issue, or interface config conflict |
| Both ASAs become active                            | `show failover history` and failover link checks                       | Failover control link failed or units cannot hear hello messages                                                        |
| Standby stuck in Cold Standby                      | `show failover`                                                        | Hardware/software/license mismatch or failed config negotiation                                                         |
| Config does not replicate                          | `show running-config failover` and `show failover`                     | Standby was configured manually, failover link issue, or config sync failure                                            |
| Standby config differs from active                 | `show running-config` on both units                                    | Changes were made on standby; fix from active using `write standby`                                                     |
| Data interface shows failed                        | `show failover`                                                        | Interface down, switch VLAN issue, cable issue, missing standby IP, or monitoring problem                               |
| Unexpected switchover after one interface failure  | `show failover history`                                                | Default interface policy allows failover after one monitored interface failure                                          |
| Failover flaps repeatedly                          | `show failover history` and `show interface`                           | Aggressive timers, unstable interface, congested failover link, or bad switch path                                      |
| Stateful counters show xerr/rerr climbing          | `show failover`                                                        | Stateful link congestion, peer load, packet loss, or undersized state link                                              |
| Connections drop during failover                   | `show failover`, `show conn`, `show xlate`                             | Stateless failover, state link issue, unsupported application state, or old flow not replicated                         |
| HTTP sessions reset during failover                | `show running-config failover`                                         | `failover replication http` not enabled or application is too short-lived/sensitive                                     |
| VPN sessions fail after failover                   | `show crypto ikev1 sa`, `show crypto ikev2 sa`, `show crypto ipsec sa` | VPN state not replicated cleanly, peer sees new IP/MAC behavior, or tunnel must rekey                                   |
| Standby cannot be managed                          | `show failover` and management ACLs                                    | Standby management IP missing, ACL/SSH/ASDM config issue, or wrong active/standby IP used                               |
| Primary does not automatically become active again | `show failover history`                                                | Normal Active/Standby behavior; there is no preemption                                                                  |
| Dynamic routing adjacency only exists on active    | `show route` and routing protocol show commands                        | Normal behavior; standby does not form dynamic routing adjacencies                                                      |
| ARP issues after failover                          | `show arp` and upstream switch/router ARP tables                       | No virtual MACs, stale upstream ARP, or gratuitous ARP not accepted                                                     |
| Upstream switch sees MAC flaps                     | Switch MAC table and ASA `show failover history`                       | Cabling/L2 adjacency issue, virtual MAC conflict, or repeated failover                                                  |
| Failover link cannot be configured                 | `show running-config interface <interface>`                            | Interface already has non-failover config or is not enabled                                                             |
| Secondary rejects failover interface config        | `show running-config interface`                                        | Secondary physical/subinterface setup does not match primary                                                            |
| Failover IP subnet overlaps data network           | `show running-config failover` and `show interface ip brief`           | Failover/state subnet overlaps a data interface or NAT policy                                                           |
| Active/Active does not work in single context mode | `show mode`                                                            | Active/Active requires multiple context mode                                                                            |
| Active/Active contexts fail over unexpectedly      | `show failover group` or `show failover`                               | Failover group interface policy, ASR, context assignment, or monitored interface issue                                  |
| Debug output floods console                        | `show debug`                                                           | Failover debug left enabled; run `undebug all`                                                                          |