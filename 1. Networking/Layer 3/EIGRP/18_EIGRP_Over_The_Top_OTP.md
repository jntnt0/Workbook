Cisco’s IOS XE EIGRP OTP doc is the source for the exact OTP syntax here: CE devices use named-mode EIGRP with neighbor <remote-ip> <wan-interface> remote <hops> lisp-encap <id>, while EIGRP route reflectors use remote-neighbors source <interface> unicast-listen lisp-encap <id>, plus no next-hop-self and no split-horizon on the RR-facing interface. OTP uses EIGRP for the control plane and LISP encapsulation for the data plane across the WAN.  

EIGRP_Over_The_Top_OTP.md

EIGRP_Over_The_Top_OTP

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP DUAL, topology table, neighbor behavior, named mode, and troubleshooting to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports EIGRP neighbor formation, topology table, RTP, DUAL, successors, and route exchange |
| `All_combined_part3.md` | Book 3, Chapter 2: Advanced EIGRP Concepts, Data Structures and Algorithms | Supports DUAL convergence, passive/active routes, queries, replies, and feasible successor logic |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Named Mode | Supports named-mode EIGRP address-family structure |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | EIGRP Over the Top | Supports OTP CE syntax: `neighbor <remote-ip> <interface> remote <hops> lisp-encap <id>` |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | Configuring EIGRP Route Reflectors | Supports E-RR syntax: `remote-neighbors source <interface> unicast-listen lisp-encap <id>`, `no next-hop-self`, and `no split-horizon` |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | EIGRP OTP Support to Propagate SGT | Supports optional `cts propagate sgt` under named-mode `topology base` |
# EIGRP_Over_The_Top_OTP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EIGRP OTP | Runs one EIGRP routing domain across a WAN without relying on the provider to carry enterprise routes |
| Control plane | EIGRP advertises routes and forms remote neighbor relationships |
| Data plane | LISP encapsulates traffic across the WAN underlay |
| CE router | Customer edge router at a site participating in OTP |
| E-RR | EIGRP route reflector that receives EIGRP updates from CEs and reflects them to other CE neighbors |
| Underlay | The provider, Internet, MPLS, or routed WAN path that gives reachability between CE WAN-facing IPs |
| Overlay | The EIGRP OTP route exchange and LISP-encapsulated forwarding across that underlay |
| Remote neighbor | EIGRP neighbor reached over multiple underlay hops instead of a directly connected subnet |
| `remote <hops>` | Allows EIGRP to form a remote multihop neighbor relationship |
| `lisp-encap <id>` | Tells EIGRP OTP to use LISP encapsulation for data traffic |
| `network` statement | Advertises local site prefixes into OTP EIGRP |
| Route reflector model | CEs peer to E-RRs instead of building a full mesh to every other CE |
| `no next-hop-self` on E-RR | Prevents the E-RR from becoming the data-plane transit next hop |
| `no split-horizon` on E-RR | Allows the E-RR to reflect routes learned from one CE toward other CEs |
| `remote-neighbors source` | Lets the E-RR listen for unicast remote EIGRP neighbors using the WAN-facing source interface |
| Core prerequisite | CE WAN-facing addresses must already be reachable through the underlay before OTP matters |
| Main failure point | OTP fails when engineers troubleshoot EIGRP before proving underlay reachability between remote WAN IPs |
# EIGRP_Over_The_Top_OTP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm WAN-facing underlay interfaces are up | CE routers and E-RRs | `show ip interface brief` | WAN-facing interfaces are `up/up` with correct underlay IP addresses |
| 2 | Confirm LAN/site interfaces are up | CE routers | `show ip interface brief` | Site-facing interfaces are `up/up` with correct local prefix addresses |
| 3 | Confirm underlay reachability between CE and E-RR WAN IPs | CE routers | `ping <E_RR_WAN_IP> source <CE_WAN_INTERFACE>` | Ping succeeds before OTP configuration |
| 4 | Confirm underlay reachability from E-RR to each CE WAN IP | E-RR | `ping <CE_WAN_IP> source <E_RR_WAN_INTERFACE>` | Ping succeeds to every CE WAN IP |
| 5 | Confirm underlay route to remote CE or E-RR WAN IP | CE routers and E-RR | `show ip route <REMOTE_WAN_IP>` | Route exists through the WAN underlay |
| 6 | Confirm no accidental classic EIGRP process conflicts exist | All OTP routers | `show running-config | section router eigrp` | Existing EIGRP config is documented before named-mode OTP is added |
| 7 | Enter global configuration mode on the first CE | CE router | `configure terminal` | Router enters global configuration mode |
| 8 | Create named EIGRP process on the CE | CE router | `router eigrp <PROCESS_NAME>` | Router enters named EIGRP mode |
| 9 | Enter IPv4 unicast address family on the CE | CE router | `address-family ipv4 unicast autonomous-system <AS_NUMBER>` | CE enters EIGRP address-family mode |
| 10 | Configure OTP neighbor toward the E-RR | CE router | `neighbor <E_RR_WAN_IP> <CE_WAN_INTERFACE> remote <MAX_HOPS> lisp-encap <LISP_ID>` | CE is configured to form a remote EIGRP OTP neighbor with the E-RR |
| 11 | Advertise the CE local LAN/site prefix into EIGRP OTP | CE router | `network <LOCAL_SITE_NETWORK> <WILDCARD_MASK>` | CE injects local site prefix into the OTP EIGRP process |
| 12 | Advertise additional local prefixes if needed | CE router | `network <ADDITIONAL_LOCAL_NETWORK> <WILDCARD_MASK>` | Additional site prefixes are added to the EIGRP topology table |
| 13 | Avoid advertising the provider WAN if it should remain underlay-only | CE router | Planning step | WAN/provider infrastructure routes are not accidentally advertised as enterprise overlay prefixes |
| 14 | Exit CE configuration mode | CE router | `end` | CE returns to privileged EXEC mode |
| 15 | Repeat CE OTP neighbor and network configuration on every site CE | All CE routers | Same CE commands | Every CE points at the E-RR WAN IP and advertises its local site prefixes |
| 16 | Enter global configuration mode on the E-RR | E-RR | `configure terminal` | E-RR enters global configuration mode |
| 17 | Create named EIGRP process on the E-RR | E-RR | `router eigrp <PROCESS_NAME>` | E-RR enters named EIGRP mode |
| 18 | Enter IPv4 unicast address family on the E-RR | E-RR | `address-family ipv4 unicast autonomous-system <AS_NUMBER>` | E-RR enters EIGRP address-family mode |
| 19 | Enter the E-RR WAN-facing address-family interface | E-RR | `af-interface <E_RR_WAN_INTERFACE>` | E-RR enters EIGRP AF interface configuration |
| 20 | Disable next-hop-self on the E-RR WAN interface | E-RR | `no next-hop-self` | E-RR reflects routes without making itself the data-plane next hop |
| 21 | Disable split horizon on the E-RR WAN interface | E-RR | `no split-horizon` | E-RR can reflect routes learned from one CE toward other CEs |
| 22 | Exit AF interface mode | E-RR | `exit-af-interface` | E-RR returns to address-family mode |
| 23 | Enable remote-neighbor listening with LISP encapsulation on the E-RR | E-RR | `remote-neighbors source <E_RR_WAN_INTERFACE> unicast-listen lisp-encap <LISP_ID>` | E-RR listens for unicast remote EIGRP OTP neighbors |
| 24 | Advertise E-RR local routes if the E-RR is also acting as a CE | E-RR | `network <E_RR_LOCAL_NETWORK> <WILDCARD_MASK>` | E-RR local site prefixes are advertised if intended |
| 25 | Exit E-RR configuration mode | E-RR | `end` | E-RR returns to privileged EXEC mode |
| 26 | Verify named-mode OTP configuration on CEs | CE routers | `show running-config | section router eigrp` | CE shows named EIGRP, address-family, `neighbor ... remote ... lisp-encap`, and local `network` statements |
| 27 | Verify named-mode OTP configuration on E-RR | E-RR | `show running-config | section router eigrp` | E-RR shows `remote-neighbors source`, `no next-hop-self`, and `no split-horizon` |
| 28 | Verify remote EIGRP neighbor formation | CE routers and E-RR | `show ip eigrp neighbors` | CE routers form EIGRP neighbor adjacency with the E-RR |
| 29 | Verify EIGRP protocol state | CE routers and E-RR | `show ip protocols` | Correct named process, AS number, network statements, and remote neighbor behavior are visible |
| 30 | Verify remote site routes enter EIGRP topology table | CE routers | `show ip eigrp topology` | Remote CE site prefixes appear in passive state |
| 31 | Verify remote site routes enter the RIB | CE routers | `show ip route eigrp` | Remote site prefixes appear as EIGRP routes |
| 32 | Verify exact route to remote site prefix | CE routers | `show ip route <REMOTE_SITE_PREFIX>` | Route points to expected remote CE next hop, not the E-RR, when `no next-hop-self` is working |
| 33 | Verify LISP-related state if supported by platform | CE routers | `show ip lisp` | LISP state exists or platform shows OTP/LISP forwarding support |
| 34 | Test CE-to-CE site reachability | Source CE or host | `ping <REMOTE_SITE_IP> source <LOCAL_SITE_INTERFACE>` | Ping succeeds across OTP overlay |
| 35 | Verify CE-to-CE forwarding path | Source CE or host | `traceroute <REMOTE_SITE_IP>` | Path follows expected OTP/LISP overlay behavior |
| 36 | Verify E-RR is not carrying user traffic as transit if it should only reflect routes | E-RR | Interface counters or path trace review | E-RR reflects control plane and does not become unwanted data-plane transit |
| 37 | Optionally enable SGT propagation over OTP if TrustSec is in scope | CE routers and E-RR | `topology base` then `cts propagate sgt` | OTP carries SGT information across the Layer 3 WAN |
| 38 | Verify SGT propagation configuration if enabled | CE routers and E-RR | `show running-config | section router eigrp` | `cts propagate sgt` appears under `topology base` |
| 39 | Check logs for OTP/EIGRP instability | CE routers and E-RR | `show logging | include EIGRP|DUAL|LISP|NBRCHANGE` | No repeated neighbor resets or DUAL instability remains |
| 40 | Save the configuration | All OTP routers | `write memory` | Configuration is saved |
# EIGRP_Over_The_Top_OTP_Skeleton
! Baseline underlay checks
show ip interface brief
show ip route <REMOTE_WAN_IP>
ping <REMOTE_WAN_IP> source <LOCAL_WAN_INTERFACE>
show running-config | section router eigrp
! CE router OTP configuration
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   neighbor <E_RR_WAN_IP> <CE_WAN_INTERFACE> remote <MAX_HOPS> lisp-encap <LISP_ID>
   network <LOCAL_SITE_NETWORK> <WILDCARD_MASK>
   network <ADDITIONAL_LOCAL_NETWORK> <WILDCARD_MASK>
  exit-address-family
 end
