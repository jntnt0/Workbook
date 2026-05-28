
# DMVPN_QoS_Overlay_Treatment_Mental_Model

| Concept | Operational Meaning |

|---|---|

| QoS over DMVPN | QoS is traffic treatment applied to traffic crossing a DMVPN overlay |

| DMVPN first | Underlay, tunnel, NHRP, routing, and optional IPsec must work before QoS matters |

| Classification | Identify traffic classes such as voice, video, routing, critical data, bulk data, and default |

| Marking | Set or preserve DSCP values so downstream devices can treat traffic consistently |

| Queuing | Decide which traffic gets priority during congestion |

| LLQ | Low-latency queue for real-time traffic such as voice, but it must be capped |

| CBWFQ | Reserves bandwidth for non-priority classes during congestion |

| Shaping | Slows traffic to the real WAN/transport rate so queuing happens on your router, not inside the provider cloud |

| Policing | Drops or remarks traffic above a configured rate; less friendly than shaping for WAN egress |

| Tunnel bandwidth | Logical reference used by routing protocols and QoS math; it does not create physical bandwidth |

| MTU/MSS | GRE and IPsec overhead reduce usable payload size; bad MTU breaks applications before QoS can help |

| `qos pre-classify` | Lets QoS classify based on inner packet headers before GRE/IPsec encapsulation hides them |

| Physical egress policy | Usually where WAN shaping and queuing are applied because congestion occurs on the transport-facing interface |

| Tunnel policy | Can be useful for overlay-facing classification or per-tunnel designs, but hardware/platform support must be checked |

| Per-tunnel QoS | DMVPN-specific method to apply different output policies per spoke using NHRP groups |

| Hard rule | QoS does not create bandwidth; it only decides which packets suffer first during congestion |

# DMVPN_QoS_Overlay_Treatment_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Confirm underlay NBMA reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA reachability works before QoS is added |

| 2 | Confirm tunnel state | Hub, Spokes | `show ip interface brief | include Tunnel` | Tunnel interfaces show `up/up` |

| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | Peers show `UP` |

| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub and spokes have expected tunnel-to-NBMA mappings |

| 5 | Confirm overlay routing works | Hub, Spokes | `show ip route`<br>`ping <remote-lan-ip>` | Remote LAN reachability works before QoS is added |

| 6 | Confirm whether IPsec is enabled | Hub, Spokes | `show running-config interface Tunnel<id> | include tunnel protection` | You know whether encryption will hide inner headers |

| 7 | Confirm tunnel MTU and MSS baseline | Hub, Spokes | `show running-config interface Tunnel<id> | include mtu|mss` | Tunnel has sane MTU/MSS values before QoS testing |

| 8 | Configure tunnel MTU if missing | Hub, Spokes | `interface Tunnel<id>`<br>`ip mtu 1400` | Tunnel accounts for GRE/IPsec overhead |

| 9 | Configure TCP MSS adjustment if missing | Hub, Spokes | `interface Tunnel<id>`<br>`ip tcp adjust-mss 1360` | TCP flows avoid fragmentation across the overlay |

| 10 | Set tunnel bandwidth reference | Hub, Spokes | `interface Tunnel<id>`<br>`bandwidth <kbps>` | Routing and QoS have a realistic tunnel bandwidth reference |

| 11 | Enable QoS pre-classification when GRE/IPsec encapsulation would hide inner traffic | Hub, Spokes | `interface Tunnel<id>`<br>`qos pre-classify` | QoS can classify traffic by original inner headers before encapsulation |

| 12 | Identify real WAN egress bandwidth | Hub, Spokes | Design check | Real shaping rate is known, usually below physical interface speed |

| 13 | Define voice class | Hub, Spokes | `class-map match-any DMVPN_VOICE`<br>`match dscp ef` | Voice traffic can be matched by DSCP EF |

| 14 | Define video class if required | Hub, Spokes | `class-map match-any DMVPN_VIDEO`<br>`match dscp af41 af42 af43` | Video traffic can be matched by DSCP AF41/AF42/AF43 |

