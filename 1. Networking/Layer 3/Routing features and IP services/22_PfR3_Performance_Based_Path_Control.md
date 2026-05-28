Cisco’s current IOS XE 17.x IP Routing Configuration Guide says PfRv3/IWAN is not supported from IOS XE 17.4 onward, so treat this as legacy IWAN/PfRv3 lab syntax unless your image supports it.  

PfR3_Performance_Based_Path_Control.md
# PfR3_Performance_Based_Path_Control
# PfR3_Performance_Based_Path_Control_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PfRv3 | Performance Routing Version 3 is Cisco IWAN path-control logic for choosing WAN paths based on application class and path performance |
| Path control | PfRv3 can move selected traffic classes away from a degraded path instead of waiting for routing to fail |
| Not a routing protocol | PfRv3 does not replace BGP, EIGRP, OSPF, DMVPN, or static routing; it influences forwarding after reachability exists |
| Hub Master Controller | Central hub role where enterprise policies, site prefixes, and traffic classes are defined |
| Hub Border Router | Hub WAN edge role where external WAN paths terminate and report path information to the hub master |
| Branch Master Controller | Branch-site decision role that receives policy from the hub and makes local optimization decisions |
| Branch Border Router | Branch WAN edge role that participates in measurement and forwarding for local WAN exits |
| Domain | PfRv3 control scope; all participating devices must use the same domain name |
| VRF | PfRv3 configuration is built under a domain VRF, usually `vrf default` in simple labs |
| Source interface | Loopback used for stable PfRv3 peering between MC and BR roles |
| Site prefixes | Prefixes that identify the local site |
| Enterprise prefix | Prefix range considered internal enterprise reachability |
| Path name | Human-readable WAN path label such as `MPLS` or `INET`; path names must be consistent across policy and border interfaces |
| Path ID | Numeric identifier for a path/interface inside the PfRv3 domain |
| Traffic class | Application or DSCP-matched traffic bucket controlled by policy |
| Policy thresholds | Delay, loss, jitter, or byte-loss thresholds define when a path is out of policy |
| Path preference | Preferred path order for a traffic class, with fallback behavior if the preferred path violates policy |
| Load balance | Allows non-policy/default traffic to be balanced across eligible external paths |
| Performance Monitor | PfRv3 uses performance monitoring and smart probes to learn path quality |
| PfR route override | Routing table may show `p` flags or next-hop overrides when PfRv3 controls forwarding |
| Operational warning | PfRv3 is legacy IWAN-era technology and is not supported from IOS XE 17.4 onward |
# PfR3_Performance_Based_Path_Control_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm image support before starting | All PfRv3 Routers | `show version`<br>`domain ?` | Platform and IOS XE image support PfRv3 domain configuration |
| 2 | Confirm base routing is already functional | All PfRv3 Routers | `show ip route` | Hub and branch loopbacks, WAN tunnel endpoints, and site prefixes are reachable |
| 3 | Confirm loopbacks exist for PfRv3 source interfaces | All PfRv3 Routers | `show ip interface brief | include Loopback` | Each MC and BR role has a stable loopback address |
| 4 | Confirm WAN tunnels or external interfaces are up | Hub BR / Branch BR | `show ip interface brief` | MPLS, INET, DMVPN, or WAN-facing interfaces are `up/up` |
| 5 | Confirm WAN path routing before PfRv3 | Hub BR / Branch BR | `show ip route <REMOTE_SITE_PREFIX>` | Remote site prefixes are reachable before PfRv3 controls path selection |
| 6 | Confirm CEF forwarding before PfRv3 | Hub BR / Branch BR | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Normal routing path is known before PfRv3 changes forwarding |
| 7 | Create enterprise prefix list on the hub master | Hub MC | `conf t`<br>`ip prefix-list PL-ENTERPRISE seq 10 permit <ENTERPRISE_PREFIX>/<LEN> le <MAX_LEN>`<br>`end` | Enterprise address space is defined for PfRv3 policy scope |
| 8 | Create hub site prefix list on the hub master | Hub MC | `conf t`<br>`ip prefix-list PL-HUB-SITE seq 10 permit <HUB_SITE_PREFIX>/<LEN> le <MAX_LEN>`<br>`end` | Hub-local prefixes are defined |
| 9 | Configure PfRv3 domain and VRF on the hub master | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`end` | Domain and VRF context exist |
| 10 | Configure hub master role | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`end` | Device enters hub master controller role |
| 11 | Configure hub master source interface | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   source-interface Loopback0`<br>`end` | Hub MC uses Loopback0 for PfRv3 peering |
| 12 | Configure enterprise prefix scope | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   enterprise-prefix prefix-list PL-ENTERPRISE`<br>`end` | Hub MC knows the enterprise prefix boundary |
| 13 | Configure hub site prefixes | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   site-prefixes prefix-list PL-HUB-SITE`<br>`end` | Hub site prefixes are published into the PfRv3 domain |
| 14 | Enable default traffic load balancing if intended | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   load-balance`<br>`end` | Default class can be load balanced across eligible paths |
| 15 | Configure DSCP monitoring interval if needed | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   monitor-interval <SECONDS> dscp <DSCP_VALUE>`<br>`end` | PfRv3 monitors the selected DSCP class interval |
| 16 | Configure voice traffic class | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   class VOICE sequence 10`<br>`    match dscp ef policy custom`<br>`    path-preference MPLS fallback INET`<br>`end` | Voice traffic is matched and prefers MPLS with INET fallback |
| 17 | Configure voice delay threshold | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   class VOICE sequence 10`<br>`    priority 1 one-way-delay threshold 150 msec`<br>`end` | Voice class treats one-way delay above threshold as policy violation |
| 18 | Configure voice packet-loss threshold | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   class VOICE sequence 10`<br>`    priority 2 packet-loss-rate threshold 5.0 percent`<br>`end` | Voice class tracks packet loss threshold |
| 19 | Configure voice byte-loss threshold | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   class VOICE sequence 10`<br>`    priority 2 byte-loss-rate threshold 5.0 percent`<br>`end` | Voice class tracks byte-loss threshold |
| 20 | Configure video traffic class | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   class VIDEO sequence 20`<br>`    match dscp af41 policy custom`<br>`    path-preference INET fallback MPLS`<br>`    priority 1 one-way-delay threshold 150 msec`<br>`    priority 2 packet-loss-rate threshold 5.0 percent`<br>`    priority 2 byte-loss-rate threshold 5.0 percent`<br>`end` | Video traffic prefers INET unless performance violates policy |
| 21 | Configure critical-data traffic class | Hub MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master hub`<br>`   class CRITICAL sequence 30`<br>`    match dscp af31 policy custom`<br>`    path-preference MPLS fallback INET`<br>`    priority 1 one-way-delay threshold 600 msec`<br>`    priority 2 packet-loss-rate threshold 10.0 percent`<br>`    priority 2 byte-loss-rate threshold 10.0 percent`<br>`end` | Critical data is classified and given path preference |
| 22 | Optional: configure branch-to-branch behavior on hub or branch master where required | Branch MC | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master branch`<br>`   branch-to-branch`<br>`end` | Branch-to-branch optimization behavior is enabled where supported |
| 23 | Configure PfRv3 domain and border role on hub border router | Hub BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  border`<br>`end` | Hub BR enters border role |
| 24 | Configure hub border source interface | Hub BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  border`<br>`   source-interface Loopback0`<br>`end` | Hub BR uses Loopback0 for PfRv3 peering |
| 25 | Point hub border to the hub master controller | Hub BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  border`<br>`   master <HUB_MC_LOOPBACK_IP>`<br>`end` | Hub BR registers to the hub MC |
| 26 | Mark MPLS tunnel as a PfRv3 path | Hub BR | `conf t`<br>`interface <MPLS_TUNNEL_INTERFACE>`<br>` bandwidth <MPLS_BANDWIDTH_KBPS>`<br>` load-interval 30`<br>` domain <DOMAIN_NAME> path MPLS path-id 1`<br>`end` | MPLS tunnel is an external PfRv3 path |
| 27 | Mark internet tunnel as a PfRv3 path | Hub BR | `conf t`<br>`interface <INET_TUNNEL_INTERFACE>`<br>` bandwidth <INET_BANDWIDTH_KBPS>`<br>` load-interval 30`<br>` domain <DOMAIN_NAME> path INET path-id 2`<br>`end` | INET tunnel is an external PfRv3 path |
| 28 | Optional: mark internet-bound path on hub internet edge designs | Hub BR | `conf t`<br>`interface <INET_INTERFACE>`<br>` domain <DOMAIN_NAME> path INET path-id 2 internet-bound`<br>`end` | Interface is treated as internet-bound where supported and intended |
| 29 | Configure branch domain and border role | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  border`<br>`end` | Branch router has PfRv3 border role |
| 30 | Configure branch border source interface | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  border`<br>`   source-interface Loopback0`<br>`end` | Branch border uses Loopback0 for PfRv3 peering |
| 31 | Configure branch border to use local master when MC and BR are co-located | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  border`<br>`   master local`<br>`end` | Branch border registers to the local branch MC |
| 32 | Configure branch master role | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master branch`<br>`end` | Router enters branch master role |
| 33 | Configure branch master source interface | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master branch`<br>`   source-interface Loopback0`<br>`end` | Branch MC uses Loopback0 for PfRv3 peering |
| 34 | Configure hub address under branch master | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master branch`<br>`   hub <HUB_MC_LOOPBACK_IP>`<br>`end` | Branch MC learns policy from the hub MC |
| 35 | Configure branch traffic-class scale limit if required | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master branch`<br>`   traffic-class-max <MAX_CLASSES>`<br>`end` | Branch MC limits learned traffic classes to the configured value |
| 36 | Configure route update dampener if required | Branch MC/BR | `conf t`<br>`domain <DOMAIN_NAME>`<br>` vrf default`<br>`  master branch`<br>`   route-update-dampner <SECONDS>`<br>`end` | PfRv3 route update behavior is dampened |
| 37 | Mark branch MPLS tunnel as a PfRv3 path if the platform/design requires explicit path marking | Branch BR | `conf t`<br>`interface <MPLS_TUNNEL_INTERFACE>`<br>` bandwidth <MPLS_BANDWIDTH_KBPS>`<br>` load-interval 30`<br>` domain <DOMAIN_NAME> path MPLS path-id 1`<br>`end` | Branch MPLS tunnel is associated with the PfRv3 domain |
| 38 | Mark branch INET tunnel as a PfRv3 path if the platform/design requires explicit path marking | Branch BR | `conf t`<br>`interface <INET_TUNNEL_INTERFACE>`<br>` bandwidth <INET_BANDWIDTH_KBPS>`<br>` load-interval 30`<br>` domain <DOMAIN_NAME> path INET path-id 2`<br>`end` | Branch INET tunnel is associated with the PfRv3 domain |
| 39 | Verify hub master status | Hub MC | `show domain <DOMAIN_NAME> master status` | Hub MC operational status is `Up` |
| 40 | Verify hub master peering | Hub MC | `show domain <DOMAIN_NAME> master peering` | Hub MC peering is enabled and border/branch services are visible |
| 41 | Verify hub master exits | Hub MC | `show domain <DOMAIN_NAME> master exits` | MPLS and INET external paths appear with expected BR and interface information |
| 42 | Verify hub master policy | Hub MC | `show domain <DOMAIN_NAME> master policy` | VOICE, VIDEO, CRITICAL, and default classes appear with expected match and path-preference |
| 43 | Verify hub border status | Hub BR | `show domain <DOMAIN_NAME> border status` | Border instance status is `UP` and connection to master is `UP` |
| 44 | Verify border peering | Hub BR / Branch BR | `show domain <DOMAIN_NAME> border peering` | Border peering is enabled and points to the correct MC |
| 45 | Verify site-prefix database | Hub MC / Branch MC / BR | `show domain <DOMAIN_NAME> master site-prefix`<br>`show domain <DOMAIN_NAME> border site-prefix` | Local and remote site prefixes are learned or configured correctly |
| 46 | Verify PfRv3 channels | Hub BR / Branch BR | `show domain <DOMAIN_NAME> border channels` | Channels are reachable for expected DSCP/site/interface combinations |
| 47 | Verify traffic classes on master | Hub MC / Branch MC | `show domain <DOMAIN_NAME> master traffic-classes summary`<br>`show domain <DOMAIN_NAME> master traffic-classes` | Traffic classes are learned and have active policy state |
| 48 | Verify performance-monitor objects were auto-created | BR | `show flow monitor type performance-monitor` | PfRv3-generated performance-monitor objects are present on WAN egress interfaces |
| 49 | Verify PfRv3 route influence in routing table | BR | `show ip route <REMOTE_SITE_PREFIX>` | Route output may show PfRv3 override indicator where PfRv3 is controlling next hop |
| 50 | Verify CEF path for a controlled traffic class | BR | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` | Forwarding path matches expected PfRv3-selected exit |
| 51 | Generate traffic matching a configured DSCP class | Test Host / Router | `ping <DESTINATION_IP> tos <TOS_VALUE>` | Traffic class counters and path selection can be observed |
| 52 | Create path impairment in a lab | WAN Path Device | `! Introduce delay, loss, or shut a test tunnel` | Preferred path violates policy or becomes unreachable |
| 53 | Verify traffic class moves to fallback path | Hub MC / Branch MC | `show domain <DOMAIN_NAME> master traffic-classes` | Affected class changes path according to policy and fallback order |
| 54 | Restore path impairment | WAN Path Device | `! Remove delay, loss, or no shut tunnel` | Path becomes compliant again |
| 55 | Verify failback behavior | Hub MC / Branch MC | `show domain <DOMAIN_NAME> master traffic-classes` | Traffic class returns based on fallback timer and dampening behavior |
| 56 | Save the working configuration | All PfRv3 Routers | `copy running-config startup-config` | PfRv3 domain configuration survives reload |
# PfR3_Performance_Based_Path_Control_Skeleton
conf t
!
ip prefix-list PL-ENTERPRISE seq 10 permit <ENTERPRISE_PREFIX>/<LEN> le <MAX_LEN>
ip prefix-list PL-HUB-SITE seq 10 permit <HUB_SITE_PREFIX>/<LEN> le <MAX_LEN>
!
domain <DOMAIN_NAME>
 vrf default
  master hub
   source-interface Loopback0
   enterprise-prefix prefix-list PL-ENTERPRISE
   site-prefixes prefix-list PL-HUB-SITE
   load-balance
   monitor-interval <SECONDS> dscp ef
   class VOICE sequence 10
    match dscp ef policy custom
    path-preference MPLS fallback INET
    priority 1 one-way-delay threshold 150 msec
    priority 2 packet-loss-rate threshold 5.0 percent
    priority 2 byte-loss-rate threshold 5.0 percent
   exit
   class VIDEO sequence 20
    match dscp af41 policy custom
    path-preference INET fallback MPLS
    priority 1 one-way-delay threshold 150 msec
    priority 2 packet-loss-rate threshold 5.0 percent
    priority 2 byte-loss-rate threshold 5.0 percent
   exit
   class CRITICAL sequence 30
    match dscp af31 policy custom
    path-preference MPLS fallback INET
    priority 1 one-way-delay threshold 600 msec
    priority 2 packet-loss-rate threshold 10.0 percent
    priority 2 byte-loss-rate threshold 10.0 percent
   exit
  exit
 exit
exit
!
end
write memory
# Hub_Master_Controller_Skeleton
conf t
!
interface Loopback0
 description PFRV3_HUB_MC_SOURCE
 ip address <HUB_MC_LOOPBACK_IP> 255.255.255.255
!
ip prefix-list PL-ENTERPRISE seq 10 permit <ENTERPRISE_PREFIX>/<LEN> le <MAX_LEN>
ip prefix-list PL-HUB-SITE seq 10 permit <HUB_SITE_PREFIX>/<LEN> le <MAX_LEN>
!
domain <DOMAIN_NAME>
 vrf default
  master hub
   source-interface Loopback0
   enterprise-prefix prefix-list PL-ENTERPRISE
   site-prefixes prefix-list PL-HUB-SITE
   load-balance
   class VOICE sequence 10
    match dscp ef policy custom
    path-preference MPLS fallback INET
    priority 1 one-way-delay threshold 150 msec
    priority 2 packet-loss-rate threshold 5.0 percent
    priority 2 byte-loss-rate threshold 5.0 percent
   exit
   class VIDEO sequence 20
    match dscp af41 policy custom
    path-preference INET fallback MPLS
    priority 1 one-way-delay threshold 150 msec
    priority 2 packet-loss-rate threshold 5.0 percent
    priority 2 byte-loss-rate threshold 5.0 percent
   exit
  exit
 exit
exit
!
end
write memory
# Hub_Border_Router_Skeleton
conf t
!
interface Loopback0
 description PFRV3_HUB_BR_SOURCE
 ip address <HUB_BR_LOOPBACK_IP> 255.255.255.255
!
domain <DOMAIN_NAME>
 vrf default
  border
   source-interface Loopback0
   master <HUB_MC_LOOPBACK_IP>
  exit
 exit
exit
!
interface <MPLS_TUNNEL_INTERFACE>
 description PFRV3_PATH_MPLS
 bandwidth <MPLS_BANDWIDTH_KBPS>
 load-interval 30
 domain <DOMAIN_NAME> path MPLS path-id 1
!
interface <INET_TUNNEL_INTERFACE>
 description PFRV3_PATH_INET
 bandwidth <INET_BANDWIDTH_KBPS>
 load-interval 30
 domain <DOMAIN_NAME> path INET path-id 2
!
end
write memory
# Branch_Master_And_Border_CoLocated_Skeleton
conf t
!
interface Loopback0
 description PFRV3_BRANCH_MC_BR_SOURCE
 ip address <BRANCH_LOOPBACK_IP> 255.255.255.255
!
domain <DOMAIN_NAME>
 vrf default
  border
   source-interface Loopback0
   master local
  exit
  master branch
   source-interface Loopback0
   hub <HUB_MC_LOOPBACK_IP>
   traffic-class-max <MAX_CLASSES>
   route-update-dampner <SECONDS>
  exit
 exit
exit
!
interface <MPLS_TUNNEL_INTERFACE>
 description PFRV3_BRANCH_PATH_MPLS
 bandwidth <MPLS_BANDWIDTH_KBPS>
 load-interval 30
 domain <DOMAIN_NAME> path MPLS path-id 1
!
interface <INET_TUNNEL_INTERFACE>
 description PFRV3_BRANCH_PATH_INET
 bandwidth <INET_BANDWIDTH_KBPS>
 load-interval 30
 domain <DOMAIN_NAME> path INET path-id 2
!
end
write memory
# PfR3_Advanced_Master_Tuning_Skeleton
conf t
!
domain <DOMAIN_NAME>
 vrf default
  master hub
   advanced
    fallback-timer <MINUTES>
    threshold-variance <PERCENT>
   exit
  exit
 exit
exit
!
end
write memory
# PfR3_Smart_Probe_Burst_Skeleton
conf t
!
domain <DOMAIN_NAME>
 vrf default
  master hub
   advanced
    smart-probes burst packets every <INTERVAL_SECONDS>
   exit
  exit
 exit
exit
!
end
write memory
# PfR3_Performance_Based_Path_Control_Verification_Commands
show version
show ip interface brief
show running-config | section ^domain
show running-config interface Loopback0
show running-config interface <MPLS_TUNNEL_INTERFACE>
show running-config interface <INET_TUNNEL_INTERFACE>
show ip route
show ip route <REMOTE_SITE_PREFIX>
show ip route <HUB_MC_LOOPBACK_IP>
show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>
show domain <DOMAIN_NAME> master status
show domain <DOMAIN_NAME> master peering
show domain <DOMAIN_NAME> master exits
show domain <DOMAIN_NAME> master policy
show domain <DOMAIN_NAME> master site-prefix
show domain <DOMAIN_NAME> master traffic-classes summary
show domain <DOMAIN_NAME> master traffic-classes
show domain <DOMAIN_NAME> border status
show domain <DOMAIN_NAME> border peering
show domain <DOMAIN_NAME> border site-prefix
show domain <DOMAIN_NAME> border channels
show domain <DOMAIN_NAME> border channels parent route
show domain <DOMAIN_NAME> border pmi
show flow monitor type performance-monitor
show platform pfrv3 rp active smart-probe
show platform pfrv3 fp active smart-probe
show interfaces <MPLS_TUNNEL_INTERFACE>
show interfaces <INET_TUNNEL_INTERFACE>
show policy-map interface
show access-lists
ping <REMOTE_SITE_IP> source <LOCAL_SITE_IP_OR_INTERFACE>
traceroute <REMOTE_SITE_IP> source <LOCAL_SITE_IP>
debug platform software pfrv3 smart-probe
debug platform software pfrv3 channel
debug platform software pfrv3 route-control
debug platform software pfrv3 site-prefix
show debugging
undebug all
# PfR3_Performance_Based_Path_Control_Rollback
conf t
!
interface <MPLS_TUNNEL_INTERFACE>
 no domain <DOMAIN_NAME> path MPLS path-id 1
 no domain <DOMAIN_NAME> path MPLS
!
interface <INET_TUNNEL_INTERFACE>
 no domain <DOMAIN_NAME> path INET path-id 2
 no domain <DOMAIN_NAME> path INET
!
no domain <DOMAIN_NAME>
!
no ip prefix-list PL-ENTERPRISE
no ip prefix-list PL-HUB-SITE
!
end
write memory
# Hub_Master_Controller_Rollback
conf t
!
domain <DOMAIN_NAME>
 vrf default
  master hub
   no class VOICE sequence 10
   no class VIDEO sequence 20
   no class CRITICAL sequence 30
   no load-balance
   no enterprise-prefix prefix-list PL-ENTERPRISE
   no site-prefixes prefix-list PL-HUB-SITE
   no source-interface Loopback0
  exit
  no master hub
 exit
exit
!
no ip prefix-list PL-ENTERPRISE
no ip prefix-list PL-HUB-SITE
!
end
write memory
# Hub_Border_Router_Rollback
conf t
!
interface <MPLS_TUNNEL_INTERFACE>
 no domain <DOMAIN_NAME> path MPLS path-id 1
 no domain <DOMAIN_NAME> path MPLS
!
interface <INET_TUNNEL_INTERFACE>
 no domain <DOMAIN_NAME> path INET path-id 2
 no domain <DOMAIN_NAME> path INET
!
domain <DOMAIN_NAME>
 vrf default
  border
   no master <HUB_MC_LOOPBACK_IP>
   no source-interface Loopback0
  exit
  no border
 exit
exit
!
end
write memory
# Branch_Master_And_Border_Rollback
conf t
!
interface <MPLS_TUNNEL_INTERFACE>
 no domain <DOMAIN_NAME> path MPLS path-id 1
 no domain <DOMAIN_NAME> path MPLS
!
interface <INET_TUNNEL_INTERFACE>
 no domain <DOMAIN_NAME> path INET path-id 2
 no domain <DOMAIN_NAME> path INET
!
domain <DOMAIN_NAME>
 vrf default
  border
   no master local
   no source-interface Loopback0
  exit
  no border
  master branch
   no hub <HUB_MC_LOOPBACK_IP>
   no traffic-class-max <MAX_CLASSES>
   no route-update-dampner <SECONDS>
   no source-interface Loopback0
  exit
  no master branch
 exit
exit
!
end
write memory
# PfR3_Debug_Rollback
undebug all
# PfR3_Performance_Based_Path_Control_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `domain` command is missing | Image or platform does not support PfRv3 | `show version` and `domain ?` | Use a supported legacy IOS XE image or move to SD-WAN application-aware routing |
| PfRv3 configuration is accepted but never becomes operational | Base routing or loopback reachability is broken | `show ip route <PEER_LOOPBACK>` | Fix underlay, DMVPN, BGP/EIGRP/OSPF, or static routing first |
| Hub master status is down | Hub MC source interface or domain VRF is wrong | `show domain <DOMAIN_NAME> master status` | Correct `source-interface`, VRF, and loopback reachability |
| Border cannot connect to master | Wrong master IP or loopback reachability failure | `show domain <DOMAIN_NAME> border status` | Correct `master <HUB_MC_LOOPBACK_IP>` and routing to MC loopback |
| Domain peers reject each other | Domain name mismatch | `show running-config | section ^domain` | Use the same domain name on all PfRv3 devices |
| Border role missing | `border` not configured under domain VRF | `show running-config | section ^domain` | Configure `domain <NAME>`, `vrf default`, `border` |
| Branch local MC and BR do not peer | `master local` missing under border role | `show domain <DOMAIN_NAME> border peering` | Configure `master local` when branch MC and BR are co-located |
| Branch does not receive hub policy | Branch master has wrong hub IP or cannot reach hub MC | `show domain <DOMAIN_NAME> master peering` | Correct `hub <HUB_MC_LOOPBACK_IP>` and routing |
| External paths do not appear | WAN interfaces lack PfRv3 `domain path` configuration | `show domain <DOMAIN_NAME> master exits` | Add `domain <DOMAIN_NAME> path <PATH_NAME> path-id <ID>` under WAN interfaces |
| Path names do not match policy | Interface path name differs from `path-preference` name | Interface config and `show domain <DOMAIN_NAME> master policy` | Make path names consistent, such as `MPLS` and `INET` |
| Path name command is rejected | Path name is too long or unsupported syntax is used | Interface config parser help | Use a short supported path name |
| Traffic class never appears | Match DSCP or application classification does not see traffic | `show domain <DOMAIN_NAME> master traffic-classes` | Verify DSCP marking, NBAR/app recognition, and traffic direction |
| Default traffic is not performance controlled | Default class is load-balanced, not measured like custom classes | `show domain <DOMAIN_NAME> master policy` | Create explicit traffic class for traffic that needs SLA-based control |
| Voice stays on bad primary path | Threshold is too loose or no alternate path is compliant | `show domain <DOMAIN_NAME> master traffic-classes` | Tighten thresholds or fix alternate path performance |
| Traffic moves too often | Thresholds are too tight or fallback timer/dampening is weak | `show domain <DOMAIN_NAME> master traffic-classes` | Tune thresholds, fallback timer, and dampening |
| Traffic never fails back | Fallback timer is disabled or primary remains out of policy | `show domain <DOMAIN_NAME> master traffic-classes` | Restore fallback timer and fix primary path performance |
| Smart probes fail | Path/channel not reachable or blocked | `show domain <DOMAIN_NAME> border channels` | Fix tunnel reachability, ACLs, CoPP, or path MTU |
| Performance monitor objects are missing | PfRv3 did not generate passive monitoring on WAN egress | `show flow monitor type performance-monitor` | Verify border role, WAN path config, and operational status |
| Site prefixes are missing | Site prefix learning or configured prefix list is wrong | `show domain <DOMAIN_NAME> master site-prefix` | Correct `site-prefixes prefix-list` or routing visibility |
| Enterprise prefix is too broad | PfRv3 classifies unintended destinations as enterprise traffic | `show domain <DOMAIN_NAME> master site-prefix` | Tighten `enterprise-prefix` prefix list |
| Enterprise prefix is too narrow | Internal prefixes treated as internet or outside domain | `show domain <DOMAIN_NAME> master site-prefix` | Add missing internal prefixes to enterprise prefix list |
| PfRv3 route override surprises routing table | PfRv3 is controlling next hop for a traffic class | `show ip route <PREFIX>` and `show domain <DOMAIN_NAME> master traffic-classes` | Verify this is intended path control, not a routing protocol failure |
| CEF path does not match expected policy | Tested flow does not match traffic class or DSCP | `show ip cef exact-route <SOURCE_IP> <DESTINATION_IP>` and traffic-class output | Test with the correct source, destination, DSCP, and direction |
| Interface utilization is wrong | Bandwidth command is missing or inaccurate on WAN path | `show interfaces <WAN_INTERFACE>` | Configure realistic `bandwidth <KBPS>` on WAN interfaces |
| Loss/delay test gives no policy movement | Impairment not on the path used by the measured class | `show domain <DOMAIN_NAME> border channels` | Impair the actual path/channel used by the traffic class |
| Debug output is too noisy | PfRv3 debug left running | `show debugging` | Run `undebug all` |
| Config disappears after reload | Configuration was not saved | `show startup-config | section ^domain` | Reconfigure and save with `copy running-config startup-config` |
| Upgrade breaks PfRv3 | IOS XE release no longer supports PfRv3 | `show version` | Use supported legacy release for labs or migrate to SD-WAN application-aware routing |
# Index
PfR3_Performance_Based_Path_Control.md
PfR3_Performance_Based_Path_Control
PfR3_Performance_Based_Path_Control_Mental_Model
PfR3_Performance_Based_Path_Control_Configuration_Checklist
PfR3_Performance_Based_Path_Control_Skeleton
Hub_Master_Controller_Skeleton
Hub_Border_Router_Skeleton
Branch_Master_And_Border_CoLocated_Skeleton
PfR3_Advanced_Master_Tuning_Skeleton
PfR3_Smart_Probe_Burst_Skeleton
PfR3_Performance_Based_Path_Control_Verification_Commands
PfR3_Performance_Based_Path_Control_Rollback
Hub_Master_Controller_Rollback
Hub_Border_Router_Rollback
Branch_Master_And_Border_Rollback
PfR3_Debug_Rollback
PfR3_Performance_Based_Path_Control_Failure_Checks