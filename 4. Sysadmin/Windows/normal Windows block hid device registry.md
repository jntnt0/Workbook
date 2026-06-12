# Blocking Specific HID Devices via Registry Policy

## Overview

Use this to block specific Human Interface Devices (HID) from being used on a Windows machine by denying their hardware IDs through registry policy. Common use cases include blocking unauthorized mice, keyboards, or other USB input devices in managed environments.

This method uses the Windows Device Installation Restrictions policy, which prevents the listed devices from installing or functioning even if they are physically connected.

---

## Step 1 — Find the Hardware ID of the Device to Block

1. Connect the device
2. Open **Device Manager** (`devmgmt.msc`)
3. Locate the device — it will typically appear under **Human Interface Devices**
4. Right-click the device > **Properties**
5. Go to the **Details** tab
6. In the **Property** dropdown, select **Hardware IDs**
7. Copy the ID strings listed — use the most specific one at the top of the list

Example hardware ID format:

```
HID\VID_045E&PID_096F
HID\VID_046D&PID_C34B
```

---

## Step 2 — Create the Registry File

Create a `.reg` file with the following structure. Add one numbered entry per device ID you want to block:

```registry
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions]
"DenyDeviceIDs"=dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs]
"1"="HID\\<hardware-id-1>"
"2"="HID\\<hardware-id-2>"
"3"="HID\\<hardware-id-3>"
```

> **Note:** Backslashes must be doubled (`\\`) inside `.reg` files.

---

## Step 3 — Apply the Registry File

Double-click the `.reg` file and confirm the prompt, or apply silently via command line:

```powershell
reg import .\block-devices.reg
```

No reboot is required for the policy to take effect on newly connected devices. Already-installed devices may require a reboot or manual device disable/enable cycle.

---

## Step 4 — Verify

1. Open **Device Manager**
2. Connect the blocked device
3. It should either fail to install or show an error state
4. Alternatively, confirm the registry keys were written:

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs"
```

---

## Removing a Block

To unblock a device, delete its numbered entry from the registry key:

```powershell
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs" -Name "1"
```

Or remove all device blocks entirely:

```powershell
Remove-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyDeviceIDs" -Recurse
```

---

## Notes

- This method applies locally via registry. In a domain environment, the equivalent Group Policy setting is under **Computer Configuration > Administrative Templates > System > Device Installation > Device Installation Restrictions > Prevent installation of devices that match any of these device IDs**
- The `DenyDeviceIDs` DWORD value must be set to `1` for the deny list to be enforced
- Hardware IDs are case-sensitive — copy them exactly from Device Manager
- Multiple IDs for the same physical device may be listed in Device Manager — blocking the most specific ID (top of the list) is sufficient in most cases