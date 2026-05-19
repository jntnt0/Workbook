# ROUTING & CONTROL PLANE (IOS XE)

## 🚨 PROBLEM: “Traffic to destination fails”

---

## ⚡ STEP 1 — ROUTE

**show ip route <destination>**

- ❌ No route → **FIX RIB**
- ✅ Route exists → go next

## ⚡ STEP 2 — CORRECT ROUTE?
**show ip route**
- ❌ Wrong route → **AD / METRIC / POLICY**
- ✅ Correct → go next
- 
---

## ⚡ STEP 3 — NEXT-HOP RESOLUTION
**show ip route <next-hop-ip>**
- ❌ Not resolved → **RECURSION**
- ✅ Resolved → go next
---

## ⚡ STEP 4 — EXIT INTERFACE
**show ip interface brief**
- ❌ Not up/up → **INTERFACE**
- ✅ up/up → go next

---

## ⚡ STEP 5 — L2 (ARP)

**show ip arp <next-hop-ip>**
- ❌ Incomplete → **L2 / VLAN / TRUNK**
- ✅ MAC learned → go next
---

## ⚡ STEP 6 — FORWARDING (CEF)
**show ip cef <prefix>**

- ❌ Drop / unresolved → **CEF**
- ✅ OK → go next

---

## ⚡ STEP 7 — DATA PLANE TEST
**ping <destination> source <src>**
- ❌ Fail → go next
- ✅ Works → intermittent / higher layer

---

## ⚡ STEP 8 — PATH

**traceroute <destination>**
- ❌ Wrong path → **POLICY / AD**
- ❌ Stops → **UPSTREAM**
- ✅ Correct → go next
---

## ⚡ STEP 9 — CONTROL PLANE (IF DYNAMIC)

**show ip ospf neighbor**  
**show ip eigrp neighbors**  
**show ip bgp summary**

- ❌ Not FULL / Established → **CONTROL PLANE**
- ✅ Up → go next