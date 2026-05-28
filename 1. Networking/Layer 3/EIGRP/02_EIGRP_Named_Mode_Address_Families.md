EIGRP_Named_Mode_Address_Families.md

EIGRP_Named_Mode_Address_Families

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | EIGRP source mapping | Points EIGRP depth to `All_combined_part3.md`, Book 3 |
| `All_combined_part3.md` | Book 3: EIGRP for IP | Supports EIGRP concepts, DUAL behavior, topology table logic, metrics, summarization, filtering, redistribution, stub, and authentication |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Named Mode | Supports named-mode structure: `router eigrp <process-name>`, `address-family ipv4 unicast autonomous-system <as-number>`, `af-interface`, and `topology base` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Network Statement | Confirms the `network` command enables EIGRP on matching interfaces and adds connected networks to the EIGRP topology table |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Passive Interfaces for Named Mode | Supports `af-interface default`, `passive-interface`, and per-interface `no passive-interface` syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Confirming Interfaces, Verifying Neighbor Adjacencies, Displaying Installed EIGRP Routes | Supports verification commands: `show ip eigrp interfaces`, `show ip eigrp neighbors`, `show ip protocols`, and `show ip route eigrp` |
# EIGRP_Named_Mode_Address_Families_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Named mode | EIGRP configuration is centralized under a named process instead of scattered between router mode and interface mode |
| Process name | The value after `router eigrp` is only a local process name; it is not automatically the EIGRP AS number |
| Address family | The address-family binds EIGRP to IPv4 or IPv6 and defines the real autonomous system number |
| Autonomous system | Must match between routers that should become EIGRP neighbors |
| Network statement | Enables EIGRP on interfaces whose IP addresses match the statement |
| Wildcard mask | Controls which interfaces join the EIGRP address family |
| `af-interface default` | Applies interface-level EIGRP settings to all EIGRP-enabled interfaces |
| Specific `af-interface` | Overrides the default interface settings for one interface |
| Passive interface | Advertises the connected network but suppresses EIGRP hello packets and neighbor formation |
| `topology base` | Holds topology and route-selection settings such as maximum paths, variance, redistribution, distance, and active timers |
| Wide metrics | Named mode uses EIGRP wide metrics by default; `show ip protocols` shows K6 when wide metrics are active |
| Common failure point | Most named-mode failures come from confusing process name with AS number, missing address-family, wrong network wildcard, passive router-facing interfaces, or editing classic mode instead of named mode |
# EIGRP_Named_Mode_Address_Families_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm routed interfaces are up before enabling EIGRP | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm local connected routes exist | All routers | `show ip route connected` | Local LAN, loopback, and transit prefixes appear as connected routes |
| 3 | Enter global configuration mode | All routers | `configure terminal` | Router enters `(config)#` mode |
| 4 | Create the named EIGRP process | All routers | `router eigrp <PROCESS_NAME>` | Router enters named EIGRP configuration mode |
| 5 | Create the IPv4 unicast address family and bind it to the real AS number | All routers | `address-family ipv4 unicast autonomous-system <AS_NUMBER>` | Router enters `(config-router-af)#` mode |
| 6 | Set a stable EIGRP router ID | All routers | `eigrp router-id <ROUTER_ID>` | EIGRP uses the manually defined router ID |
| 7 | Enable EIGRP on a specific transit interface using host wildcard matching | All routers | `network <INTERFACE_IP> 0.0.0.0` | Only the exact matching interface joins the address family |
| 8 | Enable EIGRP on a full subnet when appropriate | All routers | `network <SUBNET_ID> <WILDCARD_MASK>` | All matching local interfaces join the address family |
| 9 | Enable EIGRP for loopback advertisement if loopbacks are used | All routers | `network <LOOPBACK_IP> 0.0.0.0` | Loopback prefix is added to the EIGRP topology table |
| 10 | Apply a safe default that prevents accidental neighbor formation | All routers | `af-interface default` | Router enters default EIGRP interface submode |
| 11 | Make all EIGRP-enabled interfaces passive by default | All routers | `passive-interface` | EIGRP suppresses hello packets on all matched interfaces by default |
| 12 | Exit default interface submode | All routers | `exit-af-interface` | Router returns to address-family mode |
| 13 | Enter the router-facing interface submode | All routers | `af-interface <ROUTER_FACING_INTERFACE>` | Router enters interface-specific EIGRP submode |
| 14 | Allow neighbor formation on the router-facing interface | All routers | `no passive-interface` | Interface can send and receive EIGRP hellos |
| 15 | Exit the specific interface submode | All routers | `exit-af-interface` | Router returns to address-family mode |
| 16 | Enter the base topology submode | All routers | `topology base` | Router enters EIGRP topology configuration mode |
| 17 | Confirm or set the intended ECMP path count if needed | All routers | `maximum-paths <NUMBER>` | EIGRP allows the intended number of equal-cost paths |
| 18 | Exit topology submode | All routers | `exit-af-topology` | Router returns to address-family mode |
| 19 | Exit address-family mode | All routers | `exit-address-family` | Router returns to named EIGRP process mode |
| 20 | Exit configuration mode | All routers | `end` | Router returns to privileged EXEC mode |
| 21 | Verify named-mode configuration structure | All routers | `show running-config | section router eigrp` | Output shows `router eigrp <PROCESS_NAME>`, `address-family`, `af-interface`, `topology base`, and `network` statements |
| 22 | Verify intended EIGRP interfaces are active | All routers | `show ip eigrp interfaces` | Router-facing interfaces appear; passive interfaces do not appear as active EIGRP interfaces |
| 23 | Verify detailed interface behavior | All routers | `show ip eigrp interfaces detail` | Hello/hold timers, split horizon, authentication status, and topology advertisement are visible |
| 24 | Verify neighbor formation | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 25 | Verify named-mode protocol state | All routers | `show ip protocols` | Output shows the EIGRP address family, AS number, router ID, K values, passive interfaces, network statements, and AD values |
| 26 | Verify EIGRP topology entries | All routers | `show ip eigrp topology` | Expected prefixes appear in passive state with successors |
| 27 | Verify EIGRP routes enter the routing table | All routers | `show ip route eigrp` | Remote internal EIGRP routes appear as `D` |
| 28 | Verify end-to-end reachability | Edge routers/hosts | `ping <REMOTE_IP>` | Ping succeeds across the EIGRP domain |
| 29 | Verify forwarding path | Edge routers/hosts | `traceroute <REMOTE_IP>` | Path follows expected EIGRP next hops |
| 30 | Save the working configuration | All routers | `write memory` | Configuration is saved |
# EIGRP_Named_Mode_Address_Families_Skeleton
! Baseline verification
show ip interface brief
show ip route connected
! Named-mode EIGRP configuration
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   eigrp router-id <ROUTER_ID>
   ! Enable EIGRP on intended interfaces
   network <TRANSIT_INTERFACE_IP_1> 0.0.0.0
   network <TRANSIT_INTERFACE_IP_2> 0.0.0.0
   network <LOOPBACK_IP> 0.0.0.0
   ! Safe passive-interface model
   af-interface default
    passive-interface
   exit-af-interface
   ! Allow neighbor formation only on router-facing links
   af-interface <ROUTER_FACING_INTERFACE_1>
    no passive-interface
   exit-af-interface
   af-interface <ROUTER_FACING_INTERFACE_2>
    no passive-interface
   exit-af-interface
   ! Topology/RIB behavior
   topology base
    maximum-paths <NUMBER>
   exit-af-topology
  exit-address-family
 end
