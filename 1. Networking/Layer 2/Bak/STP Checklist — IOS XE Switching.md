|      |                                     |                                                 |                                             |                                      |                                              |
| ---- | ----------------------------------- | ----------------------------------------------- | ------------------------------------------- | ------------------------------------ | -------------------------------------------- |
| Step | Check                               | IOS XE Command                                  | Expected Result                             | Failure Indicator                    | Notes                                        |
| 1    | Confirm STP is running              | show spanning-tree summary                      | STP mode shown and VLANs participating      | No STP info / unexpected mode        | Start here before looking at ports           |
| 2    | Identify STP mode                   | show spanning-tree summary                      | Rapid PVST+, PVST, or MST expected          | Wrong STP mode                       | Mode mismatch can change behavior            |
| 3    | Check root bridge per VLAN          | show spanning-tree vlan <vlan-id>               | Expected switch is root                     | Wrong switch elected root            | Most common â€œSTP is weirdâ€ issue         |
| 4    | Confirm local root status           | show spanning-tree vlan <vlan-id> root          | Root bridge matches design                  | Unexpected root bridge ID            | Use on multiple switches                     |
| 5    | Check bridge priority               | show spanning-tree vlan <vlan-id>               | Root has lowest priority / best BID         | Default priority everywhere          | Root election left to chance                 |
| 6    | Check root port                     | show spanning-tree vlan <vlan-id>               | Non-root switch has one root port           | No root port / wrong root port       | Indicates bad cost/path/design               |
| 7    | Check designated ports              | show spanning-tree vlan <vlan-id>               | Downstream-facing ports are designated      | Unexpected blocking/alternate        | May reveal topology loop or bad root         |
| 8    | Check blocked/alternate ports       | show spanning-tree blockedports                 | Blocking only where expected                | Access port blocked unexpectedly     | STP may be protecting from loop              |
| 9    | Check port roles and states         | show spanning-tree interface <interface> detail | Role/state makes sense                      | Root/alternate/designated unexpected | Best command for per-port truth              |
| 10   | Check trunks participating in VLANs | show interfaces trunk                           | VLANs allowed and active                    | VLAN missing from trunk              | STP only matters for VLANs carried           |
| 11   | Check VLAN existence                | show vlan brief                                 | VLAN exists on switch                       | VLAN missing                         | STP per VLAN requires VLAN locally present   |
| 12   | Check EtherChannel consistency      | show etherchannel summary                       | Port-channel up, members bundled            | Members standalone/suspended         | Broken EtherChannel can create STP weirdness |
| 13   | Check PortFast on edge ports        | show spanning-tree interface <interface> detail | Edge host ports show PortFast               | Access ports slow to forward         | Use only on true edge ports                  |
| 14   | Check BPDU Guard                    | show spanning-tree summary                      | BPDU Guard enabled where expected           | Edge ports err-disabled              | BPDU received on PortFast edge               |
| 15   | Check err-disabled ports            | show interfaces status err-disabled             | No unexpected err-disabled ports            | Port down due to bpduguard           | Usually means loop/device plugged in         |
| 16   | Check root protection               | show running-config interface <interface>       | Root Guard only on intended ports           | Root Guard on wrong uplink           | Can block legitimate root path               |
| 17   | Check loop protection               | show spanning-tree inconsistentports            | No inconsistent ports                       | Ports listed inconsistent            | Root Guard/Loop Guard blocking               |
| 18   | Check topology changes              | show spanning-tree detail                       | Stable, low recent TC count                 | Frequent topology changes            | Flapping links or unstable edge              |
| 19   | Check MAC movement symptoms         | show mac address-table dynamic                  | MACs stable on expected ports               | MAC moves between ports              | Possible loop or bad L2 design               |
| 20   | Confirm final forwarding path       | show spanning-tree vlan <vlan-id>               | One logical L2 path, no unexpected blocking | Wrong link forwarding/blocking       | Final sanity check                           |

