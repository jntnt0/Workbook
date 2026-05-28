### 🧩 START: “Traffic to destination fails”

---

## 1️⃣ Does the route exist?

**Command:**  
show ip route <destination>

- ❌ **NO** → go to **[RIB FAILURE]**
- ✅ **YES** → continue

---

## 2️⃣ Is the correct route selected?

**Command:**  
show ip route

- ❌ Wrong route (AD/metric issue) → **[RIB FAILURE]**
- ✅ Correct route → continue

---

## 3️⃣ Is next-hop resolved?

**Command:**  
show ip route <next-hop-ip>

- ❌ Not resolved → **[RECURSION FAILURE]**
- ✅ Resolved → continue

---

## 4️⃣ Is exit interface up?

**Command:**  
show ip interface brief

- ❌ Interface down → **[INTERFACE FAILURE]**
- ✅ up/up → continue

---

## 5️⃣ Is ARP working?

**Command:**  
show ip arp <next-hop-ip>

- ❌ Incomplete/missing → **[ARP FAILURE]**
- ✅ MAC learned → continue

---

## 6️⃣ Is CEF programmed?

**Command:**  
show ip cef <prefix>

- ❌ Drop / unresolved → **[CEF FAILURE]**
- ✅ Valid adjacency → continue

---

## 7️⃣ Does ping work?

**Command:**  
ping <destination> source <source>

- ❌ Fail → continue
- ✅ Works → problem is intermittent / higher layer

---

## 8️⃣ Does traceroute match expected path?

**Command:**  
traceroute <destination>

- ❌ Wrong path → **[PATH FAILURE]**
- ❌ Stops early → **[UPSTREAM FAILURE]**
- ✅ Correct → continue

---

## 9️⃣ Is control-plane neighbor up (if dynamic routing)?

**Command:**  
show ip ospf neighbor  
show ip eigrp neighbors  
show ip bgp summary

- ❌ Not established → **[CONTROL PLANE FAILURE]**
- ✅ Established → continue

---

## 🔻 FAILURE DOMAIN DRILL-DOWN

---

### 🔴 RIB FAILURE

- show ip route
- show running-config | include ^ip route
- show ip protocols

👉 Causes:

- Missing route
- Wrong AD / metric
- Filtering / redistribution issue

---

### 🟠 RECURSION FAILURE

- show ip route <next-hop-ip>

👉 Causes:

- No route to next-hop
- Broken dependency chain

---

### 🟡 INTERFACE FAILURE

- show ip interface brief
- show interfaces <int>

👉 Causes:

- Admin down
- Physical link down
- SVI inactive (no VLAN members)

---

### 🟢 ARP FAILURE

- show ip arp
- show mac address-table

👉 Causes:

- VLAN mismatch
- Trunk issue
- Wrong subnet

---

### 🔵 CEF FAILURE

- show ip cef <prefix>
- show adjacency <int> detail

👉 Causes:

- Drop adjacency
- Unresolved adjacency
- Hardware programming issue

---

### 🟣 CONTROL PLANE FAILURE

- show ip ospf neighbor
- show ip bgp summary
- show ip protocols

👉 Causes:

- ASN mismatch (BGP)
- Timer mismatch
- Authentication mismatch
- Passive interface
- Network statement missing

---

### ⚫ PATH FAILURE

- traceroute <destination>
- ping <hop-by-hop>

👉 Causes:

- PBR
- ACL / firewall
- NAT
- Asymmetric routing

---

### ⚪ UPSTREAM FAILURE

👉 Everything local checks out

👉 Causes:

- Return path broken
- ISP / remote side issue

---

# 🧠 TAC Mental Model (this is the key)

Every issue falls into ONE of these:

- Route missing → **RIB**
- Next-hop unknown → **Recursion**
- Can't exit → **Interface**
- Can't reach L2 → **ARP**
- Can't forward → **CEF**
- Neighbor down → **Control Plane**
- Traffic wrong → **Path**

---

# ⚡ How to use this FAST

You don’t run everything.

You **branch instantly**:

- No route? → skip everything → fix RIB
- Route exists but no ARP? → skip control plane → fix L2
- Neighbor down? → ignore CEF → fix protocol