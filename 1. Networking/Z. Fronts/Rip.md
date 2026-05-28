

01_RIP_RIPv2_Core_And_Network_Activation.md

  

Mental model:

RIP is a distance-vector IGP. RIPv2 carries subnet mask information, but Cisco `network` statements are still classful activation statements. The `network` command enables RIP on matching interfaces and advertises connected networks from those interfaces. `no auto-summary` is mandatory in modern discontiguous or subnetted labs.

  

Related labs:

- rip-v2-two-routers-final

02_RIP_Three_Router_Distance_Vector_Propagation.md

  

Mental model:

RIP does not build adjacencies like OSPF or EIGRP. It learns routes through periodic route advertisements from neighbors. The metric is hop count, 15 is the last usable hop, and 16 means unreachable. This note should make the user see RIP as "neighbor sends a table, receiver increments hop count, best route wins."

  

Related labs:

- rip-three-routers-final

03_RIP_Passive_Interface_Boundary_Control.md

  

Mental model:

Passive-interface in RIP suppresses routing updates out of an interface while still allowing that connected network to be advertised through other RIP-enabled interfaces. This is not an adjacency-control feature like OSPF. It is an update-suppression and boundary-control feature.

  

Related labs:

- rip-passive-interface-final

04_RIP_Default_Route_Origination.md

  

Mental model:

A default route in RIP is just another advertised route, normally `0.0.0.0/0`, used to point downstream routers toward an exit path. The upstream router must have a valid default route or be explicitly configured to originate one, depending on the method used. The key verification is not just seeing gateway of last resort locally, but confirming downstream routers learned the default via RIP.

  

Related labs:

- rip-default-route-final

05_RIP_Maximum_Paths_ECMP.md

  

Mental model:

RIP can install multiple equal-hop routes to the same destination if the metrics are equal and `maximum-paths` allows it. This is not unequal-cost balancing. RIP only sees hop count, so two paths that are very different physically can still look equal to RIP.

  

Related labs:

- rip-maximum-paths-final

06_RIP_Offset_List_Metric_Control.md

  

Mental model:

An offset-list manipulates RIP hop-count metrics by adding cost inbound or outbound. It does not filter the route by itself. It biases path selection by making one learned or advertised route look farther away. This note should focus on direction: inbound affects what this router installs, outbound affects what neighbors hear.

  

Related labs:

- rip-offset-list-final

07_RIP_Summarization_Boundary.md

  

Mental model:

Summarization hides specific routes behind a shorter aggregate. It reduces route table size, but it also changes forwarding behavior. In RIP, summarization must be treated carefully because the protocol has weak loop-prevention compared to link-state protocols. The key idea: summarize only at a real boundary where the summarizing router owns reachability to the more-specific routes.

  

Related labs:

- rip-summarization-final

08_RIP_Summary_Loop_And_Blackhole_Prevention.md

  

Mental model:

A summary route can create a lie if the router advertising the summary does not actually have valid reachability to all more-specific routes inside that summary. The result can be a routing loop or blackhole. This note should address the danger of overbroad summaries, missing discard routes, split horizon expectations, poison behavior, and why verification must include traceroute, route ownership, and failure testing.

  

Related labs:

- rip-summary-loop-final

# Final_Order_Index

01_RIP_RIPv2_Core_And_Network_Activation.md

02_RIP_Three_Router_Distance_Vector_Propagation.md

03_RIP_Passive_Interface_Boundary_Control.md

04_RIP_Default_Route_Origination.md

05_RIP_Maximum_Paths_ECMP.md

06_RIP_Offset_List_Metric_Control.md

07_RIP_Summarization_Boundary.md

08_RIP_Summary_Loop_And_Blackhole_Prevention.md

This is the right order. Do not put rip-default-route-final first just because default routes are simple. You need base RIPv2 operation and multi-router propagation before default origination makes operational sense.