## Overview

Use this when a Windows machine is stuck in a broken Azure AD / Microsoft account authentication state. Symptoms include repeated sign-in prompts, SSO failures, or Office/Azure CLI refusing to authenticate despite valid credentials.

This procedure clears the Web Account Manager (WAM) token broker cache and resets the AAD Broker Plugin app package.

---

## Step 1 — Clear the Token Broker Cache

Open File Explorer and navigate to each path below. Delete everything inside the folder, not the folder itself.

**Path 1:**

```
%LOCALAPPDATA%\Packages\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\AC\TokenBroker\
```

**Path 2:**

```
%LOCALAPPDATA%\Packages\Microsoft.AAD.BrokerPlugin_cw5n1h2txyewy\AC\
```

> **If Windows refuses to delete because files are locked**, perform the deletion in Safe Mode:
> 
> 1. Hold **Shift** and click **Restart**
> 2. Navigate to **Troubleshoot** > **Advanced options** > **Startup Settings** > **Restart**
> 3. Press **4** to boot into Safe Mode
> 4. Delete the folder contents
> 5. Reboot back into normal mode

---

## Step 2 — Reset the AAD Broker Plugin

1. Open **Settings** > **Apps** > **Installed apps**
2. Search for **Microsoft AAD Broker Plugin**
3. Click the three-dot menu > **Advanced options**
4. In order: click **Terminate**, then **Repair**, then **Reset**

---

## Step 3 — Reboot

Reboot the machine after completing both steps before testing authentication.

---

## Verification

After reboot, sign in to the affected application again. You will be prompted to authenticate from scratch — this is expected. A clean token will be issued and cached.

---

## Common Scenarios

|Symptom|Notes|
|---|---|
|Repeated MFA prompts despite successful login|WAM cache corrupted or stale token persisting|
|Azure CLI `az login` hangs or fails silently|Token broker interference — clear cache then retry|
|Office apps looping on sign-in screen|Reset broker plugin after clearing cache|
|Files locked during deletion|Perform deletion in Safe Mode|