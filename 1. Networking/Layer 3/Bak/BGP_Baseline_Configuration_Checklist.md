

|      |                                                            |                    |                                                                                                          |                                                                            |
| ---- | ---------------------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Step | Task                                                       | Device             | Command                                                                                                  | Expected Result                                                            |
| 1    | Verify interface state before configuring BGP              | All BGP routers    | show ip interface brief                                                                                  | Peer-facing interfaces are up/up                                           |
| 2    | Verify IP reachability to the neighbor                     | Local router       | ping <neighbor-ip>                                                                                       | Ping succeeds before BGP is configured                                     |
| 3    | Verify the local routing table                             | Local router       | show ip route                                                                                            | Connected/static routes needed for BGP peering are present                 |
| 4    | Enter BGP configuration mode                               | Local router       | conf t then router bgp <local-as>                                                                        | Router enters BGP process configuration                                    |
| 5    | Set a stable BGP router ID                                 | Local router       | bgp router-id <router-id>                                                                                | Router ID is manually defined and predictable                              |
| 6    | Define the BGP neighbor and remote AS                      | Local router       | neighbor <neighbor-ip> remote-as <remote-as>                                                             | Neighbor is created under the BGP process                                  |
| 7    | Add a neighbor description                                 | Local router       | neighbor <neighbor-ip> description <description>                                                         | Neighbor purpose is documented in running config                           |
| 8    | Configure iBGP loopback peering, if used                   | iBGP routers       | neighbor <neighbor-loopback-ip> update-source Loopback0                                                  | BGP sources the session from the loopback                                  |
| 9    | Configure eBGP loopback peering, if used                   | eBGP routers       | neighbor <neighbor-loopback-ip> ebgp-multihop <hop-count>                                                | eBGP can form beyond directly connected interfaces                         |
| 10   | Enter IPv4 address-family mode                             | Local router       | address-family ipv4 unicast                                                                              | Router enters IPv4 unicast BGP address-family                              |
| 11   | Activate the IPv4 neighbor                                 | Local router       | neighbor <neighbor-ip> activate                                                                          | Neighbor is enabled for IPv4 unicast exchange                              |
| 12   | Advertise a specific local network                         | Local router       | network <prefix> mask <subnet-mask>                                                                      | Prefix is eligible for BGP advertisement if it exists in the routing table |
| 13   | Verify advertised network exists in RIB                    | Local router       | show ip route <prefix>                                                                                   | Exact prefix/mask exists in the local routing table                        |
| 14   | Create a Null0 route for an aggregate, if needed           | Local router       | ip route <aggregate-prefix> <mask> Null0                                                                 | Aggregate route exists so BGP can advertise it                             |
| 15   | Advertise the aggregate prefix                             | Local router       | router bgp <local-as> then address-family ipv4 unicast then network <aggregate-prefix> mask <mask>       | Aggregate prefix appears in local BGP table                                |
| 16   | Build an outbound prefix-list, if filtering advertisements | Local router       | ip prefix-list <name> permit <prefix>/<length>                                                           | Only intended prefixes are matched                                         |
| 17   | Build a route-map for outbound policy                      | Local router       | route-map <name> permit 10 then match ip address prefix-list <name>                                      | Route-map matches the permitted prefixes                                   |
| 18   | Apply outbound BGP policy                                  | Local router       | router bgp <local-as> then address-family ipv4 unicast then neighbor <neighbor-ip> route-map <name> out  | Only approved prefixes are advertised                                      |
| 19   | Build an inbound prefix-list, if filtering learned routes  | Local router       | ip prefix-list <name> permit <prefix>/<length>                                                           | Only intended received prefixes are matched                                |
| 20   | Apply inbound BGP policy                                   | Local router       | router bgp <local-as> then address-family ipv4 unicast then neighbor <neighbor-ip> prefix-list <name> in | Only approved routes are accepted                                          |
| 21   | Exit address-family mode                                   | Local router       | exit-address-family                                                                                      | Returns to BGP global configuration mode                                   |
| 22   | Save the configuration                                     | Local router       | end then write memory                                                                                    | BGP configuration is saved                                                 |
| 23   | Verify BGP neighbor state                                  | Local router       | show ip bgp summary                                                                                      | Neighbor state shows a prefix count, not Idle or Active                    |
| 24   | Verify received BGP routes                                 | Local router       | show ip bgp                                                                                              | Learned and advertised prefixes appear in the BGP table                    |
| 25   | Verify BGP routes installed in routing table               | Local router       | show ip route bgp                                                                                        | Best BGP routes are installed in the IP routing table                      |
| 26   | Verify neighbor details                                    | Local router       | show ip bgp neighbors <neighbor-ip>                                                                      | Session state, timers, capabilities, and policy are visible                |
| 27   | Verify advertised routes                                   | Local router       | show ip bgp neighbors <neighbor-ip> advertised-routes                                                    | Intended prefixes are being sent to the neighbor                           |
| 28   | Verify received routes                                     | Local router       | show ip bgp neighbors <neighbor-ip> routes                                                               | Intended prefixes are being received from the neighbor                     |
| 29   | Soft refresh BGP after policy changes                      | Local router       | clear ip bgp <neighbor-ip> soft in or clear ip bgp <neighbor-ip> soft out                                | Policy updates apply without tearing down the session                      |
| 30   | Test end-to-end forwarding                                 | Source host/router | ping <remote-prefix-ip> then traceroute <remote-prefix-ip>                                               | Traffic reaches the remote network through the expected path               |

Basic eBGP skeleton:
`
```
conf t

  

router bgp <local-as>

  

 bgp router-id <router-id>

  

 neighbor <neighbor-ip> remote-as <remote-as>

  

 neighbor <neighbor-ip> description <description>

  

 address-family ipv4 unicast

  

  neighbor <neighbor-ip> activate

  

  network <local-prefix> mask <subnet-mask>

  

 exit-address-family

  

end

  

write memory

```

Basic iBGP loopback skeleton:

```
conf t

interface Loopback0

 ip address <loopback-ip> 255.255.255.255

exit

router bgp <local-as>

 bgp router-id <router-id>

 neighbor <peer-loopback-ip> remote-as <same-as>

 neighbor <peer-loopback-ip> update-source Loopback0

 neighbor <peer-loopback-ip> description <description>

 address-family ipv4 unicast

  neighbor <peer-loopback-ip> activate

  network <local-prefix> mask <subnet-mask>

 exit-address-family

end

write memory

```