! EIGRP Route Reflector configuration
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <E_RR_WAN_INTERFACE>
    no next-hop-self
    no split-horizon
   exit-af-interface
   remote-neighbors source <E_RR_WAN_INTERFACE> unicast-listen lisp-encap <LISP_ID>
   network <E_RR_LOCAL_NETWORK> <WILDCARD_MASK>
  exit-address-family
 end
! Optional SGT propagation over OTP
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    cts propagate sgt
   exit-af-topology
  exit-address-family
 end
! Verification
show running-config | section router eigrp
show ip protocols
show ip eigrp neighbors
show ip eigrp topology
show ip route eigrp
show ip route <REMOTE_SITE_PREFIX>
show ip lisp
show logging | include EIGRP|DUAL|LISP|NBRCHANGE
ping <REMOTE_SITE_IP> source <LOCAL_SITE_INTERFACE>
traceroute <REMOTE_SITE_IP>
! Save
write memory
# EIGRP_Over_The_Top_OTP_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms underlay and site-facing interface state | WAN and LAN interfaces are `up/up` |
| `show ip route <REMOTE_WAN_IP>` | Confirms underlay reachability to remote CE or E-RR WAN IP | Route exists through the WAN underlay |
| `ping <REMOTE_WAN_IP> source <LOCAL_WAN_INTERFACE>` | Proves underlay reachability before OTP troubleshooting | Ping succeeds |
| `show running-config | section router eigrp` | Confirms named-mode OTP configuration | Shows address-family, OTP neighbor, LISP encapsulation, E-RR settings, and network statements |
| `show ip protocols` | Confirms EIGRP process state | Correct process name, AS number, network statements, and remote neighbor behavior |
| `show ip eigrp neighbors` | Confirms OTP remote neighbor formation | CE routers form adjacencies with the E-RR |
| `show ip eigrp topology` | Confirms remote site prefixes are learned by EIGRP | Remote prefixes appear in passive state |
| `show ip route eigrp` | Confirms OTP-learned routes entered the RIB | Remote site prefixes appear as EIGRP routes |
| `show ip route <REMOTE_SITE_PREFIX>` | Confirms exact next-hop result | Route points to the correct remote CE path, not an unintended transit |
| `show ip lisp` | Checks LISP state where supported | Platform shows LISP or OTP-related forwarding state |
| `show logging | include EIGRP|DUAL|LISP|NBRCHANGE` | Checks control-plane instability | No recurring EIGRP neighbor resets, DUAL issues, or LISP errors |
| `ping <REMOTE_SITE_IP> source <LOCAL_SITE_INTERFACE>` | Confirms overlay reachability between sites | Ping succeeds |
| `traceroute <REMOTE_SITE_IP>` | Confirms forwarding path | Traffic follows expected overlay behavior |
# EIGRP_Over_The_Top_OTP_Rollback
! Remove CE OTP neighbor while leaving named EIGRP process intact
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   no neighbor <E_RR_WAN_IP> <CE_WAN_INTERFACE> remote <MAX_HOPS> lisp-encap <LISP_ID>
  exit-address-family
 end
