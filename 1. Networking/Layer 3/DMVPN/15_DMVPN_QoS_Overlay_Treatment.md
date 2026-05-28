
DMVPN_QoS_Overlay_Treatment.md

# DMVPN_QoS_Overlay_Treatment_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 12, Quality of Service | Supports MQC, DSCP, CBWFQ, LLQ, policing, shaping, and QoS verification |
| `_KB_INDEX.md` | IOS XE QoS Configuration Guide | Points IOS XE MQC, class-map, policy-map, shaping, queuing, marking, policing, and `show policy-map interface` syntax to `iosxe_combined_pdfs_.md` |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports the requirement that DMVPN, NHRP, tunnel state, and routing are stable before QoS is layered on |
| `All_combined_part3.md` | Book 6, Chapter 12, QoS | Supports QoS classification, marking, queuing, shaping, policing, and verification workflows |
| `iosxe_combined_pdfs_.md` | QoS Configuration Guide | Supports MQC syntax including `class-map`, `policy-map`, `priority`, `bandwidth`, `shape average`, `service-policy`, and `show policy-map interface` |

# DMVPN_QoS_Overlay_Treatment_Mental_Model

| Concept | Operational Meaning |
|---|---|
| QoS over DMVPN | QoS classifies, marks, queues, shapes, or polices traffic that crosses a DMVPN overlay |
| DMVPN first | QoS does not fix broken underlay, NHRP, tunnel state, routing, or IPsec |
| Classification | Identifies traffic classes such as voice, video, routing, critical data, bulk, and default |
| Marking | Sets or preserves DSCP values so downstream devices can treat traffic consistently |
| LLQ | Strict priority queue for real-time traffic, usually voice, but it must be capped |
| CBWFQ | Reserves bandwidth for non-priority traffic classes during congestion |
| Shaping | Slows egress traffic to the real WAN rate so queuing happens on your router instead of inside the provider cloud |
| Policing | Drops or remarks traffic above a rate; useful at boundaries but harsher than shaping |
| Tunnel bandwidth | Logical reference used by routing protocols and QoS math; it does not create physical bandwidth |
| MTU and MSS | GRE and IPsec overhead reduce usable payload size, so MTU/MSS must be handled before blaming QoS |
| QoS pre-classify | Allows classification based on original inner headers before GRE or IPsec encapsulation hides them |
| Physical egress policy | Common place for WAN shaping and queuing because the transport-facing interface is usually the bottleneck |
| Tunnel policy | Can be used in lab or platform-supported designs, but physical egress is usually cleaner for WAN congestion |
| Per-tunnel QoS | DMVPN method that applies different hub output policies per spoke using NHRP groups |
| Hard rule | QoS does not create bandwidth; it decides which traffic is protected when congestion happens |

