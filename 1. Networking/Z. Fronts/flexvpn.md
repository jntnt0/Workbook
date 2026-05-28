FlexVPN_Note_Order.md

FlexVPN_Mechanism_Note_Order

Use this order. Do not make one note per lab. Make mechanism notes, then attach labs underneath.

Cisco’s own FlexVPN framing supports this order: FlexVPN is a unified IKEv2 framework for site-to-site, remote access, hub-and-spoke, and spoke-to-spoke partial mesh, built around the tunnel interface model.   Smart defaults are a simplification layer that prebuilds common IKEv2/IPsec objects such as authorization policy, proposal, policy, IPsec profile, and transform set.  

|   |   |   |   |
|---|---|---|---|
|Order|Mechanism Note|Mental Model To Address|Related Labs|
|01|FlexVPN_IKEv2_IPsec_VTI_Core.md|FlexVPN is not “just IPsec.” It is IKEv2 control plane plus IPsec data protection attached to routed tunnel interfaces. The tunnel is what routing sees. IKEv2/IPsec is what protects it.|flexvpn-site-to-site smart defaults-final; flexvpn-site-to-site without smart defaults-final; flexvpn site-to-site pki authentication-final; flexvpn-hub-and-spoke-final; flexvpn-site-to-site-ikev2-routing-final; flexvpn-spoke-to-spoke-final; flexvpn-spoke-to-spoke-bgp-pool-final; mpls-over-flexvpn-final|
|02|FlexVPN_Site_To_Site_Smart_Defaults.md|Smart defaults let you build a working FlexVPN with fewer explicit crypto objects. The mental model is “Cisco supplies sane default crypto scaffolding, you still define identity, authentication, tunnel, and routing.”|flexvpn-site-to-site smart defaults-final|
|03|FlexVPN_Site_To_Site_Explicit_IKEv2_IPsec.md|Without smart defaults, every moving part becomes visible: IKEv2 proposal, IKEv2 policy, keyring or PKI, IKEv2 profile, transform set, IPsec profile, tunnel protection. This is the real dependency chain.|flexvpn-site-to-site without smart defaults-final|
|04|FlexVPN_PKI_Certificate_Authentication.md|PKI changes the authentication model, not the tunnel model. The tunnel still uses IKEv2/IPsec/VTI, but peer trust is based on certificates, trustpoints, subject/SAN identity, and RSA signature instead of PSK.|flexvpn site-to-site pki authentication-final|
|05|FlexVPN_IKEv2_Profile_Identity_And_Authorization.md|The IKEv2 profile is the match and policy brain. It decides which peer matches, what local identity is sent, how the peer authenticates, what AAA or authorization policy applies, and whether a virtual-template is used.|flexvpn-site-to-site smart defaults-final; flexvpn-site-to-site without smart defaults-final; flexvpn site-to-site pki authentication-final; flexvpn-site-to-site-ikev2-routing-final; flexvpn-hub-and-spoke-final; flexvpn-remote-access-anyconnect-ipsec-final|
|06|FlexVPN_IKEv2_Route_Authorization.md|FlexVPN can advertise routes as part of IKEv2 authorization instead of relying only on a routing protocol over the tunnel. The key idea is that route exchange can be tied to IKEv2 SA establishment. Cisco’s dynamic-peer example uses route set interface and route set access-list inside the IKEv2 authorization policy.|flexvpn-site-to-site-ikev2-routing-final|
|07|FlexVPN_DVTI_Virtual_Template_And_Virtual_Access.md|A virtual-template is a reusable tunnel template. When a dynamic peer connects, IOS XE clones it into a virtual-access interface. This is the bridge from static site-to-site FlexVPN into scalable hub/spoke and remote-access behavior. Cisco describes virtual-template as the template used to create dynamic virtual-access interfaces.|flexvpn-hub-and-spoke-final; flexvpn-remote-access-anyconnect-ipsec-final; flexvpn-spoke-to-spoke-final; flexvpn-spoke-to-spoke-bgp-pool-final|
|08|FlexVPN_Hub_And_Spoke.md|Hub-and-spoke FlexVPN is server/client behavior. Spokes authenticate to the hub, receive or form routed overlay reachability, and normally use the hub as transit unless spoke-to-spoke shortcuts are added.|flexvpn-hub-and-spoke-final|
|09|FlexVPN_Remote_Access_AnyConnect_IKEv2.md|Remote access FlexVPN is not site-to-site with a laptop. It adds user authentication, address assignment, AnyConnect profile delivery, EAP/AAA, and client policy. Cisco’s AnyConnect IKEv2 headend flow uses AAA, a trustpoint, local address pool, IKEv2 authorization policy, AnyConnect XML profile, and an IKEv2 profile tied to a virtual-template.|flexvpn-remote-access-anyconnect-ipsec-final|
|10|FlexVPN_Spoke_To_Spoke_NHRP_Shortcut.md|Spoke-to-spoke is the shortcut mechanism. Initial reachability may go through the hub, then NHRP resolves the remote spoke and allows direct crypto between spokes. The key mental model is redirect, resolution, virtual-access creation, IPsec SA creation, next-hop override, then shortcut route installation.|flexvpn-spoke-to-spoke-final|
|11|FlexVPN_Spoke_To_Spoke_BGP_Pool.md|This is the routing-specific spoke-to-spoke variant. Keep the note focused on how BGP-learned reachability, address pools, next-hop behavior, and shortcut formation interact. Do not bury this inside the generic spoke-to-spoke note.|flexvpn-spoke-to-spoke-bgp-pool-final|
|12|MPLS_Over_FlexVPN.md|FlexVPN becomes the encrypted transport. MPLS/VPN control-plane and label-forwarding behavior ride across the protected overlay. The failure model changes: first prove FlexVPN tunnel and crypto, then prove MPLS label exchange, VPNv4/VRF routing, and CEF/label forwarding.|mpls-over-flexvpn-final|
|13|FlexVPN_Verification_And_Troubleshooting_Workflow.md|Verification must follow the stack: interface up, tunnel up, IKEv2 SA ready, IPsec SAs active, encaps/decaps incrementing, routing present, application traffic passing. Cisco’s FlexVPN verification flow uses show ip interface brief, show crypto ikev2 sa, and show crypto ipsec sa to prove tunnel, IKEv2, and encrypted traffic.|all listed FlexVPN labs|

Final Order Only

01_FlexVPN_IKEv2_IPsec_VTI_Core.md

02_FlexVPN_Site_To_Site_Smart_Defaults.md

03_FlexVPN_Site_To_Site_Explicit_IKEv2_IPsec.md

04_FlexVPN_PKI_Certificate_Authentication.md

05_FlexVPN_IKEv2_Profile_Identity_And_Authorization.md

06_FlexVPN_IKEv2_Route_Authorization.md

07_FlexVPN_DVTI_Virtual_Template_And_Virtual_Access.md

08_FlexVPN_Hub_And_Spoke.md

09_FlexVPN_Remote_Access_AnyConnect_IKEv2.md

10_FlexVPN_Spoke_To_Spoke_NHRP_Shortcut.md

11_FlexVPN_Spoke_To_Spoke_BGP_Pool.md

12_MPLS_Over_FlexVPN.md

13_FlexVPN_Verification_And_Troubleshooting_Workflow.md

The important correction: remote-access-anyconnect-ipsec should not be placed after spoke-to-spoke just because it feels “advanced.” It belongs after DVTI/virtual-template because the mental model depends on dynamic virtual-access, address pools, AAA, and client profile behavior. Spoke-to-spoke and MPLS-over-FlexVPN should stay later because they depend on the base tunnel and routing models being clean first.