| 15 | Define routing/control class if required | Hub, Spokes | `class-map match-any DMVPN_CONTROL`<br>`match dscp cs6` | Control-plane or routing-marked traffic can be matched |

| 16 | Define critical-data class | Hub, Spokes | `class-map match-any DMVPN_CRITICAL_DATA`<br>`match dscp af31 af32 af33` | Critical application traffic can be matched |

| 17 | Define scavenger class if required | Hub, Spokes | `class-map match-any DMVPN_SCAVENGER`<br>`match dscp cs1` | Low-priority traffic can be matched |

| 18 | Create child queuing policy | Hub, Spokes | `policy-map DMVPN_CHILD_QOS` | Child policy-map is created |

| 19 | Add voice LLQ treatment | Hub, Spokes | `class DMVPN_VOICE`<br>`priority percent <percent>` | Voice gets strict priority during congestion, capped by configured percent |

| 20 | Add video bandwidth treatment | Hub, Spokes | `class DMVPN_VIDEO`<br>`bandwidth percent <percent>` | Video receives reserved bandwidth during congestion |

| 21 | Add critical-data bandwidth treatment | Hub, Spokes | `class DMVPN_CRITICAL_DATA`<br>`bandwidth percent <percent>` | Critical data receives reserved bandwidth during congestion |

| 22 | Add scavenger policing or minimal bandwidth if required | Hub, Spokes | `class DMVPN_SCAVENGER`<br>`bandwidth percent <low-percent>` | Scavenger traffic receives low-priority treatment |

| 23 | Add default class behavior | Hub, Spokes | `class class-default`<br>`fair-queue` | Default traffic is fairly queued if platform supports it |

| 24 | Create parent shaping policy | Hub, Spokes | `policy-map DMVPN_PARENT_SHAPE` | Parent policy-map is created |

| 25 | Shape to real WAN egress rate | Hub, Spokes | `class class-default`<br>`shape average <bps>` | Router shapes below real WAN/Internet uplink rate |

| 26 | Attach child policy under parent shaper | Hub, Spokes | `service-policy DMVPN_CHILD_QOS` | Queuing happens inside the shaped rate |

| 27 | Apply parent policy outbound on physical WAN interface | Hub, Spokes | `interface <wan-underlay-interface>`<br>`service-policy output DMVPN_PARENT_SHAPE` | QoS shaping and queuing are active where congestion is likely |

| 28 | Alternative: apply policy outbound on tunnel if the lab/platform expects tunnel policy | Hub, Spokes | `interface Tunnel<id>`<br>`service-policy output DMVPN_CHILD_QOS` | Tunnel policy is attached, if supported and intentionally used |

| 29 | If classifying inbound LAN traffic, create marking policy | LAN Edge Routers | `policy-map DMVPN_MARKING` | Marking policy is created before traffic enters the DMVPN edge |

| 30 | Mark voice traffic if not already marked | LAN Edge Routers | `class DMVPN_VOICE`<br>`set dscp ef` | Voice receives DSCP EF marking |

| 31 | Mark critical traffic if not already marked | LAN Edge Routers | `class DMVPN_CRITICAL_DATA`<br>`set dscp af31` | Critical traffic receives DSCP marking |

| 32 | Apply marking policy inbound on LAN-facing interface if required | LAN Edge Routers | `interface <lan-interface>`<br>`service-policy input DMVPN_MARKING` | Traffic is marked before entering DMVPN tunnel path |

| 33 | If using DMVPN per-tunnel QoS, define NHRP group on spoke | Spokes | `interface Tunnel<id>`<br>`ip nhrp group <group-name>` | Spoke advertises its NHRP QoS group to the hub |

| 34 | If using per-tunnel QoS, map NHRP group to service-policy on hub | Hub | `interface Tunnel<id>`<br>`ip nhrp map group <group-name> service-policy output <policy-name>` | Hub applies the selected outbound QoS policy to spokes in that NHRP group |

| 35 | Verify service policy is attached to WAN interface | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Output policy is visible with class counters |

| 36 | Verify tunnel policy if used | Hub, Spokes | `show policy-map interface Tunnel<id>` | Tunnel policy is visible with class counters if supported |

