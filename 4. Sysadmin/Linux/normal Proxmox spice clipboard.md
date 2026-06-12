# Clipboard Sharing in Proxmox VMs (SPICE)

## Overview

Clipboard sharing between a Proxmox VM and the host requires SPICE display protocol and the SPICE guest agent installed inside the VM. The browser-based noVNC console does not support reliable clipboard sharing — use Virt-Viewer over SPICE instead.

---

## Prerequisites

|Requirement|Detail|
|---|---|
|VM display|Set to SPICE in Proxmox hardware settings|
|Guest agent|`spice-vdagent` inside Linux VM, or SPICE Guest Tools for Windows|
|SPICE client|Virt-Viewer installed on the local machine|
|VM|Must be running a GUI desktop session|

---

## Step 1 — Install the SPICE Agent Inside the VM

**Linux:**

```bash
sudo apt update
sudo apt install spice-vdagent
sudo systemctl start spice-vdagentd
```

> `spice-vdagentd` starts automatically with the desktop session — `systemctl enable` is not required.

**Windows:**

Install the official SPICE Guest Tools ISO available from the Proxmox downloads page.

---

## Step 2 — Configure VM Display in Proxmox

1. Select the VM in the Proxmox GUI
2. Go to **Hardware** > **Display**
3. Set **Display type** to **SPICE**
4. Leave **Clipboard** at **Default** — do not set it to VNC
5. Confirm and reboot the VM

---

## Step 3 — Connect via Virt-Viewer

1. In the Proxmox GUI, open the VM and go to the **Console** tab
2. Click **Launch SPICE client** — this downloads a `.vv` file
3. Open the `.vv` file with Virt-Viewer on your local machine

This opens a native SPICE console with clipboard sharing enabled.

> The `.vv` file can be saved and reused. Re-download it if the Proxmox IP or VM port changes.

---

## Step 4 — Using Clipboard Sharing

|Direction|Method|
|---|---|
|Host to Linux VM|`Ctrl+Shift+V` or middle-click to paste|
|Host to Windows VM|Standard `Ctrl+V`|
|VM to Host|Copy inside VM, paste normally on host|

Clipboard sharing is bi-directional once the SPICE agent is running and Virt-Viewer is the active console.

---

## Troubleshooting

**Check agent status:**

```bash
systemctl status spice-vdagentd
```

**Confirm process is running:**

```bash
ps aux | grep spice-vdagentd
```

|Symptom|Likely Cause|Fix|
|---|---|---|
|Clipboard not working|Connected via browser noVNC console|Switch to Virt-Viewer SPICE client|
|Clipboard not working|`spice-vdagentd` not running|Start the service manually|
|`.vv` file fails to connect|IP or port changed|Re-download `.vv` from Proxmox console tab|