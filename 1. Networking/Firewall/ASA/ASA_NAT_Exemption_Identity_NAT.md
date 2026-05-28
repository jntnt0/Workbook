# ASA_NAT_Exemption_Identity_NAT_Mental_Model

| Concept                         | Operational Meaning                                                                                         |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| NAT exemption                   | ASA explicitly matches traffic and leaves the real source and destination unchanged                         |
| Identity NAT                    | Real object maps to itself; translated object is the same as original object                                |
| Manual NAT                      | Required when the NAT rule must match both source and destination                                           |
| Section 1 NAT                   | Best place for VPN NAT exemption because it must beat broad Dynamic PAT rules                               |
| VPN interesting traffic         | Usually local LAN to remote LAN, or inside LAN to remote-access VPN pool                                    |
| Crypto ACL alignment            | VPN selectors should usually match the real, untranslated networks                                          |
| `no-proxy-arp`                  | Prevents ASA from answering ARP for identity NAT translations where it should not                           |
| `route-lookup`                  | Lets ASA use the routing table to determine egress behavior instead of blindly using the NAT interface pair |
| NAT exemption is not permission | ACLs, VPN policy, crypto ACLs, routing, and tunnel state still matter                                       |
# ASA_NAT_Exemption_Identity_NAT_Configuration_Checklist

| Step | Task                                                                         | Device        | Command                                                                                                                                                           | Expected Result                                                                    |
| ---: | ---------------------------------------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
|    1 | Confirm routed ASA baseline is working                                       | ASA           | `show interface ip brief`                                                                                                                                         | Inside and outside interfaces are up/up                                            |
|    2 | Confirm logical interface names                                              | ASA           | `show nameif`                                                                                                                                                     | Interfaces are named correctly, usually `inside` and `outside`                     |
|    3 | Confirm local protected network route                                        | ASA           | `show route <local-network-ip>`                                                                                                                                   | ASA knows the local protected network                                              |
|    4 | Confirm remote protected network route or VPN path expectation               | ASA           | `show route <remote-network-ip>`                                                                                                                                  | ASA has expected path or VPN crypto map path for remote network                    |
|    5 | Review existing NAT rules                                                    | ASA           | `show nat`                                                                                                                                                        | NAT table sections are visible                                                     |
|    6 | Review NAT rule order and hit counts                                         | ASA           | `show nat detail`                                                                                                                                                 | Existing NAT rules and hit counts are visible                                      |
|    7 | Confirm broad Dynamic PAT exists if this exemption is needed                 | ASA           | `show nat detail`                                                                                                                                                 | Broad PAT exists and would otherwise translate the traffic                         |
|    8 | Identify local exempt network                                                | ASA / Notes   | `<local-network> <local-mask>`                                                                                                                                    | Local side of exempt traffic is known                                              |
|    9 | Identify remote exempt network                                               | ASA / Notes   | `<remote-network> <remote-mask>`                                                                                                                                  | Remote side of exempt traffic is known                                             |
|   10 | Confirm NAT exemption must be above broad PAT                                | ASA / Notes   | Section 1 Manual NAT                                                                                                                                              | Identity NAT should be checked before Auto NAT Dynamic PAT                         |
|   11 | Enter global configuration mode                                              | ASA           | `configure terminal`                                                                                                                                              | ASA enters configuration mode                                                      |
|   12 | Create local network object                                                  | ASA           | `object network <local-object>`                                                                                                                                   | ASA enters local network object mode                                               |
|   13 | Define local network                                                         | ASA           | `subnet <local-network> <local-mask>`                                                                                                                             | Local object represents the real protected local network                           |
|   14 | Create remote network object                                                 | ASA           | `object network <remote-object>`                                                                                                                                  | ASA enters remote network object mode                                              |
|   15 | Define remote network                                                        | ASA           | `subnet <remote-network> <remote-mask>`                                                                                                                           | Remote object represents the real protected remote network                         |
|   16 | Configure identity NAT exemption                                             | ASA           | `nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup`                   | Traffic from local to remote is not translated                                     |
|   17 | Add sequence number if strict rule order is needed                           | ASA           | `nat (inside,outside) <sequence-number> source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | Identity NAT is placed before broader NAT rules                                    |
|   18 | Configure reverse identity NAT only if design requires explicit reverse rule | ASA           | `nat (outside,inside) source static <remote-object> <remote-object> destination static <local-object> <local-object> no-proxy-arp route-lookup`                   | Reverse-direction exemption exists if required by the lab/design                   |
|   19 | Verify crypto ACL uses real networks if this is site-to-site VPN             | ASA           | `show running-config access-list <crypto-acl>`                                                                                                                    | Crypto ACL matches local real network to remote real network                       |
|   20 | Verify remote-access VPN pool object if this is RA VPN exemption             | ASA           | `show running-config object network <vpn-pool-object>`                                                                                                            | VPN pool object exists if exempting inside-to-VPN-client traffic                   |
|   21 | Save configuration                                                           | ASA           | `write memory`                                                                                                                                                    | NAT exemption configuration is saved                                               |
|   22 | Verify NAT table placement                                                   | ASA           | `show nat`                                                                                                                                                        | Identity NAT appears in Manual NAT Section 1 before broad PAT                      |
|   23 | Verify NAT details                                                           | ASA           | `show nat detail`                                                                                                                                                 | Identity NAT rule shows untranslated source and destination objects                |
|   24 | Simulate exempt traffic                                                      | ASA           | `packet-tracer input inside ip <local-host-ip> 0 <remote-host-ip> 0 detailed`                                                                                     | NAT phase matches identity NAT and leaves addresses unchanged                      |
|   25 | Generate real traffic                                                        | Host / Router | `ping <remote-host-ip>`                                                                                                                                           | Traffic attempts to cross VPN or exempt path                                       |
|   26 | Verify NAT hit count increments                                              | ASA           | `show nat detail`                                                                                                                                                 | Identity NAT rule hit count increases                                              |
|   27 | Verify xlate behavior                                                        | ASA           | `show xlate`                                                                                                                                                      | Identity translation appears or traffic is shown untranslated depending ASA output |
|   28 | Verify connection state                                                      | ASA           | `show conn`                                                                                                                                                       | Connection appears for local-to-remote flow                                        |
|   29 | Verify NATed connection detail                                               | ASA           | `show conn long`                                                                                                                                                  | Addresses remain real/untranslated for exempt traffic                              |
|   30 | Check drops if traffic fails                                                 | ASA           | `show asp drop`                                                                                                                                                   | Drop reason points to NAT, ACL, route, VPN, inspection, or adjacency issue         |
# ASA_NAT_Exemption_Identity_NAT_Site_To_Site_Skeleton

```text
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