| 37 | Verify class-map and policy-map syntax | Hub, Spokes | `show class-map`<br>`show policy-map` | Expected classes and actions exist |

| 38 | Generate traffic for each class | Test Hosts/Routers | Voice/video/data test traffic or marked pings | Matching traffic crosses DMVPN |

| 39 | Confirm class counters increment | Hub, Spokes | `show policy-map interface <interface>` | Correct class packet/byte counters increase |

| 40 | Confirm DSCP marking is preserved or rewritten as intended | Hub, Spokes | `show policy-map interface <interface>` or packet capture | Marking behavior matches design |

| 41 | Confirm shaping activates during congestion | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Shaper counters and queue statistics increase under load |

| 42 | Confirm priority class is not overused | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Priority queue is active but not starving other classes |

| 43 | Confirm routing and DMVPN remain stable under load | Hub, Spokes | `show dmvpn`<br>`show ip nhrp`<br>`show ip route` | QoS policy does not destabilize the overlay |

| 44 | Test MTU after QoS and encryption are active | Hub, Spokes | `ping <remote-lan-ip> size <size> df-bit` | No unexpected fragmentation/black-hole behavior |

| 45 | Save working QoS-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_QoS_Overlay_Treatment_Skeleton

! Prerequisite:

! DMVPN, NHRP, routing, and optional IPsec must already work.

! Verify first:

! show dmvpn

! show ip nhrp

! show ip route

! ping <remote-lan-ip>

  

! Tunnel overhead and classification support

conf t

interface Tunnel<id>

 bandwidth <kbps>

 ip mtu 1400

 ip tcp adjust-mss 1360

 qos pre-classify

end

  

! Class maps

conf t

class-map match-any DMVPN_VOICE

 match dscp ef

!

class-map match-any DMVPN_VIDEO

 match dscp af41 af42 af43

!

class-map match-any DMVPN_CONTROL

 match dscp cs6

!

class-map match-any DMVPN_CRITICAL_DATA

 match dscp af31 af32 af33

!

class-map match-any DMVPN_SCAVENGER

 match dscp cs1

end

  

! Child queuing policy

conf t

policy-map DMVPN_CHILD_QOS

 class DMVPN_VOICE

  priority percent <voice-percent>

 class DMVPN_VIDEO

  bandwidth percent <video-percent>

 class DMVPN_CONTROL

  bandwidth percent <control-percent>

 class DMVPN_CRITICAL_DATA

  bandwidth percent <critical-percent>

 class DMVPN_SCAVENGER

  bandwidth percent <scavenger-percent>

 class class-default

  fair-queue

end

  

! Parent shaper

conf t

policy-map DMVPN_PARENT_SHAPE

 class class-default

  shape average <wan-egress-bps>

  service-policy DMVPN_CHILD_QOS

end

  

! Apply to physical WAN/transport interface, most common egress bottleneck

conf t

interface <wan-underlay-interface>

 service-policy output DMVPN_PARENT_SHAPE

end

  

! Optional marking policy on LAN ingress

conf t

policy-map DMVPN_MARKING

 class DMVPN_VOICE

  set dscp ef

 class DMVPN_CRITICAL_DATA

  set dscp af31

 class DMVPN_SCAVENGER

  set dscp cs1

end

  

conf t

interface <lan-interface>

 service-policy input DMVPN_MARKING

end

  

! Optional tunnel-applied policy if lab/platform requires it

conf t

interface Tunnel<id>

 service-policy output DMVPN_CHILD_QOS

end

  

! Optional DMVPN per-tunnel QoS model

! Spoke

conf t

interface Tunnel<id>

 ip nhrp group <group-name>

end

  

! Hub

conf t

interface Tunnel<id>

 ip nhrp map group <group-name> service-policy output <policy-name>

end

# DMVPN_QoS_Overlay_Treatment_Verification_Commands

| Check | Device | Command | Good Output |

|---|---|---|---|

| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show `UP` |

| NHRP state | Hub, Spokes | `show ip nhrp` | Expected tunnel-to-NBMA mappings exist |

| Overlay routing | Hub, Spokes | `show ip route` | Remote prefixes are reachable |