# DMVPN_QoS_Overlay_Treatment_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before QoS is configured |
| 2 | Confirm tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | DMVPN peers show up |
| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub and spokes have expected mappings |
| 5 | Confirm overlay routing works | Hub, Spokes | `show ip route` | Remote overlay prefixes are reachable |
| 6 | Test application path before QoS | Hub, Spokes | `ping <remote-lan-ip>` | Traffic works before QoS treatment |
| 7 | Confirm whether IPsec is enabled | Hub, Spokes | `show running-config interface Tunnel<id>` | Tunnel protection status is known |
| 8 | Confirm tunnel MTU and MSS | Hub, Spokes | `show running-config interface Tunnel<id>` | MTU and MSS values are known |
| 9 | Configure tunnel MTU and MSS if missing | Hub, Spokes | See `Tunnel_Overhead_Baseline_Config` | Tunnel accounts for GRE/IPsec overhead |
| 10 | Configure tunnel bandwidth reference | Hub, Spokes | See `Tunnel_Overhead_Baseline_Config` | QoS and routing have realistic bandwidth reference |
| 11 | Enable QoS pre-classify when needed | Hub, Spokes | See `Tunnel_QoS_Preclassify_Config` | QoS can classify original inner traffic |
| 12 | Identify real WAN egress rate | Hub, Spokes | Design check | Shaping rate is based on actual provider or lab bottleneck |
| 13 | Define voice class | Hub, Spokes | See `QoS_Class_Map_Config` | Voice traffic can be matched |
| 14 | Define video class | Hub, Spokes | See `QoS_Class_Map_Config` | Video traffic can be matched |
| 15 | Define control traffic class | Hub, Spokes | See `QoS_Class_Map_Config` | Routing or control traffic can be matched |
| 16 | Define critical data class | Hub, Spokes | See `QoS_Class_Map_Config` | Critical application traffic can be matched |
| 17 | Define scavenger class if required | Hub, Spokes | See `QoS_Class_Map_Config` | Low-priority traffic can be matched |
| 18 | Build child queuing policy | Hub, Spokes | See `DMVPN_Child_QoS_Policy` | Classes have priority, bandwidth, or fair queue treatment |
| 19 | Configure LLQ for voice | Hub, Spokes | See `DMVPN_Child_QoS_Policy` | Voice gets capped strict priority treatment |
| 20 | Configure bandwidth for video | Hub, Spokes | See `DMVPN_Child_QoS_Policy` | Video gets reserved bandwidth |
| 21 | Configure bandwidth for control traffic | Hub, Spokes | See `DMVPN_Child_QoS_Policy` | Control traffic gets protected bandwidth |
| 22 | Configure bandwidth for critical data | Hub, Spokes | See `DMVPN_Child_QoS_Policy` | Critical data gets reserved bandwidth |
| 23 | Configure class-default behavior | Hub, Spokes | See `DMVPN_Child_QoS_Policy` | Default traffic gets fair or default treatment |
| 24 | Build parent shaping policy | Hub, Spokes | See `DMVPN_Parent_Shaper_Policy` | Traffic is shaped before child queues are used |
| 25 | Apply parent policy to WAN egress | Hub, Spokes | See `Apply_QoS_To_WAN_Egress` | QoS is active where congestion is expected |
| 26 | Apply tunnel policy only if lab or platform requires it | Hub, Spokes | See `Optional_Apply_QoS_To_Tunnel` | Tunnel policy is applied intentionally |
| 27 | Build marking policy if ingress marking is required | LAN Edge Routers | See `DMVPN_Ingress_Marking_Policy` | Traffic is marked before entering the overlay |
| 28 | Apply marking policy to LAN ingress | LAN Edge Routers | See `Apply_Marking_To_LAN_Ingress` | Traffic is marked at the trust boundary |
| 29 | Configure NHRP group on spoke if using per-tunnel QoS | Spokes | See `DMVPN_Per_Tunnel_QoS_Spoke_Group` | Spoke advertises QoS group to hub |
| 30 | Map NHRP group to output policy on hub | Hub | See `DMVPN_Per_Tunnel_QoS_Hub_Map` | Hub applies selected policy to that spoke group |
| 31 | Verify policy attachment | Hub, Spokes | `show policy-map interface <interface>` | Policy appears on the intended interface |
| 32 | Verify class-map and policy-map definitions | Hub, Spokes | `show class-map` and `show policy-map` | Expected classes and actions exist |
| 33 | Generate traffic for each class | Test Hosts/Routers | Test traffic | Traffic crosses DMVPN in each class |
| 34 | Verify class counters increment | Hub, Spokes | `show policy-map interface <interface>` | Correct class packet counters increase |
| 35 | Verify DSCP marking behavior | Hub, Spokes | `show policy-map interface <interface>` | Marking is preserved or rewritten as intended |
| 36 | Verify shaping behavior under load | Hub, Spokes | `show policy-map interface <wan-interface>` | Shaper activates during congestion |
| 37 | Verify priority queue is not overused | Hub, Spokes | `show policy-map interface <wan-interface>` | Priority traffic is protected but not starving other classes |
| 38 | Verify DMVPN remains stable under load | Hub, Spokes | `show dmvpn` and `show ip nhrp` | QoS does not destabilize overlay |
| 39 | Test MTU after QoS and IPsec are active | Hub, Spokes | `ping <remote-lan-ip> size <size> df-bit` | No unexpected black-hole behavior |
| 40 | Save working QoS-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_QoS_Overlay_Treatment_Skeleton

