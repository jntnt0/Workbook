Use this order. It is mechanism-first, not lab-first.

|   |   |   |   |
|---|---|---|---|
|Order|Mechanism note|Mental model the note must address|Related lab names|
|00|QoS_MQC_Class_Map_Policy_Map_Service_Policy.md|QoS starts with classification, then policy action, then interface attachment. Class-map identifies traffic, policy-map defines treatment, service-policy makes it real.|qos-cbwfq-final, qos-llq-final, qos-police-exceed-drop-final, qos-shape-average-final, qos-wred-final|
|01|QoS_CBWFQ_Bandwidth_Guarantee.md|CBWFQ gives classes minimum bandwidth during congestion. It does not create more bandwidth. It decides who gets protected when the link is congested.|qos-cbwfq-final|
|02|QoS_LLQ_Strict_Priority_Real_Time_Traffic.md|LLQ is CBWFQ plus a strict priority queue for delay-sensitive traffic. Voice gets serviced first, but the priority queue must be policed so it does not starve other classes.|qos-llq-final|
|03|QoS_Policing_Exceed_Drop.md|Policing enforces a rate by measuring traffic against a token bucket. Conforming traffic passes. Exceeding traffic can be dropped or remarked. It is abrupt rate enforcement, not buffering.|qos-police-exceed-drop-final|
|04|QoS_Shaping_Shape_Average.md|Shaping smooths traffic by buffering excess packets and sending them later. It is usually used outbound, especially when the physical interface is faster than the provider rate.|qos-shape-average-final|
|05|QoS_WRED_Congestion_Avoidance.md|WRED is not queue priority. It is early, probability-based dropping to prevent full queue tail-drop, mainly useful for TCP flows and differentiated drop precedence.|qos-wred-final|
|06|QoS_IntServ_RSVP_Admission_Control.md|RSVP is admission control. Instead of just treating packets differently after congestion starts, RSVP attempts to reserve resources along the path before admitting the flow. Cisco describes RSVP as the standard signaling protocol for end-to-end bandwidth reservation.|qos-rsvp-final|
|07|RSVP_DSBM_Shared_LAN_Admission_Control.md|DSBM is RSVP/SBM for shared IEEE 802-style LAN segments. One elected DSBM owns reservation admission control for the segment so multiple RSVP devices do not overcommit shared LAN bandwidth. Cisco documents ip rsvp dsbm candidate and show ip rsvp sbm for this role.|rsvp-dsbm-final|
|08|SIP_ALG_NAT_Firewall_SIP_Fixup.md|SIP ALG is not QoS scheduling. It is Layer 7 SIP/NAT/firewall helper behavior that inspects SIP signaling and may open NAT/firewall state for voice flows. Keep it near voice/NAT notes, not buried under CBWFQ/LLQ. Cisco IOS XE docs state SIP NAT support is enabled by default on port 5060 and can be disabled with no ip nat service sip.|qos-sip-alg-final|

Better final order:

00_QoS_MQC_Class_Map_Policy_Map_Service_Policy.md

01_QoS_CBWFQ_Bandwidth_Guarantee.md

02_QoS_LLQ_Strict_Priority_Real_Time_Traffic.md

03_QoS_Policing_Exceed_Drop.md

04_QoS_Shaping_Shape_Average.md

05_QoS_WRED_Congestion_Avoidance.md

06_QoS_IntServ_RSVP_Admission_Control.md

07_RSVP_DSBM_Shared_LAN_Admission_Control.md

08_SIP_ALG_NAT_Firewall_SIP_Fixup.md

The key correction: qos-sip-alg-final should not become a normal QoS note. It belongs as a voice/NAT/firewall helper mechanism. LLQ is the QoS voice treatment note; SIP ALG is the SIP signaling/NAT inspection note.