! Verification
show running-config | section router eigrp
show ip eigrp interfaces
show ip eigrp interfaces detail
show ip eigrp neighbors
show ip protocols
show ip eigrp topology
show ip route eigrp
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Named_Mode_Address_Families_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state and addressing | EIGRP-facing interfaces are `up/up` |
| `show ip route connected` | Confirms local prefixes exist before EIGRP advertises them | Transit, LAN, and loopback prefixes appear as connected |
| `show running-config | section router eigrp` | Confirms named-mode hierarchy | Shows `router eigrp <PROCESS_NAME>`, `address-family ipv4 unicast autonomous-system <AS>`, `af-interface`, `topology base`, and `network` statements |
| `show ip eigrp interfaces` | Confirms active EIGRP interfaces | Router-facing interfaces appear with expected peer counts |
| `show ip eigrp interfaces detail` | Confirms detailed interface behavior | Shows hello interval, hold time, split horizon, authentication state, and advertised topology |
| `show ip eigrp neighbors` | Confirms adjacency formation | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms protocol-level named-mode state | Shows EIGRP address-family process, AS number, router ID, K values, passive interfaces, network statements, and AD values |
| `show ip protocols | include AS|K|Router-ID|Metric` | Confirms AS, router ID, and metric style quickly | Named mode shows K6 in the metric weight output |
| `show ip eigrp topology` | Confirms EIGRP has the expected topology entries | Prefixes appear in passive state with successors |
| `show ip eigrp topology <PREFIX>/<PREFIX_LENGTH>` | Confirms route attributes for one prefix | Shows successor, feasible distance, reported distance, metric attributes, and originating router |
| `show ip route eigrp` | Confirms EIGRP routes were installed in the RIB | Internal EIGRP routes appear as `D`; external routes appear as `D EX` |
| `show ip route <PREFIX>` | Confirms exact RIB winner and next hop | Route points to expected EIGRP next hop and outgoing interface |
| `ping <REMOTE_IP>` | Confirms reachability | Ping succeeds |
| `traceroute <REMOTE_IP>` | Confirms forwarding path | Hops follow the expected EIGRP path |
# EIGRP_Named_Mode_Address_Families_Rollback
! Remove the entire named EIGRP process
configure terminal
 no router eigrp <PROCESS_NAME>