```text
# Tunnel_Overhead_Baseline_Config
conf t
interface Tunnel<id>
 bandwidth <kbps>
 ip mtu 1400
 ip tcp adjust-mss 1360
end
```

```text
# Tunnel_QoS_Preclassify_Config
conf t
interface Tunnel<id>
 qos pre-classify
end
```

```text
# QoS_Class_Map_Config
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
```

```text
# DMVPN_Child_QoS_Policy
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
```

```text
# DMVPN_Parent_Shaper_Policy
conf t
policy-map DMVPN_PARENT_SHAPE
 class class-default
  shape average <wan-egress-bps>
  service-policy DMVPN_CHILD_QOS
end
```

```text
# Apply_QoS_To_WAN_Egress
conf t
interface <wan-underlay-interface>
 service-policy output DMVPN_PARENT_SHAPE
end
```

```text
# Optional_Apply_QoS_To_Tunnel
! Use only when the lab or platform supports and expects tunnel-applied QoS.

conf t
interface Tunnel<id>
 service-policy output DMVPN_CHILD_QOS
end
```

```text
# DMVPN_Ingress_Marking_Policy
conf t
policy-map DMVPN_MARKING
 class DMVPN_VOICE
  set dscp ef
 class DMVPN_VIDEO
  set dscp af41
 class DMVPN_CONTROL
  set dscp cs6
 class DMVPN_CRITICAL_DATA
  set dscp af31
 class DMVPN_SCAVENGER
  set dscp cs1
end
```

```text
# Apply_Marking_To_LAN_Ingress
conf t
interface <lan-interface>
 service-policy input DMVPN_MARKING
end
```

```text
# DMVPN_Per_Tunnel_QoS_Spoke_Group
conf t
interface Tunnel<id>
 ip nhrp group <group-name>
end
```

```text
# DMVPN_Per_Tunnel_QoS_Hub_Map
conf t
interface Tunnel<id>
 ip nhrp map group <group-name> service-policy output <policy-name>
end
```

```text
# Optional_Policing_Template
! Use policing at boundaries when excess traffic should be dropped or remarked.
! Shaping is usually better than policing for WAN egress.

conf t
policy-map DMVPN_POLICING_EXAMPLE
 class DMVPN_SCAVENGER
  police <bps>
 class class-default
  fair-queue
end
```

```text
# QoS_Order_Reminder
! Correct order:
! 1. Prove underlay NBMA reachability.
! 2. Prove DMVPN and NHRP.
! 3. Prove overlay routing.
! 4. Prove IPsec if used.
! 5. Fix MTU and MSS.
! 6. Add marking, shaping, queuing, or per-tunnel QoS.
! 7. Verify counters under real or simulated congestion.
```

```text
# QoS_Placement_Reminder
! Most common WAN model:
! - Mark traffic at LAN ingress or trust existing DSCP.
! - Use qos pre-classify on tunnel if GRE/IPsec hides inner headers.
! - Shape and queue on WAN egress.
! - Use per-tunnel QoS on hub only when spoke-specific treatment is required.
```

