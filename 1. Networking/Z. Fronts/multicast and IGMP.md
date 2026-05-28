Source basis checked first:

|   |   |
|---|---|
|Source|What it supports|
|_KB_INDEX.md|Points multicast primarily to All_combined_part3.md|
|All_combined_part3.md, CCIE R&S Foundations Ch14|IGMP, PIM dense, PIM sparse, static RP, Auto-RP, BSR, RPF behavior|
|All_combined_part3.md, Advanced BGP Ch11|MSDP, Anycast RP, SA messages, peer-RPF|
|ENCOR / CiscoPress supplemental material|PIM-SSM, PIM-BIDIR, multicast design notes|
|Wireless / campus multicast supplemental material|IGMP snooping as Layer 2 multicast containment|

Do not make these one note per lab. That would be messy. Make mechanism notes, then attach the labs.

Recommended multicast note order

|   |   |   |   |
|---|---|---|---|
|Order|Mechanism note|Mental model to address|Related labs|
|01|Multicast_RPF_And_Mroute_Baseline.md|Multicast is not forwarded like unicast. A router accepts multicast only if the packet arrives on the interface it would use to reach the source. show ip mroute is the truth table: (*,G), (S,G), incoming interface, RPF neighbor, outgoing interface list.|multicast-rpf-final|
|02|Multicast_RPF_Tunnel_And_Overlay_Failure.md|Tunnels break multicast when the unicast RPF path points somewhere different from where multicast actually arrives. The fix is usually routing/RPF correction, not random PIM tuning.|multcast-rpf-tunnel-failure-final|
|03|IGMP_Receiver_Membership_And_Version_Behavior.md|IGMP is host-to-router membership signaling. It does not route multicast. IGMP tells the first-hop router which groups have interested receivers on a segment. IGMPv1 lacks leave behavior and relies more heavily on timers. IGMPv2 adds explicit leave and querier election.|multicast-igmp-v1-final, igmp-v2-final, igmp-version-2-final|
|04|IGMP_Filtering_Receiver_Admission_Control.md|IGMP filtering controls who is allowed to join which multicast groups. This is receiver admission control, not multicast routing policy.|igmp-filter-final|
|05|IGMP_Proxy_Receiver_Delegation.md|IGMP proxy lets one router act upstream as a multicast receiver on behalf of downstream hosts. Mentally treat it as receiver-state delegation, not full PIM routing.|multicast-igmp-proxy-test2-final|
|06|IGMP_Snooping_Layer2_Multicast_Containment.md|IGMP snooping is switch behavior. The switch listens to IGMP reports and forwards multicast only toward interested ports instead of flooding the VLAN. It does not replace PIM or IGMP querier behavior.|multicast-igmp-snooping-final|
|07|PIM_Dense_Mode_Flood_And_Prune.md|PIM dense mode assumes receivers are everywhere, floods first, then prunes unwanted branches. Simple, noisy, and mostly useful for understanding multicast mechanics or small controlled labs.|multicast-pim-dense-mode-final|
|08|PIM_Sparse_Mode_RP_And_Shared_Tree.md|PIM sparse mode assumes receivers are not everywhere. Receivers join toward an RP first using a shared tree. Source traffic registers to the RP, then routers may move toward shortest-path tree behavior.|multicast-pim-sparse-mode-final|
|09|PIM_DR_Receiver_Segment_And_Last_Hop_Router.md|On a multiaccess LAN, the PIM designated router is responsible for representing local receivers upstream. Do not confuse IGMP querier with PIM DR. They solve different problems and can be different routers.|multicast-pim-designated-router-final, multicast-pim-sparse-receiver-behind-router-final|
|10|PIM_Prune_Override_And_Multiaccess_Receiver_Protection.md|On shared segments, one router pruning does not always mean all routers are done with the flow. Prune override prevents one receiver path from killing traffic needed by another router on the same segment.|multicast-pim-prune-override-final|
|11|PIM_Register_And_Accept_Register_Policy.md|In sparse mode, the first-hop router encapsulates source traffic in PIM register messages to the RP. accept-register controls which sources/groups the RP accepts. This is source admission control at the RP.|multicast-pim-accept-register-final|
|12|PIM_Sparse_Dense_Mode_And_Dense_Mode_Fallback.md|Sparse-dense mode uses sparse mode when an RP is known and dense mode when an RP is unknown. This exists largely because Auto-RP discovery groups need dense-mode transport unless autorp listener is used. Dangerous if you do not understand fallback.|multicast-pim-sparse-dense-mode-final|
|13|PIM_Auto_RP_Candidate_RP_And_Mapping_Agent.md|Auto-RP is dynamic RP discovery. Candidate RPs announce themselves, mapping agents select and advertise RP mappings. The key mental model is that routers must learn group-to-RP mapping before sparse-mode joins can work.|multicast-pim-auto-rp-final, multicast-autorp-spoke-mapping-agent-final|
|14|Multicast_Admin_Boundary_And_Scope_Control.md|ip multicast boundary limits multicast scope. It is a containment tool for administratively scoped groups and Auto-RP leakage control. Think “where multicast is allowed to spread.”|multicast-ip-boundary-final|
|15|PIM_BSR_Candidate_RP_And_BSR_Election.md|BSR is another dynamic RP discovery mechanism. Candidate BSRs distribute candidate RP information. Unlike Auto-RP, it is native PIMv2 and does not require the Auto-RP dense-mode discovery workaround.|multicast-pim-bsr-final|
|16|PIM_BIDIR_Shared_Tree_And_Designated_Forwarder.md|BIDIR PIM uses shared trees in both directions and avoids source-specific state. It does not use the normal PIM register process. The important role is the designated forwarder toward the RP.|multicast-pim-bidir-final|
|17|PIM_SSM_Source_Specific_Multicast.md|SSM removes the RP from the model. Receivers join (S,G) directly for a known source. IGMPv3 matters here because the receiver must specify the source.|multicast-ssm-final|
|18|MSDP_Source_Active_Exchange_And_Anycast_RP.md|MSDP lets RPs share source-active information. Anycast RP uses multiple RPs with the same RP address, while MSDP keeps source awareness synchronized between them.|multicast-anycast-msdp-final|
|19|MSDP_SA_Filtering_And_Peer_RPF.md|MSDP SA filtering controls which source-active messages are accepted or advertised. Peer-RPF prevents SA loops and depends heavily on the MSDP/BGP topology lining up correctly.|multicast-msdp-sa-filter-final|
|20|Multicast_VRF_MRIB_And_RP_Scope.md|VRFs isolate multicast routing state just like unicast routing state. Treat each VRF as having its own multicast control plane, RPF logic, RP scope, and receiver/source reachability.|multicast-vrf-final|
|21|Multicast_Over_DMVPN_Phase3.md|DMVPN multicast is not magic. You still need NHRP, tunnel reachability, RPF correctness, PIM neighbor behavior, and hub/spoke forwarding expectations. Phase 3 shortcuts can make RPF and control-plane expectations easy to misunderstand.|multicast-dmvpn-phase3-final|

Cleaner grouping by study phase

|   |   |
|---|---|
|Phase|Notes|
|1|Multicast_RPF_And_Mroute_Baseline.md, Multicast_RPF_Tunnel_And_Overlay_Failure.md|
|2|IGMP notes: version behavior, filtering, proxy, snooping|
|3|Core PIM notes: dense mode, sparse mode, DR, prune override, register policy|
|4|RP discovery notes: sparse-dense mode, Auto-RP, boundary, BSR|
|5|Advanced multicast notes: BIDIR, SSM, MSDP, Anycast RP, SA filtering|
|6|Segmented/overlay notes: VRF multicast, DMVPN multicast|

Blunt answer: this order is better than alphabetizing the labs or copying Cisco feature order. Multicast becomes understandable only when you keep the mental stack straight:

Receiver membership: IGMP

Layer 2 containment: IGMP snooping

Router tree building: PIM

Source acceptance gate: RPF

Shared-tree rendezvous: RP

Dynamic RP learning: Auto-RP / BSR

No-RP source-specific model: SSM

Multi-RP/source exchange: MSDP / Anycast RP

Segmentation and overlays: VRF / DMVPN