STP Sanity Decision Table

|                               |                                    |                                      |
| ----------------------------- | ---------------------------------- | ------------------------------------ |
| Observation                   | Meaning                            | Fix Direction                        |
| Wrong root bridge             | Root election not controlled       | Set STP priority on intended root    |
| All switches default priority | Root is accidental                 | Configure primary/secondary root     |
| Access port blocking          | STP sees possible loop             | Check cabling, PortFast, BPDU source |
| Trunk missing VLAN            | VLAN not participating across path | Fix allowed VLAN list                |
| EtherChannel members separate | STP sees multiple physical links   | Fix LACP/EtherChannel                |
| Port err-disabled             | Protection feature triggered       | Check cause, fix, then recover       |
| Inconsistent port             | STP guard feature blocking         | Check Root Guard / Loop Guard        |
| Frequent topology changes     | L2 instability                     | Find flapping port or edge device    |
| Root port unexpected          | Cost/path/root placement issue     | Check root bridge and path cost      |
| Host delay on link-up         | No PortFast on edge                | Enable PortFast on true host ports   |

Assigning Basic STP Root Configuration — IOS XE

|      |                                     |                                             |                                                |
| ---- | ----------------------------------- | ------------------------------------------- | ---------------------------------------------- |
| Step | Task                                | IOS XE Command                              | Expected Result                                |
| 1    | Enter config mode                   | conf t                                      | Global config mode                             |
| 2    | Set STP mode                        | spanning-tree mode rapid-pvst               | Rapid PVST+ enabled                            |
| 3    | Set primary root for VLAN           | spanning-tree vlan <vlan-id> root primary   | Switch becomes preferred root                  |
| 4    | Set secondary root on backup switch | spanning-tree vlan <vlan-id> root secondary | Backup switch preferred if primary fails       |
| 5    | Exit config                         | end                                         | Back to privileged EXEC                        |
| 6    | Verify root                         | show spanning-tree vlan <vlan-id>           | This switch is root or points to expected root |
| 7    | Verify summary                      | show spanning-tree summary                  | STP mode and status correct                    |
Rapid STP War Sheet

|                                 |                                        |                                                     |
| ------------------------------- | -------------------------------------- | --------------------------------------------------- |
| Question                        | Command                                | Good Answer                                         |
| Who is root?                    | show spanning-tree vlan <vlan-id> root | Expected root switch                                |
| Is this switch root?            | show spanning-tree vlan <vlan-id>      | â€œThis bridge is the rootâ€ only on intended root |
| Which port points to root?      | show spanning-tree vlan <vlan-id>      | Expected uplink is root port                        |
| What is blocked?                | show spanning-tree blockedports        | Only expected redundant link                        |
| Is a guard blocking me?         | show spanning-tree inconsistentports   | Empty                                               |
| Did BPDU Guard fire?            | show interfaces status err-disabled    | No unexpected ports                                 |
| Is trunk carrying VLAN?         | show interfaces trunk                  | VLAN listed as allowed and active                   |
| Is EtherChannel healthy?        | show etherchannel summary              | SU and member ports P                               |
| Are topology changes happening? | show spanning-tree detail              | No constant recent changes                          |
Bottom-line sanity rule

|                                  |                                                               |
| -------------------------------- | ------------------------------------------------------------- |
| Requirement                      | What It Means                                                 |
| Root bridge is intentional       | You know exactly which switch is root per VLAN                |
| Root ports make sense            | Non-root switches point toward the root over expected uplinks |
| Blocked ports make sense         | Only redundant paths are blocked                              |
| Edge ports are protected         | Host ports use PortFast + BPDU Guard                          |
| Trunks carry only intended VLANs | No accidental VLAN sprawl                                     |
| EtherChannels are bundled        | STP sees one logical link, not multiple loose links           |
| No constant topology changes     | L2 is stable                                                  |