# DMVPN_QoS_Overlay_Treatment_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show up |
| NHRP state | Hub, Spokes | `show ip nhrp` | Expected mappings exist |
| Overlay routing | Hub, Spokes | `show ip route` | Remote prefixes are reachable |
| Tunnel MTU and MSS | Hub, Spokes | `show running-config interface Tunnel<id>` | MTU and MSS are configured as intended |
| QoS pre-classify | Hub, Spokes | `show running-config interface Tunnel<id>` | `qos pre-classify` is present if needed |
| Class maps | Hub, Spokes | `show class-map` | Expected class maps exist |
| Policy maps | Hub, Spokes | `show policy-map` | Parent, child, and marking policies exist |
| WAN policy attachment | Hub, Spokes | `show running-config interface <wan-interface>` | Output service-policy is applied |
| Tunnel policy attachment | Hub, Spokes | `show running-config interface Tunnel<id>` | Tunnel service-policy exists only if intended |
| WAN policy counters | Hub, Spokes | `show policy-map interface <wan-interface>` | Parent and child class counters appear |
| Tunnel policy counters | Hub, Spokes | `show policy-map interface Tunnel<id>` | Tunnel policy counters appear if policy is applied |
| LAN marking counters | LAN Edge Routers | `show policy-map interface <lan-interface>` | Marking counters increase |
| Class match behavior | Hub, Spokes | `show policy-map interface <interface>` | Traffic lands in the expected class |
| Shaper behavior | Hub, Spokes | `show policy-map interface <wan-interface>` | Shaper counters activate under congestion |
| Priority queue behavior | Hub, Spokes | `show policy-map interface <wan-interface>` | Voice class matches expected traffic and stays capped |
| DSCP behavior | Hub, Spokes | `show policy-map interface <interface>` | DSCP is preserved or rewritten as intended |
| Per-tunnel QoS group | Spokes | `show running-config interface Tunnel<id>` | `ip nhrp group` is present if used |
| Per-tunnel hub mapping | Hub | `show running-config interface Tunnel<id>` | `ip nhrp map group` is present if used |
| MTU validation | Hub, Spokes | `ping <remote-lan-ip> size <size> df-bit` | Large traffic does not black-hole unexpectedly |
| Stability under load | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN remains stable during traffic load |

# DMVPN_QoS_Overlay_Treatment_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove WAN egress service policy | Hub, Spokes | See `Rollback_WAN_Egress_Service_Policy` | Parent shaper is removed from WAN interface |
| 2 | Remove tunnel output policy if used | Hub, Spokes | See `Rollback_Tunnel_Service_Policy` | Tunnel output policy is removed |
| 3 | Remove LAN ingress marking policy if used | LAN Edge Routers | See `Rollback_LAN_Ingress_Marking` | LAN marking policy is removed |
| 4 | Remove spoke NHRP QoS group if used | Spokes | See `Rollback_Spoke_NHRP_QoS_Group` | Spoke no longer advertises QoS group |
| 5 | Remove hub NHRP group-to-policy mapping if used | Hub | See `Rollback_Hub_NHRP_QoS_Map` | Hub no longer maps group to policy |
| 6 | Remove QoS pre-classify only if no longer required | Hub, Spokes | See `Rollback_QoS_Preclassify` | Pre-classification is removed |
| 7 | Remove parent shaping policy | Hub, Spokes | See `Rollback_Parent_Shaper` | Parent policy-map is deleted |
| 8 | Remove child QoS policy | Hub, Spokes | See `Rollback_Child_QoS_Policy` | Child policy-map is deleted |
| 9 | Remove marking policy | Hub, Spokes | See `Rollback_Marking_Policy` | Marking policy-map is deleted |
| 10 | Remove class maps if no longer used | Hub, Spokes | See `Rollback_QoS_Class_Maps` | Class maps are deleted |
| 11 | Restore tunnel bandwidth if changed only for lab | Hub, Spokes | See `Rollback_Tunnel_Bandwidth` | Tunnel bandwidth returns to baseline |
| 12 | Keep MTU and MSS unless fully resetting tunnel | Hub, Spokes | Design check | GRE/IPsec overhead handling is not accidentally removed |
| 13 | Verify policies are gone | Hub, Spokes | `show policy-map interface` | Removed policies no longer appear |
| 14 | Verify DMVPN still works | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN remains stable |

```text
# Rollback_WAN_Egress_Service_Policy
conf t
interface <wan-underlay-interface>
 no service-policy output DMVPN_PARENT_SHAPE
end
```

```text
# Rollback_Tunnel_Service_Policy
conf t
interface Tunnel<id>
 no service-policy output DMVPN_CHILD_QOS
end
```

```text
# Rollback_LAN_Ingress_Marking
conf t
interface <lan-interface>
 no service-policy input DMVPN_MARKING
end
```