end
! Verify the named process is removed
show running-config | section router eigrp
show ip eigrp neighbors
show ip eigrp interfaces
show ip route eigrp
! Expected result:
! No named EIGRP process remains.
! No EIGRP neighbors remain.
! No EIGRP-enabled interfaces remain.
! No EIGRP-learned routes remain in the routing table.
write memory
# EIGRP_Named_Mode_Address_Families_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| No EIGRP neighbors form | Process name was mistaken for the AS number | `show running-config | section router eigrp` | Ensure the real AS is configured under `address-family ipv4 unicast autonomous-system <AS_NUMBER>` |
| No EIGRP neighbors form | Address family is missing | `show running-config | section router eigrp` | Add `address-family ipv4 unicast autonomous-system <AS_NUMBER>` |
| No EIGRP neighbors form | AS numbers do not match | `show ip protocols` | Match the address-family AS number on routers that should peer |
| No EIGRP neighbors form | Router-facing interface is passive | `show ip protocols` | Under `af-interface <interface>`, configure `no passive-interface` |
| No EIGRP neighbors form | Interface not matched by any network statement | `show ip eigrp interfaces` | Add `network <interface-ip> 0.0.0.0` or the correct subnet wildcard |
| Too many interfaces run EIGRP | Network statement is too broad | `show ip eigrp interfaces` | Replace broad wildcard statements with exact interface matches |
| Passive LAN is not visible in `show ip eigrp interfaces` | Normal passive-interface behavior | `show ip protocols` and `show ip eigrp topology` | Confirm the connected prefix is still advertised through the topology table |
| Routes are in the topology table but not the RIB | Another route has better AD or metric | `show ip route <prefix>` | Compare RIB winner and adjust design only if needed |
| Neighbor forms but routes are missing | Prefix not added to EIGRP topology table | `show ip eigrp topology` | Add the missing interface with a correct `network` statement |
| Neighbor forms but traffic fails | Return route is missing | `show ip route <source-prefix>` on remote router | Ensure reverse path is advertised |
| Named mode config exists but classic commands were edited instead | Wrong configuration hierarchy | `show running-config | section router eigrp` | Put named-mode settings under address-family, af-interface, or topology base as appropriate |
| Metrics look different from classic mode routers | Named mode uses wide metrics | `show ip protocols | include K` | Keep metric styles and K values consistent across the design |
| Neighbor resets after changing K values | K-value mismatch | `show ip protocols | include K` | Restore matching K values across neighbors |
| `maximum-paths` change does not work as expected | Command placed in wrong hierarchy | `show running-config | section router eigrp` | Put `maximum-paths` under `topology base` in named mode |
# Index
# Source_Basis
# EIGRP_Named_Mode_Address_Families_Mental_Model
# EIGRP_Named_Mode_Address_Families_Configuration_Checklist
# EIGRP_Named_Mode_Address_Families_Skeleton
# EIGRP_Named_Mode_Address_Families_Verification_Commands
# EIGRP_Named_Mode_Address_Families_Rollback
# EIGRP_Named_Mode_Address_Families_Failure_Checks