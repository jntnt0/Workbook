Source basis: _KB_INDEX.md points IS-IS to iosxe_combined_pdfs_.md, IOS XE IS-IS Configuration Guide, lines 370-2,797. That is the right primary source for these.

  

Do not order these by lab filename. Order them by dependency.

|   |   |   |   |
|---|---|---|---|
|Order|Mechanism Note|Mental Model That Must Be Addressed|Related Lab Names|
|01|IS-IS_Process_NET_System_ID_And_Interface_Enablement.md|IS-IS does not use OSPF-style network statements. The process is created under router isis, the NET defines area plus system ID, and interfaces participate with ip router isis. No interface enablement, no adjacency, no LSPs, no routes.|is-is-single-area-two-routers-final|
|02|IS-IS_Single_Area_LSP_Flooding_And_SPF.md|In one IS-IS area, routers share the same flooding domain. Adjacencies create LSP exchange, the LSDB becomes consistent, then SPF installs routes. The four-router lab proves propagation beyond one neighbor.|is-is-single-area-four-routers-final|
|03|IS-IS_Point_To_Point_Circuit_Behavior.md|Point-to-point IS-IS links are direct adjacencies. No DIS, no pseudonode, no LAN representation. Use this to simplify Ethernet links that are really only between two routers.|is-is-point-to-point-final|
|04|IS-IS_Broadcast_DIS_And_Pseudonode.md|On broadcast multiaccess links, IS-IS elects a DIS and represents the LAN as a pseudonode in the LSDB. The pseudonode is not a router. It is a database abstraction used to reduce topology complexity.|is-is-pseudonode-final|
|05|IS-IS_Level_1_Level_2_And_Area_Boundaries.md|Level 1 is intra-area. Level 2 is inter-area backbone behavior. L1 routers normally know their area and follow attached L1/L2 routers for outside reachability. This is the first real area-design note.|is-is-two-areas-final|
|06|IS-IS_L2_To_L1_Route_Leaking.md|Route leaking intentionally moves selected Level 2 reachability into Level 1 so L1 routers do not rely only on the default/attached-bit path. This is controlled reachability injection, not normal flooding.|is-is-route-leaking-final|
|07|IS-IS_Summarization_At_Level_Boundaries.md|Summarization reduces prefix detail crossing level boundaries. Specifics stay local, summaries are advertised outward. Bad summaries can blackhole traffic, so verification must check both the summary route and the more-specific local routes.|is-is-summarization-final|
|08|IS-IS_Redistribution_External_Route_Injection.md|Redistribution injects routes from another source into IS-IS. The dangerous parts are metric, level, route-map control, and loop prevention. Tags matter because they let you identify and control redistributed routes later.|is-is-redistribution-final|
|09|IS-IS_Filtering_Route_Map_And_Tag_Control.md|IS-IS is link-state, so do not think of filtering as casually hiding routes inside the same area. Filtering belongs at controlled boundaries: redistribution, route leaking, route install, or route-map/tag policy points.|is-is-filtering-final|
|10|IS-IS_Authentication_HMAC_SHA_And_Scope.md|IS-IS authentication protects adjacency and routing information integrity. It does not encrypt payload traffic. Scope matters: interface, area/process, Level 1, Level 2, or both. Mismatched keys or modes break adjacency or LSP acceptance.|is-is-authentication-final|

The important correction: is-is-authentication-final should not be first just because security is important. You need baseline adjacency, circuit behavior, and level behavior first, otherwise the authentication failure modes will look like generic neighbor problems. Filter and redistribution should also stay late because they are policy controls layered on top of a working IS-IS domain.