| Tunnel MTU/MSS | Hub, Spokes | `show running-config interface Tunnel<id> | include mtu|mss` | Tunnel has expected MTU/MSS values |

| QoS pre-classify | Hub, Spokes | `show running-config interface Tunnel<id> | include qos pre-classify` | `qos pre-classify` is present if classification across GRE/IPsec is needed |

| Class maps | Hub, Spokes | `show class-map` | Expected traffic classes exist |

| Policy maps | Hub, Spokes | `show policy-map` | Expected parent, child, marking, and class actions exist |

| WAN service policy | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Parent shaper and child classes appear |

| Tunnel service policy if used | Hub, Spokes | `show policy-map interface Tunnel<id>` | Tunnel policy appears if intentionally applied |

| Interface attachment | Hub, Spokes | `show running-config interface <interface> | include service-policy` | Correct service policy is applied in correct direction |

| Marking counters | LAN Edge Routers | `show policy-map interface <lan-interface>` | Marking class counters increase |

| Queue counters | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Class packet/byte counters increase under traffic |

| Shaping counters | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Shaper activates under congestion |

| Priority queue behavior | Hub, Spokes | `show policy-map interface <wan-underlay-interface>` | Priority class matches expected traffic and does not dominate the link |

| DSCP preservation | Hub, Spokes | Packet capture or `show policy-map interface` | DSCP values are preserved or rewritten as intended |

| Per-tunnel QoS group | Hub, Spokes | `show running-config interface Tunnel<id> | include nhrp group|map group` | Spoke group and hub group-to-policy mapping exist if used |

| MTU validation | Hub, Spokes | `ping <remote-lan-ip> size <size> df-bit` | Confirms traffic is not black-holed due to overhead |

| Stability under load | Hub, Spokes | `show dmvpn`<br>`show ip nhrp`<br>`show ip route` | DMVPN remains stable while QoS policy is active |

# DMVPN_QoS_Overlay_Treatment_Rollback

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Remove WAN egress service policy | Hub, Spokes | `interface <wan-underlay-interface>`<br>`no service-policy output DMVPN_PARENT_SHAPE` | Parent shaper is removed from WAN egress |

| 2 | Remove tunnel output policy if used | Hub, Spokes | `interface Tunnel<id>`<br>`no service-policy output DMVPN_CHILD_QOS` | Tunnel service policy is removed |

| 3 | Remove LAN ingress marking policy if used | LAN Edge Routers | `interface <lan-interface>`<br>`no service-policy input DMVPN_MARKING` | Marking policy is removed from LAN ingress |

| 4 | Remove per-tunnel QoS group from spoke if used | Spokes | `interface Tunnel<id>`<br>`no ip nhrp group <group-name>` | Spoke no longer advertises NHRP QoS group |

| 5 | Remove NHRP group-to-policy mapping from hub if used | Hub | `interface Tunnel<id>`<br>`no ip nhrp map group <group-name> service-policy output <policy-name>` | Hub no longer maps that group to an output policy |

| 6 | Remove QoS pre-classify if no longer required | Hub, Spokes | `interface Tunnel<id>`<br>`no qos pre-classify` | Pre-classification is removed |

| 7 | Remove parent shaping policy | Hub, Spokes | `no policy-map DMVPN_PARENT_SHAPE` | Parent policy-map is deleted |

| 8 | Remove child queuing policy | Hub, Spokes | `no policy-map DMVPN_CHILD_QOS` | Child policy-map is deleted |

| 9 | Remove marking policy | Hub, Spokes | `no policy-map DMVPN_MARKING` | Marking policy-map is deleted |

| 10 | Remove class maps if no longer used | Hub, Spokes | `no class-map DMVPN_VOICE`<br>`no class-map DMVPN_VIDEO`<br>`no class-map DMVPN_CONTROL`<br>`no class-map DMVPN_CRITICAL_DATA`<br>`no class-map DMVPN_SCAVENGER` | QoS class maps are deleted |

| 11 | Restore tunnel bandwidth if it was only changed for QoS testing | Hub, Spokes | `interface Tunnel<id>`<br>`bandwidth <original-kbps>` | Tunnel bandwidth returns to baseline |