write memory
```

# ASA_NAT_Exemption_Identity_NAT_With_Sequence_Skeleton

```text
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) <sequence-number> source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

write memory
```


# ASA_NAT_Exemption_Identity_NAT_Remote_Access_Pool_Skeleton

```text
configure terminal

object network <inside-local-object>
 subnet <inside-network> <inside-mask>

object network <vpn-pool-object>
 subnet <vpn-pool-network> <vpn-pool-mask>

nat (inside,outside) source static <inside-local-object> <inside-local-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup

write memory
```

# ASA_NAT_Exemption_Identity_NAT_Multiple_Remote_Networks_Skeleton

```text
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object-group network <remote-object-group>
 network-object <remote-network-1> <remote-mask-1>
 network-object <remote-network-2> <remote-mask-2>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object-group> <remote-object-group> no-proxy-arp route-lookup

write memory
```

# ASA_NAT_Exemption_Identity_NAT_Verification_Commands

| Task                               | Command                                                                                          | Expected Result                                                            |
| ---------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| Verify interface state             | `show interface ip brief`                                                                        | Inside and outside interfaces are up/up                                    |
| Verify nameifs                     | `show nameif`                                                                                    | Interface names match the NAT rule interface pair                          |
| Verify NAT table order             | `show nat`                                                                                       | Identity NAT appears before broad PAT                                      |
| Verify NAT details                 | `show nat detail`                                                                                | Identity NAT rule shows local-to-local and remote-to-remote static mapping |
| Verify packet-tracer NAT phase     | `packet-tracer input inside ip <local-host-ip> 0 <remote-host-ip> 0 detailed`                    | NAT phase matches identity NAT                                             |
| Verify ICMP-specific packet-tracer | `packet-tracer input inside icmp <local-host-ip> 8 0 <remote-host-ip> detailed`                  | ICMP flow is allowed if policy permits                                     |
| Verify TCP-specific packet-tracer  | `packet-tracer input inside tcp <local-host-ip> <src-port> <remote-host-ip> <dst-port> detailed` | TCP flow is allowed if policy permits                                      |
| Verify NAT hit count               | `show nat detail`                                                                                | Identity NAT hit count increments after connection build                   |
| Verify translation table           | `show xlate`                                                                                     | Exempt traffic is not PATed to outside interface                           |
| Verify connection table            | `show conn`                                                                                      | Connection appears for exempt traffic                                      |
| Verify detailed connection         | `show conn long`                                                                                 | Real source and destination are visible                                    |
| Verify crypto ACL if VPN           | `show access-list <crypto-acl>`                                                                  | Crypto ACL matches real local and remote networks                          |
| Verify IPsec SAs if VPN            | `show crypto ipsec sa`                                                                           | Encrypt/decrypt counters increment for VPN traffic                         |
| Verify IKE SA if VPN               | `show crypto ikev1 sa` or `show crypto ikev2 sa`                                                 | IKE tunnel is established                                                  |
| Verify drops                       | `show asp drop`                                                                                  | No relevant drop reason increments for intended traffic                    |

# ASA_NAT_Exemption_Identity_NAT_Crypto_ACL_Alignment

| NAT Exemption State                            | Crypto ACL Should Match                        | Operational Meaning                                                          |
| ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------- |
| Identity NAT exists                            | Real local network to real remote network      | Normal and preferred VPN design                                              |
| Identity NAT missing and traffic is translated | Post-NAT/global addresses                      | Source warns crypto ACL must match post-NAT addresses if NAT is performed    |
| Broad PAT catches VPN traffic                  | Outside interface IP to remote network         | Usually wrong; fix identity NAT order instead of changing crypto ACL blindly |
| Remote-access VPN exemption                    | Inside real network to VPN pool real addresses | ASA should not PAT inside-to-VPN-client traffic                              |
# ASA_NAT_Exemption_Identity_NAT_Rollback

| Step | Task                                       | Device | Command                                                                                                                                                              | Expected Result                                  |
| ---: | ------------------------------------------ | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
|    1 | Identify identity NAT rule                 | ASA    | `show nat detail`                                                                                                                                                    | Exact NAT rule and object names are identified   |
|    2 | Remove identity NAT rule                   | ASA    | `no nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup`                   | Identity NAT rule is removed                     |
|    3 | Remove sequenced identity NAT rule if used | ASA    | `no nat (inside,outside) <sequence-number> source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | Sequenced identity NAT rule is removed           |
|    4 | Remove reverse identity NAT if configured  | ASA    | `no nat (outside,inside) source static <remote-object> <remote-object> destination static <local-object> <local-object> no-proxy-arp route-lookup`                   | Reverse identity NAT rule is removed             |
|    5 | Remove local object if unused              | ASA    | `no object network <local-object>`                                                                                                                                   | Local object is removed if no longer referenced  |
|    6 | Remove remote object if unused             | ASA    | `no object network <remote-object>`                                                                                                                                  | Remote object is removed if no longer referenced |
|    7 | Remove remote object-group if unused       | ASA    | `no object-group network <remote-object-group>`                                                                                                                      | Remote object-group is removed                   |
|    8 | Clear translations                         | ASA    | `clear xlate`                                                                                                                                                        | Existing translations are cleared                |
|    9 | Clear stale connections                    | ASA    | `clear conn address <local-host-ip>`                                                                                                                                 | Existing connection state is cleared             |
|   10 | Verify identity NAT is gone                | ASA    | `show nat`                                                                                                                                                           | Identity NAT rule no longer appears              |
|   11 | Save rollback state                        | ASA    | `write memory`                                                                                                                                                       | Rollback is saved                                |
# ASA_NAT_Exemption_Identity_NAT_Failure_Checks