! Remove CE local network advertisements if they were created only for OTP
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   no network <LOCAL_SITE_NETWORK> <WILDCARD_MASK>
   no network <ADDITIONAL_LOCAL_NETWORK> <WILDCARD_MASK>
  exit-address-family
 end
! Remove E-RR remote-neighbor listener
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   no remote-neighbors source <E_RR_WAN_INTERFACE> unicast-listen lisp-encap <LISP_ID>
  exit-address-family
 end
! Restore E-RR interface behavior if it was changed only for OTP
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <E_RR_WAN_INTERFACE>
    next-hop-self
    split-horizon
   exit-af-interface
  exit-address-family
 end
! Remove optional SGT propagation if configured only for OTP
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   topology base
    no cts propagate sgt
   exit-af-topology
  exit-address-family
 end
! Remove entire named EIGRP OTP process if the whole mechanism is being backed out
configure terminal
 no router eigrp <PROCESS_NAME>
end
! Verify rollback
show running-config | section router eigrp
show ip eigrp neighbors
show ip route eigrp
show ip eigrp topology
show ip route <REMOTE_SITE_PREFIX>
show ip lisp
ping <REMOTE_SITE_IP> source <LOCAL_SITE_INTERFACE>
traceroute <REMOTE_SITE_IP>
! Expected result:
! OTP remote neighbor relationship is removed.
! OTP-learned remote site routes disappear.
! Underlay reachability remains intact unless separately removed.
! EIGRP process remains only if intentionally retained.
write memory
# EIGRP_Over_The_Top_OTP_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| CE does not form OTP neighbor with E-RR | Underlay reachability is missing | `ping <E_RR_WAN_IP> source <CE_WAN_INTERFACE>` | Fix WAN underlay route before touching EIGRP |
| CE has no route to E-RR WAN IP | Missing static, BGP, or provider route in underlay | `show ip route <E_RR_WAN_IP>` | Add or repair underlay routing |
| E-RR does not accept CE neighbor | `remote-neighbors source` missing or wrong interface | `show running-config | section router eigrp` | Configure `remote-neighbors source <E_RR_WAN_INTERFACE> unicast-listen lisp-encap <ID>` |
| CE neighbor command has no effect | Wrong CE WAN interface specified | `show ip interface brief` and `show running-config | section router eigrp` | Use the interface that owns the CE underlay source IP |
| Neighbor does not form | AS number mismatch | `show ip protocols` | Match `address-family ipv4 unicast autonomous-system <AS_NUMBER>` |
| Neighbor does not form | Process name confusion | `show running-config | section router eigrp` | Remember process name is local; AS number must match |
| Neighbor does not form | `remote <MAX_HOPS>` too low | `traceroute <E_RR_WAN_IP>` | Increase remote maximum hops to cover underlay path |
| Neighbor forms but remote site routes are missing | CE local networks not advertised | `show ip protocols` and `show ip eigrp topology` | Add correct `network <LOCAL_SITE_NETWORK> <WILDCARD>` statements |
| Remote site routes appear but traffic fails | LISP/data-plane encapsulation or return path issue | `show ip route <REMOTE_SITE_PREFIX>` and remote `show ip route <SOURCE_PREFIX>` | Verify both directions have routes and platform supports OTP/LISP forwarding |
| Remote site route next hop is E-RR | `no next-hop-self` missing on E-RR | E-RR `show running-config | section router eigrp` | Configure `no next-hop-self` under E-RR WAN `af-interface` |
| E-RR receives routes but does not reflect them | Split horizon still enabled on E-RR WAN AF interface | E-RR `show running-config | section router eigrp` | Configure `no split-horizon` under E-RR WAN `af-interface` |
| CE routes are learned but not installed in RIB | Better route from another protocol exists | `show ip route <REMOTE_SITE_PREFIX>` | Compare AD and metric before blaming OTP |
| EIGRP topology shows active routes | DUAL query or neighbor instability | `show ip eigrp topology active` | Fix missing neighbor replies, unstable underlay, or route reachability |
| LISP verification command unavailable | Platform image lacks LISP show support or OTP feature support | `show version` and `show ip lisp ?` | Use a supported IOS XE image or document the feature gap in CML |
| `neighbor ... remote ... lisp-encap` command unavailable | Platform or image does not support EIGRP OTP | CLI help under EIGRP address-family | Use supported IOS XE platform/image |
| `remote-neighbors source` command unavailable | Platform or image does not support E-RR OTP behavior | CLI help under EIGRP address-family | Use a supported IOS XE platform/image |
| WAN/provider routes leak into enterprise EIGRP | Broad `network` statement | `show ip protocols` and `show ip route eigrp` | Narrow EIGRP `network` statements to enterprise site prefixes |
| E-RR becomes unwanted transit path | `no next-hop-self` missing or forwarding design wrong | `show ip route <REMOTE_SITE_PREFIX>` on CE | Restore E-RR route-reflector behavior and verify next-hop handling |
| Ping fails but route exists | Source interface or return route problem | `ping <REMOTE_SITE_IP> source <LOCAL_SITE_INTERFACE>` and remote `show ip route <LOCAL_SITE_PREFIX>` | Fix reverse route or use correct source interface |
| Repeated EIGRP neighbor resets | Underlay loss, MTU, CPU, timer, auth, or feature instability | `show logging | include EIGRP|DUAL|LISP|NBRCHANGE` | Fix underlay first, then EIGRP OTP |
| SGT propagation does not work | `cts propagate sgt` missing or TrustSec not otherwise configured | `show running-config | section router eigrp` | Configure SGT propagation only if TrustSec is actually in scope |
# Index
# Source_Basis
# EIGRP_Over_The_Top_OTP_Mental_Model
# EIGRP_Over_The_Top_OTP_Configuration_Checklist
# EIGRP_Over_The_Top_OTP_Skeleton
# EIGRP_Over_The_Top_OTP_Verification_Commands
# EIGRP_Over_The_Top_OTP_Rollback
# EIGRP_Over_The_Top_OTP_Failure_Checks