| 12 | Keep MTU/MSS unless you are fully resetting the tunnel | Hub, Spokes | Design check | Do not remove MTU/MSS if GRE/IPsec overhead still exists |

| 13 | Verify policies are gone | Hub, Spokes | `show policy-map interface`<br>`show running-config | include service-policy` | Removed policies no longer appear on interfaces |

| 14 | Verify DMVPN remains stable after QoS rollback | Hub, Spokes | `show dmvpn`<br>`show ip nhrp`<br>`ping <remote-lan-ip>` | DMVPN and routing still work |

# DMVPN_QoS_Overlay_Treatment_Failure_Checks

| Symptom | Likely Cause | Check | Fix |

|---|---|---|---|

| Policy counters stay at zero | Policy applied to wrong interface or wrong direction | `show policy-map interface`<br>`show run interface <interface>` | Apply policy to actual egress bottleneck in the correct direction |

| Traffic all lands in class-default | Class-map does not match current markings or headers are hidden by GRE/IPsec | `show policy-map interface`<br>`show run interface Tunnel<id>` | Fix match criteria or add `qos pre-classify` |

| Voice quality still poor | No real congestion control, wrong shaper rate, or priority queue too small | `show policy-map interface <wan-interface>` | Shape below provider rate and tune priority percent carefully |

| Priority queue starves other traffic | LLQ percentage is too high or voice/video class is overmatching | `show policy-map interface` | Lower priority percent and tighten class-map matching |

| Shaping never activates | Shape rate is above real bottleneck or policy is on wrong interface | `show policy-map interface <wan-interface>` | Apply shaper to correct egress and set rate below actual WAN speed |

| Drops occur in wrong class | Bad DSCP marking or wrong class-map order | `show policy-map interface`<br>`show class-map` | Fix marking policy and class-map matches |

| QoS works before IPsec but fails after IPsec | Inner headers are hidden after encryption | `show run interface Tunnel<id> | include qos pre-classify` | Add `qos pre-classify` on the tunnel |

| Routing protocol packets are delayed | Control traffic is not classified or bandwidth is starved | `show policy-map interface`<br>`show ip protocols` | Add a control class or reserve enough bandwidth |

| DMVPN drops under load | Shaper rate, queue limits, CPU, or MTU are wrong | `show policy-map interface`<br>`show process cpu`<br>`show interface Tunnel<id>` | Tune shaper, queues, MTU/MSS, or reduce load |

| Large packets fail | GRE/IPsec overhead and MTU/MSS not handled | `ping <remote-lan-ip> size <size> df-bit` | Configure `ip mtu 1400` and `ip tcp adjust-mss 1360` |

| DSCP markings disappear | Marking is overwritten by policy or not trusted at ingress | `show policy-map interface` | Fix ingress marking/trust boundary and outbound policy |

| Per-tunnel QoS does not apply | NHRP group missing on spoke or group-to-policy missing on hub | `show run interface Tunnel<id> | include nhrp group|map group` | Configure spoke `ip nhrp group` and hub `ip nhrp map group` |

| One spoke gets wrong treatment | Spoke is in wrong NHRP group or using wrong tunnel | `show run interface Tunnel<id>` | Correct NHRP group assignment |

| QoS config is accepted but no hardware counters move | Platform does not support that action/interface combination | `show policy-map interface` | Use platform-supported MQC action or apply policy to supported interface |

| Engineer blames QoS before DMVPN | Overlay is unstable before treatment is applied | `show dmvpn`<br>`show ip nhrp`<br>`show ip route` | Fix DMVPN and routing first, then QoS |

Index

Source_Basis  
DMVPN_QoS_Overlay_Treatment_Mental_Model  
DMVPN_QoS_Overlay_Treatment_Configuration_Checklist  
DMVPN_QoS_Overlay_Treatment_Skeleton  
DMVPN_QoS_Overlay_Treatment_Verification_Commands  
DMVPN_QoS_Overlay_Treatment_Rollback  
DMVPN_QoS_Overlay_Treatment_Failure_Checks