| Symptom                                                 | Command                                              | What Usually Broke                                                                                   |
| ------------------------------------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| VPN traffic gets PATed                                  | `packet-tracer ... detailed` and `show nat detail`   | Identity NAT missing, wrong interface pair, or broad PAT matched first                               |
| Identity NAT has zero hits                              | `show nat detail`                                    | Local object, remote object, source interface, or destination interface does not match the real flow |
| Packet-tracer matches Dynamic PAT instead               | `show nat`                                           | Identity NAT is below a broader rule or placed in the wrong section                                  |
| Crypto ACL counters do not increment                    | `show access-list <crypto-acl>`                      | Traffic is translated or does not match the VPN interesting traffic ACL                              |
| IPsec encrypt/decrypt counters stay zero                | `show crypto ipsec sa`                               | NAT exemption, crypto ACL, route, or tunnel selector mismatch                                        |
| One side encrypts but other side does not decrypt       | `show crypto ipsec sa` on both peers                 | Proxy ID/crypto ACL mismatch or NAT mismatch between peers                                           |
| NAT exemption works but ping fails                      | `show conn`, `show asp drop`, `show crypto ipsec sa` | ACL, VPN SA, route, inspection, or remote-side firewall issue                                        |
| Identity NAT rule appears but wrong traffic is exempted | `show nat detail`                                    | Object definitions are too broad or wrong remote object is used                                      |
| Remote-access VPN client cannot reach inside            | `show nat detail` and `show vpn-sessiondb`           | Missing identity NAT between inside network and VPN pool, or VPN filter blocks traffic               |
| ASA shows no route to remote network                    | `show route <remote-host-ip>`                        | VPN route/crypto map behavior, static route, or route lookup expectation is wrong                    |
| `route-lookup` changes behavior unexpectedly            | `packet-tracer ... detailed`                         | Routing table chooses a different egress path than expected                                          |
| Proxy ARP issue appears                                 | `show arp`                                           | Missing or incorrect `no-proxy-arp` behavior for identity NAT design                                 |
| Real traffic fails but packet-tracer allows             | `show conn`, `show xlate`, `show asp drop`, captures | Stale xlate/conn, remote peer, host firewall, or return path issue                                   |
| Identity NAT removed and VPN breaks                     | `show nat detail`                                    | Broad Dynamic PAT now translates VPN interesting traffic                                             |