```text
# Rollback_Spoke_NHRP_QoS_Group
conf t
interface Tunnel<id>
 no ip nhrp group <group-name>
end
```

```text
# Rollback_Hub_NHRP_QoS_Map
conf t
interface Tunnel<id>
 no ip nhrp map group <group-name> service-policy output <policy-name>
end
```

```text
# Rollback_QoS_Preclassify
conf t
interface Tunnel<id>
 no qos pre-classify
end
```

```text
# Rollback_Parent_Shaper
conf t
no policy-map DMVPN_PARENT_SHAPE
end
```

```text
# Rollback_Child_QoS_Policy
conf t
no policy-map DMVPN_CHILD_QOS
end
```

```text
# Rollback_Marking_Policy
conf t
no policy-map DMVPN_MARKING
end
```

```text
# Rollback_QoS_Class_Maps
conf t
no class-map DMVPN_VOICE
no class-map DMVPN_VIDEO
no class-map DMVPN_CONTROL
no class-map DMVPN_CRITICAL_DATA
no class-map DMVPN_SCAVENGER
end
```

```text
# Rollback_Tunnel_Bandwidth
conf t
interface Tunnel<id>
 bandwidth <original-kbps>
end
```

# DMVPN_QoS_Overlay_Treatment_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Policy counters stay at zero | Policy is on wrong interface or wrong direction | `show policy-map interface` | Apply policy to real egress bottleneck |
| All traffic lands in class-default | Class-map does not match traffic or inner headers are hidden | `show policy-map interface <interface>` | Fix class-map or use `qos pre-classify` |
| Voice still sounds bad | No congestion control, wrong shaper rate, or LLQ too small | `show policy-map interface <wan-interface>` | Shape below WAN rate and tune priority percent |
| Voice starves other traffic | LLQ percentage too high or class overmatches traffic | `show policy-map interface <wan-interface>` | Lower priority percent and tighten match rules |
| Shaper never activates | Shape rate is above actual bottleneck or applied to wrong interface | `show policy-map interface <wan-interface>` | Apply to correct egress and lower shape rate |
| Drops occur in wrong class | DSCP marking or class order is wrong | `show policy-map interface <interface>` | Fix marking policy and class-map matches |
| QoS worked before IPsec but not after | GRE/IPsec hides original packet headers | `show running-config interface Tunnel<id>` | Add `qos pre-classify` |
| Routing protocol packets suffer | Control traffic is not classified or bandwidth is too low | `show policy-map interface <wan-interface>` | Add control class or reserve bandwidth |
| DMVPN drops under load | Bad shaper rate, queue limits, CPU, or MTU | `show dmvpn` and `show policy-map interface` | Tune shaper, queues, MTU, or reduce load |
| Large packets fail | GRE/IPsec overhead not handled | `ping <remote-lan-ip> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| DSCP markings disappear | Marking is overwritten or ingress trust boundary is wrong | `show policy-map interface <interface>` | Fix ingress marking or trust boundary |
| Per-tunnel QoS does not apply | NHRP group or hub group map missing | `show running-config interface Tunnel<id>` | Configure spoke group and hub map group |
| One spoke gets wrong policy | Spoke is assigned to wrong NHRP group | `show running-config interface Tunnel<id>` | Correct `ip nhrp group` |
| QoS commands are accepted but no counters move | Platform does not support that action or interface combination | `show policy-map interface` | Use supported action or apply policy to supported interface |
| Engineer blames QoS too early | DMVPN or routing is unstable before QoS | `show dmvpn` and `show ip nhrp` | Fix DMVPN first, then QoS |

##### Source_Basis
# DMVPN_QoS_Overlay_Treatment_Mental_Model
# DMVPN_QoS_Overlay_Treatment_Configuration_Checklist
# DMVPN_QoS_Overlay_Treatment_Skeleton
# DMVPN_QoS_Overlay_Treatment_Verification_Commands
# DMVPN_QoS_Overlay_Treatment_Rollback
# DMVPN_QoS_Overlay_Treatment_Failure_Checks